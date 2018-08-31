## Table of Contents
1. [문서 개요](#1-문서-개요)
  - 1.1. [목적](#11-목적)
  - 1.2. [범위](#12-범위)
  - 1.3. [시스템 구성도](#13-시스템-구성도)
  - 1.4. [참고자료](#14-참고자료)
2. [MySQL 서비스팩 설치](#2-mysql-서비스팩-설치)
  - 2.1. [설치전 준비사항](#21-설치전-준비사항)
  - 2.2. [MySQL 서비스 릴리즈 업로드](#22-mysql-서비스-릴리즈-업로드)
  - 2.3. [MySQL 서비스 Deployment 파일 수정 및 배포](#23-mysql-서비스-deployment-파일-수정-및-배포)
  - 2.4. [MySQL 서비스 브로커 등록](#24-mysql-서비스-브로커-등록)
3. [MySQL 연동 Sample Web App 설명](#3-mysql-연동-sample-web-app-설명)
  - 3.1. [Sample Web App 구조](#31-sample-web-app-구조)
  - 3.2. [PaaS-TA에서 서비스 신청](#32-개방형-클라우드-플랫폼에서-서비스-신청)
  - 3.3. [Sample Web App에 서비스 바인드 신청 및 App 확인](#33-sample-web-app에-서비스-바인드-신청-및-app-확인)
4. [MySQL Client 툴 접속](#4-mysql-client-툴-접속)
  - 4.1. [HeidiSQL 설치 및 연결](#41-heidisql-설치-및-연결)

# 1. 문서 개요
### 1.1. 목적

본 문서(MySQL 서비스팩 설치 가이드)는 전자정부표준프레임워크 기반의 PaaS-TA에서 제공되는 서비스팩인 MySQL 서비스팩을 Bosh2.0을 이용하여 설치 하는 방법과 PaaS-TA의 SaaS 형태로 제공하는 Application 에서 MySQL 서비스를 사용하는 방법을 기술하였다.
PaaS-TA 3.5 버전부터는 Bosh2.0 기반으로 deploy를 진행하며 기존 Bosh1.0 기반으로 설치를 원할경우에는 PaaS-TA 3.1 이하 버전의 문서를 참고한다.

### 1.2. 범위
설치 범위는 MySQL 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다.

### 1.3. 시스템 구성도
본 문서의 설치된 시스템 구성도이다. MySQL Server, MySQL 서비스 브로커, Proxy로 최소사항을 구성하였다.

![시스템구성도][mysql_vsphere_1.3.01]

| 구분 | Resource Pool | 스펙 |
|--------|-------|-------|
| paasta-mysql-broker | minimal | 1vCPU / 1GB RAM / 8GB Disk |
| proxy | minimal | 1vCPU / 1GB RAM / 8GB Disk |
| mysql | minimal | 1vCPU / 1GB RAM / 8GB Disk +8GB(영구적 Disk) |

### 1.4. 참고자료
[**http://bosh.io/docs**](http://bosh.io/docs)  
[**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/)

# 2. MySQL 서비스팩 설치

### 2.1. 설치전 준비사항

본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다.
서비스팩 설치를 위해서는 먼저 BOSH CLI v2 가 설치 되어 있어야 하고 BOSH 에 로그인이 되어 있어야 한다.<br>
BOSH CLI v2 가 설치 되어 있지 않을 경우 먼저 BOSH2.0 설치 가이드 문서를 참고 하여 BOSH CLI v2를 설치를 하고 사용법을 숙지 해야 한다.<br>

- BOSH2.0 사용자 가이드
>BOSH2 사용자 가이드 : **<https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/v3.5/Use-Guide/Bosh/PaaS-TA_BOSH2_%EC%82%AC%EC%9A%A9%EC%9E%90_%EA%B0%80%EC%9D%B4%EB%93%9Cv1.0.md>**

>BOSH CLI V2 사용자 가이드 : **<https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/v3.5/Use-Guide/Bosh/PaaS-TA_BOSH_CLI_V2_%EC%82%AC%EC%9A%A9%EC%9E%90_%EA%B0%80%EC%9D%B4%EB%93%9Cv1.0.md>**

- PaaS-TA에서 제공하는 압축된 릴리즈 파일들을 다운받는다. (PaaSTA-Deployment.zip, PaaSTA-Sample-Apps.zip, PaaSTA-Services.zip)

- 다운로드 위치
>PaaSTA-Deployment : **<https://paas-ta.kr/data/packages/3.5/deployment.zip>**  
>PaaSTA-Sample-Apps : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Sample-Apps.zip>**


### 2.2. MySQL 서비스 릴리즈 업로드

-	업로드 되어 있는 릴리즈 목록을 확인한다.

- **사용 예시**
 $ bosh -e micro-bosh releases
 Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

Name                              Version   Commit Hash  
binary-buildpack                  1.0.21*   d714741  
bpm                               0.9.0*    c9b7136  
caas-release                      1.0*      empty+  
capi                              1.62.0*   22a608c  
cf-networking                     2.8.0*    479f4a66  
cf-smoke-tests                    40.0.5*   d6aaf1f  
cf-syslog-drain                   7.0*      71b995a  
cflinuxfs2                        1.227.0*  60128e1  
consul                            195*      67cdbcd  
diego                             2.13.0*   b5644d9  
dotnet-core-buildpack             2.1.3*    46a41cd  
garden-runc                       1.15.1*   75107e7+  
go-buildpack                      1.8.25*   40c60a0  
haproxy                           8.8.0*    9292573  
java-buildpack                    4.13*     c2749d3  
loggregator                       103.0*    05da4e3d  
loggregator-agent                 2.0*      2382c90  
nats                              24*       30e7a82  
nodejs-buildpack                  1.6.28*   4cfdb7b  
paas-ta-portal-release            2.0*      non-git  
paasta-delivery-pipeline-release  1.0*      b3ee8f48+  
paasta-pinpoint                   2.0*      2dbb8bf3+  
php-buildpack                     4.3.57*   efc48f3  
postgres                          29*       5de4d63d+  
python-buildpack                  1.6.18*   bcc4f26  
routing                           0.179.0*  18155a5  
ruby-buildpack                    1.7.21*   9d69600  
silk                              2.9.0*    eebed55  
staticfile-buildpack              1.4.29*   8a82e63  
statsd-injector                   1.3.0*    39e5179  
uaa                               60.2*     ebb5895  

(*) Currently deployed
(+) Uncommitted changes

31 releases

Succeeded

<br>Mysql 서비스 릴리즈가 업로드 되어 있지 않은 것을 확인<br>

<br>
-	MySQL 서비스 릴리즈 파일을 업로드한다.

>$ bosh upload release <br>
>※	본 샘플은 직접 릴리즈 파일을 생성하도록 안내하고 있으므로 'bosh upload release'만 입력한다. 릴리즈 파일을 직접 생성하지 않고 다운로드 받거나 릴리즈 파일 다운로드 URL이 존재하는 경우는 파일 경로 또는 다운로드 URL을 아래와 같이 입력한다.<br>
>$ bosh upload release {서비스 릴리즈 파일 PATH 또는 URL}<br>

![update_mysql_vsphere_41]
![update_mysql_vsphere_42]
![update_mysql_vsphere_43]
![update_mysql_vsphere_44]
![update_mysql_vsphere_45]

-	업로드 된 MySQL 릴리즈를 확인한다.

>$ bosh releases<br>
>RSA 1024 bit CA certificates are loaded due to old openssl compatibility<br>
>Acting as user 'admin' on 'bosh'<br>
><br>
>+--------------------------------------+-----------+-------------+<br>
>| Name                                 | Versions  | Commit Hash |<br>
>+--------------------------------------+-----------+-------------+<br>
>| cf                                   | 247*      | af4efe9f+   |<br>
>| cflinuxfs2-rootfs                    | 1.40.0*   | 19fe09f4+   |<br>
>| diego                                | 1.1.0*    | 2298c8d4    |<br>
>| empty-release                        | 1+dev.1*  | 00000000    |<br>
>| etcd                                 | 86*       | 2dfbef00+   |<br>
>| garden-runc                          | 1.0.3*    | c6c4c73c    |<br>
>| paasta-cubrid                        | 2.0*      | 85e3f01e+   |<br>
>| paasta-eclipse-che                   | 2.0*      | 00000000    |<br>
>| paasta-glusterfs                     | 2.0*      | 85e3f01e+   |<br>
>| paasta-mysql                         | 2.0*      | 85e3f01e+   |<br>
>| paasta-portal-object-storage-release | 0+dev.1*  | 00000000    |<br>
>| paasta-redis                         | 2.0       | 2d766084+   |<br>
>| swift-test-2                         | 0+dev.1*  | 00000000    |<br>
>+--------------------------------------+-----------+-------------+<br>
>(*) Currently deployed<br>
>(+) Uncommitted changes<br>
><br>
>Releases total: 13<br>
><br>
>Mysql 서비스 릴리즈가 업로드 되어 있는 것을 확인<br>



### 2.3. MySQL 서비스 Deployment 파일 수정 및 배포

BOSH Deployment manifest 는 components 요소 및 배포의 속성을 정의한 YAML 파일이다.
Deployment manifest 에는 sotfware를 설치 하기 위해서 어떤 Stemcell (OS, BOSH agent) 을 사용할것이며 Release (Software packages, Config templates, Scripts) 이름과 버전, VMs 용량, Jobs params 등을 정의가 되어 있다.

-	PaaSTA-Deployment.zip 파일 압축을 풀고 폴더안에 있는 IaaS별 MySQL Deployment 파일을 복사한다.<br>
    예) vsphere 일 경우 paasta_mysql_vsphere_2.0.yml를 복사<br>
    다운로드 받은 Deployment Yml 파일을 확인한다.


>$ ls –all<br>
><br>
> total 851588<br>
> drwxrwxr-x  5 inception inception      4096 Jan  9 10:18 .<br>
> drwxrwxr-x 11 inception inception      4096 Dec 21 09:28 ..<br>
><br>
> -rw-r--r--  1 inception inception      6614 Jan  6 16:14 paasta_cubrid_vsphere_2.0.yml<br>
> -rw-rw-r--  1 inception inception      6382 Jan  9 10:18 paasta_mysql_vsphere_2.0.yml<br>


<br>

- Director UUID를 확인한다.

>BOSH CLI가 배포에 대한 모든 작업을 허용하기위한 현재 대상 BOSH Director의 UUID와 일치해야한다. ‘bosh status’ CLI 을 통해서 현재 BOSH Director 에 target 되어 있는 UUID를 확인할 수 있다.

>`$ bosh status`

![update_mysql_vsphere_46]

<br>

- Deploy시 사용할 Stemcell을 확인한다.

>`$ bosh stemcells`

![update_mysql_vsphere_47]

>Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드를 해야 한다.

<br>

-	Deployment 파일을 서버 환경에 맞게 수정한다. (vsphere 용으로 설명, 다른 IaaS는 해당 Deployment 파일의 주석내용을 참고)

```yml
# paasta-mysql-vsphere 설정 파일 내용
name: paasta-mysql-service                              # 서비스 배포이름(필수)
director_uuid: d363905f-eaa0-4539-a461-8c1318498a32     # bosh status 에서 확인한 Director UUID을 입력(필수)

releases:
- name: paasta-mysql                                    # 서비스 릴리즈 이름(필수)
  version: 2.0                                          # 서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전

update:
  canaries: 1                            # canary 인스턴스 수(필수)
  canary_watch_time: 30000-600000        # canary 인스턴스가 수행하기 위한 대기 시간(필수)
  max_in_flight: 1                       # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)
  update_watch_time: 30000-600000        # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)

compilation:                             # 컴파일시 필요한 가상머신의 속성(필수)
  cloud_properties:                      # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성 (instance_type, availability_zone), 직접 cpu,disk,ram 사이즈를 넣어도 됨
    cpu: 4
    disk: 20480
    ram: 4096
  network: default                       # Networks block에서 선언한 network 이름(필수)
  reuse_compilation_vms: true            # 컴파일지 VM 재사용 여부(옵션)
  workers: 4                             # 컴파일 하는 가상머신의 최대수(필수)

jobs:
- instances: 1                           # job 인스턴스 수(필수)
  name: mysql_z1
  networks:                              # 네트워크 구성정보
  - name: default
    static_ips:                          # 사용할 IP addresses 정의(필수): MySQL 서버 IP
    - 10.30.40.191
    #- 10.30.40.192
    #- 10.30.40.193
  persistent_disk: 8192                  # 영구적 디스크 사이즈 정의(옵션): 10G
  properties:
    admin_password: admin                # MySQL 어드민 패스워드
    cluster_ips:                         # 클러스터 구성시 IPs(필수)
    - 10.30.40.191
    #- 10.30.40.192
    #- 10.30.40.193
    network_name: default
    seeded_databases: null
    syslog_aggregator: null
    collation_server: utf8_unicode_ci    # Mysql CharSet
    character_set_server: utf8
  release: paasta-mysql
  resource_pool: services-small
  template: mysql

- instances: 1
  name: proxy                                # 작업 이름(필수): proxy
  networks:
  - name: default
    static_ips: 10.30.40.194
  properties:
    cluster_ips:
    - 10.30.40.191
    #- 10.30.40.192
    #- 10.30.40.193
    external_host: 115.68.46.30.xip.io       # PaaS-TA 설치시 설정한 외부 호스트 정보(필수)
    nats:                                    # PaaS-TA 설치시 설치한 nats 정보 (필수)
      machines:
      - 10.30.110.31
      password: nats
      port: 4222
      user: nats
    network_name: default
    proxy:                                   # proxy 정보 (필수)
      api_password: admin
      api_username: api
      api_force_https: false
    syslog_aggregator: null
  release: paasta-mysql
  resource_pool: services-small
  template: proxy

- instances: 1
  name: paasta-mysql-java-broker                     # 작업 이름(필수): 서비스 브로커
  networks:
  - name: default
    static_ips: 10.30.40.195
  properties:                                        # Mysql 정보
    jdbc_ip: 10.30.40.194
    jdbc_pwd: admin
    jdbc_port: 3306
    log_dir: paasta-mysql-java-broker
    log_file: paasta-mysql-java-broker
    log_level: INFO
  release: paasta-mysql
  resource_pool: services-small
  template: op-mysql-java-broker

- instances: 1
  lifecycle: errand                                 # bosh deploy시 vm에 생성되어 설치 되지 않고 bosh errand 로 실행할때 설정, 주로 테스트 용도에 쓰임
  name: broker-registrar                            # 작업 이름: 서비스 브로커 등록
  networks:
  - name: default
  properties:
    broker:
      host: 10.30.40.195
      name: mysql-service-broker
      password: cloudfoundry
      username: admin
      protocol: http
      port: 8080
    cf:
      admin_password: admin
      admin_username: admin
      api_url: https://api.115.68.46.30.xip.io
      skip_ssl_validation: true
  release: paasta-mysql
  resource_pool: services-small
  template: broker-registrar

- instances: 1
  lifecycle: errand
  name: broker-deregistrar
  networks:
  - name: default
  properties:
    broker:
      name: mysql-service-broker
    cf:
      admin_password: admin
      admin_username: admin
      api_url: https://api.115.68.46.30.xip.io
      skip_ssl_validation: true
  release: paasta-mysql
  resource_pool: services-small
  template: broker-deregistrar

meta:
  apps_domain: 115.68.46.30.xip.io
  environment: null
  external_domain: 115.68.46.30.xip.io
  nats:
    machines:
    - 10.30.110.31
    password: nats
    port: 4222
    user: nats
  syslog_aggregator: null

networks:
- name: default
  subnets:
  - cloud_properties:
      name: Internal                          # vsphere 에서 사용하는 network 이름(필수)
    dns:
    - 8.8.8.8
    gateway: 10.30.20.23
    name: default_unused
    range: 10.30.0.0/16
    reserved:
    - 10.30.0.1 - 10.30.0.10                  # 설치시 제외할 IP 설정
    static:
    - 10.30.40.190 - 10.30.40.200             # 사용 가능한 IP 설정
  type: manual
properties: {}
resource_pools:                               # 배포시 사용하는 resource pools를 명시하며 여러 개의 resource pools 을 사용할 경우 name 은 unique 해야함(필수)
- cloud_properties:                           # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성을 설명 (instance_type, availability_zone), 직접 cpu, disk, 메모리 설정가능
    cpu: 1
    disk: 8192
    ram: 1024
  name: services-small                        # 고유한 resource pool 이름
  network: default
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent     # stemcell 이름(필수)
    version: "3309"                                  # stemcell 버전(필수)
```
<br>

-	MySQL 서비스팩을 배포한다.

>$ bosh deploy<br>
>※	40분 ~ 1시간 정도 소요된다.
![update_mysql_vsphere_49]
![update_mysql_vsphere_50]

<br>

-	배포된 MySQL 서비스팩을 확인한다.

>$ bosh vms paasta-mysql-service<br>
> <br>
> RSA 1024 bit CA certificates are loaded due to old openssl compatibility<br>
> Acting as user 'admin' on deployment 'paasta-mysql-service' on 'bosh'<br>
> <br>
> Director task 1396<br>
> <br>
> Task 1396 done<br>
> <br>
> +-------------------------------------------------------------------+---------+-----+----------------+--------------+<br>
> | VM                                                                | State   | AZ  | VM Type        | IPs          |<br>
> +-------------------------------------------------------------------+---------+-----+----------------+--------------+<br>
> | mysql_z1/0 (00293380-ab04-4b45-8670-6a3891c30c05)                 | running | n/a | services-small | 10.30.40.191 |<br>
> | paasta-mysql-java-broker/0 (fb9723c6-1a56-4a39-b4ef-cdf9296696f0) | running | n/a | services-small | 10.30.40.195 |<br>
> | proxy/0 (481685c7-1c23-4fe6-8f84-b3049878f1b7)                    | running | n/a | services-small | 10.30.40.194 |<br>
> +-------------------------------------------------------------------+---------+-----+----------------+--------------+<br>
> <br>
> VMs total: 3<br>

<br>


### 2.4. MySQL 서비스 브로커 등록
Mysql 서비스팩 배포가 완료 되었으면 Application에서 서비스 팩을 사용하기 위해서 먼저 MySQL 서비스 브로커를 등록해 주어야 한다.  
서비스 브로커 등록시 PaaS-TA에서 서비스브로커를 등록할 수 있는 사용자로 로그인이 되어 있어야 한다.

##### 서비스 브로커 목록을 확인한다.

>`$ cf service-brokers`

>![update_mysql_vsphere_16]

<br>

##### MySQL 서비스 브로커를 등록한다.

>`$ cf create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{서비스팩 URL(IP)}`
  
  서비스팩 이름 : 서비스 팩 관리를 위해 PaaS-TA에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스팩 리스트의 명칭이다.
  서비스팩 사용자ID / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID입니다. 서비스팩도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.
  서비스팩 URL : 서비스팩이 제공하는 API를 사용할 수 있는 URL을 입력한다.

>`$ cf create-service-broker mysql-service-broker admin cloudfoundry http://10.0.0.95:8080`

>![update_mysql_vsphere_17]

<br>

##### 등록된 MySQL 서비스 브로커를 확인한다.

>`$ cf service-brokers`

>![update_mysql_vsphere_18]

<br>

##### 접근 가능한 서비스 목록을 확인한다.

>`$ cf service-access`

>![update_mysql_vsphere_19]

>서비스 브로커 생성시 디폴트로 접근을 허용하지 않는다.

<br>

##### 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)

>`$ cf enable-service-access Mysql-DB`

>`$ cf service-access`

>![update_mysql_vsphere_20]

# 3. MySQL 연동 Sample Web App 설명
본 Sample Web App은 PaaS-TA에 배포되며 MySQL의 서비스를 Provision과 Bind를 한 상태에서 사용이 가능하다.

### 3.1. Sample Web App 구조

Sample Web App은 PaaS-TA에 App으로 배포가 된다. App을 배포하여 구동시 Bind 된 MySQL 서비스 연결정보로 접속하여 초기 데이터를 생성하게 된다. 배포 완료 후 정상적으로 App 이 구동되면 브라우져나 curl로 해당 App에 접속 하여 MySQL 환경정보(서비스 연결 정보)와 초기 적재된 데이터를 보여준다.

Sample Web App 구조는 다음과 같다.

| 이름 | 설명
| ---- | ------------
| src | Sample 소스 디렉토리
| manifest | PaaS-TA에 app 배포시 필요한 설정을 저장하는 파일
| pom.xml | 메이븐 project 설정 파일
| target | 메이븐 빌드시 생성되는 디렉토리(war 파일, classes 폴더 등)

##### PaaSTA-Sample-Apps을 다운로드 받고 Service 폴더안에 있는 MySQL Sample Web App인 hello-spring-mysql를복사한다.

>`$ ls -all`

>![update_mysql_vsphere_21]

<br>

### 3.2. PaaS-TA에서 서비스 신청
Sample Web App에서 MySQL 서비스를 사용하기 위해서는 서비스 신청(Provision)을 해야 한다.

*참고: 서비스 신청시 PaaS-TA에서 서비스를 신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.

<br>

##### 먼저 PaaS-TA Marketplace에서 서비스가 있는지 확인을 한다.

>`$ cf marketplace`

>![update_mysql_vsphere_22]

<br>

##### Marketplace에서 원하는 서비스가 있으면 서비스 신청(Provision)을 한다.

>`$ cf create-service {서비스명} {서비스플랜} {내서비스명}`

>서비스명 : p-mysql로 Marketplace에서 보여지는 서비스 명칭이다.
>서비스플랜 : 서비스에 대한 정책으로 plans에 있는 정보 중 하나를 선택한다. MySQL 서비스는 10 connection, 100 connection 를 지원한다.
>내 서비스명 : 내 서비스에서 보여지는 명칭이다. 이 명칭을 기준으로 환경설정정보를 가져온다.

>`$ cf create-service 'Mysql-DB' Mysql-Plan2-100con mysql-service-instance

>![update_mysql_vsphere_23]
<br>

##### 생성된 MySQL 서비스 인스턴스를 확인한다.

>`$ cf services`

>![update_mysql_vsphere_24]

<br>

### 3.3. Sample Web App에 서비스 바인드 신청 및 App 확인
서비스 신청이 완료되었으면 Sample Web App 에서는 생성된 서비스 인스턴스를 Bind 하여 App에서 MySQL 서비스를 이용한다. 
*참고: 서비스 Bind 신청시 PaaS-TA에서 서비스 Bind신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.

<br>

##### Sample Web App 디렉토리로 이동하여 manifest 파일을 확인한다.

>`$ cd hello-spring-mysql`

>`$ vi manifest.yml`

```yml
---
applications:
- name: hello-spring-mysql       #배포할 App 이름
  memory: 512M                # 배포시 메모리 사이즈
  instances: 1                    # 배포 인스턴스 수
path: target/hello-spring-mysql-1.0.0-BUILD-SNAPSHOT.war      #배포하는 App 파일 PATH
```

>참고: target/hello-spring-mysql-1.0.0-BUILD-SNAPSHOT.war파일이 존재 하지 않을 경우 mvn 빌드를 수행 하면 파일이 생성된다.

<br>

##### --no-start 옵션으로 App을 배포한다.  

>--no-start: App 배포시 구동은 하지 않는다.

>`$ cf push --no-start`

>![update_mysql_vsphere_25]

<br>

##### 배포된 Sample App을 확인하고 로그를 수행한다.
>`$ cf apps`

>![update_mysql_vsphere_26]

>`$ cf logs {배포된 App명}`

>`$ cf logs hello-spring-mysql`

>![update_mysql_vsphere_27]

<br>

##### Sample Web App에서 생성한 서비스 인스턴스 바인드 신청을 한다.

>`$ cf bind-service hello-spring-mysql mysql-service-instance`

>![update_mysql_vsphere_28]

<br>

##### 바인드가 적용되기 위해서 App을 재기동한다.

>`$ cf restart hello-spring-mysql`

>![update_mysql_vsphere_29]

>(참고) 바인드 후 App구동시 Mysql 서비스 접속 에러로 App 구동이 안될 경우 보안 그룹을 추가한다.  

<br>

##### rule.json 화일을 만들고 아래와 같이 내용을 넣는다.
>`$ vi rule.json`

```json
[
  {
    "protocol": "tcp",
    "destination": "10.0.0.63",
    "ports": "3306"
  }
]
```
<br>

##### 보안 그룹을 생성한다.

>`$ cf create-security-group p-mysql rule.json`

>![update_mysql_vsphere_30]

<br>

##### 모든 App에 Mysql 서비스를 사용할수 있도록 생성한 보안 그룹을 적용한다.

>`$ cf bind-running-security-group p-mysql`

>![update_mysql_vsphere_31]

<br>

##### App을 리부팅 한다.

>`$ cf restart hello-spring-mysql`

>![update_mysql_vsphere_32]

<br>

##### App이 정상적으로 MySQL 서비스를 사용하는지 확인한다.

>curl 로 확인

>`$ curl hello-spring-mysql.52.71.64.39.xip.io`

>![update_mysql_vsphere_33]

> 브라우져에서 확인
>![update_mysql_vsphere_34]

# 4. MySQL Client 툴 접속

Application에 바인딩 된 MySQL 서비스 연결정보는 Private IP로 구성되어 있기 때문에 MySQL Client 툴에서 직접 연결할수 없다. 따라서 MySQL Client 툴에서 SSH 터널, Proxy 터널 등을 제공하는 툴을 사용해서 연결하여야 한다. 본 가이드는 SSH 터널을 이용하여 연결 하는 방법을 제공하며 MySQL Client 툴로써는 오픈 소스인 HeidiSQL로 가이드한다. HeidiSQL 에서 접속하기 위해서 먼저 SSH 터널링 할수 있는 VM 인스턴스를 생성해야한다. 이 인스턴스는 SSH로 접속이 가능해야 하고 접속 후 Open PaaS 에 설치한 서비스팩에 Private IP 와 해당 포트로 접근이 가능하도록 시큐리티 그룹을 구성해야 한다. 이 부분은 vSphere관리자 및 OpenPaaS 운영자에게 문의하여 구성한다.

### 4.1. HeidiSQL 설치 및 연결

HeidiSQL 프로그램은 무료로 사용할 수 있는 오픈소스 소프트웨어이다.

##### HeidiSQL을 다운로드 하기 위해 아래 URL로 이동하여 설치파일을 다운로드 한다.

>[http://www.heidisql.com/download.php](http://www.heidisql.com/download.php)

>![mysql_vsphere_4.1.01]

<br>

##### 다운로드한 설치파일을 실행한다.

>![mysql_vsphere_4.1.02]

<br>

##### HeidSQL 설치를 위한 안내사항이다. Next 버튼을 클릭한다.

>![mysql_vsphere_4.1.03]

<br>

##### 프로그램 라이선스에 관련된 내용이다. 동의(I accept the agreement)에 체크 후 Next 버튼을 클릭한다.

>![mysql_vsphere_4.1.04]

<br>

##### HeidiSQL을 설치할 경로를 설정 후 Next 버튼을 클릭한다.

>별도의 경로 설정이 필요 없을 경우 default로 C드라이브 Program Files 폴더에 설치가 된다.

>![mysql_vsphere_4.1.05]

<br>

##### 설치 완료 후 시작메뉴에 HeidiSQL 바로가기 아이콘의 이름을 설정하는 과정이다.  
>Next 버튼을 클릭하여 다음 과정을 진행한다.

>![mysql_vsphere_4.1.06]

<br>

##### 체크박스가 4개가 있다. 아래의 경우를 고려하여 체크 및 해제를 한다.
> 
  바탕화면에 바로가기 아이콘을 생성할 경우  
  sql확장자를 HeidiSQL 프로그램으로 실행할 경우  
  heidisql 공식 홈페이지를 통해 자동으로 update check를 할 경우  
  heidisql 공식 홈페이지로 자동으로 버전을 전송할 경우

> 체크박스에 체크 설정/해제를 완료했다면 Next 버튼을 클릭한다.

>![mysql_vsphere_4.1.07]

<br>

##### 설치를 위한 모든 설정이 한번에 출력된다. 확인 후 Install 버튼을 클릭하여 설치를 진행한다.

>![mysql_vsphere_4.1.08]

<br>

##### Finish 버튼 클릭으로 설치를 완료한다.

>![mysql_vsphere_4.1.09]

<br>

##### HeidiSQL을 실행했을 때 처음 뜨는 화면이다. 이 화면에서 Server에 접속하기 위한 profile을 설정/저장하여 접속할 수 있다. 신규 버튼을 클릭한다.

>![mysql_vsphere_4.1.10]

<br>

##### 어떤 Server에 접속하기 위한 Connection 정보인지 별칭을 입력한다.

>![mysql_vsphere_4.1.11]

<br>

##### 네트워크 유형의 목록에서 MySQL(SSH tunel)을 선택한다.

>![mysql_vsphere_4.1.12]

<br>

##### 아래 붉은색 영역에 접속하려는 서버 정보를 모두 입력한다.

>![mysql_vsphere_4.1.13]

>서버 정보는 Application에 바인드되어 있는 서버 정보를 입력한다. cf env <app_name> 명령어로 이용하여 확인한다.

>**예)** $cf env hello-spring-mysql

>![mysql_vsphere_4.1.14]

<br>

##### - SSH 터널 탭을 클릭하고 OpenPaaS 운영 관리자에게 제공받은 SSH 터널링 가능한 서버 정보를 입력한다. plink.exe 위치 입력은 Putty에서 제공하는 plink.exe 실행 위치를 넣어주고 만일 해당 파일이 없을 경우 plink.exe 내려받기 링크를 클릭하여 다운받는다. 로컬 포트 정보는 임의로 넣고 열기 버튼을 클릭하면 Mysql 데이터베이스에 접속한다.

>(참고) 만일 개인 키로 접속이 가능한 경우에는 openstack용 Open PaaS Mysql 서비스팩 설치 가이드를 참고한다.

>![mysql_vsphere_4.1.15]

<br>

##### 접속이 완료되면 좌측에 스키마 정보가 나타난다. 하지만 초기설정은 테이블, 뷰, 프로시져, 함수, 트리거, 이벤트 등 모두 섞여 있어서 한눈에 구분하기가 힘들어서 접속한 DB 별칭에 마우스 오른쪽 클릭 후 "트리 방식 옵션" - "객체를 유형별로 묶기"를 클릭하면 아래 화면과 같이 각 유형별로 구분이된다.

>![mysql_vsphere_4.1.16]

<br>

##### 우측 화면에 쿼리 탭을 클릭하여 Query문을 작성한 후 실행 버튼(삼각형)을 클릭한다.  

>쿼리문에 이상이 없다면 정상적으로 결과를 얻을 수 있을 것이다.

>![mysql_vsphere_4.1.17]


[mysql_vsphere_1.3.01]:/Service-Guide/images/mysql/mysql_vsphere_1.3.01.png
[mysql_vsphere_2.2.01]:/Service-Guide/images/mysql/mysql_vsphere_2.2.01.png
[mysql_vsphere_2.2.02]:/Service-Guide/images/mysql/mysql_vsphere_2.2.02.png
[mysql_vsphere_2.2.03]:/Service-Guide/images/mysql/mysql_vsphere_2.2.03.png
[mysql_vsphere_2.2.04]:/Service-Guide/images/mysql/mysql_vsphere_2.2.04.png
[mysql_vsphere_2.2.05]:/Service-Guide/images/mysql/mysql_vsphere_2.2.05.png
[mysql_vsphere_2.2.06]:/Service-Guide/images/mysql/mysql_vsphere_2.2.06.png
[mysql_vsphere_2.2.07]:/Service-Guide/images/mysql/mysql_vsphere_2.2.07.png
[mysql_vsphere_2.2.08]:/Service-Guide/images/mysql/mysql_vsphere_2.2.08.png
[mysql_vsphere_2.3.01]:/Service-Guide/images/mysql/mysql_vsphere_2.3.01.png
[mysql_vsphere_2.3.02]:/Service-Guide/images/mysql/mysql_vsphere_2.3.02.png
[mysql_vsphere_2.3.03]:/Service-Guide/images/mysql/mysql_vsphere_2.3.03.png
[mysql_vsphere_2.3.04]:/Service-Guide/images/mysql/mysql_vsphere_2.3.04.png
[mysql_vsphere_2.3.05]:/Service-Guide/images/mysql/mysql_vsphere_2.3.05.png
[mysql_vsphere_2.3.06]:/Service-Guide/images/mysql/mysql_vsphere_2.3.06.png
[mysql_vsphere_2.3.07]:/Service-Guide/images/mysql/mysql_vsphere_2.3.07.png

[mysql_vsphere_2.4.01]:/Service-Guide/images/mysql/mysql_vsphere_2.4.01.png
[mysql_vsphere_2.4.02]:/Service-Guide/images/mysql/mysql_vsphere_2.4.02.png
[mysql_vsphere_2.4.03]:/Service-Guide/images/mysql/mysql_vsphere_2.4.03.png
[mysql_vsphere_2.4.04]:/Service-Guide/images/mysql/mysql_vsphere_2.4.04.png
[mysql_vsphere_2.4.05]:/Service-Guide/images/mysql/mysql_vsphere_2.4.05.png
[mysql_vsphere_3.1.01]:/Service-Guide/images/mysql/mysql_vsphere_3.1.01.png
[mysql_vsphere_3.2.01]:/Service-Guide/images/mysql/mysql_vsphere_3.2.01.png
[mysql_vsphere_3.2.02]:/Service-Guide/images/mysql/mysql_vsphere_3.2.02.png
[mysql_vsphere_3.2.03]:/Service-Guide/images/mysql/mysql_vsphere_3.2.03.png
[mysql_vsphere_3.3.01]:/Service-Guide/images/mysql/mysql_vsphere_3.3.01.png
[mysql_vsphere_3.3.02]:/Service-Guide/images/mysql/mysql_vsphere_3.3.02.png
[mysql_vsphere_3.3.03]:/Service-Guide/images/mysql/mysql_vsphere_3.3.03.png
[mysql_vsphere_3.3.04]:/Service-Guide/images/mysql/mysql_vsphere_3.3.04.png
[mysql_vsphere_3.3.05]:/Service-Guide/images/mysql/mysql_vsphere_3.3.05.png
[mysql_vsphere_3.3.06]:/Service-Guide/images/mysql/mysql_vsphere_3.3.06.png
[mysql_vsphere_3.3.07]:/Service-Guide/images/mysql/mysql_vsphere_3.3.07.png
[mysql_vsphere_3.3.08]:/Service-Guide/images/mysql/mysql_vsphere_3.3.08.png
[mysql_vsphere_3.3.09]:/Service-Guide/images/mysql/mysql_vsphere_3.3.09.png
[mysql_vsphere_4.1.01]:/Service-Guide/images/mysql/mysql_vsphere_4.1.01.png
[mysql_vsphere_4.1.02]:/Service-Guide/images/mysql/mysql_vsphere_4.1.02.png
[mysql_vsphere_4.1.03]:/Service-Guide/images/mysql/mysql_vsphere_4.1.03.png
[mysql_vsphere_4.1.04]:/Service-Guide/images/mysql/mysql_vsphere_4.1.04.png
[mysql_vsphere_4.1.05]:/Service-Guide/images/mysql/mysql_vsphere_4.1.05.png
[mysql_vsphere_4.1.06]:/Service-Guide/images/mysql/mysql_vsphere_4.1.06.png
[mysql_vsphere_4.1.07]:/Service-Guide/images/mysql/mysql_vsphere_4.1.07.png
[mysql_vsphere_4.1.08]:/Service-Guide/images/mysql/mysql_vsphere_4.1.08.png
[mysql_vsphere_4.1.09]:/Service-Guide/images/mysql/mysql_vsphere_4.1.09.png
[mysql_vsphere_4.1.10]:/Service-Guide/images/mysql/mysql_vsphere_4.1.10.png
[mysql_vsphere_4.1.11]:/Service-Guide/images/mysql/mysql_vsphere_4.1.11.png
[mysql_vsphere_4.1.12]:/Service-Guide/images/mysql/mysql_vsphere_4.1.12.png
[mysql_vsphere_4.1.13]:/Service-Guide/images/mysql/mysql_vsphere_4.1.13.png
[mysql_vsphere_4.1.14]:/Service-Guide/images/mysql/mysql_vsphere_4.1.14.png
[mysql_vsphere_4.1.15]:/Service-Guide/images/mysql/mysql_vsphere_4.1.15.png
[mysql_vsphere_4.1.16]:/Service-Guide/images/mysql/mysql_vsphere_4.1.16.png
[mysql_vsphere_4.1.17]:/Service-Guide/images/mysql/mysql_vsphere_4.1.17.png



[update_mysql_vsphere_01]:/Service-Guide/images/mysql/update_mysql_vsphere_01.png
[update_mysql_vsphere_02]:/Service-Guide/images/mysql/update_mysql_vsphere_02.png
[update_mysql_vsphere_03]:/Service-Guide/images/mysql/update_mysql_vsphere_03.png
[update_mysql_vsphere_04]:/Service-Guide/images/mysql/update_mysql_vsphere_04.png
[update_mysql_vsphere_05]:/Service-Guide/images/mysql/update_mysql_vsphere_05.png
[update_mysql_vsphere_06]:/Service-Guide/images/mysql/update_mysql_vsphere_06.png
[update_mysql_vsphere_07]:/Service-Guide/images/mysql/update_mysql_vsphere_07.png
[update_mysql_vsphere_08]:/Service-Guide/images/mysql/update_mysql_vsphere_08.png
[update_mysql_vsphere_09]:/Service-Guide/images/mysql/update_mysql_vsphere_09.png
[update_mysql_vsphere_10]:/Service-Guide/images/mysql/update_mysql_vsphere_10.png
[update_mysql_vsphere_11]:/Service-Guide/images/mysql/update_mysql_vsphere_11.png
[update_mysql_vsphere_12]:/Service-Guide/images/mysql/update_mysql_vsphere_12.png
[update_mysql_vsphere_13]:/Service-Guide/images/mysql/update_mysql_vsphere_13.png
[update_mysql_vsphere_14]:/Service-Guide/images/mysql/update_mysql_vsphere_14.png
[update_mysql_vsphere_15]:/Service-Guide/images/mysql/update_mysql_vsphere_15.png
[update_mysql_vsphere_16]:/Service-Guide/images/mysql/update_mysql_vsphere_16.png
[update_mysql_vsphere_17]:/Service-Guide/images/mysql/update_mysql_vsphere_17.png
[update_mysql_vsphere_18]:/Service-Guide/images/mysql/update_mysql_vsphere_18.png
[update_mysql_vsphere_19]:/Service-Guide/images/mysql/update_mysql_vsphere_19.png
[update_mysql_vsphere_20]:/Service-Guide/images/mysql/update_mysql_vsphere_20.png
[update_mysql_vsphere_21]:/Service-Guide/images/mysql/update_mysql_vsphere_21.png
[update_mysql_vsphere_22]:/Service-Guide/images/mysql/update_mysql_vsphere_22.png
[update_mysql_vsphere_23]:/Service-Guide/images/mysql/update_mysql_vsphere_23.png
[update_mysql_vsphere_24]:/Service-Guide/images/mysql/update_mysql_vsphere_24.png
[update_mysql_vsphere_25]:/Service-Guide/images/mysql/update_mysql_vsphere_25.png
[update_mysql_vsphere_26]:/Service-Guide/images/mysql/update_mysql_vsphere_26.png
[update_mysql_vsphere_27]:/Service-Guide/images/mysql/update_mysql_vsphere_27.png
[update_mysql_vsphere_28]:/Service-Guide/images/mysql/update_mysql_vsphere_28.png
[update_mysql_vsphere_29]:/Service-Guide/images/mysql/update_mysql_vsphere_29.png
[update_mysql_vsphere_30]:/Service-Guide/images/mysql/update_mysql_vsphere_30.png
[update_mysql_vsphere_31]:/Service-Guide/images/mysql/update_mysql_vsphere_31.png
[update_mysql_vsphere_32]:/Service-Guide/images/mysql/update_mysql_vsphere_32.png
[update_mysql_vsphere_33]:/Service-Guide/images/mysql/update_mysql_vsphere_33.png
[update_mysql_vsphere_34]:/Service-Guide/images/mysql/update_mysql_vsphere_34.png

[update_mysql_vsphere_35]:/Service-Guide/images/mysql/update_mysql_vsphere_35.png
[update_mysql_vsphere_36]:/Service-Guide/images/mysql/update_mysql_vsphere_36.png
[update_mysql_vsphere_37]:/Service-Guide/images/mysql/update_mysql_vsphere_37.png
[update_mysql_vsphere_38]:/Service-Guide/images/mysql/update_mysql_vsphere_38.png
[update_mysql_vsphere_39]:/Service-Guide/images/mysql/update_mysql_vsphere_39.png
[update_mysql_vsphere_40]:/Service-Guide/images/mysql/update_mysql_vsphere_40.png
[update_mysql_vsphere_41]:/Service-Guide/images/mysql/update_mysql_vsphere_41.png
[update_mysql_vsphere_42]:/Service-Guide/images/mysql/update_mysql_vsphere_42.png
[update_mysql_vsphere_43]:/Service-Guide/images/mysql/update_mysql_vsphere_43.png
[update_mysql_vsphere_44]:/Service-Guide/images/mysql/update_mysql_vsphere_44.png
[update_mysql_vsphere_45]:/Service-Guide/images/mysql/update_mysql_vsphere_45.png
[update_mysql_vsphere_46]:/Service-Guide/images/mysql/update_mysql_vsphere_46.png
[update_mysql_vsphere_47]:/Service-Guide/images/mysql/update_mysql_vsphere_47.png
[update_mysql_vsphere_48]:/Service-Guide/images/mysql/update_mysql_vsphere_48.png
[update_mysql_vsphere_49]:/Service-Guide/images/mysql/update_mysql_vsphere_49.png
[update_mysql_vsphere_50]:/Service-Guide/images/mysql/update_mysql_vsphere_50.png

