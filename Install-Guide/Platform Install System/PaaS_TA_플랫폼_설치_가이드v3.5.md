## Table of Contents

1. [개요](#1)
  * [목적](#2)
  * [범위](#3)
  * [참고자료](#4)
2. [paasta-3.5](#5)
	* [paasta-3.1](#6)
	* [paasta-3.5](#7)
3. [paasta-3.5 설치](#8)
	* [pre-requsite](#9)
    * [Stemcell upload](#10)
    * [cloud-config](#11)
    * [paasta설치 환경 설정](#12)
    * [paasta deploy shell](#13)
    	*  [deploy-aws.sh](#14)
    	*  [deploy-openstack.sh](#15)
    	*  [deploy-azure.sh](#16)
    	*  [deploy-google.sh](#17)
    	*  [deploy-vsphere.sh](#18)
        *  [deploy-bosh-lite.sh](#19)
        *  [option file](#20)
    * [paasta deploy shell](#21)

## Executive Summary

본 문서(설치가이드)는 paasta 를 설치 하기 위한 가이드를 제공하는데 그목적이 있다.

# <div id='1'/>1.  문서 개요 

## <div id='2'/>1.1.  목적
본 문서(설치가이드)는 paasta 를 설치 하기 위한 가이드를 제공하는데 그목적이 있다.

## <div id='3'/>1.2.  범위
Paasta-3.5에서 사용하는 설치 방식은 기존 bosh1과 다르다. Bosh2에서 제공하는 bosh-deployment를 기반으로 bosh를 설치 된 환경에서 paasta-3.5 를 설치 한다.
3.1 버전에서는 파스타 설치 자동화 및 모니터링 기능을 제공했지만 3.5에서는 수동 설치 가이드만을 제공한다. 
향후 2018년 12월 4.0에서 설치 자동화 및 모니터링 기능을 제공할 예정이다. 

본문서는 cf-deployment v2.9.0을 기준으로 작성 되었다.

## <div id='4'/>1.3.  참고자료

본 문서는 Cloud Foundry의 BOSH Document와 Cloud Foundry Document를 참고로 작성하였다.

BOSH Document: [http://bosh.io](http://bosh.io)

Cloud Foundry Document: [https://docs.cloudfoundry.org/](https://docs.cloudfoundry.org/)

BOSH DEPLOYMENT: [https://github.com/cloudfoundry/bosh-deployment](https://github.com/cloudfoundry/bosh-deployment)

CF DEPLOYMENT: [https://github.com/cloudfoundry/cf-deployment](https://github.com/cloudfoundry/cf-deployment)



# <div id='5'/>2. paasta-3.5
Pasta-3.1까지는 bosh-init을 통하여 Bosh를 생성하고, bosh1 cli를 통하여 PaaS-TA Controller, Container를 생성하였다. 



## <div id='6'/>2.1.	paasta-3.1

BBosh1은 bosh-init을 통하여 Bosh를 생성하고, bosh1 cli를 통하여 PaaS-TA Controller, Container를 생성하였다.

![PaaSTa_BOSH_Use_Guide_Image1]

## <div id='7'/>2.2.	paasta-3.5
paasta-3.5는 bosh2를 기반으로 설치된다. Bosh2를 사용하여 Bosh생성 후 paasta-deployment를 활용하여 paasta를 deploy한다. Paasta-3.1 버전까지는  PaaS-TA Container, Controller를 별도로 deployment로 설치 해야 했지만 3.5부터는 paasta deployment 하나로 통합 되었으며, 한번에 PaaS-TA를 설치 할 수 있다.

![PaaSTa_BOSH_Use_Guide_Image2]

# <div id='8'/>3.	paasta-3.5 설치

## <div id='9'/>3.1.	pre-requsite

1.	PaaS-ta 3.5를 설치 하기 위해서는 bosh 설치과정에서 언급한 것 처럼 관련 deployment, release , stemcell을 파스타 사이트에서 다운로드 받아 정해진 경로에 복사 해야 한다.
2.	Bosh를 bosh2 기반으로 설치 되어 있어야 한다.
3.	Paasta-3.5설치하는 환경은 bosh를 deploy한 inception(설치 환경)에서 작업 해야 한다.

### <div id='10'/>3.2.	Stemcell upload

Paasta-3.5는 stemcell 3586.26을 기반으로 한다. Bosh login 후 stemcell을 upload 한다. Stemcell은 Deploy될 때 생성되는 PaaS-TA VM Image이다.
Bosh Login 후 다음 명령어를 수행하여 stemcel을 upload 한다.
director_name은 bosh를 설치할때 사용한 director_name을 입력한다.
```
$ cd ~/workspace/paasta-3.5/stemcells
$ bosh -e {director_name} upload-stemcell bosh-aws-xen-hvm-ubuntu-trusty-go_agent-3586.26.tgz (aws)
$ bosh -e {director_name} upload-stemcell bosh-openstack-kvm-ubuntu-trusty-go_agent-3586.26.tgz (openstack)
$ bosh -e {director_name} upload-stemcell bosh-vsphere-esxi-ubuntu-trusty-go_agent-3586.26.tgz (vsphere)
$ bosh -e {director_name} upload-stemcell bosh-azure-hyperv-ubuntu-trusty-go_agent-3586.26.tgz (azure)
$ bosh -e {director_name} upload-stemcell bosh-google-kvm-ubuntu-trusty-go_agent-3586.26.tgz (google)

$ bosh -e {director_name} stemcells
```

### <div id='11'/>3.3.	cloud-config

Paasta 설치하기 위한 iaas 관련 network/storage/vm 관련 설정들을 정의 한다. IaaS/network/disk등 상황에 따라 설정이 다르다. paasta-deployment.yml은 cloud-config설정에 따라 paasta-vm을 설치 한다.
PaaS-TA Deploy전에 cloud-config가 Bosh에 Upload 되어야 한다.
PaaS-TA는 iaas별 cloud-config 예제를 제공하며, paasta를 설치 하려면 cloud-config.yml을 iaas상황에 맞게 수정 해야 한다.

다음은 openstack기준 cloud-config설정 예제이다. paasta-3.5 deployment를 다운로드 받으면 cloud-config 디렉토리 이하에 iaas별 cloud-config 예제를 볼 수 있다.

![PaaSTa_INSTALL_Use_Guide_Image]


```
## azs는 가용 zone에 대한 설정으로 paasta가 설치될 zone에 대한 정의이다.
azs:
- name: z1
  cloud_properties:
    availability_zone: zone1
- name: z2
  cloud_properties:
    availability_zone: zone2
- name: z3
  cloud_properties:
    availability_zone: zone3
- name: z4
  cloud_properties:
    availability_zone: zone1
- name: z5
  cloud_properties:
    availability_zone: zone2
- name: z6
  cloud_properties:
    availability_zone: zone3

## vm_type은 paasta vm이 설치될 spec을 정의 하며 instance_type은 openstack flavor에 설정 되어 있어야 한다.
vm_types:
- name: minimal
  cloud_properties:
    instance_type: m1.small
- name: default 
  cloud_properties:
    instance_type: m1.monitoring
- name: small
  cloud_properties:
    instance_type: m1.monitoring
- name: medium
  cloud_properties:
    instance_type: m1.medium
- name: medium-memory-8GB
  cloud_properties:
    instance_type: m1.medium-memory 
- name: large
  cloud_properties:
    instance_type: m1.large
- name: xlarge
  cloud_properties:
    instance_type: m1.xlarge
- name: small-50GB
  cloud_properties:
    instance_type: m1.medium
- name: small-50GB-ephemeral-disk 
  cloud_properties:
    instance_type: m1.medium
- name: small-100GB-ephemeral-disk
  cloud_properties:
    instance_type: m1.large
- name: small-highmem-100GB-ephemeral-disk 
  cloud_properties:
    instance_type: m1.large
- name: small-highmem-16GB
  cloud_properties:
    instance_type: m1.large-memory

## compilation 은 PaaS-TA 설치 시 Compile VM이 생성될 zone 및 vm type을 설정한다.
compilation:
  az: z3
  network: default
  reuse_compilation_vms: true
  vm_type: large
  workers: 5

## disk_type은 paasta vm이 사용할 persistence disk에대한 정의이다.
disk_types:
- disk_size: 1024
  name: default
- disk_size: 1024
  name: 1GB
- disk_size: 5120
  name: 5GB
- disk_size: 10240
  name: 10GB
- disk_size: 30720
  name: 30GB
- disk_size: 51200
  name: 50GB
- disk_size: 102400
  name: 100GB
- disk_size: 1048576
  name: 1TB

- cloud_properties:
    type: SSD1 
  disk_size: 2000
  name: 2GB_GP2
- cloud_properties:
    type: SSD1 
  disk_size: 5000
  name: 5GB_GP2
- cloud_properties:
    type: SSD1 
  disk_size: 10000
  name: 10GB_GP2
- cloud_properties:
    type: SSD1 
  disk_size: 50000
  name: 50GB_GP2

## network은 paasta vm이 생성될 network관련 정의이다.openstack의 subnet및 security_group, dns, gateway 등에 대한 정보를 설정한다.
networks:
- name: default
  subnets:
  - az: z1
    cloud_properties:
      name: random
      net_id: 51b96a68-aded-4e73-aa44-f44a812b9b30
      security_groups:
      - openpaas
    dns:
    - 8.8.8.8
    gateway: 10.20.10.1
    range: 10.20.10.0/24
    reserved:
    - 10.20.10.2 - 10.20.10.10
    static:
    - 10.20.10.11 - 10.20.10.30
  - az: z2
    cloud_properties:
      name: random
      net_id: 51b96a68-aded-4e73-aa44-f44a812b9b30
      security_groups:
      - openpaas
    dns:
    - 8.8.8.8
    gateway: 10.20.20.1
    range: 10.20.20.0/24
    reserved:
    - 10.20.20.2 - 10.20.20.10
    static:
    - 10.20.20.11 - 10.20.20.30
  - az: z3
    cloud_properties:
      name: random
      net_id: 51b96a68-aded-4e73-aa44-f44a812b9b30
      security_groups:
      - openpaas
    dns:
    - 8.8.8.8
    gateway: 10.20.30.1
    range: 10.20.30.0/24
    reserved:
    - 10.20.30.2 - 10.20.30.10
    static:
    - 10.20.30.11 - 10.20.30.30
  - az: z4
    cloud_properties:
      name: random
      net_id: 51b96a68-aded-4e73-aa44-f44a812b9b30
      security_groups:
      - openpaas
    dns:
    - 8.8.8.8
    gateway: 10.20.40.1
    range: 10.20.40.0/24
    reserved:
    - 10.20.40.2 - 10.20.40.10
    static:
    - 10.20.40.11 - 10.20.40.30
  - az: z5
    cloud_properties:
      name: random
      net_id: 51b96a68-aded-4e73-aa44-f44a812b9b30
      security_groups:
      - openpaas
    dns:
    - 8.8.8.8
    gateway: 10.20.50.1
    range: 10.20.50.0/24
    reserved:
    - 10.20.50.2 - 10.20.50.10
    static:
    - 10.20.50.11 - 10.20.50.30
  - az: z6
    cloud_properties:
      name: random
      net_id: 51b96a68-aded-4e73-aa44-f44a812b9b30
      security_groups:
      - openpaas
    dns:
    - 8.8.8.8
    gateway: 10.20.60.1
    range: 10.20.60.0/24
    reserved:
    - 10.20.60.2 - 10.20.60.10
    static:
    - 10.20.60.11 - 10.20.60.30
- name: vip 
  type: vip

## vm_extentions는 관련 보안 그룹 및 로드 밸런서와 같은 임의의 IaaS 특정 구성을 지정할 수있는 클라우드 구성의 가상 컴퓨터 구성이다.
vm_extensions:
- cloud_properties:
    ports:
    - host: 3306
  name: mysql-proxy-lb
- name: cf-router-network-properties
- name: cf-tcp-router-network-properties
- name: diego-ssh-proxy-network-properties
- name: cf-haproxy-network-properties 
- cloud_properties:
    ephemeral_disk:
      size: 51200
      type: gp2
  name: small-50GB 
- cloud_properties:
    ephemeral_disk:
      size: 102400
      type: gp2
  name: small-highmem-100GB 

```

Cloud-config설정 정보를 BOSH에 update 한다.

```
$ cd ~/workspace/paasta-3.5/deployment/cloud-config
$ bosh –e {director_name} update-cloud-config {iaas}_cloud_config.yml
$ bosh –e {director_name} cloud-config # bosh에 update된 cloud-config 확인
```

upload된 cloud-config 확인 한다.

```
$ bosh –e {director_name} cloud-config  
```

다음은 openstack에서 정의한 flavor type 예제 이다.
![PaaSTa_FLAVOR_Image]




### <div id='12'/>3.4.	paasta설치 환경 설정

~/workspace/paasta-3.5/deployment/bosh-deployment 이하 디렉토리에는 iaas별 paasta를 설 치 하는 shell이 존재한다. Shell 파일을 이용하여 bosh를 설치 한다.
파일명은 deploy-{iaaS-name}.sh 로 만들어 졌다. 

```
$ cd ~/workspace/paasta-3.5/deployment/paasta-deployment-3.5
$ chmod 755 *.sh
```


<table>
<tr>
<td>deploy-aws.sh</td>
<td>amazon 설치 paasta 설치 shell</td>
</tr>
<tr>
<td>deploy-openstack.sh</td>
<td>openstack 설치 paasta 설치 shell</td>
</tr>   
<tr>
<td>deploy-vsphere.sh</td>
<td>vsphere 설치 paasta 설치 shell</td>
</tr>
<tr>
<td>deploy-gcp.sh</td>
<td>GCP(google) 설치 paasta 설치 shell</td>
</tr>
<tr>
<td>deploy-azure.sh</td>
<td>Azure 설치 bosh 설치 shell</td>
</tr>
<tr>
<td>deploy-bosh-lite.sh</td>
<td>Bosh-lite(local Test용) 설치 paasta 설치 shell</td>
</tr>
</table>

설치 shell 파일은 각 iaas별로 존재하며 bosh 설치 시 명령어는 deploy 로 시작한다. Shell 이 아닌 bosh command로 실행이 가능하며 설치하는 IaaS 환경에 따라 Option들이 달라진다.

```
$ bosh –e {director_name} –d paasta deploy {deploy.yml}
```

설치 option

<table>
<tr>
<td>-e</td>
<td>BOSH Director anme</td>
</tr>
<tr>
<td>-d</td>
<td>Deployment name (paasta 는 deployment name paasta)</td>
</tr>   
<tr>
<td>-o</td>
<td>Paasta 설치시 사용하는 option 파일로 아래 기능을 기본으로 제공한다.(iaas별속성, 	haproxy 사용여부
,	database 선정 )
</td>
</tr>
<tr>
<td>-v</td>
<td>paasta 설치시 사용되는 yml 파일 또는 option 파일에 변수에 값을 설정할 경우 사용 할 수 있다. Yml, optionfile 속성에 따라 필수인 경우와 option인 경우가 있다.</td>
</tr>
</table>

## <div id='13'/>3.5.	paasta deploy shell

 - --vars-store : paasta deploy시 사용하는 option으로 paasta 내부 인증 정보를 담고 있다. 해당 경로에 파일이 없으면 bosh-cli가 자동 생성 해주며 자동 생성해 준다. 그리고 deployment-vars.yml 파일에 기록된다. 

 - paasta-deployment.yml 파일은 paasta를 deploy하는 manifest file이다. paasta vm에대한 설치 정의를 하게 된다. vm중 singleton-blobstore, database 의 azs(zone)을 변경 하면 조직, 스페이스, app의 정보가 모두 삭제된다. 

아래 option들은 존재 하지 않으면 bosh-cli가 자동 생성 해준다. 만약 존재하는 deployment-vars.yml 파일에 기록되지 않는다.

<table>
<tr>
<td>cf_admin_password</td>
<td>paasta admin password</td>
</tr>
<tr>
<td>cc_db_encryption_key</td>
<td>paasta database 암호화 키 (version upgrade시 동일한 key이어야 함)</td>
</tr>   
<tr>
<td>uaa_database_password</td>
<td>uaadb database admin pwd
</td>
</tr>
<tr>
<td>cc_database_password</td>
<td>cloud_controller(api) database admin pwd</td>
</tr>
</table>

### <div id='14'/>3.5.1. deploy-aws.sh
```
bosh -e {director_name} -d paasta deploy paasta-deployment.yml \  # paasta manifest file
   --vars-store deployment-vars.yml \              # 인증 정보 보관 파일(중요, backup 필요)
   -o operations/aws.yml \                         # aws 설정
   -o operations/use-compiled-releases.yml \
   -o operations/use-haproxy.yml \                 # haproxy 사용여부
   -o operations/use-haproxy-public-network.yml \  # haproxy public ip 사용여부
   -o operations/use-postgres.yml \                # paasta database type 설정 (3.5버전 이하에서 migration시 필수)
   -v inception_os_user_name=ubuntu \          # home user명 (release file path와 연관성 있음. /home/ubuntu/paasta-3.5 이하 release 파일들의 경로 설정)
   -v haproxy_public_ip=52.199.190.1 \             # pasta public ip
   -v haproxy_public_network_name=vip \   
   -v cf_admin_password=admin \                    # paasta-admin password
   -v cc_db_encryption_key=db-encryption-key \     # database 암호화 키 (version upgrade시 동일한 key이어야 함)
   -v uaa_database_password=uaa_admin \            # uaadb database pwd
   -v cc_database_password=cc_admin \              # ccdb database pwd
   -v system_domain=52.199.190.1.xip.io            # domain (xip.io를 사용하는 경우 haproxy public_ip와 동일하게 한다)
```

### <div id='15'/>3.5.2. deploy-openstack.sh
```
bosh -e {director_name} -d paasta deploy paasta-deployment.yml \   # paasta manifest file
   --vars-store deployment-vars.yml \              # 인증 정보 보관 파일(중요, backup 필요)
   -o operations/openstack.yml \                  # openstack 설정
   -o operations/use-compiled-releases.yml \      # compile된 release 파일 정보(offline)
   -o operations/use-haproxy.yml \
   -o operations/use-haproxy-public-network.yml \
   -o operations/use-postgres.yml \              # database postgres 선택
   -v inception_os_user_name=ubuntu \          # home user명 (release file path와 연관성 있음. /home/ubuntu/paasta-3.5 이하 release 파일들의 경로 설정)
   -v haproxy_public_ip=52.199.190.1 \           # paasta public ip
   -v haproxy_public_network_name=vip \   
   -v cf_admin_password=admin \                  # paasta-admin password
   -v cc_db_encryption_key=db-encryption-key \   # version upgrade시 동일한 key이어야 함
   -v uaa_database_password=uaa_admin \          # uaadb database pwd
   -v cc_database_password=cc_admin \            # ccdb database pwd
   -v system_domain=52.199.190.1.xip.io          # domain (xip.io를 사용하는 경우 haproxy public_ip와 동일하게 한다)
```

### <div id='16'/>3.5.3. deploy-azure.sh
```
bosh -e {director_name} -d paasta deploy paasta-deployment.yml \
   --vars-store deployment-vars.yml \              # 인증 정보 보관 파일(중요, backup 필요)
   -o operations/azure.yml \
   -o operations/use-compiled-releases.yml \
   -o operations/use-haproxy.yml \
   -o operations/use-haproxy-public-network.yml \
   -o operations/use-postgres.yml \
   -v inception_os_user_name=ubuntu \          # home user명 (release file path와 연관성 있음. /home/ubuntu/paasta-3.5 이하 release 파일들의 경로 설정)
   -v haproxy_public_ip=52.231.156.110 \        # paasta public ip
   -v haproxy_public_network_name=vip \
   -v cf_admin_password=admin \                 # cf-admin password
   -v cc_db_encryption_key=db-encryption-key \  # version upgrade시 동일한 key이어야 함
   -v uaa_database_password=uaa_admin \         # uaadb database pwd
   -v cc_database_password=cc_admin \           # ccdb database pwd
   -v system_domain=52.231.156.110.xip.io       # domain (xip.io를 사용하는 경우 haproxy public_ip와 동일하게 한다)
```

### <div id='17'/>3.5.4. deploy-google.sh

```
bosh -e {director_name} -d paasta deploy paasta-deployment.yml \
   --vars-store deployment-vars.yml \              # 인증 정보 보관 파일(중요, backup 필요)
   -o operations/use-compiled-releases.yml \
   -o operations/use-haproxy.yml \
   -o operations/use-haproxy-public-network.yml \
   -o operations/use-postgres.yml \
   -v inception_os_user_name=ubuntu \          # home user명 (release file path와 연관성 있음. /home/ubuntu/paasta-3.5 이하 release 파일들의 경로 설정)
   -v haproxy_public_ip=35.200.2.244 \          # paasta public ip
   -v haproxy_public_network_name=vip \
   -v cf_admin_password=admin \                 # cf-admin password
   -v cc_db_encryption_key=db-encryption-key \  # version upgrade시 동일한 key이어야 함
   -v uaa_database_password=uaa_admin \         # uaadb database pwd
   -v cc_database_password=cc_admin \           # ccdb database pwd
   -v system_domain=35.200.2.244.xip.io         # domain  (xip.io를 사용하는 경우 haproxy public_ip와 동일하게 한다)
```


### <div id='18'/>3.5.5. deploy-vshpere.sh

```
bosh -e {director_name} -d paasta deploy paasta-deployment.yml \
   --vars-store deployment-vars.yml \              # 인증 정보 보관 파일(중요, backup 필요)
   -o operations/use-compiled-releases.yml \
   -o operations/use-haproxy.yml \
   -o operations/use-haproxy-public-network.yml \
   -o operations/use-postgres.yml \
   -v inception_os_user_name=ubuntu \          # home user명 (release file path와 연관성 있음. /home/ubuntu/paasta-3.5 이하 release 파일들의 경로 설정)
   -v haproxy_public_ip=35.200.2.244 \          # paasta public ip
   -v haproxy_public_network_name=vip \
   -v cf_admin_password=admin \                 # cf-admin password
   -v cc_db_encryption_key=db-encryption-key \  # version upgrade시 동일한 key이어야 함
   -v uaa_database_password=uaa_admin \         # uaadb database pwd
   -v cc_database_password=cc_admin \           # ccdb database pwd
   -v system_domain=35.200.2.244.xip.io         # domain  (xip.io를 사용하는 경우 haproxy public_ip와 동일하게 한다)
```

#### <div id='19'/>3.5.6. deploy-bosh-lite.sh

```
bosh -e {director_name} -d paasta deploy paasta-deployment.yml \
   --vars-store deployment-vars.yml \              # 인증 정보 보관 파일(중요, backup 필요)
   -o operations/bosh-lite.yml \
   -o operations/use-compiled-releases.yml \
   -o operations/use-postgres.yml \
   -v cf_admin_password=admin \
   -v inception_os_user_name=ubuntu \          # home user명 (release file path와 연관성 있음. /home/ubuntu/paasta-3.5 이하 release 파일들의 경로 설정)
   -v cf_admin_password=admin \                 # cf-admin password
   -v cc_db_encryption_key=db-encryption-key \  # version upgrade시 동일한 key이어야 함
   -v uaa_database_password=uaa_admin \         # uaadb database pwd
   -v cc_database_password=cc_admin \           # ccdb database pwd
   -v system_domain=10.244.0.34.xip.io          # domain (haproxy public_ip와 동일하게 한다)
```


#### <div id='20'/>3.5.7. option file

-  operations/use-compiled-releases.yml : paasta release 에서 제공되는 파일로 download, compile 없이 빠른 설치가 가능하다.
-  operations/use-postgres.yml: 3.5 이전 버전에서 migration 시 필수 
-  operations/use-haproxy.yml: haproxy 사용여부로 use-haproxy-public-network.yml 파일과 같시 사용되어야 한다. 아래 value option은 use_haproxy.yml과 함께 쓰여야 하는 option value이다.

   -v haproxy_private_ip=10.0.0.12

   -v haproxy_public_ip=xxx.xxx.xxx.xxx

   -v haproxy_public_network_name=vip

- db_encryption_key: 3.5 이전 버전에서 migration 시 paasta-controller.yml의 db_encryption_key에 있는 값이 동일 해야 한다.

### <div id='21'/>3.6.	paasta deploy 

```
$ cd ~/workspace/paasta-3.5/deployment/paasta-deployment-3.5
$ ./deploy-{iaas}.sh
```

### <div id='22'/>3.7.	paasta deploy 확인

```
bosh -e {director_name} vms
```

![PaaSTa_VMS_Guide_Image]

### <div id='23'/>2.8.	paasta login 

cf cli를 설치 하고 paasta에 로그인 한다.
api target 은 paasta deploy시 지정해주었던 system_domain 명을 사용한다.
ex) cf api api.{system_domain} --skip-ssl-validation

```
cf api api.domain_name --skip-ssl-validation  #api target 지정
cf login                                      # login
```


![PaaSTa_LOGIN_Guide_Image]

[PaaSTa_BOSH_Use_Guide_Image1]:../images/paasta-3.5/bosh1.png
[PaaSTa_BOSH_Use_Guide_Image2]:../images/paasta-3.5/bosh2.png
[PaaSTa_INSTALL_Use_Guide_Image]:../images/paasta-3.5/cloud-config.png
[PaaSTa_VMS_Guide_Image]:../images/paasta-3.5/paasta-vms.png
[PaaSTa_LOGIN_Guide_Image]:../images/paasta-3.5/paasta-login.png
[PaaSTa_FLAVOR_Image]:../images/paasta-3.5/flavor.png