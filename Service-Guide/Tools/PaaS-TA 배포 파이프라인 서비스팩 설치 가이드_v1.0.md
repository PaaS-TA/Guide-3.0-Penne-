# [PaaS-TA 배포 파이프라인 서비스팩 설치 가이드]

## 목차
1. [문서 개요](#1)
	* [1.1. 목적](#11)
	* [1.2. 범위](#12)
	* [1.3. 시스템 구성도](#13)
  	* [1.4. 참고 자료](#14)
2. [배포 파이프라인 서비스팩 설치](#2)
	* [2.1. 설치 전 준비사항](#21)  
	* [2.2. 배포 파이프라인 서비스 릴리즈 업로드](#22)
	* [2.3. 배포 파이프라인 서비스 릴리즈 Deployment 파일 수정 및 배포](#23)
	* [2.4. 배포 파이프라인 서비스 브로커 등록](#24)
  	* [2.5. 배포 파이프라인 UAA Client Id 등록](#25)

# <div id='1'/> 1. 문서 개요

### <div id='11'/> 1.1 목적
본 문서(배포 파이프라인 서비스팩 설치 가이드)는 개방형 PaaS 플랫폼 고도화 및 개발자 지원 환경 기반의 Open PaaS에서 제공되는 서비스팩인 배포 파이프라인 서비스팩을 Bosh를 이용하여 설치 및 서비스 등록하는 방법을 기술하였다.

### <div id='12'/> 1.2 범위
설치 범위는 배포 파이프라인 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다.

### <div id='13'/> 1.3 시스템 구성도
본 문서의 설치된 시스템 구성도입니다. 배포 파이프라인 Server, 형상관리 서비스 브로커로 최소사항을 구성하였다.
![1-1-3]
<table>
  <tr>
    <td>VM 명</td>
    <td>인스턴스 수</td>
		<td>vCPU 수</td>
		<td>메모리(GB)</td>
		<td>디스크(GB)</td>
  </tr>
  <tr>
    <td>HAProxy</td>
    <td>1</td>
	  <td>1</td>
	  <td>2</td>
	  <td>Root 4G</td>
  </tr>
  <tr>
		<td>WEB UI</td>
		<td>N</td>
		<td>1</td>
		<td>2</td>
		<td>Root 4G</td>
  </tr>
	<tr>
		<td>Service broker</td>
		<td>1</td>
		<td>1</td>
		<td>2</td>
		<td>Root 4G</td>
  </tr>
	<tr>
		<td>Common API</td>
		<td>N</td>
		<td>1</td>
		<td>2</td>
		<td>Root 4G</td>
  </tr>
	<tr>
		<td>DeliveryPipeline API</td>
		<td>N</td>
		<td>1</td>
		<td>2</td>
		<td>Root 4G</td>
  </tr>
	<tr>
		<td>Inspection API</td>
		<td>N</td>
		<td>1</td>
		<td>2</td>
		<td>Root 4G</td>
  </tr>
	<tr>
		<td>Storage API</td>
		<td>1</td>
		<td>1</td>
		<td>2</td>
		<td>Root 4G</td>
  </tr>
	<tr>
		<td>Scheduler</td>
		<td>1</td>
		<td>1</td>
		<td>2</td>
		<td>Root 4G</td>
  </tr>
	<tr>
		<td>DeliveryPipeline</td>
		<td>N</td>
		<td>1</td>
		<td>2</td>
		<td>Root 8G + 영구디스크 10G</td>
  </tr>
	<tr>
		<td>Inspection</td>
		<td>1</td>
		<td>1</td>
		<td>2</td>
		<td>Root 4G</td>
  </tr>
	<tr>
		<td>Storage</td>
		<td>1</td>
		<td>1</td>
		<td>4</td>
		<td>Root 4G + 영구디스크 50G</td>
  </tr>
	<tr>
		<td>DBMS(mariadb)</td>
		<td>1</td>
		<td>1</td>
		<td>4</td>
		<td>Root 6G + 영구디스크 4G</td>
  </tr>
	<tr>
		<td>Postgres</td>
		<td>1</td>
		<td>1</td>
		<td>2</td>
		<td>Root 6G + 영구디스크 4G</td>
  </tr>
</table>

### <div id='14'/> 1.4 참고 자료
> http://bosh.io/docs

> http://docs.cloudfoundry.org/

# <div id='2'/> 2. 배포 파이프라인 서비스팩 설치

### <div id='21'/> 2.1 설치 전 준비사항

본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다.
서비스팩 설치를 위해서는 먼저 BOSH CLI 가 설치 되어 있어야 하고 BOSH에 로그인 및 타깃 설정이 되어 있어야 한다.
BOSH CLI 가 설치되어 있지 않을 경우 먼저 BOSH 설치 가이드 문서를 참고하여 BOSH CLI를 설치해야 한다.
PaaS-TA에서 제공하는 압축된 릴리즈 파일들을 다운로드한다. (PaaSTA-Deployment.zip, PaaSTA-Sample-Apps.zip, PaaSTA-Services.zip)

※	설치에 필요한 모든 다운로드 파일 및 문서는 다음 Url에서 찾을 수 있다.
 > https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/master/Download_Page.md


### <div id='22'/> 2.2 배포 파이프라인 서비스 릴리즈 업로드

- 다운로드한 PaaSTA-Services.zip 압축 파일 안에 paasta-delivery-pipeline-release-1.0.tgz 를 bosh에 업로드한다.

- Release Root 디렉토리로 이동하여 업로드 되어 있는 릴리즈 목록을 확인한다.
```
$ cd ../../
$ bosh releases

RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on 'bosh'

+-----------------------------------------+----------+-------------+
| Name                                    | Versions | Commit Hash |
+-----------------------------------------+----------+-------------+
| paasta-cflinuxfs2-rootfs                | 2.0*     | fcdc42ec    |
|                                         | 3.0*     | 7276c2c8    |
| paasta-container                        | 2.0*     | b857e171    |
|                                         | 3.0*     | 4d3e57ce    |
| paasta-controller                       | 2.0*     | 0f315314    |
|                                         | 3.0*     | 32b488da    |
| paasta-cubrid                           | 2.0*     | 85e3f01e+   |
| paasta-etcd                             | 2.0*     | 20622a4a    |
| paasta-garden-runc                      | 2.0*     | ea5f5d4d+   |
|                                         | 3.0*     | 08cfdc2c    |
| paasta-glusterfs                        | 2.0*     | 85e3f01e+   |
| paasta-influxdb-grafana                 | 3.0*     | 00000000    |
| paasta-loggregator                      | 3.0*     | d4ee801f    |
| paasta-logsearch                        | 3.0*     | 00000000    |
| paasta-metrics-collector                | 3.0*     | 00000000    |
| paasta-mongodb-shard                    | 2.0      | 85e3f01e+   |
| paasta-mysql                            | 2.0*     | 85e3f01e+   |
| paasta-pinpoint                         | 2.0*     | 2dbb8bf3+   |
| paasta-portal-release			  | 1.0*     | 00000000    |
| paasta-rabbitmq                         | 2.0*     | 21516d49+   |
| paasta-redis                            | 2.0*     | 2d766084+   |
| paasta-sourcecontrol-release            | 1.0*     | badc45e9+   |
| paasta-web-ide                          | 2.0      | 00000000    |
+-----------------------------------------+----------+-------------+
(*) Currently deployed
(+) Uncommitted changes

Releases total: 19

```

- 배포 파이프라인 서비스 릴리즈를 업로드한다.
```
$ bosh upload release paasta-delivery-pipeline-release-1.0.tgz
```

- 업로드된 배포 파이프라인 릴리즈를 확인한다.
```
$ bosh releases

RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on 'bosh'

+-----------------------------------------+----------+-------------+
| Name                                    | Versions | Commit Hash |
+-----------------------------------------+----------+-------------+
| paasta-cflinuxfs2-rootfs                | 2.0*     | fcdc42ec    |
|                                         | 3.0*     | 7276c2c8    |
| paasta-container                        | 2.0*     | b857e171    |
|                                         | 3.0*     | 4d3e57ce    |
| paasta-controller                       | 2.0*     | 0f315314    |
|                                         | 3.0*     | 32b488da    |
| paasta-cubrid                           | 2.0*     | 85e3f01e+   |
| paasta-delivery-pipeline-release        | 1.0*     | 7f4eb247+   |
| paasta-etcd                             | 2.0*     | 20622a4a    |
| paasta-garden-runc                      | 2.0*     | ea5f5d4d+   |
|                                         | 3.0*     | 08cfdc2c    |
| paasta-glusterfs                        | 2.0*     | 85e3f01e+   |
| paasta-influxdb-grafana                 | 3.0*     | 00000000    |
| paasta-loggregator                      | 3.0*     | d4ee801f    |
| paasta-logsearch                        | 3.0*     | 00000000    |
| paasta-metrics-collector                | 3.0*     | 00000000    |
| paasta-mongodb-shard                    | 2.0      | 85e3f01e+   |
| paasta-mysql                            | 2.0*     | 85e3f01e+   |
| paasta-pinpoint                         | 2.0*     | 2dbb8bf3+   |
| paasta-portal-release		          | 1.0*     | 00000000    |
| paasta-rabbitmq                         | 2.0*     | 21516d49+   |
| paasta-redis                            | 2.0*     | 2d766084+   |
| paasta-sourcecontrol-release            | 1.0*     | badc45e9+   |
| paasta-web-ide                          | 2.0      | 00000000    |
+-----------------------------------------+----------+-------------+
(*) Currently deployed
(+) Uncommitted changes

Releases total: 20

```
- 배포 파이프라인 서비스 릴리즈가 업로드되어 있는 것을 확인

### <div id='23'/> 2.3 배포 파이프라인 서비스 Deployment 파일 수정 및 배포

BOSH Deployment manifest는 components 요소 및 배포의 속성을 정의한 YAML  파일이다.
Deployment manifest에는 sotfware를 설치하기 위해서 어떤 Stemcell (OS, BOSH agent) 을 사용할 것이며 Release (Software packages, Config templates, Scripts) 이름과 버전, VMs 용량, Jobs params 등을 정의가 되어 있다.

- 다운로드 받은 Deployment Yml 파일을 확인한다. (paasta-delivery-pipeline-vsphere-1.0.yml)
```
$ ls –all

inception@inception:~/bosh_space/chsong-work/delivery-pipeline-release/deployment$ ls -all
total 52
drwxrwxr-x  2 inception inception  4096 Nov 23 14:54 .
drwxrwxr-x 13 inception inception  4096 Nov 22 14:59 ..
-rw-rw-r--  1 inception inception 11926 Nov 23 14:54 paasta-delivery-pipeline-vsphere-1.0.yml
inception@inception:~/bosh_space/chsong-work/delivery-pipeline-release/deployment$

```

- Director UUID를 확인한다. BOSH CLI가 배포에 대한 모든 작업을 허용하기 위한 현재 대상 BOSH Director의 UUID와 일치해야 한다. ‘bosh status’ CLI 을 통해서 현재 BOSH Director에 target 되어 있는 UUID를 확인할수 있다.

- Deploy 시 사용할 Stemcell을 확인한다.
```
$ bosh stemcells

RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on 'bosh'

+------------------------------------------+---------------+----------+-----------------------------------------+
| Name                                     | OS            | Version  | CID                                     |
+------------------------------------------+---------------+----------+-----------------------------------------+
| bosh-vsphere-esxi-ubuntu-trusty-go_agent | ubuntu-trusty | 3215.4*  | sc-c8fa499a-5e85-4437-ab72-ef5462b01500 |
| bosh-vsphere-esxi-ubuntu-trusty-go_agent | ubuntu-trusty | 3309     | sc-dcfebaa0-15cf-416b-8cfc-80bd41399298 |
| bosh-vsphere-esxi-ubuntu-trusty-go_agent | ubuntu-trusty | 3312     | sc-4fe82b3c-a91d-4bb5-8685-2119049a824e |
| bosh-vsphere-esxi-ubuntu-trusty-go_agent | ubuntu-trusty | 3363.10* | sc-54cf5ec2-47cd-4664-b42e-7461f0f3ea17 |
| bosh-vsphere-esxi-ubuntu-trusty-go_agent | ubuntu-trusty | 3363.12  | sc-da7a2eda-873c-491c-a81c-52c0cf77eac3 |
| bosh-vsphere-esxi-ubuntu-trusty-go_agent | ubuntu-trusty | 3421.4*  | sc-99eaa7e6-a194-4517-a571-220a68106820 |
| bosh-vsphere-esxi-ubuntu-trusty-go_agent | ubuntu-trusty | 3445.2*  | sc-e45ff8e8-5a60-45aa-a365-e30532197e55 |
+------------------------------------------+---------------+----------+-----------------------------------------+

(*) Currently in-use

Stemcells total: 7

Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell 버전을 업로드를 해야 한다.
```

- Deployment 파일을 서버 환경에 맞게 수정한다. (vsphere 용으로 설명, 다른 IaaS는 해당 Deployment 파일의 주석 내용을 참고)
```
---
name: paasta-delivery-pipeline # 서비스 배포이름(필수) bosh deployments 로 확인가능한 이름
director_uuid: <%= `bosh status --uuid` %>  # Director UUID을 입력(필수) bosh status 명령으로 확인가능

release:
  name: paasta-delivery-pipeline-release #서비스 릴리즈 이름(필수) bosh releases로 확인가능
  version: latest   #서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전

compilation:          # 컴파일시 필요한 가상머신의 속성(필수)
  workers: 6          # 컴파일 하는 가상머신의 최대수(필수)
  network: default    # Networks block에서 선언한 network 이름(필수)
  cloud_properties:   # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성 (instance_type, availability_zone), 직접 cpu,disk,ram 사이즈를 넣어도 됨
    ram: 4096
    disk: 8192
    cpu: 4

# this section describes how updates are handled
update:
  canaries: 1                 # canary 인스턴스 수(필수)
  canary_watch_time: 120000    # canary 인스턴스가 수행하기 위한 대기 시간(필수)
  update_watch_time: 120000    # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)
  max_in_flight: 4            # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)

networks:                     # 네트워크 블록에 나열된 각 서브 블록이 참조 할 수있는 작업이 네트워크 구성을 지정, 네트워크 구성은 네트워크 담당자에게 문의 하여 작성 요망
- name: default
  subnets:
  - cloud_properties:
      name: Internal          # vsphere 에서 사용하는 network 이름(필수)
    dns:                      # DNS 정보
    - 10.30.20.24
    - 8.8.8.8
    gateway: 10.30.20.23
    name: default_unusedls
    range: 10.30.0.0/16
    reserved:                 # 설치시 제외할 IP 설정
    - 10.30.20.0 - 10.30.20.22
    - 10.30.20.24 - 10.30.20.255
    - 10.30.40.0 - 10.30.40.255
    - 10.30.60.0 - 10.30.60.112
    static:
    - 10.30.111.30 - 10.30.111.200          #사용 가능한 IP 설정

- name: public_network    # setting for public network
  type: manual
  subnets:
  - cloud_properties:
      name: External
    dns:
    - 8.8.8.8
    gateway: 115.68.46.209
    range: 115.68.46.208/28     # 115.68.46.178 ~ 190
    static:
    - 115.68.46.220

resource_pools:               # 배포시 사용하는 resource pools를 명시하며 여러 개의 resource pools 을 사용할 경우 name 은 unique 해야함(필수)
  - name: small      # 고유한 resource pool 이름
    network: default
    stemcell:
      name: bosh-vsphere-esxi-ubuntu-trusty-go_agent  # stemcell 이름(필수)
      version: 3445.2           # stemcell 버전(필수)
    cloud_properties:         # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성을 설명 (instance_type, availability_zone), 직접 cpu, disk, 메모리 설정가능
      cpu: 1
      disk: 4096
      ram: 2048
    env:
      bosh:
        password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0

  - name: small_api      # 고유한 resource pool 이름
    network: default
    stemcell:
      name: bosh-vsphere-esxi-ubuntu-trusty-go_agent  # stemcell 이름(필수)
      version: 3445.2           # stemcell 버전(필수)
    cloud_properties:         # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성을 설명 (instance_type, availability_zone), 직접 cpu, disk, 메모리 설정가능
      cpu: 1
      disk: 4096
      ram: 1024
    env:
      bosh:
        password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0


  - name: medium      # 고유한 resource pool 이름
    network: default
    stemcell:
      name: bosh-vsphere-esxi-ubuntu-trusty-go_agent  # stemcell 이름(필수)
      version: 3445.2           # stemcell 버전(필수)
    cloud_properties:       # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성을 설명 (instance_type, availability_zone), 직접 cpu, disk, 메모리 설정가능
      cpu: 1
      disk: 4096
      ram: 4096
    env:
      bosh:
        password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0


jobs:
- name: mariadb
  template: mariadb
  instances: 1
  resource_pool: small
  persistent_disk: 2048
  networks:
  - name: default
    static_ips:
    - 10.30.111.68

- name: postgres
  template: postgres
  instances: 1
  resource_pool: small
  persistent_disk: 4096
  networks:
  - name: default
    static_ips:
    - 10.30.111.82

- name: inspection
  template: inspection
  instances: 1
  resource_pool: small
  networks:
  - name: default
    static_ips:
    - 10.30.111.69

- name: haproxy
  template: haproxy
  instances: 1
  resource_pool: small
  networks:
  - name: default
    static_ips:
    - 10.30.111.70

- name: ci_server
  template: ci_server
  instances: 1
  resource_pool: medium
  persistent_disk: 4096
  networks:
  - name: default
    static_ips:
    - 10.30.111.71

- name: binary_storage
  template: binary_storage
  instances: 1
  persistent_disk: 10240
  resource_pool: medium
  networks:
  - name: default
    static_ips:
    - 10.30.111.39

- name: delivery-pipeline-common-api
  template: delivery-pipeline-common-api
  instances: 1
  resource_pool: small_api
  networks:
  - name: default
    static_ips:
    - 10.30.111.66

- name: delivery-pipeline-inspection-api
  template: delivery-pipeline-inspection-api
  instances: 1
  resource_pool: small_api
  networks:
  - name: default
    static_ips:
    - 10.30.111.62

- name: delivery-pipeline-binary-storage-api
  template: delivery-pipeline-binary-storage-api
  instances: 1
  resource_pool: small_api
  networks:
  - name: default
    static_ips:
    - 10.30.111.61

- name: delivery-pipeline-api
  template: delivery-pipeline-api
  instances: 1
  resource_pool: small_api
  networks:
  - name: default
    static_ips:
    - 10.30.111.65

- name: delivery-pipeline-service-broker
  template: delivery-pipeline-service-broker
  instances: 1
  resource_pool: small_api
  networks:
  - name: default
    static_ips:
    - 10.30.111.64

- name: delivery-pipeline-ui
  template: delivery-pipeline-ui
  instances: 1
  resource_pool: small_api
  networks:
  - name: default
    static_ips:
    - 10.30.111.67
  - name: public_network    # Public network 설정
    default: [dns, gateway]
    static_ips: 0.0.0.0  # change it!

- name: delivery-pipeline-scheduler
  template: delivery-pipeline-scheduler
  instances: 1
  resource_pool: small_api
  networks:
  - name: default
    static_ips:
    - 10.30.111.63

properties:
  cf:
    uaa:
      oauth:
        info:
          uri: https://uaa.(domain)/userinfo
        token:
          check:
            uri: https://uaa.(domain)/check_token
          access:
            uri: https://uaa.(domain)/oauth/token
        logout:
          uri: https://uaa.(domain)/logout.do
        authorization:
          uri: https://uaa.(domain)/oauth/authorize
        client:
          id: pipeclient
          secret: xxxxx
    api:
      url: https://api.(domain)/v2/service_instances/[SUID]/permissions

  ci_server:
    password: 'xxxx'
    admin_user:
      username: 'admin'
      password: 'xxxx'
    http_url: 'http://10.30.111.71'
    http_port: 8088
    ajp13_port: 8009
    ssh:
      username: vcap
      password: xxxxxx
      port: 22
    credentials:
      scope: "GLOBAL"
      url: "/credentials/store/system/domain/_"
      class_name: "com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl"
    workspace:
      path: "/var/vcap/store/jenkins_master/workspace/"

  mariadb:
    port: 3306
    admin_user:
      password: "xxxxx"
    host: 10.30.111.68
    host_names:
    - mariadb0
    host_ips:
    - 10.30.111.68
    datasource:
      url: jdbc:mysql://10.30.111.68:3306/delivery_pipeline?autoReconnect=true&useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Seoul&useLegacyDatetimeCode=false
      username: root
      password: "xxxx"
      driver_class_name: com.mysql.cj.jdbc.Driver
    jpa:
      database:
        name: mysql

  postgres:
    port: 5432
    host: 10.30.111.82
    vcap_password: xxxxx
    datasource:
      url: jdbc:postgresql://10.30.111.82:5432/sonar
      username: "xxxx"
      password: "xxxx"
      database: "xxxx"

  inspection:
    url: 10.30.111.69
    http_url: 'http://10.30.111.69'
    http_port: 9000
    admin:
      username: admin
      password: xxxx

  binary_storage:
    proxy_ip: 10.30.111.39              # 프록시 서버  IP  (swift-keystone job의 static_ip, Object Storage 접속 IP)
    proxy_port: 10008                   # 프록시 서버 Port (Object Storage 접속 Port)
    default_username: paasta-pipeline   # 최초 생성되는 유저이름(Object Storage 접속 유저이름)
    default_password: paasta-pipeline   # 최초 생성되는 유저 비밀번호(Object Storage 접속 유저 비밀번호)
    default_tenantname: paasta-pipeline # 최초 생성되는 테넌트 이름(Object Storage 접속 테넌트 이름)
    default_email: email@email.com      # 최소 생성되는 유저의 이메일
    container: delivery-pipeline-container
    auth_port: 5000

  common_api:
    url: 10.30.111.66:8080
    authorization:
      id: admin
      password: xxxxx
    server:
      port: 8080
    logging:
      level: DEBUG

  pipeline_api:
    url: 10.30.111.65:8080
    authorization:
      id: admin
      password: PaaS-TA
    http:
      multipart:
        max_file_size: 1000Mb
        max_request_size: 1000Mb
    server:
      port: 8080
    logging:
      level: DEBUG

  inspection_api:
    http:
      multipart:
        max_file_size: 1000Mb
        max_request_size: 1000Mb
    server:
      port: 8080
    logging:
      level: DEBUG
    url: 10.30.111.62:8080
    authorization:
      id: admin
      password: xxxxx

  binary_storage_api:
    http:
      multipart:
        max_file_size: 1000Mb
        max_request_size: 1000Mb
    server:
      port: 8080
    logging:
      level: INFO
    url: 10.30.111.61:8080
    authorization:
      id: admin
      password: xxxx

  pipeline_ui:
    http:
      multipart:
        max_file_size: 1000Mb
        max_request_size: 1000Mb
    server:
      port: 8080
    logging:
      level: DEBUG

  pipeline_scheduler:
    server:
      port: 8080
    logging:
      level: DEBUG
    quartz:
      scheduler:
        instance_name: paastaDeliveryPipelineScheduler
        instance_id: AUTO
      thread_pool:
        thread_count: 5
    job:
      start_delay: 0
      repeat_interval: 5000
      description: PaaS-TA Delivery Pipeline Scheduler
      key: StatisticsJob

  pipeline_service_broker:
    server:
      port: 9090
    logging:
      controller:
        level: DEBUG
      service:
        level: DEBUG
    dashboard:
      url: {url}

```

-	Deploy 할 deployment manifest 파일을 BOSH에 지정한다.
```
$ bosh deployment {Deployment manifest 파일 PATH}

inception@inception:~/bosh_space/chsong-work/delivery-pipeline-release/deployment$ bosh deployment paasta-delivery-pipeline-vsphere-1.0.yml
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Deployment set to '/mnt/bosh-space/chsong-work/delivery-pipeline-release/deployment/paasta-delivery-pipeline-vsphere-1.0.yml'
inception@inception:~/bosh_space/chsong-work/delivery-pipeline-release/deployment$

```
- 배포 파이프라인 서비스팩 배포 설정 및 배포한다.
```
$ bosh deploy (or bosh deploy --no-redact)

inception@inception:~/bosh_space/chsong-work/delivery-pipeline-release$ bosh deploy
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on deployment 'paasta-delivery-pipeline' on 'bosh'
Getting deployment properties from director...

Detecting deployment changes

Please review all changes carefully

Deploying
---------
Are you sure you want to deploy? (type 'yes' to continue): yes

Director task 2567236
Deprecation: Ignoring cloud config. Manifest contains 'networks' section.

  Started preparing deployment > Preparing deployment. Done (00:00:01)

  Started preparing package compilation > Finding packages to compile. Done (00:00:00)

  Started compiling packages
  Started compiling packages > delivery-pipeline-scheduler/01dea40e305407b6b107a8fe230bb37dadadca87
  Started compiling packages > delivery-pipeline-ui/1b2be30a4a31372a8ef0961e58a2f1407cdd6111
  Started compiling packages > delivery-pipeline-service-broker/abcf7fb1531e23a7b94d4013c6e7732f8772ad3d
  Started compiling packages > delivery-pipeline-api/12e1e5a06a51d42a8d14ea9e190d6798015a6d6a
  Started compiling packages > delivery-pipeline-binary-storage-api/a6ef9d12bcdb75f607e65c4823ad234ad225f9de
  Started compiling packages > delivery-pipeline-inspection-api/cb36be29250ebf5faf1c826741ab6f2f80a06853
     Done compiling packages > delivery-pipeline-ui/1b2be30a4a31372a8ef0961e58a2f1407cdd6111 (00:07:38)
  Started compiling packages > delivery-pipeline-common-api/c9fc3def1a799ddb4323206f17d26561cdc87468
     Done compiling packages > delivery-pipeline-inspection-api/cb36be29250ebf5faf1c826741ab6f2f80a06853 (00:10:38)
  Started compiling packages > swift-all-in-one/af76e98d69570ca8c886b3c7fcb23c3a1ccbbe40
     Done compiling packages > delivery-pipeline-service-broker/abcf7fb1531e23a7b94d4013c6e7732f8772ad3d (00:11:09)
  Started compiling packages > python/4e255efa754d91b825476b57e111345f200944e1
     Done compiling packages > delivery-pipeline-binary-storage-api/a6ef9d12bcdb75f607e65c4823ad234ad225f9de (00:12:40)
  Started compiling packages > ci-server/2a578577a17c12ec257aa242383cda25eb143d9c
     Done compiling packages > delivery-pipeline-api/12e1e5a06a51d42a8d14ea9e190d6798015a6d6a (00:13:22)
  Started compiling packages > sshpass/1b75eb8cd625da33e67fe33deba7b675796adb83
     Done compiling packages > delivery-pipeline-scheduler/01dea40e305407b6b107a8fe230bb37dadadca87 (00:14:21)
  Started compiling packages > maven/4b5bbaf9046d7bf024bbd34b54508c8ee84d79e8
     Done compiling packages > delivery-pipeline-common-api/c9fc3def1a799ddb4323206f17d26561cdc87468 (00:15:25)
  Started compiling packages > cf-cli/ead908636133c1f141562c33b2adc7ec182d2320
     Done compiling packages > swift-all-in-one/af76e98d69570ca8c886b3c7fcb23c3a1ccbbe40 (00:17:33)
  Started compiling packages > git/f900481c1d55f12cf907f739dc98c1cfcfeedc86
     Done compiling packages > sshpass/1b75eb8cd625da33e67fe33deba7b675796adb83 (00:17:22)
  Started compiling packages > gradle-3.5/5cd9ebb9f4d42250621db2ab754f53e47c972209
     Done compiling packages > python/4e255efa754d91b825476b57e111345f200944e1 (00:21:14)
  Started compiling packages > gradle-2.14.1/176372db7e820d54a29f7d72d8230e14179a4ace
     Done compiling packages > maven/4b5bbaf9046d7bf024bbd34b54508c8ee84d79e8 (00:18:55)
  Started compiling packages > haproxy/14b0441f6d68c89612f53ce4334a65c80d601e51
     Done compiling packages > cf-cli/ead908636133c1f141562c33b2adc7ec182d2320 (00:18:52)
  Started compiling packages > sonarqube/93eee5695e88afa04e327e1718f54ed48a077bb8
     Done compiling packages > ci-server/2a578577a17c12ec257aa242383cda25eb143d9c (00:34:09)
  Started compiling packages > java/86d8b8f8115418addf836753c1735abe547d4105
     Done compiling packages > git/f900481c1d55f12cf907f739dc98c1cfcfeedc86 (00:18:59)
  Started compiling packages > postgres/db393c97abb20a2c0d6537fabb8d429b494b9cb7
     Done compiling packages > gradle-2.14.1/176372db7e820d54a29f7d72d8230e14179a4ace (00:17:57)
  Started compiling packages > mariadb/59a218308c6c7dcf8795b531b53aa4a1c666ce00
     Done compiling packages > gradle-3.5/5cd9ebb9f4d42250621db2ab754f53e47c972209 (00:23:28)
     Done compiling packages > haproxy/14b0441f6d68c89612f53ce4334a65c80d601e51 (00:21:20)
     Done compiling packages > sonarqube/93eee5695e88afa04e327e1718f54ed48a077bb8 (00:14:47)
     Done compiling packages > java/86d8b8f8115418addf836753c1735abe547d4105 (00:11:36)
     Done compiling packages > postgres/db393c97abb20a2c0d6537fabb8d429b494b9cb7 (00:13:25)
     Done compiling packages > mariadb/59a218308c6c7dcf8795b531b53aa4a1c666ce00 (00:10:54)
     Done compiling packages (01:01:14)

  Started creating missing vms
  Started creating missing vms > mariadb/94a5a94a-cad0-4914-8589-19fecb53d761 (0)
  Started creating missing vms > postgres/6daa2ddd-bf05-4526-ba35-b0da97af249b (0)
  Started creating missing vms > inspection/2cde9424-9722-4e40-955a-70ec62e00126 (0)
  Started creating missing vms > haproxy/f2dfceb9-d3a3-4458-91b9-48c6314f7ddd (0)
  Started creating missing vms > ci_server/9a3be94e-569b-4f11-949d-78f0c9f3182b (0)
  Started creating missing vms > binary_storage/e8892948-a140-4b9b-bfb6-f8fb291a55fb (0)
  Started creating missing vms > delivery-pipeline-common-api/d9754843-da73-4316-80c9-1f49dd981faa (0)
  Started creating missing vms > delivery-pipeline-binary-storage-api/175fd5bc-d0fc-4d6f-93d0-064841ea7edd (0)
  Started creating missing vms > delivery-pipeline-inspection-api/5fe19fec-c1e6-4c84-8471-a6dba780f262 (0)
  Started creating missing vms > delivery-pipeline-service-broker/264cc8f9-df4a-4aad-b432-62dda30e182f (0)
  Started creating missing vms > delivery-pipeline-api/13c00678-8eda-4d92-990b-3862c9536632 (0)
  Started creating missing vms > delivery-pipeline-ui/2602a8d0-0d37-4d0f-bda5-bd848c6fff3c (0)
  Started creating missing vms > delivery-pipeline-scheduler/60934e83-9ead-49c0-a557-a20a244864f5 (0)
     Done creating missing vms > delivery-pipeline-service-broker/264cc8f9-df4a-4aad-b432-62dda30e182f (0) (00:14:37)
     Done creating missing vms > delivery-pipeline-inspection-api/5fe19fec-c1e6-4c84-8471-a6dba780f262 (0) (00:15:28)
     Done creating missing vms > delivery-pipeline-binary-storage-api/175fd5bc-d0fc-4d6f-93d0-064841ea7edd (0) (00:15:29)
     Done creating missing vms > binary_storage/e8892948-a140-4b9b-bfb6-f8fb291a55fb (0) (00:19:32)
     Done creating missing vms > ci_server/9a3be94e-569b-4f11-949d-78f0c9f3182b (0) (00:20:51)
     Done creating missing vms > inspection/2cde9424-9722-4e40-955a-70ec62e00126 (0) (00:20:52)
     Done creating missing vms > postgres/6daa2ddd-bf05-4526-ba35-b0da97af249b (0) (00:21:50)
     Done creating missing vms > mariadb/94a5a94a-cad0-4914-8589-19fecb53d761 (0) (00:23:52)
     Done creating missing vms > haproxy/f2dfceb9-d3a3-4458-91b9-48c6314f7ddd (0) (00:23:52)
     Done creating missing vms > delivery-pipeline-api/13c00678-8eda-4d92-990b-3862c9536632 (0) (00:24:13)
     Done creating missing vms > delivery-pipeline-ui/2602a8d0-0d37-4d0f-bda5-bd848c6fff3c (0) (00:25:37)
     Done creating missing vms > delivery-pipeline-common-api/d9754843-da73-4316-80c9-1f49dd981faa (0) (00:25:38)
     Done creating missing vms > delivery-pipeline-scheduler/60934e83-9ead-49c0-a557-a20a244864f5 (0) (00:25:48)
     Done creating missing vms (00:25:48)

 Started updating instance mariadb > mariadb/94a5a94a-cad0-4914-8589-19fecb53d761 (0) (canary). Done (00:06:32)
  Started updating instance postgres > postgres/6daa2ddd-bf05-4526-ba35-b0da97af249b (0) (canary). Done (00:04:58)
  Started updating instance inspection > inspection/2cde9424-9722-4e40-955a-70ec62e00126 (0) (canary). Done (00:02:46)
  Started updating instance haproxy > haproxy/f2dfceb9-d3a3-4458-91b9-48c6314f7ddd (0) (canary). Done (00:02:28)
  Started updating instance ci_server > ci_server/9a3be94e-569b-4f11-949d-78f0c9f3182b (0) (canary)

..
..
..


Task 2567236 done

Started		2017-11-23 07:11:07 UTC
Finished	2017-11-23 07:11:07 UTC
Duration	00:00:00

deploy 'paasta-delivery-pipeline'

```

- 배포된 배포 파이프라인 서비스팩을 확인한다.
```
$ bosh vms paasta-delivery-pipeline

RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on deployment 'paasta-delivery-pipeline' on 'bosh'

Director task 2567345

Task 2567345 done

+-------------------------------------------------------------------------------+---------+-----+-----------+---------------+
| VM                                                                            | State   | AZ  | VM Type   | IPs           |
+-------------------------------------------------------------------------------+---------+-----+-----------+---------------+
| binary_storage/0 (e8892948-a140-4b9b-bfb6-f8fb291a55fb)                       | running | n/a | medium    | 10.30.111.39  |
| ci_server/0 (9a3be94e-569b-4f11-949d-78f0c9f3182b)                            | running | n/a | medium    | 10.30.111.71  |
| delivery-pipeline-api/0 (13c00678-8eda-4d92-990b-3862c9536632)                | running | n/a | small_api | 10.30.111.65  |
| delivery-pipeline-binary-storage-api/0 (175fd5bc-d0fc-4d6f-93d0-064841ea7edd) | running | n/a | small_api | 10.30.111.61  |
| delivery-pipeline-common-api/0 (d9754843-da73-4316-80c9-1f49dd981faa)         | running | n/a | small_api | 10.30.111.66  |
| delivery-pipeline-inspection-api/0 (5fe19fec-c1e6-4c84-8471-a6dba780f262)     | running | n/a | small_api | 10.30.111.62  |
| delivery-pipeline-scheduler/0 (60934e83-9ead-49c0-a557-a20a244864f5)          | running | n/a | small_api | 10.30.111.63  |
| delivery-pipeline-service-broker/0 (264cc8f9-df4a-4aad-b432-62dda30e182f)     | running | n/a | small_api | 10.30.111.64  |
| delivery-pipeline-ui/0 (2602a8d0-0d37-4d0f-bda5-bd848c6fff3c)                 | running | n/a | small_api | 10.30.111.67  |
|                                                                               |         |     |           | 115.68.46.220 |
| haproxy/0 (f2dfceb9-d3a3-4458-91b9-48c6314f7ddd)                              | running | n/a | small     | 10.30.111.70  |
| inspection/0 (2cde9424-9722-4e40-955a-70ec62e00126)                           | running | n/a | small     | 10.30.111.69  |
| mariadb/0 (94a5a94a-cad0-4914-8589-19fecb53d761)                              | stopped | n/a | small     | 10.30.111.68  |
| postgres/0 (6daa2ddd-bf05-4526-ba35-b0da97af249b)                             | running | n/a | small     | 10.30.111.82  |
+-------------------------------------------------------------------------------+---------+-----+-----------+---------------+

VMs total: 13

```

### <div id='24'/> 2.4 배포 파이프라인 서비스 브로커 등록

배포 파이프라인 서비스팩 배포가 완료되었으면 파스-타 포탈에서 서비스 팩을 사용하기 위해서 먼저 배포 파이프라인 서비스 브로커를 등록해 주어야 한다.
서비스 브로커 등록 시 개방형 클라우드 플랫폼에서 서비스 브로커를 등록할 수 있는 사용자로 로그인이 되어있어야 한다.

- 서비스 브로커 목록을 확인한다.
```
$ cf service-brokers

Getting service brokers as admin...

name                          url
cubrid-service-broker         http://10.30.60.22:8080
glusterfs-service             http://10.30.120.197:8080
mysql-service-broker          http://10.30.40.195:8080
paasta-redis-broker           http://10.30.60.71:12350
```

- 배포 파이프라인 서비스 브로커를 등록한다.
```
$ cf create-service-broker {서비스팩 이름}{서비스팩 사용자ID}{서비스팩 사용자비밀번호} http://{서비스팩 URL}

-	서비스팩 이름 : 서비스 팩 관리를 위해 개방형 클라우드 플랫폼에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스팩 리스트의 명칭이다.
-	서비스팩 사용자ID / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID입니다. 서비스팩도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.
-	서비스팩 URL : 서비스팩이 제공하는 API를 사용할 수 있는 URL을 입력한다.

$ cf create-service-broker delivery-pipeline admin cloudfoundry http://10.30.111.64:9090

```
- 등록된 배포 파이프라인 서비스 브로커를 확인한다.
```
$ cf service-brokers

Getting service brokers as admin...

name                          url
cubrid-service-broker         http://10.30.60.22:8080
glusterfs-service             http://10.30.120.197:8080
delivery-pipeline             http://10.30.111.64:9090
mysql-service-broker          http://10.30.40.195:8080
paasta-redis-broker           http://10.30.60.71:12350

```
- 접근 가능한 서비스 목록을 확인한다.
```
$ cf service-access
서비스 브로커 생성시 디폴트로 접근을 허용하지 않는다.
Getting service access as admin...
broker: paasta-redis-broker
   service   plan           access   orgs
   redis     shared-vm      all
   redis     dedicated-vm   all

broker: cubrid-service-broker
   service    plan    access   orgs
   CubridDB   utf8    all
   CubridDB   euckr   all

broker: glusterfs-service
   service     plan               access   orgs
   glusterfs   glusterfs-5Mb      all
   glusterfs   glusterfs-100Mb    all
   glusterfs   glusterfs-1000Mb   all

broker: mysql-service-broker
   service    plan                 access   orgs
   Mysql-DB   Mysql-Plan1-10con    all
   Mysql-DB   Mysql-Plan2-100con   all

broker: rabbitmq-service-broker
   service      plan       access   orgs
   p-rabbitmq   standard   all

broker: delivery-pipeline
   service             plan      access   orgs
   delivery-pipeline   default   none

```

- 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)
```
$ cf enable-service-access delivery-pipeline default
Getting service access as admin...
broker: paasta-redis-broker
   service   plan           access   orgs
   redis     shared-vm      all
   redis     dedicated-vm   all

broker: cubrid-service-broker
   service    plan    access   orgs
   CubridDB   utf8    all
   CubridDB   euckr   all

broker: glusterfs-service
   service     plan               access   orgs
   glusterfs   glusterfs-5Mb      all
   glusterfs   glusterfs-100Mb    all
   glusterfs   glusterfs-1000Mb   all

broker: mysql-service-broker
   service    plan                 access   orgs
   Mysql-DB   Mysql-Plan1-10con    all
   Mysql-DB   Mysql-Plan2-100con   all

broker: rabbitmq-service-broker
   service      plan       access   orgs
   p-rabbitmq   standard   all

broker: delivery-pipeline
   service             plan      access   orgs
   delivery-pipeline   default   all

```

### <div id='25'/> 2.5 배포 파이프라인 UAA Client Id 등록

UAA 포털 계정 등록 절차에 대한 순서입니다.
아래와 같은 URL 일 경우, http://localhost:8084 까지만 입력 여러 개 입력시 ","를 이용하여 다중 입력 가능하며, 루트 도메인만 입력하면 됩니다.
예) http://localhost:8084/dashboard/198c1dd4-6eec-4387-bf73-2e968005ccc1


- 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)
```
$ uaac target

inception@inception-new:~$ uaac target

Target: https://uaa.115.68.46.186.xip.io
Context: admin, from client admin

```

- URL을 변경하고 싶을 경우 [uaac target https://uaa.115.68.46.186.xip.io]과 같이 입력하여 변경 가능합니다. (uaac target [URL])

- 로그인을 한다.
```
$ uaac token client get
계정 : admin
비번: admin-secret

inception@inception-new:~$ uaac token client get
Client ID: admin
Client secret: ************

Successfully fetched token via client credentials grant.
Target: https://uaa.115.68.46.186.xip.io
Context: admin, from client admin

```
- 파이프라인 포탈 계정 생성을 한다.
```
$ uaac client add pipeclient -s clientsecret --redirect_uri "[URL]" /
--scope "cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" /
--authorized_grant_types "authorization_code , client_credentials , refresh_token" /
--authorities="uaa.resource" /
--autoapprove="openid , cloud_controller_service_permissions.read"

```

[1-1-3]:/Service-Guide/images/pipeline/Delivery_Pipeline_Architecture.jpg
