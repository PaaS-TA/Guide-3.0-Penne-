
## Table of Contents
1. [문서 개요](#1)
     * [1.1. 목적](#2)
     * [1.2. 범위](#3)
     * [1.3. 시스템 구성도](#4)
     * [1.4. 참고자료](#5)
2. [Cubrid 서비스팩 설치](#6)
     * [2.1. 설치전 준비사항](#7)
     * [2.2. Cubrid 서비스 릴리즈 업로드](#8)
     * [2.3. Cubrid 서비스 Deployment 파일 수정 및 배포](#9)
     * [2.4. Cubrid 서비스 브로커 등록](#10)
3. [Cubrid 연동 Sample App 설명](#11)
     * [3.1. Sample App 구조](#12)
     * [3.2. PaaS-TA에서 서비스 신청](#13)
     * [3.3. Sample App에 서비스 바인드 신청 및 App 확인](#14)
4. [Cubrid Client 툴 접속](#15)
     * [4.1. Putty 다운로드 및 터널링](#16)
     * [4.2. Cubrid Manager 설치 및 연결](#17)

     


# <div id='1'> 1. 문서 개요


### <div id='2'> 1.1. 목적
      
본 문서(Cubrid 서비스팩 설치 가이드)는 전자정부표준프레임워크 기반의 PaaS-TA에서 제공되는 서비스팩인 Cubrid 서비스팩을 Bosh를 이용하여 설치 하는 방법과 PaaS-TA의 SaaS 형태로 제공하는 Application 에서 Cubrid 서비스를 사용하는 방법을 기술하였다.


### <div id='3'> 1.2. 범위 

설치 범위는 Cubrid 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다. 


### <div id='4'> 1.3. 시스템 구성도
본 문서의 설치된 시스템 구성도입니다. Cubrid Server, Cubrid 서비스 브로커로 최소사항을 구성하였다.  
![시스템 구성도][1-3-0-0]

<table>
  <tr>
    <td>구분</td>
    <td>스펙</td>
  </tr>
   <tr>
    <td>(cubrid_resource_pools)</td>
    <td>1vCPU / 1GB RAM / 8GB Disk</td>
  </tr>
  <tr>
    <td>paasta-cubrid-broker</td>
    <td>(cubrid_resource_pools)</td>
  </tr>
  <tr>
    <td>Cubrid</td>
    <td>(cubrid_resource_pools)</td>
  </tr>
</table>


### <div id='5'> 1.4. 참고자료
**<http://bosh.io/docs>**  
**<http://docs.cloudfoundry.org/>**


# <div id='6'>  2. Cubrid 서비스팩 설치


### <div id='7'> 2.1. 설치전 준비사항
본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다.  
서비스팩 설치를 위해서는 먼저 BOSH CLI 가 설치 되어 있어야 하고 BOSH 에 로그인 및 target 설정이 되어 있어야 한다.<br>
BOSH CLI 가 설치 되어 있지 않을 경우 먼저 BOSH 설치 가이드 문서를 참고 하여 BOSH CLI를 설치 해야 한다.<br>

- PaaS-TA에서 제공하는 압축된 릴리즈 파일들을 다운받는다. (PaaSTA-Deployment.zip, PaaSTA-Sample-Apps.zip, PaaSTA-Services.zip)

- 다운로드 위치  
>PaaSTA-Services : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Services.zip>**  
>PaaSTA-Deployment : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Deployment.zip>**  
>PaaSTA-Sample-Apps : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Sample-Apps.zip>**

<br>


### <div id='8'>  2.2. Cubrid 서비스 릴리즈 업로드

- Release Root 디렉토리로 이동하여 업로드 되어 있는 릴리즈 목록을 확인한다.

>$ cd ../../<br>
>$ bosh releases<br>
<br>
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
>| openpaas-paasta-pinpoint             | 2.0*      | 34e02d07+   |<br>
>| openpaas-redis                       | 1.0*      | af975e0f    |<br>
>| paasta-eclipse-che                   | 2.0*      | 00000000    |<br>
>| paasta-glusterfs                     | 2.0*      | 85e3f01e+   |<br>
>| paasta-mysql                         | 2.0*      | 85e3f01e+   |<br>
>| paasta-portal-object-storage-release | 0+dev.1*  | 00000000    |<br>
>| paasta-redis                         | 2.0       | 2d766084+   |<br>
>+--------------------------------------+-----------+-------------+<br>
>(*) Currently deployed<br>
>(+) Uncommitted changes<br>
><br>
>Releases total: 13<br>
>Cubrid 서비스 릴리즈가 업로드 되어 있지 않은 것을 확인

- Cubrid 서비스 릴리즈를 업로드한다.

>$ bosh upload release {서비스 릴리즈 파일 PATH}<br>
><br>
>$ bosh upload release paasta-cubrid-2.0.tgz<br>
><br>
>RSA 1024 bit CA certificates are loaded due to old openssl compatibility<br>
>Acting as user 'admin' on 'bosh'<br>
><br>
>Verifying manifest...<br>
>Extract manifest                                             OK<br>
>Manifest exists                                              OK<br>
>Release name/version                                         OK<br>
><br>
>File exists and readable                                     OK<br>
>Read package 'cubrid' (1 of 4)                               OK<br>
>Package 'cubrid' checksum                                    OK<br>
>Read package 'java7' (2 of 4)                                OK<br>
>Package 'java7' checksum                                     OK<br>
>Read package 'cli' (3 of 4)                                  OK<br>
>Package 'cli' checksum                                       OK<br>
>Read package 'cubrid_broker' (4 of 4)                        OK<br>
>Package 'cubrid_broker' checksum                             OK<br>
>Package dependencies                                         OK<br>
>Checking jobs format                                         OK<br>
>Read job 'cubrid' (1 of 4), version 45427cb5f3b6c86df80bb2de38c12e98aec7b95f OK<br>
>Job 'cubrid' checksum                                        OK<br>
>Extract job 'cubrid'                                         OK<br>
>Read job 'cubrid' manifest                                   OK<br>
>Check template 'cubrid_ctl.erb' for 'cubrid'                 OK<br>
>Check template 'cubrid.conf.erb' for 'cubrid'                OK<br>
>Check template 'cubrid_broker.conf.erb' for 'cubrid'         OK<br>
>Check template 'cubrid_broker_init.sql.erb' for 'cubrid'     OK<br>
>Check template '.cubrid.sh.erb' for 'cubrid'                 OK<br>
>Job 'cubrid' needs 'cubrid' package                          OK<br>
>Monit file for 'cubrid'                                      OK<br>
>Read job 'cubrid_broker_deregistrar' (2 of 4), version 49602e528fa68a557ece12688b6b278a1134ed27 OK<br>
>Job 'cubrid_broker_deregistrar' checksum                     OK<br>
>Extract job 'cubrid_broker_deregistrar'                      OK<br>
>Read job 'cubrid_broker_deregistrar' manifest                OK<br>
>Check template 'errand.sh.erb' for 'cubrid_broker_deregistrar' OK<br>
>Job 'cubrid_broker_deregistrar' needs 'cli' package          OK<br>
>Monit file for 'cubrid_broker_deregistrar'                   OK<br>
>Read job 'cubrid_broker' (3 of 4), version c73852bd8115be216e32b69bc6ee5f0bb5444b06 OK<br>
>Job 'cubrid_broker' checksum                                 OK<br>
>Extract job 'cubrid_broker'                                  OK<br>
>Read job 'cubrid_broker' manifest                            OK<br>
>Check template 'bin/cubrid_broker_ctl' for 'cubrid_broker'   OK<br>
>Check template 'bin/monit_debugger' for 'cubrid_broker'      OK<br>
>Check template 'data/properties.sh.erb' for 'cubrid_broker'  OK<br>
>Check template 'helpers/ctl_setup.sh' for 'cubrid_broker'    OK<br>
>Check template 'helpers/ctl_utils.sh' for 'cubrid_broker'    OK<br>
>Check template 'config/cubrid_broker.yml.erb' for 'cubrid_broker' OK<br>
>Check template 'config/application-mvc.properties.erb' for 'cubrid_broker' OK<br>
>Check template 'config/datasource.properties.erb' for 'cubrid_broker' OK<br>
>Check template 'config/logback.xml.erb' for 'cubrid_broker'  OK<br>
>Check template 'config/bosh.pem.erb' for 'cubrid_broker'     OK<br>
>Job 'cubrid_broker' needs 'cubrid_broker' package            OK<br>
>Job 'cubrid_broker' needs 'java7' package                    OK<br>
>Monit file for 'cubrid_broker'                               OK<br>
>Read job 'cubrid_broker_registrar' (4 of 4), version 8a89128e95a9707a30697bda7a9d7a678a2fd109 OK<br>
>Job 'cubrid_broker_registrar' checksum                       OK<br>
>Extract job 'cubrid_broker_registrar'                        OK<br>
>Read job 'cubrid_broker_registrar' manifest                  OK<br>
>Check template 'errand.sh.erb' for 'cubrid_broker_registrar' OK<br>
>Job 'cubrid_broker_registrar' needs 'cli' package            OK<br>
>Monit file for 'cubrid_broker_registrar'                     OK<br>
><br>
>Release info<br>
>------------<br>
>Name:    paasta-cubrid<br>
>Version: 2.0<br>
><br>
>Packages<br>
>  - cubrid (36065bb22d1e816657d176c902246231347361e2)<br>
>  - java7 (cb28502f6e89870255182ea76e9029c7e9ec1862)<br>
>  - cli (24305e50a638ece2cace4ef4803746c0c9fe4bb0)<br>
>  - cubrid_broker (25717cfb95347c7ca5ed1e6cbdda701315789cfc)<br>
><br>
>Jobs<br>
>  - cubrid (45427cb5f3b6c86df80bb2de38c12e98aec7b95f)<br>
>  - cubrid_broker_deregistrar (49602e528fa68a557ece12688b6b278a1134ed27)<br>
>  - cubrid_broker (c73852bd8115be216e32b69bc6ee5f0bb5444b06)<br>
>  - cubrid_broker_registrar (8a89128e95a9707a30697bda7a9d7a678a2fd109)<br>
><br>
>License<br>
>  - none<br>
><br>
>Checking if can repack release for faster upload...<br>
>cubrid (36065bb22d1e816657d176c902246231347361e2) UPLOAD<br>
>java7 (cb28502f6e89870255182ea76e9029c7e9ec1862) SKIP<br>
>cli (24305e50a638ece2cace4ef4803746c0c9fe4bb0) SKIP<br>
>cubrid_broker (25717cfb95347c7ca5ed1e6cbdda701315789cfc) UPLOAD<br>
>cubrid (45427cb5f3b6c86df80bb2de38c12e98aec7b95f) UPLOAD<br>
>cubrid_broker_deregistrar (49602e528fa68a557ece12688b6b278a1134ed27) UPLOAD<br>
>cubrid_broker (c73852bd8115be216e32b69bc6ee5f0bb5444b06) UPLOAD<br>
>cubrid_broker_registrar (8a89128e95a9707a30697bda7a9d7a678a2fd109) UPLOAD<br>
>Release repacked (new size is 181.8M)<br>
><br>
>Uploading release<br>
>release-repac:  96%<br> |ooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo         | 174.5MB  26.1MB/s ETA:  00:00:00<br>
>Director task 1285<br>
>  Started extracting release > Extracting release. Done (00:00:02)<br>
><br>
>  Started verifying manifest > Verifying manifest. Done (00:00:00)<br>
><br>
>  Started resolving package dependencies > Resolving package dependencies. Done (00:00:00)<br>
><br>
>  Started creating new packages<br>
>  Started creating new packages > cubrid/36065bb22d1e816657d176c902246231347361e2. Done (00:00:03)<br>
>  Started creating new packages > java7/cb28502f6e89870255182ea76e9029c7e9ec1862. Done (00:00:03)<br>
>  Started creating new packages > cli/24305e50a638ece2cace4ef4803746c0c9fe4bb0. Done (00:00:00)<br>
>  Started creating new packages > cubrid_broker/25717cfb95347c7ca5ed1e6cbdda701315789cfc. Done (00:00:01)<br>
>     Done creating new packages (00:00:07)<br>
><br>
>  Started creating new jobs<br>
>  Started creating new jobs > cubrid/45427cb5f3b6c86df80bb2de38c12e98aec7b95f. Done (00:00:00)<br>
>  Started creating new jobs > cubrid_broker_deregistrar/49602e528fa68a557ece12688b6b278a1134ed27. Done (00:00:00)<br>
>  Started creating new jobs > cubrid_broker/c73852bd8115be216e32b69bc6ee5f0bb5444b06. Done (00:00:00)<br>
>  Started creating new jobs > cubrid_broker_registrar/8a89128e95a9707a30697bda7a9d7a678a2fd109. Done (00:00:00)<br>
>     Done creating new jobs (00:00:00)<br>
><br>
>  Started release has been created > paasta-cubrid/2.0. Done (00:00:00)<br>
><br>
>Task 1285 done<br>
><br>
>Started		2017-01-06 06:52:33 UTC<br>
>Finished	2017-01-06 06:52:42 UTC<br>
>Duration	00:00:09<br>
>release-repac:  96%<br> |ooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo         | 174.9MB  10.9MB/s Time: 00:00:16<br>
><br>
>Release uploaded<br>
<br>

- 업로드 된 Cubrid 릴리즈를 확인한다. 

>$ bosh releases<br>
><br>
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
>| openpaas-paasta-pinpoint             | 2.0*      | 34e02d07+   |<br>
>| openpaas-redis                       | 1.0*      | af975e0f    |<br>
>| paasta-cubrid                        | 2.0       | 85e3f01e+   |<br>
>| paasta-eclipse-che                   | 2.0*      | 00000000    |<br>
>| paasta-glusterfs                     | 2.0*      | 85e3f01e+   |<br>
>| paasta-mysql                         | 2.0*      | 85e3f01e+   |<br>
>| paasta-portal-object-storage-release | 0+dev.1*  | 00000000    |<br>
>| paasta-redis                         | 2.0       | 2d766084+   |<br>
>+--------------------------------------+-----------+-------------+<br>
>(*) Currently deployed<br>
>(+) Uncommitted changes<br>
><br>
>Releases total: 14<br>
>Cubrid 서비스 릴리즈가 업로드 되어 있는 것을 확인<br>
<br>


### <div id='9'> 2.3.  Cubrid 서비스 Deployment 파일 수정 및 배포
BOSH Deployment manifest 는 components 요소 및 배포의 속성을 정의한 YAML  파일이다.
Deployment manifest 에는 sotfware를 설치 하기 위해서 어떤 Stemcell (OS, BOSH agent) 을 사용할것이며 Release (Software packages, Config templates, Scripts) 이름과 버전, VMs 용량, Jobs params 등을 정의가 되어 있다.

- PaaSTA-Deployment.zip 파일 압축을 풀고 폴더안에 있는 IaaS별 Cubrid Deployment 파일을 복사한다.
  예) vsphere 일 경우 paasta_cubrid_vsphere_2.0.yml를 복사
  다운로드 받은 Deployment Yml 파일을 확인한다. (pasta_cubrid_vsphere_2.0.yml)


>$ ls –all<br>
><br>
>total 851588<br>
>drwxrwxr-x  5 inception inception      4096 Jan  9 10:18 .<br>
>drwxrwxr-x 11 inception inception      4096 Dec 21 09:28 ..<br>
><br>
>-rw-r--r--  1 inception inception      6614 Jan  6 16:14 paasta_cubrid_vsphere_2.0.yml<br>
>-rw-rw-r--  1 inception inception      6382 Jan  9 10:18 paasta_mysql_vsphere_2.0.yml<br>
<br>

- Director UUID를 확인한다.
- BOSH CLI가 배포에 대한 모든 작업을 허용하기위한 현재 대상 BOSH Director의 UUID와 일치해야한다. ‘bosh status’ CLI 을 통해서 현재 BOSH Director 에 target 되어 있는 UUID를 확인할수 있다.

><div>$ bosh status
>![2-1-0-9]

- Deploy시 사용할 Stemcell을 확인한다.
 
><div>$ bosh stemcells
>![2-1-0-10]
><div>Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell 3215.4 버전을 업로드를 해야 한다.

- Deployment 파일을 서버 환경에 맞게 수정한다. (vsphere 용으로 설명, 다른 IaaS는 해당 Deployment 파일의 주석내용을 참고)

>$ vi paasta-cubrid-vsphere-1.0.yml 
```yaml
# paasta-cubrid-vsphere 설정 파일 내용
---
name: paasta-cubrid-service  # 서비스 배포이름(필수)
director_uuid: 0bc8d3c2-e032-4c7e-a99c-e23eea7091fc #bosh status 에서 확인한 Director UUID을 입력(필수)
>
release:
  name: paasta-cubrid  #서비스 릴리즈 이름(필수)
  version: 1.0   #서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전
>
compilation:          # 컴파일시 필요한 가상머신의 속성(필수)
  workers: 4          # 컴파일 하는 가상머신의 최대수(필수)
  network: default    # Networks block에서 선언한 network 이름(필수)
  cloud_properties:   # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성 (instance_type, availability_zone), 직접 cpu,disk,ram 사이즈를 넣어도 됨
    ram: 2048
    disk: 8096
    cpu: 2
>
# this section describes how updates are handled
update:
  canaries: 1                 # canary 인스턴스 수(필수)
  canary_watch_time: 30000    # canary 인스턴스가 수행하기 위한 대기 시간(필수)
  update_watch_time: 30000    # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)
  max_in_flight: 4            # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)
>
networks:                     # 네트워크 블록에 나열된 각 서브 블록이 참조 할 수있는 작업이 네트워크 구성을 지정, 네트워크 구성은 네트워크 담당자에게 문의 하여 작성 요망
- name: default
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
    - 10.30.0.1 - 10.30.20.22
    - 10.30.20.24 - 10.30.40.255
    static:
    - 10.30.60.11 - 10.30.60.70 #사용 가능한 IP 설정
>
resource_pools:               # 배포시 사용하는 resource pools를 명시하며 여러 개의 resource pools 을 사용할 경우 name 은 unique 해야함(필수)
  - name: cubrid_resource_pools # 고유한 resource pool 이름
    network: default
    stemcell:
      name: bosh-vsphere-esxi-ubuntu-trusty-go_agent  # stemcell 이름(필수)
      version: 3215.4                # stemcell 버전(3215.4 버전으로 사용, aws 3263.28)
    cloud_properties:         # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성을 설명 (instance_type, availability_zone), 직접 cpu, disk, 메모리 설정가능
      cpu: 1
      disk: 8192
      ram: 4096
>
jobs:
  - name: cubrid  #작업 이름(필수): Cubrid 서버
    template: cubrid  # job template 이름(필수)
    instances: 1  # job 인스턴스 수(필수)
    resource_pool: cubrid_resource_pools  # resource_pools block에 정의한 resource pool 이름(필수)
    persistent_disk: 16384  # 영구적 디스크 사이즈 정의(옵션): 16G
    networks:     # 네트워크 구성정보
    - name: default   # Networks block에서 선언한 network 이름(필수)
      static_ips:
      - 10.30.60.23   # 사용할 IP addresses 정의(필수) Cubrid 서버 IP
  - name: cubrid_broker   #작업 이름(필수): Cubrid 서비스 브로커
    template: cubrid_broker  # job template 이름(필수)
    instances: 1  # job 인스턴스 수(필수)
    resource_pool: cubrid_resource_pools  # resource_pools block에 정의한 resource pool 이름(필수)
    networks: # 네트워크 구성정보
    - name: default   # Networks block에서 선언한 network 이름(필수)
      static_ips:
      - 10.30.60.22   # 사용할 IP addresses 정의(필수)
  - name : cubrid_broker_registrar  # 작업 이름: 서비스 브로커 등록
    template : cubrid_broker_registrar
    instances: 1
    lifecycle: errand   # bosh deploy시 vm에 생성되어 설치 되지 않고 bosh errand 로 실행할때 설정, 주로 테스트 용도에 쓰임
    resource_pool: cubrid_resource_pools
    networks:
    - name: default
    properties:
      broker: # 서비스 브로커 설정 정보
        host: 10.30.60.22 # 서비스 브로커 IP 
        name: CubridDB  # CF에서 서비스 브로커를 생성시 생기는 서비스 이름 브로커에 고정되어있는 값
        password: cloudfoundry  # 브로커 접근 아이디 비밀번호(필수)
        username: admin   # 브로커 접근 아이디(필수)
        protocol: http
        port: 8080  # 브로커 포트
      cf:
        admin_password: admin   # CF 사용자의 패스워드
        admin_username: admin   # CF 사용자 이름
        api_url: https://api.115.68.46.30.xip.io   # CF 설치시 설정한 api uri 정보(필수)
    release: paasta-cubrid
  - name : cubrid_broker_deregistrar  # 작업 이름: 서비스 브로커 삭제
    template : cubrid_broker_deregistrar
    instances: 1
    lifecycle: errand
    resource_pool: cubrid_resource_pools
    networks:
    - name: default
    properties:
      broker:
        host: 10.30.60.22
        name: CubridDB
        password: cloudfoundry
        username: admin
        protocol: http
        port: 8080
      cf:
        admin_password: admin
        admin_username: admin
        api_url: https://api.115.68.46.30.xip.io
    release: openpaas-cubrid
>
meta:
  apps_domain: 115.68.46.30.xip.io   # CF 설치시 설정한 apps 도메인 정보
  environment: null
  external_domain: 115.68.46.30.xip.io   # CF 설치시 설정한 외부 도메인 정보
  nats: # CF 설치시 설정한 nats 정보
    machines:
    - 10.30.40.11
    password: admin
    port: 4222
    user: nats
  syslog_aggregator: null
>
properties:
  cubrid:   # Cubrid 설정 정보
    max_clients: 200
  cubrid_broker:  # Cubrid Servcice Broker 설정 정보
    cubrid_ip: 10.30.60.23 # Cubrid IP
    cubrid_db_port: 30000 # Cubrid Port
    cubrid_db_name: cubrid_broker   # Cubrid service 관리를 위한 데이터베이스 이름
    cubrid_db_user: dba   # 브로커 관리용 데이터베이스 접근 사용자이름
    cubrid_db_passwd: openpaas  # 브로커 관리용 데이터베이스 접근 사용자 비밀번호
    cubrid_ssh_port: 22   # Cubrid가 설치된 서버 SSH 접속 포트
    cubrid_ssh_user: root # Cubrid가 설치된 서버 SSH 접속 사용자 이름
    cubrid_ssh_passwd: c1oudc0w # Cubrid가 설치된 서버 SSH 접속 사용자 비밀번호
    cubrid_ssh_sudo_passwd: c1oudc0w # Cubrid가 설치된 서버 sudo 비밀번호
```

- Deploy 할 deployment manifest 파일을 BOSH 에 지정한다.

>$ bosh deployment {Deployment manifest 파일 PATH}<br>
><br>
>$ bosh deployment paasta_cubrid_vsphere_2.0.yml<br>
><br>
>inception@inception:~/bosh-space/deployment$ bosh deployment paasta_cubrid_vsphere_2.0.yml <br>
>RSA 1024 bit CA certificates are loaded due to old openssl compatibility<br>
>Deployment set to '/mnt/bosh-space/deployment/paasta_cubrid_vsphere_2.0.yml'<br>


- Cubrid 서비스팩을 배포한다.

>$ bosh deploy<br>
><br>
>RSA 1024 bit CA certificates are loaded due to old openssl compatibility<br>
>Acting as user 'admin' on deployment 'paasta-cubrid-service' on 'bosh'<br>
>Getting deployment properties from director...<br>
>Unable to get properties list from director, trying without it...<br>
><br>
>Detecting deployment changes<br>
>----------------------------<br>
>resource_pools:<br>
>- name: cubrid_resource_pools<br>
>  network: default<br>
>  stemcell:<br>
>   name: bosh-vsphere-esxi-ubuntu-trusty-go_agent<br>
>    version: '3263.8'<br>
>  cloud_properties:<br>
>    cpu: 1<br>
>    disk: 8192<br>
>    ram: 2048<br>
><br>
>compilation:<br>
>  workers: 4<br>
>  network: default<br>
>  cloud_properties:<br>
>    ram: 2048<br>
>    disk: 8096<br>
>    cpu: 2<br>
><br>
>networks:<br>
>- name: default<br>
>  subnets:<br>
>  - cloud_properties:<br>
>      name: Internal<br>
>    dns:<br>
>    - 8.8.8.8<br>
>    gateway: 10.30.20.23<br>
>    name: default_unused<br>
>    range: 10.30.0.0/16<br>
>    reserved:<br>
>    - 10.30.0.1 - 10.30.20.22<br>
>    - 10.30.20.24 - 10.30.40.255<br>
>    static:<br>
>    - 10.30.60.11 - 10.30.60.70<br>
><br>
>releases:<br>
>- name: paasta-cubrid<br>
>  version: '2.0'<br>
><br>
>update:<br>
>  canaries: 1<br>
>  canary_watch_time: 120000<br>
>  update_watch_time: 120000<br>
>  max_in_flight: 4<br>
><br>
>jobs:<br>
>- name: cubrid<br>
>  template: cubrid<br>
>  instances: 1<br>
>  resource_pool: cubrid_resource_pools<br>
>  networks:<br>
>  - name: default<br>
>    static_ips:<br>
>    - 10.30.60.23<br>
>- name: cubrid_broker<br>
>  template: cubrid_broker<br>
>  instances: 1<br>
>  resource_pool: cubrid_resource_pools<br>
>  networks:<br>
>  - name: default<br>
>    static_ips:<br>
>    - 10.30.60.22<br>
>- name: cubrid_broker_registrar<br>
>  template: cubrid_broker_registrar<br>
>  instances: 1<br>
>  lifecycle: errand<br>
>  resource_pool: cubrid_resource_pools<br>
>  networks:<br>
>  - name: default<br>
>  properties:<br>
>    broker:<br>
>      host: "<redacted>"<br>
>      name: "<redacted>"<br>
>      password: "<redacted>"<br>
>      username: "<redacted>"<br>
>      protocol: "<redacted>"<br>
>      port: "<redacted>"<br>
>    cf:<br>
>      admin_password: "<redacted>"<br>
>      admin_username: "<redacted>"<br>
>      api_url: "<redacted>"<br>
>  release: paasta-cubrid<br>
>- name: cubrid_broker_deregistrar<br>
>  template: cubrid_broker_deregistrar<br>
>  instances: 1<br>
>  lifecycle: errand<br>
>  resource_pool: cubrid_resource_pools<br>
>  networks:<br>
>  - name: default<br>
>  properties:<br>
>    broker:<br>
>      host: "<redacted>"<br>
>      name: "<redacted>"<br>
>      password: "<redacted>"<br>
>      username: "<redacted>"<br>
>      protocol: "<redacted>"<br>
>      port: "<redacted>"<br>
>   cf:<br>
>      admin_password: "<redacted>"<br>
>      admin_username: "<redacted>"<br>
>      api_url: "<redacted>"<br>
>  release: paasta-cubrid<br>
><br>
>name: paasta-cubrid-service<br>
><br>
>director_uuid: d363905f-eaa0-4539-a461-8c1318498a32<br>
><br>
>meta:<br>
>  apps_domain: 115.68.46.30.xip.io<br>
>  environment: <br>
>  external_domain: 115.68.46.30.xip.io<br>
>  nats:<br>
>    machines:<br>
>    - 10.30.110.31<br>
>    password: nats<br>
>    port: 4222<br>
>    user: nats<br>
>  syslog_aggregator: <br>
><br>
>properties:<br>
>  cubrid:<br>
>    max_clients: "<redacted>"<br>
>  cubrid_broker:<br>
>    cubrid_ip: "<redacted>"<br>
>    cubrid_db_port: "<redacted>"<br>
>    cubrid_db_name: "<redacted>"<br>
>    cubrid_db_user: "<redacted>"<br>
>    cubrid_db_passwd: "<redacted>"<br>
>    cubrid_ssh_port: "<redacted>"<br>
>    cubrid_ssh_user: "<redacted>"<br>
>    cubrid_ssh_passwd: "<redacted>"<br>
>    cubrid_ssh_sudo_passwd: "<redacted>"<br>
>Please review all changes carefully<br>
><br>
>Deploying<br>
>---------<br>
>Are you sure you want to deploy? (type 'yes' to continue): yes<br>
><br>
>Director task 1303<br>
>Deprecation: Ignoring cloud config. Manifest contains 'networks' section.<br>
><br>
><br>
>  Started preparing deployment > Preparing deployment. Done (00:00:00)<br>
><br>
>  Started preparing package compilation > Finding packages to compile. Done (00:00:00)<br>
><br>
>  Started compiling packages<br>
>  Started compiling packages > cli/24305e50a638ece2cace4ef4803746c0c9fe4bb0<br>
>  Started compiling packages > java7/cb28502f6e89870255182ea76e9029c7e9ec1862<br>
>  Started compiling packages > cubrid/36065bb22d1e816657d176c902246231347361e2<br>
>     Done compiling packages > cli/24305e50a638ece2cace4ef4803746c0c9fe4bb0 (00:01:18)<br>
>     Done compiling packages > java7/cb28502f6e89870255182ea76e9029c7e9ec1862 (00:01:28)<br>
>  Started compiling packages > cubrid_broker/25717cfb95347c7ca5ed1e6cbdda701315789cfc<br>
>     Done compiling packages > cubrid/36065bb22d1e816657d176c902246231347361e2 (00:02:20)<br>
>     Done compiling packages > cubrid_broker/25717cfb95347c7ca5ed1e6cbdda701315789cfc (00:01:16)<br>
>     Done compiling packages (00:02:44)<br>
><br>
>  Started creating missing vms<br>
>  Started creating missing vms > cubrid/a6bf1096-1c12-4d7e-bbc8-03ba517ff5f6 (0)<br>
>  Started creating missing vms > cubrid_broker/be5bcafa-3b09-4300-a85d-c7935c3e05a7 (0)<br>
>     Done creating missing vms > cubrid/a6bf1096-1c12-4d7e-bbc8-03ba517ff5f6 (0) (00:00:53)<br>
>     Done creating missing vms > cubrid_broker/be5bcafa-3b09-4300-a85d-c7935c3e05a7 (0) (00:00:53)<br>
>     Done creating missing vms (00:00:53)<br>
><br>
>  Started updating instance cubrid > cubrid/a6bf1096-1c12-4d7e-bbc8-03ba517ff5f6 (0) (canary). Done (00:02:33)<br>
>  Started updating instance cubrid_broker > cubrid_broker/be5bcafa-3b09-4300-a85d-c7935c3e05a7 (0) (canary). Done (00:02:22)<br>
><br>
>Task 1303 done<br>
><br>
>Started		2017-01-06 07:15:20 UTC<br>
>Finished	2017-01-06 07:23:52 UTC<br>
>Duration	00:08:32<br>
><br>
>Deployed 'paasta-cubrid-service' to 'bosh'<br>



- 배포된 Cubrid 서비스팩을 확인한다.

>$ bosh vms paasta-cubrid-service<br>
><br>
>RSA 1024 bit CA certificates are loaded due to old openssl compatibility<br>
>Acting as user 'admin' on deployment 'paasta-cubrid-service' on 'bosh'<br>
><br><br>
>Director task 1312<br>
><br>
>Task 1312 done<br>
><br>
>+--------------------------------------------------------+---------+-----+-----------------------+-------------+<br>
>| VM                                                     | State   | AZ  | VM Type               | IPs         |<br>
>+--------------------------------------------------------+---------+-----+-----------------------+-------------+<br>
>| cubrid/0 (a6bf1096-1c12-4d7e-bbc8-03ba517ff5f6)        | running | n/a | cubrid_resource_pools | 10.30.60.23 |<br>
>| cubrid_broker/0 (be5bcafa-3b09-4300-a85d-c7935c3e05a7) | running | n/a | cubrid_resource_pools | 10.30.60.22 |<br>
>+--------------------------------------------------------+---------+-----+-----------------------+-------------+<br>
><br>
>VMs total: 2<br>






### <div id='10'> 2.4. Cubrid 서비스 브로커 등록
Cubrid 서비스팩 배포가 완료 되었으면 Application에서 서비스 팩을 사용하기 위해서 먼저 Cubrid 서비스 브로커를 등록해 주어야 한다.  
서비스 브로커 등록시 PaaS-TA에서 서비스브로커를 등록할 수 있는 사용자로 로그인이 되어있어야 한다.

- 서비스 브로커 목록을 확인한다.

><div>$ cf service-brokers
![2-4-0-0]

- Cubrid 서비스 브로커를 등록한다.

>$ cf create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{서비스팩 URL}  
- 서비스팩 이름 : 서비스 팩 관리를 위해 PaaS-TA에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스팩 리스트의 명칭이다.  
- 서비스팩 사용자ID / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID입니다. 서비스팩도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.  
- 서비스팩 URL : 서비스팩이 제공하는 API를 사용할 수 있는 URL을 입력한다.  
>
>$ cf create-service-broker cubrid-service-broker admin cloudfoundry http://10.30.60.22:8080  
>![2-4-1-0]

- 등록된 Cubrid 서비스 브로커를 확인한다.

><div>$ cf service-brokers
>![2-4-2-0]

- 접근 가능한 서비스 목록을 확인한다.

><div>$ cf service-access
>![2-4-3-0]
><div>서비스 브로커 생성시 디폴트로 접근을 허용하지 않는다.

- 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)

>$ cf enable-service-access CubridDB  
>$ cf service-access  
>![2-4-4-0]


#  <div id='11'> 3. Cubrid연동 Sample App 설명
본 Sample Web App은 PaaS-TA에 배포되며 Cubrid의 서비스를 Provision과 Bind를 한 상태에서 사용이 가능하다.

### <div id='12'> 3.1. Sample App 구조
Sample Web App은 PaaS-TA에 App으로 배포가 된다. App을 배포하여 구동시 Bind 된 Cubrid 서비스 연결정보로 접속하여 초기 데이터를 생성하게 된다. 배포 완료 후 정상적으로 App 이 구동되면 브라우져나 curl로 해당 App에 접속 하여 Cubrid 환경정보(서비스 연결 정보)와 초기 적재된 데이터를 보여준다.

Sample Web App 구조는 다음과 같다.
<table>
  <tr>
    <td>이름</td>
    <td>설명</td>
  </tr>
  <tr>
    <td>src</td>
    <td>Sample 소스 디렉토리</td>
  </tr>
  <tr>
    <td>manifest.yml</td>
    <td>PaaS-TA에 app 배포시 필요한 설정을 저장하는 파일</td>
  </tr>
  <tr>
    <td>pom.xml</td>
    <td>메이븐 project 설정 파일</td>
  </tr>
  <tr>
    <td>target</td>
    <td>메이븐 빌드시 생성되는 디렉토리(war 파일, classes 폴더 등)</td>
  </tr>
</table>

- PaaS-TA-Sample-Apps을 다운로드 받고 Service 폴더안에 있는 Cubrid Sample Web App인 hello-spring-cubrid를 복사한다.

><div>$ ls -all
>![3-1-0-0]


### <div id='13'> 3.2. PaaS-TA에서 서비스 신청
Sample Web App에서 Cubrid 서비스를 사용하기 위해서는 서비스 신청(Provision)을 해야 한다.
*참고: 서비스 신청시 PaaS-TA에서 서비스를신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.


- 먼저 PaaS-TA Marketplace에서 서비스가 있는지 확인을 한다.

><div>$ cf marketplace
>![3-2-0-0]

- Marketplace에서 원하는 서비스가 있으면 서비스 신청(Provision)을 한다.

>$ cf create-service {서비스명} {서비스플랜} {내서비스명}  
- 서비스명 : CubridDB로 Marketplace에서 보여지는 서비스 명칭이다.  
- 서비스플랜 : 서비스에 대한 정책으로 plans에 있는 정보 중 하나를 선택한다. Cubrid 서비스는 100mb, 1gb를 지원한다.  
- 내서비스명 : 내 서비스에서 보여지는 명칭이다. 이 명칭을 기준으로 환경설정정보를 가져온다.  
>
>$ cf create-service CubridDB utf8 cubrid-service-instance  
>![3-2-1-0]

- 생성된 Cubrid 서비스 인스턴스를 확인한다.

><div>$ cf services
>![3-2-2-0]


### <div id='14'> 3.3. Sample App에 서비스 바인드 신청 및 App 확인
서비스 신청이 완료되었으면 Sample Web App 에서는 생성된 서비스 인스턴스를 Bind 하여 App에서 Cubrid 서비스를 이용한다.  
*참고: 서비스 Bind 신청시 PaaS-TA에서 서비스 Bind신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.

- Sample Web App 디렉토리로 이동하여 manifest 파일을 확인한다.

>$ cd hello-spring-cubrid  
>$ vi manifest.yml  
```yaml
---
applications:
- name: hello-spring-cubrid # 배포할 App 이름
  memory: 512M # 배포시 메모리 사이즈
  instances: 1 # 배포 인스턴스 수
  path: target/hello-spring-cubrid-1.0.0-BUILD-SNAPSHOT.war #배포하는 App 파일 PATH
```
>참고: ./build/libs/hello-spring-cubrid.war 파일이 존재 하지 않을 경우 gradle 빌드를 수행 하면 파일이 생성된다.

- --no-start 옵션으로 App을 배포한다.  
--no-start: App 배포시 구동은 하지 않는다.

><div>$ cf push --no-start<br>
>![3-3-0-0]

- 배포된 Sample App을 확인하고 로그를 수행한다.

><div>$ cf apps<br>
>![3-3-1-0]
><div>$ cf logs {배포된 App명}<br>
>$ cf logs hello-spring-cubrid
>![3-3-2-0]

- Sample Web App에서 생성한 서비스 인스턴스 바인드 신청을 한다. 

><div>$ cf bind-service hello-spring-cubrid cubrid-service-instance
>![3-3-3-0]

- 바인드가 적용되기 위해서 App을 재기동한다.

><div>$ cf restart hello-spring-cubrid
>![3-3-4-0]  
>![3-3-4-1]  

- (참고) 바인드 후 App구동시 Cubrid 서비스 접속 에러로 App 구동이 안될 경우 보안 그룹을 추가한다.

><div>-  rule.json 화일을 만들고 아래와 같이 내용을 넣는다.<br>
><div>$ vi rule.json
```json
[
  {
    "protocol": "tcp",
    "destination": "10.30.60.23",
    "ports": "30000"
  }
]
```
><div>- 보안 그룹을 생성한다.<br>
><div>$ cf create-security-group CubridDB rule.json
>![3-3-5-0]
><div>- 모든 App에 Cubrid 서비스를 사용할수 있도록 생성한 보안 그룹을 적용한다.<br>
><div>$ cf bind-running-security-group CubridDB
>![3-3-6-0]
><div>- App을 리부팅 한다.<br>
><div>$ cf restart hello-spring-cubrid
>![3-3-7-0]

- App이 정상적으로 Cubrid 서비스를 사용하는지 확인한다.

><div>- curl 로 확인 <br>
$ curl hello-spring-cubrid.115.68.46.30.xip.io
>
>![3-3-8-0]
><div>- 브라우져에서 확인<br>
>
>![3-3-8-1]


# <div id='15'> 4. Cubrid Client 툴 접속
Application에 바인딩된 Cubrid 서비스 연결정보는 Private IP로 구성되어 있기 때문에 Cubrid Client 툴에서 직접 연결할수 없다. 따라서 Cubrid Client 툴에서 SSH 터널, Proxy 터널 등을 제공하는 툴을 사용해서 연결하여야 한다. 본 가이드는 무료 SSH 및 텔넷 접속 툴인 Putty를 이용하여 SSH 터널을 통해 연결 하는 방법을 제공하며 Cubrid Client 툴로써는 Cubrid에서 제공하는 Cubrid Manager로 가이드한다. Cubrid Manager 에서 접속하기 위해서 먼저 SSH 터널링 할수 있는 VM 인스턴스를 생성해야한다. 이 인스턴스는 SSH로 접속이 가능해야 하고 접속 후 Open PaaS 에 설치한 서비스팩에 Private IP 와 해당 포트로 접근이 가능하도록 시큐리티 그룹을 구성해야 한다. 이 부분은 vSphere관리자 및 OpenPaaS 운영자에게 문의하여 구성한다.


### <div id='16'> 4.1.  Putty 다운로드 및 터널링
Putty 프로그램은 SSH 및 텔넷 접속을 할 수 있는 무료 소프트웨어이다.

- Putty를 다운로드 하기 위해 아래 URL로 이동하여 파일을 다운로드 한다. 별도의 설치과정없이 사용할 수 있다.
**<http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html>**  
![4-1-0-0]

- 다운받은 putty.exe.파일을 더블클릭하여 실행한다.  
![4-1-1-0]

- Session 탭의 Host name과 Port란에. OpenPaaS 운영 관리자에게 제공받은 SSH 터널링 가능한 서버 정보를 입력한다.  
![4-1-2-0]

- Connection->SSH->Tunnels 탭에서 Source port(내 로컬에서 접근할 포트), Destination(터널링으로 연결할 서버정보)를 입력하고 Local, Auto를 선택 후 Add를 클릭한다.   
![4-1-3-0]  
![4-1-3-1]  
서버 정보는 Application에 바인드되어 있는 서버 정보를 입력한다. cf env <app_name> 명령어로 이용하여 확인한다.  
예) $ cf env hello-spring-cubrid  
![4-1-4-0]

- Session 탭에서 Saved Sessions에 저장할 이름을 입력하고 Save를 눌러 저장한 후 Open버튼을 누른다.  
![4-1-5-0]

- 서버 접속정보를 입력하여 접속하여 터널링을 완료한다.  
만약 ssh 인증이 Password방식이 아닌 Key인증 방식일 경우, Connection->SSH->인증탭의 '인증 개인키 파일'에 key 파일을 등록하여 인증한다.  
Key파일의 확장자가 .pem이라면 putty설치시 같이 설치된 puttygen을 사용하여 ppk파일로 변환한뒤 사용한다.  
![4-1-6-0]


### <div id='17'> 4.2.  Cubrid Manager 설치 & 연결
Cubrid Manager 프로그램은 Cubrid에서 제공하는 무료로 사용할 수 있는 소프트웨어이다.

- Cubrid Manager를 다운로드 하기 위해 아래 URL로 이동하여 설치파일을 다운로드 한다.  
**<http://ftp.cubrid.org/CUBRID_Tools/CUBRID_Manager/>**  
![4-2-0-0]

- 다운받은 파일을 더블클릭하여 실행한다.  
![4-2-1-0]

- 한국어를 선택하고 OK를 누른다.  
![4-2-2-0]

- 다음을 눌러 계속 진행한다.  
![4-2-3-0]

- 동의함을 눌러 계속 진행한다.  
![4-2-4-0]

- 바로가기 옵션을 선택 후 다음을 눌러 계속 진행한다.  
![4-2-5-0]

- 설치 경로를 입력하고 설치를 눌러 설치를 시작한다.  
![4-2-6-0]

- 설치가 완료되면 다음을 눌러 계속 진행한다.  
![4-2-7-0]

- 마침을 눌러 설치를 완료한다.  
![4-2-8-0]

- 설치된 Cubrid Manager를 실행하면 처음 나오는 화면이다. Workspace를 선택 후 OK를 눌러 실행한다. 만약 이 창을 다시보기를 원치않는다면 '기본적으로 이것을 사용하고 다시 물어 보지 않기' 옵션을 선택한다.  
![4-2-9-0]

- 관리 모드, 질의 모드 둘중 목적에 맞게 선택 후 확인을 눌러 실행한다.  
여기서는 질의 모드로 실행한다.  
![4-2-10-0]

- 연결정보를 입력하기 위해서 연결 정보 등록을 누른다.  
![4-2-11-0]

- Server에 접속하기 위한 Connection 정보를 입력한다.  
![4-2-12-0]  
서버 정보는 Application에 바인드되어 있는 서버 정보를 입력한다. cf env <app_name> 명령어로 이용하여 확인한다.  
예) $ cf env hello-spring-cubrid  
![4-2-13-0]

- 연결 테스트 버튼을 클릭하여 접속 테스트를 한다.  
![4-2-14-0]  
정보가 정상적으로 입력되었다면 '연결이 성공하였습니다.'라는 메시지가 나온다.  
확인 버튼을 눌러 창을 닫는다.  
![4-2-15-0]

- 연결 버튼을 클릭하여 접속한다  
![4-2-16-0]

- 접속이 완료되면 좌측에 스키마 정보가 나타난다.  
![4-2-17-0]

- 질의 편집기 버튼을 클릭하면 오른쪽 창에 query를 입력할 수 있는 창이 열린다.  
![4-2-18-0]

- 우측 화면에 쿼리 항목에 Query문을 작성한 후 실행 버튼(삼각형)을 클릭한다.  
쿼리문에 이상이 없다면 정상적으로 결과를 얻을 수 있을 것이다.  
![4-2-19-0]


[1-3-0-0]:/Service-Guide/images/cubrid/1-3-0-0.png
[2-1-0-0]:/Service-Guide/images/cubrid/2-1-0-0.png

[2-1-0-1]:/Service-Guide/images/cubrid/2-1-0-1.png
[2-1-0-2]:/Service-Guide/images/cubrid/2-1-0-2.png
[2-1-0-3]:/Service-Guide/images/cubrid/2-1-0-3.png
[2-1-0-4]:/Service-Guide/images/cubrid/2-1-0-4.png
[2-1-0-5]:/Service-Guide/images/cubrid/2-1-0-5.png
[2-1-0-6]:/Service-Guide/images/cubrid/2-1-0-6.png
[2-1-0-7]:/Service-Guide/images/cubrid/2-1-0-7.png
[2-1-0-8]:/Service-Guide/images/cubrid/2-1-0-8.png
[2-1-0-9]:/Service-Guide/images/cubrid/2-1-0-9.png
[2-1-0-10]:/Service-Guide/images/cubrid/2-1-0-10.png

[2-1-1-0]:/Service-Guide/images/cubrid/2-1-1-0.png
[2-2-0-0]:/Service-Guide/images/cubrid/2-2-0-0.png
[2-2-1-0]:/Service-Guide/images/cubrid/2-2-1-0.png
[2-2-2-0]:/Service-Guide/images/cubrid/2-2-2-0.png
[2-2-3-0]:/Service-Guide/images/cubrid/2-2-3-0.png
[2-2-4-0]:/Service-Guide/images/cubrid/2-2-4-0.png
[2-2-5-0]:/Service-Guide/images/cubrid/2-2-5-0.png
[2-2-5-1]:/Service-Guide/images/cubrid/2-2-5-1.png
[2-2-6-0]:/Service-Guide/images/cubrid/2-2-6-0.png
[2-2-7-0]:/Service-Guide/images/cubrid/2-2-7-0.png
[2-3-0-0]:/Service-Guide/images/cubrid/2-3-0-0.png
[2-3-1-0]:/Service-Guide/images/cubrid/2-3-1-0.png
[2-3-2-0]:/Service-Guide/images/cubrid/2-3-2-0.png
[2-3-3-0]:/Service-Guide/images/cubrid/2-3-3-0.png
[2-3-4-0]:/Service-Guide/images/cubrid/2-3-4-0.png
[2-3-4-1]:/Service-Guide/images/cubrid/2-3-4-1.png
[2-3-5-0]:/Service-Guide/images/cubrid/2-3-5-0.png
[2-3-5-1]:/Service-Guide/images/cubrid/2-3-5-1.png
[2-4-0-0]:/Service-Guide/images/cubrid/2-4-0-0.png
[2-4-1-0]:/Service-Guide/images/cubrid/2-4-1-0.png
[2-4-2-0]:/Service-Guide/images/cubrid/2-4-2-0.png
[2-4-3-0]:/Service-Guide/images/cubrid/2-4-3-0.png
[2-4-4-0]:/Service-Guide/images/cubrid/2-4-4-0.png
[3-1-0-0]:/Service-Guide/images/cubrid/3-1-0-0.png
[3-2-0-0]:/Service-Guide/images/cubrid/3-2-0-0.png
[3-2-1-0]:/Service-Guide/images/cubrid/3-2-1-0.png
[3-2-2-0]:/Service-Guide/images/cubrid/3-2-2-0.png
[3-3-0-0]:/Service-Guide/images/cubrid/3-3-0-0.png
[3-3-1-0]:/Service-Guide/images/cubrid/3-3-1-0.png
[3-3-2-0]:/Service-Guide/images/cubrid/3-3-2-0.png
[3-3-3-0]:/Service-Guide/images/cubrid/3-3-3-0.png
[3-3-4-0]:/Service-Guide/images/cubrid/3-3-4-0.png
[3-3-4-1]:/Service-Guide/images/cubrid/3-3-4-1.png
[3-3-5-0]:/Service-Guide/images/cubrid/3-3-5-0.png
[3-3-6-0]:/Service-Guide/images/cubrid/3-3-6-0.png
[3-3-7-0]:/Service-Guide/images/cubrid/3-3-7-0.png
[3-3-8-0]:/Service-Guide/images/cubrid/3-3-8-0.png
[3-3-8-1]:/Service-Guide/images/cubrid/3-3-8-1.png
[4-1-0-0]:/Service-Guide/images/cubrid/4-1-0-0.png
[4-1-1-0]:/Service-Guide/images/cubrid/4-1-1-0.png
[4-1-2-0]:/Service-Guide/images/cubrid/4-1-2-0.png
[4-1-3-0]:/Service-Guide/images/cubrid/4-1-3-0.png
[4-1-3-1]:/Service-Guide/images/cubrid/4-1-3-1.png
[4-1-4-0]:/Service-Guide/images/cubrid/4-1-4-0.png
[4-1-5-0]:/Service-Guide/images/cubrid/4-1-5-0.png
[4-1-6-0]:/Service-Guide/images/cubrid/4-1-6-0.png
[4-2-0-0]:/Service-Guide/images/cubrid/4-2-0-0.png
[4-2-1-0]:/Service-Guide/images/cubrid/4-2-1-0.png
[4-2-2-0]:/Service-Guide/images/cubrid/4-2-2-0.png
[4-2-3-0]:/Service-Guide/images/cubrid/4-2-3-0.png
[4-2-4-0]:/Service-Guide/images/cubrid/4-2-4-0.png
[4-2-5-0]:/Service-Guide/images/cubrid/4-2-5-0.png
[4-2-6-0]:/Service-Guide/images/cubrid/4-2-6-0.png
[4-2-7-0]:/Service-Guide/images/cubrid/4-2-7-0.png
[4-2-8-0]:/Service-Guide/images/cubrid/4-2-8-0.png
[4-2-9-0]:/Service-Guide/images/cubrid/4-2-9-0.png
[4-2-10-0]:/Service-Guide/images/cubrid/4-2-10-0.png
[4-2-11-0]:/Service-Guide/images/cubrid/4-2-11-0.png
[4-2-12-0]:/Service-Guide/images/cubrid/4-2-12-0.png
[4-2-13-0]:/Service-Guide/images/cubrid/4-2-13-0.png
[4-2-14-0]:/Service-Guide/images/cubrid/4-2-14-0.png
[4-2-15-0]:/Service-Guide/images/cubrid/4-2-15-0.png
[4-2-16-0]:/Service-Guide/images/cubrid/4-2-16-0.png
[4-2-17-0]:/Service-Guide/images/cubrid/4-2-17-0.png
[4-2-18-0]:/Service-Guide/images/cubrid/4-2-18-0.png
[4-2-19-0]:/Service-Guide/images/cubrid/4-2-19-0.png
[2-2-0-0-1]:/Service-Guide/images/cubrid/2-2-0-0-1.png
[2-2-4-0-1]:/Service-Guide/images/cubrid/2-2-4-0-1.png
[2-2-6-0-1]:/Service-Guide/images/cubrid/2-2-6-0-1.png
[2-2-7-0-1]:/Service-Guide/images/cubrid/2-2-7-0-1.png