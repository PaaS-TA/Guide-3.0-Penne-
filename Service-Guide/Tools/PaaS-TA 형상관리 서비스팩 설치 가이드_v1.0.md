# [PaaS-TA 형상관리 서비스팩 설치 가이드]

## 목차
1. [문서 개요](#1)
     * [1.1. 목적](#2)
     * [1.2. 범위](#3)
     * [1.3. 시스템 구성도](#4)
     * [1.4. 참고 자료](#5)
2. [형상관리 서비스팩 설치](#6)
     * [2.1. 설치 전 준비사항](#7)
     * [2.2. 형상관리 서비스 릴리즈 업로드](#8)
     * [2.3. 형상관리 서비스 Deployment 파일 수정 및 배포](#9)
     * [2.4. 형상관리 서비스 브로커 등록](#10)
     * [2.5. 형상관리 UAA Client Id 등록](#11)



# <div id='1'/> 1. 문서 개요

### <div id='2'/> 1.1 목적
본 문서(형상관리 서비스팩 설치 가이드)는 개방형 PaaS 플랫폼 고도화 및 개발자 지원 환경 기반의 Open PaaS에서 제공되는 서비스팩인 형상관리 서비스를 Bosh를 이용하여 설치하는 방법을 기술하였다.

### <div id='3'/> 1.2 범위
설치 범위는 형상관리 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다.

### <div id='4'/> 1.3 시스템 구성도
본 문서의 설치된 시스템 구성도입니다. 형상관리 Server, 형상관리 서비스 브로커로 최소 사항을 구성하였다.
![source_controller_Service_Guide01]

### <div id='5'/> 1.4 참고 자료
> http://bosh.io/docs

> http://docs.cloudfoundry.org/


# <div id='6'/> 2. 형상관리 서비스팩 설치

### <div id='7'/> 2.1. 설치 전 준비사항
본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다.
서비스팩 설치를 위해서는 먼저 BOSH CLI가 설치되어 있어야 하고 BOSH에 로그인 및 타깃 설정이 되어 있어야 한다.
BOSH CLI 가 설치되어 있지 않을 경우 먼저 BOSH 설치 가이드 문서를 참고하여 BOSH CLI를 설치해야 한다.
PaaS-TA에서 제공하는 압축된 릴리즈 파일들을 다운로드한다. (PaaSTA-Deployment.zip, PaaSTA-Sample-Apps.zip, PaaSTA-Services.zip)

- 설치에 필요한 모든 다운로드 파일 및 문서는 다음 Url에서 찾을 수 있다. 
> https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/master/Download_Page.md


### <div id='8'/> 2.2. 형상관리 서비스 릴리즈 업로드

- PaaSTA-Services.zip 파일 압축을 풀고 폴더안에 있는 Source Control 서비스 릴리즈 paasta-sourcecontrol-release-1.0.tgz 파일을 확인한다.
```
$ cd bosh-space
$ ll paasta-sourcecontrol-release-1.0.tgz
-rw-r--r-- 1 root root 566041075 Jul  5 10:37 paasta-sourcecontrol-release-1.0.tgz
```

- 형상관리 서비스 릴리즈 파일을 업로드한다.

```
$ sudo bosh create release --force --with-tarball --name paasta-sourcecontrol-release --version 1.0
$ [WARNING] Missing blobstore configuration, please update config/final.yml before making a final release
Syncing blobs...

Building DEV release
---------------------------------
Release artifact cache: /home/inception/.bosh/cache

Building license
----------------
Building license...
  Warning: Missing LICENSE or NOTICE in /mnt/bosh-space/sjchoi/paasta-sourcecontrol-releases


Building packages
-----------------
Building paasta-sourcecontrol-webUi...
  No artifact found for paasta-sourcecontrol-webUi
  Generating...
  Generated version '763bb503bd84d6c5667341a0b22511f653038514'

Building java8...
  No artifact found for java8
  Generating...
  Generated version '88ff03b20ad2f59840fbb5f74a074ca8fdc68213'

Building paasta-sourcecontrol-api...
  No artifact found for paasta-sourcecontrol-api
  Generating...
  Generated version '61df8d658a8ba1ec0ff161ed03246851daa3cf43'

Building haproxy...
  No artifact found for haproxy
  Generating...
  Generated version '14b0441f6d68c89612f53ce4334a65c80d601e51'

Building mariadb...
  No artifact found for mariadb
  Generating...
  Generated version '59a218308c6c7dcf8795b531b53aa4a1c666ce00'

Building scm-server...
  No artifact found for scm-server
  Generating...
  Generated version '811b70f26c1f1a77c9f38e73d5289b7f91ca0f99'

Building paasta-sourcecontrol-broker...
  No artifact found for paasta-sourcecontrol-broker
  Generating...
  Generated version '9de6a698df55ee185b78cde9c4db614a48cdf899'

Building tomcat...
  No artifact found for tomcat
  Generating...
  Generated version 'd4161657b5d1f7859b89a734b01c2153bb5fe7e6'

Resolving dependencies
----------------------
Dependencies resolved, correct build order is:
- haproxy
- java8
- mariadb
- paasta-sourcecontrol-api
- paasta-sourcecontrol-broker
- paasta-sourcecontrol-webUi
- scm-server
- tomcat


Building jobs
-------------
Building paasta-sourcecontrol-webUi...
  No artifact found for paasta-sourcecontrol-webUi
  Generating...
  Generated version 'e687414aeef12f2f39717ae40d22cbdb69b59a56'

Building paasta-sourcecontrol-api...
  No artifact found for paasta-sourcecontrol-api
  Generating...
  Generated version 'c27f77078064cb4bbc12fed4a258461273cd0bc2'

Building haproxy...
  No artifact found for haproxy
  Generating...
  Generated version 'a0760261393ed4e537536b49fc9afcd984a3e90d'

Building mariadb...
  No artifact found for mariadb
  Generating...
  Generated version '91e9372c71d97e4fe594a87c06fe5b3b81c2057c'

Building scm-manager...
  No artifact found for scm-manager
  Generating...
  Generated version '8d8726450250e0648d11d10f49124ad9b3ac2e79'

Building paasta-sourcecontrol-broker...
  No artifact found for paasta-sourcecontrol-broker
  Generating...
  Generated version 'b5432b2f7364290ee4b3adbb3394d64946333e89'


Building release
----------------

Generating manifest...
----------------------
Writing manifest...

Copying packages
----------------
paasta-sourcecontrol-webUi
java8
paasta-sourcecontrol-api
haproxy
mariadb
scm-server
paasta-sourcecontrol-broker
tomcat


Copying jobs
------------
paasta-sourcecontrol-webUi
paasta-sourcecontrol-api
haproxy
mariadb
scm-manager
paasta-sourcecontrol-broker

Generated /mnt/bosh-space/sjchoi/paasta-sourcecontrol-releases/dev_releases/paasta-sourcecontrol-release/paasta-sourcecontrol-release-1.0.tgz
Release size: 539.6M

Release summary
---------------
Packages
+-----------------------------+------------------------------------------+-------------+
| Name                        | Version                                  | Notes       |
+-----------------------------+------------------------------------------+-------------+
| paasta-sourcecontrol-webUi  | 763bb503bd84d6c5667341a0b22511f653038514 | new version |
| java8                       | 88ff03b20ad2f59840fbb5f74a074ca8fdc68213 | new version |
| paasta-sourcecontrol-api    | 61df8d658a8ba1ec0ff161ed03246851daa3cf43 | new version |
| haproxy                     | 14b0441f6d68c89612f53ce4334a65c80d601e51 | new version |
| mariadb                     | 59a218308c6c7dcf8795b531b53aa4a1c666ce00 | new version |
| scm-server                  | 811b70f26c1f1a77c9f38e73d5289b7f91ca0f99 | new version |
| paasta-sourcecontrol-broker | 9de6a698df55ee185b78cde9c4db614a48cdf899 | new version |
| tomcat                      | d4161657b5d1f7859b89a734b01c2153bb5fe7e6 | new version |
+-----------------------------+------------------------------------------+-------------+

Jobs
+-----------------------------+------------------------------------------+-------------+
| Name                        | Version                                  | Notes       |
+-----------------------------+------------------------------------------+-------------+
| paasta-sourcecontrol-webUi  | e687414aeef12f2f39717ae40d22cbdb69b59a56 | new version |
| paasta-sourcecontrol-api    | c27f77078064cb4bbc12fed4a258461273cd0bc2 | new version |
| haproxy                     | a0760261393ed4e537536b49fc9afcd984a3e90d | new version |
| mariadb                     | 91e9372c71d97e4fe594a87c06fe5b3b81c2057c | new version |
| scm-manager                 | 8d8726450250e0648d11d10f49124ad9b3ac2e79 | new version |
| paasta-sourcecontrol-broker | b5432b2f7364290ee4b3adbb3394d64946333e89 | new version |
+-----------------------------+------------------------------------------+-------------+

Jobs affected by changes in this release
+-----------------------------+------------------------------------------+
| Name                        | Version                                  |
+-----------------------------+------------------------------------------+
| paasta-sourcecontrol-webUi  | e687414aeef12f2f39717ae40d22cbdb69b59a56 |
| paasta-sourcecontrol-api    | c27f77078064cb4bbc12fed4a258461273cd0bc2 |
| haproxy                     | a0760261393ed4e537536b49fc9afcd984a3e90d |
| mariadb                     | 91e9372c71d97e4fe594a87c06fe5b3b81c2057c |
| scm-manager                 | 8d8726450250e0648d11d10f49124ad9b3ac2e79 |
| paasta-sourcecontrol-broker | b5432b2f7364290ee4b3adbb3394d64946333e89 |
+-----------------------------+------------------------------------------+

Release name: paasta-sourcecontrol-release
Release version: 1.0
Release manifest: /mnt/bosh-space/sjchoi/paasta-sourcecontrol-releases/dev_releases/paasta-sourcecontrol-release/paasta-sourcecontrol-release-1.0.yml
Release tarball (539.6M): /mnt/bosh-space/sjchoi/paasta-sourcecontrol-releases/dev_releases/paasta-sourcecontrol-release/paasta-sourcecontrol-release-1.0.tgz

```

- 업로드된 형상관리 릴리즈를 확인한다.
```
$bosh releases paasta-sourcecontrol-release-1.0.tgz

--------------------------------------------+----------+-------------+
| Name                                       | Versions | Commit Hash |
+--------------------------------------------+----------+-------------+
| cf-mysql                                   | 25       | 887aae6b+   |
| delivery-ci-server-release                 | 1.0*     | 116bf724+   |
| delivery-ci-server-release-2               | 1.0*     | 116bf724+   |
| empty-java-release                         | 1.0*     | 9d027a59+   |
| paasta-mysql                               | 2.0*     | 85e3f01e+   |
| paasta-pinpoint                            | 2.0      | 2dbb8bf3+   |
| paasta-portal-object-storage               | 2.0*     | 00000000    |
| paasta-rabbitmq                            | 2.0*     | 21516d49+   |
| paasta-redis                               | 2.0*     | 2d766084+   |
| paasta-sourcecontrol-release               | 1.0*     | badc45e9+   |
| paasta-swift-release                       | 0+dev.1* | 00000000    |
| paasta-web-ide                             | 2.0      | 00000000    |
| paastaDeliveryPipeline-release             | 0+dev.1* | 00000000    |
| paastaDeliveryPipeline-release-api         | 0+dev.1* | 00000000    |
| paastaDeliveryPipeline-swift               | 0+dev.1  | 00000000    |
| paastaDeliveryPipelineScheduler-release    | 1.0      | e353814b+   |
| rex-delivery-pipeline-haproxy-test-release | 1.0*     | 00000000    |
| rex-haproxy-release                        | 1.0*     | 00000000    |
| scm-manager                                | 1.0      | b6db23fe+   |
| sourcecontrol-release                      | 1.0      | 00000000    |
| taiga-release-test                         | 0+dev.1  | 00000000    |
| tomcat-release                             | 1.0*     | ba4673ab+   |
+--------------------------------------------+----------+-------------+
(*) Currently deployed
```
형상관리 서비스 릴리즈가 업로드되어 있는 것을 확인할 수 있다.


### <div id='9'/> 2.3. 형상관리 서비스 Deployment 파일 수정 및 배포
BOSH Deployment manifest는 components 요소 및 배포의 속성을 정의한 YAML 파일이다.
Deployment manifest에는 sotfware를 설치하기 위해서 어떤 Stemcell (OS, BOSH agent) 을 사용할 것이며 Release (Software packages, Config templates, Scripts) 이름과 버전, VMs 용량, Jobs params 등을 정의가 되어 있다.

- 다운로드한 Deployment.yml 파일을 확인한다. (paasta-sourcecontrol-1.0.yml)

```
$ ls –all
```

```
total 851588
drwxrwxr-x  5 inception inception      4096 Jan  9 10:18 .
drwxrwxr-x 11 inception inception      4096 Dec 21 09:28 ..
-rw-r--r--  1 inception inception      6614 Jan  6 16:14 paasta-sourcecontrol-1.0.yml

```

- Director UUID를 확인한다.
BOSH CLI가 배포에 대한 모든 작업을 허용하기 위한 현재 대상 BOSH Director의 UUID와 일치해야 한다. ‘bosh status’ CLI 을 통해서 현재 BOSH Director에 target 되어 있는 UUID를 확인할 수 있다.


```
$ bosh status
```
```
Config
             /home/lij/.bosh_config

Director
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
  Name       bosh
  URL        https://10.30.40.105:25555
  Version    260.1.0 (00000000)
  User       admin
  UUID       e0f4af33-c72d-4e7c-a4a4-2689f70e0467
  CPI        vsphere_cpi
  dns        disabled
  compiled_package_cache disabled
  snapshots  disabled

Deployment
Manifest   /home/lij/bosh-space/mongodb-release/mariadb-release-monitering.yml
```


- Deploy 시 사용할 Stemcell을 확인한다.

```
$ bosh stemcells
```
```
lij@inception-new:~/bosh-space/paasta-sourcecontrol-releases/dev_releases/paasta-sourcecontrol-release-test$ bosh stemcells
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
```

Stemcell 목록이 존재하지 않을 경우 BOSH 설치 가이드 문서를 참고하여 Stemcell 버전을 업로드를 해야 한다.

- Deployment 파일을 서버 환경에 맞게 수정한다. (vsphere 용으로 설명, 다른 IaaS는 해당 Deployment 파일의 주석 내용을 참고)

```
yaml
# 설정 파일 내용
--
  name: paasta-sourcecontrol-release    # 서비스 배포이름(필수) bosh deployments 로 확인가능한 이름
  director_uuid: e0f4af33-c72d-4e7c-a4a4-2689f70e0467   # Director UUID을 입력(필수) bosh status 명령으로 확인가능

  compilation:          # 컴파일시 필요한 가상머신의 속성(필수)          
    workers: 4          # 컴파일 하는 가상머신의 최대수(필수)
    network: default    # Networks block에서 선언한 network 이름(필수)
    cloud_properties:   # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성 (instance_type, availability_zone), 직접 cpu,disk,ram 사이즈를 넣어도 됨
      ram: 2048
      disk: 8096
      cpu: 2

  # this section describes how updates are handled
  update:
    canaries: 1                 # canary 인스턴스 수(필수)
    canary_watch_time: 120000   # canary 인스턴스가 수행하기 위한 대기 시간(필수)
    update_watch_time: 120000   # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)
    max_in_flight: 4            # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)
  networks:
  - name: default               # 네트워크 블록에 나열된 각 서브 블록이 참조 할 수있는 작업이 네트워크 구성을 지정, 네트워크 구성은 네트워크 담당자에게 문의 하여 작성 요망
    subnets:
    - cloud_properties:
        name: Internal          # vsphere 에서 사용하는 network 이름(필수)
      dns:                      # DNS 정보
      - 10.30.20.24
      - 8.8.8.8
      gateway: 10.30.20.23
      name: default_unused
      range: 10.30.0.0/16
      reserved:                 # 설치시 제외할 IP 설정
      - 10.30.20.0 - 10.30.20.22
      - 10.30.20.24 - 10.30.20.255
      - 10.30.40.0 - 10.30.40.255
      - 10.30.60.0 - 10.30.120.255
      static:
      - 10.30.130.120 - 10.30.130.140           #사용 가능한 IP 설정
  - name: op_public_network
    type: manual
    subnets:
    - cloud_properties:
        name: External
      dns:
      - 8.8.8.8
      gateway: 115.68.46.177
      range: 115.68.46.176/28
      static:
      - 115.68.46.189
      - 115.68.46.182
  - name: op_public_network1
    type: manual
    subnets:
    - cloud_properties:
        name: External
      dns:
      - 8.8.8.8
      gateway: 115.68.46.209
      range: 115.68.46.208/28
      static:
      - 115.68.46.217
      - 115.68.46.216

  resource_pools:               # 배포시 사용하는 resource pools를 명시하며 여러 개의 resource pools 을 사용할 경우 name 은 unique 해야함(필수)
  - name: resource_pools      # 고유한 resource pool 이름
    network: default
    stemcell:
      name: bosh-vsphere-esxi-ubuntu-trusty-go_agent
      version: "3421.4"
    cloud_properties:
      cpu: 1
      disk: 8192
      ram: 2048
    env:
      bosh:
        password:

  jobs:
  - name: scm-manager
    template: scm-manager
    instances: 1
    resource_pool: resource_pools          # resource_pools block에 정의한 resource pool 이름(필수)
    persistent_disk: 10240                 # 영구적 디스크 사이즈 정의(옵션)
    networks:                              # 네트워크 구성정보
    - name: default                        # Networks block에서 선언한 network 이름(필수) 내부 네트워크 설정
      static_ips:
      - 10.30.130.121                      # 사용할 내부 IP addresses 정의(필수)
  #    - 10.30.130.120                      # 사용할 내부 IP addresses 정의(필수)
  #  - name: op_public_network
  #    default:
  #    - dns
  #    - gateway
  #    static_ips: 115.68.46.189

  - name: mariadb                   
    template: mariadb              
    instances: 1                    
    resource_pool: resource_pools          
    persistent_disk: 2048           
    networks:                       
    - name: default                
      static_ips:
      - 10.30.130.122              
  #    - 10.30.130.130               
  #  - name: op_public_network1
  #    default:
  #    - dns
  #    - gateway
  #    static_ips: 115.68.46.217

  - name: haproxy
    template: haproxy
    instances: 1
    resource_pool: resource_pools         
    networks:                      
    - name: default               
      static_ips:
      - 10.30.130.123              
  #    - 10.30.130.131              

  - name: paasta-sourcecontrol-webUi
    template: paasta-sourcecontrol-webUi
    instances: 1
    resource_pool: resource_pools
    networks:
    - name: default
      static_ips:
      - 10.30.130.124
    - name: op_public_network1
      default:
      - dns
      - gateway
      static_ips: 115.68.46.217

  - name: paasta-sourcecontrol-api
    template: paasta-sourcecontrol-api
    instances: 1
    resource_pool: resource_pools
    networks:
    - name: default
      static_ips:
      - 10.30.130.125
  #    - 10.30.130.132
  #  - name: op_public_network1
  #    default:
  #    - dns
  #    - gateway
  #    static_ips: 115.68.46.216

  - name: paasta-sourcecontrol-broker
    template: paasta-sourcecontrol-broker
    instances: 1
    resource_pool: resource_pools
    networks:
    - name: default
      static_ips:
      - 10.30.130.126
  #    - 10.30.130.137

  properties:

    mariadb:
      port: 41653
      admin_user: root
        password:
      host: 10.30.130.122
      host_names:
      - mariadb0
      host_ips:
      - 10.30.130.122

    sourcontol:
      jdbc_username: "thtmzjsxmfhf"
      jdbc_password:
      database_name: "PAASTA_SRC_DB"
      port: 41653

    haproxy:                                 # paasta-sourcecontrol haproxy
      host_name: haproxy-webui               # paasta-sourcecontrol haproxy host name
      host_ip: 10.30.130.123                   # paasta-sourcecontrol haproxy host IP
      http_port: 80                          # paasta-sourcecontrol haproxy host port

    paasta-sourcecontrol-webui:             # paasta-sourcecontrol webui ¼³Á¤Á¤º¸
      host_names:                            # paasta-sourcecontrol webui host name
      - paasta-sourcecontrol-webui
      host_ips:                              # paasta-sourcecontrol webui host IP
      - 10.30.130.124


    paasta-sourcecontrol-api:             # paasta-sourcecontrol webui ¼³Á¤Á¤º¸
      host_names:                            # paasta-sourcecontrol webui host name
      - paasta-sourcecontrol-api
      host_ips:                              # paasta-sourcecontrol webui host IP
      - 10.30.130.125

  releases:
  - name: paasta-sourcecontrol-release-test
    version: latest
```

- Deploy 할 deployment manifest 파일을 BOSH에 지정한다.
```
$ bosh deployment paasta-sourcecontrol-deployment.yml
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility '/mnt/workspace/deployments/test-deployments/paasta_glusterfs_vsphere_2.0.yml'
```

- 형상관리 서비스팩 배포 설정 및 배포한다.
```
$ bosh deploy (or bosh deploy --no-redact)
```

- 배포된 형상관리 서비스팩을 확인한다.
```
$ bosh vms paasta-sourcecontrol-vsphere
```

```
bosh vms paasta-sourcecontrol-vsphere
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on deployment 'paasta-sourcecontrol-vsphere' on 'bosh'

Director task 2567695

Task 2567695 done

+----------------------------------------------------------------------+---------+-----+----------------+---------------+
| VM                                                                   | State   | AZ  | VM Type        | IPs           |
+----------------------------------------------------------------------+---------+-----+----------------+---------------+
| mariadb/0 (57397949-8fb2-4793-a94e-ae1a41820492)                     | failing | n/a | resource_pools | 10.30.130.130 |
| paasta-sourcecontrol-api/0 (d916cf6d-da82-43d7-9a27-63953bfab811)    | running | n/a | resource_pools | 10.30.130.132 |
| paasta-sourcecontrol-broker/0 (2c005294-8f1a-4cda-a420-387af140f019) | running | n/a | resource_pools | 10.30.130.137 |
| paasta-sourcecontrol-webUi/0 (d9283a5d-79ac-4d30-9c19-c6962e06ea43)  | running | n/a | resource_pools | 10.30.130.135 |
|                                                                      |         |     |                | 115.68.46.182 |
| scm-manager/0 (4d8c25a0-58bb-45c5-aa89-d7a8397b721e)                 | running | n/a | resource_pools | 10.30.130.120 |
|                                                                      |         |     |                | 115.68.46.189 |

```

### <div id='10'/> 2.4. 형상관리 서비스 브로커 등록
형상관리 서비스팩 배포가 완료되었으면 Application에서 서비스 팩을 사용하기 위해서 먼저 형상관리 서비스 브로커를 등록해 주어야 한다.
서비스 브로커 등록 시 개방형 클라우드 플랫폼에서 서비스 브로커를 등록할 수 있는 사용자로 로그인이 되어있어야 한다.

- 서비스 브로커 목록을 확인한다.

```
$ cf service-brokers
```
```
Getting service brokers as admin...

name                          url
cubrid-service-broker         http://10.30.60.22:8080
delivery-pipline              http://paas-ta-delivery-pipline-broker.115.68.46.187.xip.io
glusterfs-service             http://10.30.120.197:8080
mysql-service-broker          http://10.30.40.195:8080
paasta-redis-broker           http://10.30.60.71:12350
```


- 형상관리 서비스 브로커를 등록한다.

```
cf create-service-broker {서비스브로커 이름} {서비스브로커 사용자ID} {서비스브로커 사용자비밀번호} http://{서비스브로커 호스트}:{서비스브로커 포트}

- 서비스브로커 이름 : 서비스 브로커 관리를 위해 PaaS-TA에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스팩 리스트의 명칭이다.
- 서비스브로커 사용자ID / 비밀번호 : 서비스 브로커에 접근할 수 있는 사용자 ID이다. 서비스팩도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.
- 서비스브로커 URL : 서비스 브로커가 제공하는 API를 사용할 수 있는 URL을 입력한다.
```
```
$ cf create-service-broker p-paasta-sourcecontrol admin cloudfoundry http://10.30.130.137
lij@inception-new:~/bosh-space/paasta-sourcecontrol-releases/deployment$ cf service-brokers
Getting service brokers as admin...

name                          url
cubrid-service-broker         http://10.30.60.22:8080
delivery-pipline              http://paas-ta-delivery-pipline-broker.115.68.46.187.xip.io
glusterfs-service             http://10.30.120.197:8080
mysql-service-broker          http://10.30.40.195:8080
paasta-redis-broker           http://10.30.60.71:12350
paasta-sourcecontrol-broker   http://10.30.130.137
rabbitmq

```

- 접근 가능한 서비스 목록을 확인한다.
```
$ cf service-access
```
- 서비스 브로커 생성 시 디폴트로 접근을 허용하지 않는다.

- 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)
```
$ cf enable-service-access p-paasta-sourcecontrol
$ cf service-access
broker : paasta-sourcecontrol-broker
service                         plan              access              orgs
p-paasta-sourcecontrol          Default           all
```

서비스 브로커 등록 시 최초에는 접근을 허용하지 않는다. 따라서 access는 none으로 설정된다.


### <div id='11'/> 2.5. 형상관리 UAA Client Id 등록
UAA 포털 계정 등록 절차에 대한 순서입니다.
아래와 같은 URL 일 경우, http://localhost:8084 까지만 입력 여러 개 입력시 ","를 이용하여 다중 입력 가능하며, 루트 도메인만 입력하면 됩니다.

 - 예시
  http://localhost:8084/dashboard/198c1dd4-6eec-4387-bf73-2e968005ccc1

<br>
- 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)
```
$ uaac target
```
```
Target : https://uaa.115.68.46.186.xip.io
Context : admin, from client admin
```
URL을 변경하고 싶을 경우 [uaac target https://uaa.115.68.46.186.xip.io] 과 같이 입력하여 변경 가능합니다. (uaac target [URL])

<br>
- 로그인을 한다.

```
$ uaac token client get
계정 : admin
비밀번호 :  password
```
```
Client ID : admin
Client secret  : ***********

Successfully fetched token via client credentials grant.
Target :  https://uaa.115.68.46.186.xip.io
Context : admin, from client admin
```

<br>
-SC포탈 계정 생성을 한다.

```
$ uaac client add scclient -s password --redirect_uri "[URL]" \
scope "cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" \
--authorized_grant_types "authorization_code , client_credentials , refresh_token" \
--authorities="uaa.resource" \
--autoapprove="openid , cloud_controller_service_permissions.read"
```

[source_controller_Service_Guide01]:/Service-Guide/images/source_control/source_controller_Service_Guide01.PNG
