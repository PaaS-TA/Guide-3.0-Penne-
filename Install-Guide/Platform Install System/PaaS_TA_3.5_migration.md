## Table of Contents

1. [개요](#1)
  * [목적](#2)
  * [범위](#3)
2. [Paasta 3.5 version upgrade](#4)
	* [pre-requsite](#5)
	* [paasta-3.1 backup](#6)
    * [ccdb, uaadb backup (bosh cli 1 기준)](#7)
    * [blobstore backup](#8)
    * [ccdb, uaadb datafile edit](#9)
	* [paasta-3.5 restore](#10)
    * [pre-requisite](#11)
    * [blob-store recovery](#12)
    * [uaa database recovery](#13)
    * [cloud_controller database recovery](#14)
    * [사용자 및 App 확인 ](#15)

# <div id='1'/>1.  문서 개요 

## <div id='2'/>1.1.  목적
본 문서(설치가이드)는 파스타를 3.1 버전 이하에서 3.5로 버전 업그레이드 하는데 있다.

## <div id='3'/>1.2.  범위
본 문서(설치가이드)는 파스타를 3.1 버전 이하에서 3.5로 버전 업그레이드 방법을 가이드 하는데 있다.


# <div id='4'/>2. Paasta 3.5 version upgrade
Paasta 3.5에서는 3.1이전 버전과 배포 방식 및 version upgrade하는 방식이 바뀌 었다. Paasta 3.1 까지는 cf-release 를 기준으로 version upgrade가 되었다. 하지만 cf-release 287을 마지막으로 cloud-foundry의 bosh 및 cf deploy방식이 바뀌었다. 
Bosh는 bosh-deployment를 기반으로 bosh를 배포 하였으며, cloud-foundry는 cf-deployment를 기반으로 cf 를 배포 하게 되었다. Cli도 bosh-cli1 에서 bosh-cli2로 변경 되었다. Bosh-cli2에서는 기존 bosh cli명령어도 변경 되었다. 

이런 이유에서 paasta-3.1에서 3.5로 upgrade할 때 기존 방식으로 는 upgrade가 불가능 하다. 본 가이드는 paasta-3.1에서 3.5로 upgrade시 기존 App과 사용자/조직 정보를 migration 하는데 그 목적이 있다.

Paasta 3.5로 migration하기위해서는 ccdb, uaadb 및 blobstore data를 bakcup하여 restore해야 한다.


![PaaSTa_Migration_Image]


## <div id='5'/>2.1.	pre-requsite

1.	Backup and restore를 진행하는 동안  paasta를 사용하는 사용자가 없어야 한다. 
2.	3.1과 3.5는 System domain이 동일해야 한다.
3.	cf_admin_password가 3.1 버전과 동일해야 한다.
4.	Paasta-3.1 Paasta-controller.yml에 db_encryption_key에 있는 값이 paasta-3.5 설치시 동일해야 한다. 
5.	paasta-3.1 Backup 은 bosh cli 기준은 bosh1 이며, paasta-3.5 recovery bosh cli 기준은 bosh2 이다.


### <div id='6'/>2.2.	paasta-3.1 backup

paasta-3.1 설치된 vm에서 database data와 blobstore data를 backup 한다.


### <div id='7'/>2.2.1.	ccdb, uaadb backup (bosh cli 1 기준)

```
$ bosh ssh postgres_z1/0   #postgres database vm 접근
$ sudo su –
$ cd /var/vcap/packages/postgres_x.x.x/bin/
$ ./pg_dump -U ccadmin -p 5524 -d ccdb --data-only --column-inserts > /tmp/ccdb-data.sql     # ccdb database backup (사용자, 조직, 스페이스, app meta data)
./pg_dump -U uaaadmin -p 5524 -d uaadb --data-only --column-inserts > /tmp/uaadb-data.sql    # uaadb database backup (사용자 정보 meta data)

$ exit
$ exit   # postgres vm 에서 나온다.

$ bosh scp postgres_z1/0 --download  /tmp/ccdb-data.sql  {download_path}
$ bosh scp postgres_z1/0 --download  /tmp/uaadb-data.sql  {download_path}
$ cd {download_path}
$ mv ccdb-data.sql.postgres_z1.0 ccdb-data.sql
$ mv uaadb-data.sql.postgres_z1.0 uaadb-data.sql

```
### <div id='8'/>2.2.2.	blobstore backup

```
$ bosh ssh blobstore_z1/0   #blobstore_z1 vm 접근
$ sudo su -
$ cd /var/vcap/store  
$ sudo tar cvf blobstore.tar shared/
$ sudo mv blobstore.tar /tmp/

$ exit
$ exit   # blobstore vm 에서 나온다.

$ bosh scp blobstore_z1/0 /tmp/blobstore.tar {download_path} –download  (app 용량에 따라 시간이 걸림)

$ mv blobstore.tar.blobstore_z1.o blobstore.tar
```

### <div id='9'/>2.2.3.	ccdb, uaadb datafile edit


1)	uaadb-data.sql 파일을 열어 
identity_zone, identity_provider, schema_migrations Table insert 문장 삭제

2)	ccdb-data.sql 파일을 열어 
schema_migrations Table insert 문장 삭제



## <div id='10'/>2.3.	paasta-3.5 restore (bosh2 cli 기준)

### <div id='11'/>2.3.1.	pre-requisite

-	recover에서 사용하는 bosh cli는 bosh2 이다.
-	paasta-3.5가 설치 되어 있어야 한다.
-	System domain이 이전과 동일해야 한다.
-	Cf_admin_password가 이전과 동일해야 한다.
-	Paasta-controller.yml에 db_encryption_key에 있는 값이 paasta-3.5 설치시 동일해야 한다.
-	paasta-3.5 설치 한 상태에서 recovery해야 한다.(app, 사용자 없어야 함)
-	recovery시 아래 순서를 그대로 따라 해야 한다.

### <div id='12'/>2.3.2.	blob-store recovery

bosh cli 로그인 후 아래 명령어를 실행한다.
```
$ bosh -e {director_name} -d paasta scp {blob_file_path}/blobstore.tar singleton-blobstore:/tmp   #blob file upload

$ bosh -e {director_name}  -d paasta ssh singleton-blobstore  #blobstore vm 접속
$ sudo su - 
$ cd /tmp/
$ mv blobstore.tar /var/vcap/store
$ cd /var/vcap/store
$ rm -rf shared/cc-buildpacks/
$ tar xvf blobstore.tar
$ cd shared

$ mv xx.xxx.xx.xx.xip.io-cc-buildpacks/ cc-buildpacks   (디렉토리명 변경)
$ mv xx.xxx.xx.xx.xip.io-cc-resources cc-resources
$ mv xx.xxx.xx.xx.xip.io-cc-packages cc-packages
$ mv xx.xxx.xx.xx.xip.io-cc-droplets cc-droplets
$ exit
$ exit    # blobstore vm 에서 나온다.
```

[PaaSTa_Migration_Image]:../images/paasta-3.5/paasta-migration.png
[PaaSTa_BOSH_Use_Guide_Image2]:../images/paasta-3.5/bosh2.png
[PaaSTa_INSTALL_Use_Guide_Image]:../images/paasta-3.5/cloud-config.png


### <div id='13'/>2.3.3.	uaa database recovery
uaa는 사용자 정보 및 인증정보를 보관하고 있다.

```
$ bosh -e {director_name} -d paasta ssh database  #database vm 접속
$ cd /var/vcap/packages/postgres-x.x.x/bin/
$ ./psql -h 127.0.0.1 -p 5524 -U uaa -W uaa (password는 paasta deploy시 설정한 uaa_database_password)

Database login
# delete from external_group_mapping;
# delete from groups;
# delete from group_membership;
# delete from oauth_client_details;
# delete from users where username = 'admin';


수정한 uaadb-data.sql 에 있는 내용을 실행한다.
#  \q  (postgres exit)

$ exit 
$ exit
```

### <div id='14'/>2.3.4.	cloud_controller database recovery

```
$ bosh -d paasta ssh database
$ cd /var/vcap/packages/postgres-x.x.x/bin/
$ ./psql -h 127.0.0.1 -p 5524 -U cloud_controller -W cloud_controller (password는 paasta deploy시 설정한 cc_database_password)

Database login
# delete from buildpacks;
# delete from clock_jobs;
# delete from domains;
# delete from isolation_segments;
# delete from env_groups; 
# delete from lockings;
# delete from spaces;
# delete from organizations;
# delete from quota_definitions;
# delete from stacks;
# delete from users;
# delete from security_groups;

# 수정한 ccdb-data.sql 에 있는 내용을 실행한다.
#  \q  (postgres exit)
$ exit
$ exit
```
환경에 따라 기존 Application이 start 되는데 시간이 걸 릴 수 있다.

### <div id='15'/>2.3.5.	사용자 및 App 확인 

```
$ cf login
$ cf apps 
```
