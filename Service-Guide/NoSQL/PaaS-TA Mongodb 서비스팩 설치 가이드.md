## Table of Contents
1. [문서 개요](#1)
  - 1.1. [목적](#11)
  - 1.2. [범위](#12)
  - 1.3. [시스템 구성도](#13)
  - 1.4. [참고자료](#14)
2. [Mongodb 서비스팩 설치](#2)
  - 2.1. [설치전 준비사항](#21)
  - 2.2. [Mongodb 서비스 릴리즈 업로드](#22)
  - 2.3. [Mongodb 서비스 Deployment 파일 수정 및 배포](#23)
  - 2.4. [Mongodb 서비스 브로커 등록](#24)
3. [Mongodb 연동 Sample Web App 설명](#3)
  - 3.1. [Sample App 구조](#31)
  - 3.2. [PaaS-TA에서 서비스 신청](#32)
  - 3.3. [Sample App에 서비스 바인드 신청 및 App 확인](#33)
4. [Mongodb Client 툴 접속](#4)
  - 4.1. [MongoChef 설치 및 연결](#41)


# <div id='1'> 1. 문서 개요
### <div id='11'> 1.1. 목적

본 문서(Mongodb 서비스팩 설치 가이드)는 전자정부표준프레임워크 기반의 PaaS-TA에서 제공되는 서비스팩인 Mongodb 서비스팩을 Bosh를 이용하여 설치 하는 방법과 PaaS-TA의 SaaS 형태로 제공하는 Application 에서 Mongodb 서비스를 사용하는 방법을 기술하였다.
PaaS-TA 3.5 버전부터는 Bosh2.0 기반으로 deploy를 진행하며 기존 Bosh1.0 기반으로 설치를 원할경우에는 PaaS-TA 3.1 이하 버전의 문서를 참고한다.

### <div id='12'> 1.2. 범위
설치 범위는 Mongodb 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다.

### <div id='13'> 1.3. 시스템 구성도
본 문서의 설치된 시스템 구성도입니다. Mongodb Server, Mongodb 서비스 브로커로 최소사항을 구성하였다.

![시스템구성도][mongodb_image_02]

<table>
  <tr>
    <td>구분</td>
    <td>스펙</td>
  </tr>
  <tr>
    <td>PaaS-TA-mongodb-broker</td>
    <td>1vCPU / 1GB RAM / 8GB Disk</td>
  </tr>
  <tr>
    <td>Mongos</td>
    <td>1vCPU / 1GB RAM / 8GB Disk</td>
  </tr>
  <tr>
    <td>Mongo Config</td>
    <td>1vCPU / 1GB RAM / 8GB Disk+4GB(영구적 Disk)</td>
  </tr>
  <tr>
    <td>Mongod</td>
    <td>1vCPU / 1GB RAM /8GB Disk+4GB(영구적 Disk)</td>
  </tr>
</table>


### <div id='14'> 1.4. 참고자료
[**http://bosh.io/docs**](http://bosh.io/docs)
[**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/)



# <div id='2'> 2.  Mongodb 서비스팩 설치

### <div id='21'> 2.1.  설치전 준비사항
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

### <div id='22'> 2.2. Mongodb 서비스 릴리즈 업로드

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

-	Mongodb 서비스 릴리즈가 업로드 되어 있지 않은 것을 확인

-	Mongodb 서비스 릴리즈 파일을 업로드한다.

- **사용 예시**

		$ bosh -e micro-bosh upload-release paasta-mongodb-shard-2.0.tgz
    		
		######################################################### 100.00% 93.67 MiB/s 0s
		Task 8119

		Task 8119 | 06:47:16 | Extracting release: Extracting release (00:00:01)
		Task 8119 | 06:47:17 | Verifying manifest: Verifying manifest (00:00:00)
		Task 8119 | 06:47:17 | Resolving package dependencies: Resolving package dependencies (00:00:00)
		Task 8119 | 06:47:17 | Creating new packages: cli/668cd93a9dc828726341909df08f74dce57478fe (00:00:00)
		Task 8119 | 06:47:17 | Creating new packages: java7/a6b0fd78eadc51bfbf305a5571a710128fae6fe5 (00:00:05)
		Task 8119 | 06:47:22 | Creating new packages: mongodb/f48a8e934cc387162028629a196040c17bfa3401 (00:00:01)
		Task 8119 | 06:47:23 | Creating new packages: mongodb_broker/25bacb6034e74fc63b592c8bb6ba98cf512afbbc (00:00:00)
		Task 8119 | 06:47:23 | Creating new jobs: mongodb_broker/9d10fbfc09bb211114058ffd16dce044ff27d75e (00:00:01)
		Task 8119 | 06:47:24 | Creating new jobs: mongodb_broker_deregistrar/d437a344605d23e639a34373c4b2553841c956a5 (00:00:00)
		Task 8119 | 06:47:24 | Creating new jobs: mongodb_broker_registrar/1f2b27e672ee49de953cf718de841b3018ab9fb2 (00:00:00)
		Task 8119 | 06:47:24 | Creating new jobs: mongodb_config/1cf1de67d7ffc4c30a573de973a8573e1098d81c (00:00:00)
		Task 8119 | 06:47:24 | Creating new jobs: mongodb_master/b58a9c73a51c95a98bda70f9884ffbc995fc0a52 (00:00:00)
		Task 8119 | 06:47:24 | Creating new jobs: mongodb_shard/9e8f61155d16c2f6eb8012278f7be65c26c0c8b5 (00:00:00)
		Task 8119 | 06:47:24 | Creating new jobs: mongodb_slave/640d6953cba4ac04d17c0e3aa2af6820e5298b8a (00:00:00)
		Task 8119 | 06:47:24 | Release has been created: paasta-mongodb-shard/2.0 (00:00:00)

		Task 8119 Started  Thu Sep 13 06:47:16 UTC 2018
		Task 8119 Finished Thu Sep 13 06:47:24 UTC 2018
		Task 8119 Duration 00:00:08
		Task 8119 done

		Succeeded


-	업로드 된 Mongodb 릴리즈를 확인한다.

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
		paasta-mongodb-shard              2.0       3c41864+  
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

	    	32 releases

	    	Succeeded
		
-	Mongodb 서비스 릴리즈가 업로드 되어 있는 것을 확인

-	Deploy시 사용할 Stemcell을 확인한다.

- **사용 예시**

		$ bosh -e micro-bosh stemcells
		Name                                      Version   OS             CPI  CID  
		bosh-vsphere-esxi-ubuntu-trusty-go_agent  3586.26*  ubuntu-trusty  -    sc-109fbdb0-f663-49e8-9c30-8dbdd2e5b9b9  
		~                                         3445.2*   ubuntu-trusty  -    sc-025c70b5-7d6e-4ba3-a12b-7e71c33dad24  
		~                                         3309*     ubuntu-trusty  -    sc-22429dba-e5cc-4469-ab3a-882091573277  

		(*) Currently deployed

		3 stemcells

		Succeeded
		
>Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드를 해야 한다. (Mongodb 는 stemcell 3309 버전을 사용)


### <div id='23'> 2.3. Mongodb 서비스 Deployment 파일 수정 및 배포

BOSH Deployment manifest 는 components 요소 및 배포의 속성을 정의한 YAML 파일이다.
Deployment manifest 에는 sotfware를 설치 하기 위해서 어떤 Stemcell (OS, BOSH agent) 을 사용할것이며 Release (Software packages, Config templates, Scripts) 이름과 버전, VMs 용량, Jobs params 등을 정의가 되어 있다.

deployment 파일에서 사용하는 network, vm_type 등은 cloud config 를 활용하고 해당 가이드는 Bosh2.0 가이드를 참고한다.

-	cloud config 내용 조회

- **사용 예시**

		bosh -e micro-bosh cloud-config
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		azs:
		- cloud_properties:
		    datacenters:
		    - clusters:
		      - BD-HA:
			  resource_pool: CF_BOSH2_Pool
		      name: BD-HA
		  name: z1
		- cloud_properties:
		    datacenters:
		    - clusters:
		      - BD-HA:
			  resource_pool: CF_BOSH2_Pool
		      name: BD-HA
		  name: z2
		- cloud_properties:
		    datacenters:
		    - clusters:
		      - BD-HA:
			  resource_pool: CF_BOSH2_Pool
		      name: BD-HA
		  name: z3
		- cloud_properties:
		    datacenters:
		    - clusters:
		      - BD-HA:
			  resource_pool: CF_BOSH2_Pool
		      name: BD-HA
		  name: z4
		- cloud_properties:
		    datacenters:
		    - clusters:
		      - BD-HA:
			  resource_pool: CF_BOSH2_Pool
		      name: BD-HA
		  name: z5
		- cloud_properties:
		    datacenters:
		    - clusters:
		      - BD-HA:
			  resource_pool: CF_BOSH2_Pool
		      name: BD-HA
		  name: z6
		compilation:
		  az: z1
		  network: default
		  reuse_compilation_vms: true
		  vm_type: large
		  workers: 5
		disk_types:
		- disk_size: 1024
		  name: default
		- disk_size: 1024
		  name: 1GB
		- disk_size: 2048
		  name: 2GB
		- disk_size: 4096
		  name: 4GB
		- disk_size: 5120
		  name: 5GB
		- disk_size: 8192
		  name: 8GB
		- disk_size: 10240
		  name: 10GB
		- disk_size: 20480
		  name: 20GB
		- disk_size: 30720
		  name: 30GB
		- disk_size: 51200
		  name: 50GB
		- disk_size: 102400
		  name: 100GB
		- disk_size: 1048576
		  name: 1TB
		networks:
		- name: default
		  subnets:
		  - azs:
		    - z1
		    - z2
		    - z3
		    - z4
		    - z5
		    - z6
		    cloud_properties:
		      name: Internal
		    dns:
		    - 8.8.8.8
		    gateway: 10.30.20.23
		    range: 10.30.0.0/16
		    reserved:
		    - 10.30.0.0 - 10.30.111.40
		- name: public
		  subnets:
		  - azs:
		    - z1
		    - z2
		    - z3
		    - z4
		    - z5
		    - z6
		    cloud_properties:
		      name: External
		    dns:
		    - 8.8.8.8
		    gateway: 115.68.46.177
		    range: 115.68.46.176/28
		    reserved:
		    - 115.68.46.176 - 115.68.46.188
		    static:
		    - 115.68.46.189 - 115.68.46.190
		  type: manual
		- name: service_private
		  subnets:
		  - azs:
		    - z1
		    - z2
		    - z3
		    - z4
		    - z5
		    - z6
		    cloud_properties:
		      name: Internal
		    dns:
		    - 8.8.8.8
		    gateway: 10.30.20.23
		    range: 10.30.0.0/16
		    reserved:
		    - 10.30.0.0 - 10.30.106.255
		    static:
		    - 10.30.107.1 - 10.30.107.255
		- name: service_public
		  subnets:
		  - azs:
		    - z1
		    - z2
		    - z3
		    - z4
		    - z5
		    - z6
		    cloud_properties:
		      name: External
		    dns:
		    - 8.8.8.8
		    gateway: 115.68.47.161
		    range: 115.68.47.160/24
		    reserved:
		    - 115.68.47.161 - 115.68.47.174
		    static:
		    - 115.68.47.175 - 115.68.47.185
		  type: manual
		- name: portal_service_public
		  subnets:
		  - azs:
		    - z1
		    - z2
		    - z3
		    - z4
		    - z5
		    - z6
		    cloud_properties:
		      name: External
		    dns:
		    - 8.8.8.8
		    gateway: 115.68.46.209
		    range: 115.68.46.208/28
		    reserved:
		    - 115.68.46.216 - 115.68.46.222
		    static:
		    - 115.68.46.214
		  type: manual
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
		    disk: 51200
		  name: small-50GB
		- cloud_properties:
		    disk: 102400
		  name: small-highmem-100GB
		vm_types:
		- cloud_properties:
		    cpu: 1
		    disk: 8192
		    ram: 1024
		  name: minimal
		- cloud_properties:
		    cpu: 1
		    disk: 10240
		    ram: 2048
		  name: default
		- cloud_properties:
		    cpu: 1
		    disk: 30720
		    ram: 4096
		  name: small
		- cloud_properties:
		    cpu: 2
		    disk: 20480
		    ram: 4096
		  name: medium
		- cloud_properties:
		    cpu: 2
		    disk: 20480
		    ram: 8192
		  name: medium-memory-8GB
		- cloud_properties:
		    cpu: 4
		    disk: 20480
		    ram: 8192
		  name: large
		- cloud_properties:
		    cpu: 8
		    disk: 20480
		    ram: 16384
		  name: xlarge
		- cloud_properties:
		    cpu: 2
		    disk: 51200
		    ram: 4096
		  name: small-50GB
		- cloud_properties:
		    cpu: 2
		    disk: 51200
		    ram: 4096
		  name: small-50GB-ephemeral-disk
		- cloud_properties:
		    cpu: 4
		    disk: 102400
		    ram: 8192
		  name: small-100GB-ephemeral-disk
		- cloud_properties:
		    cpu: 4
		    disk: 102400
		    ram: 8192
		  name: small-highmem-100GB-ephemeral-disk
		- cloud_properties:
		    cpu: 8
		    disk: 20480
		    ram: 16384
		  name: small-highmem-16GB
		- cloud_properties:
		    cpu: 1
		    disk: 4096
		    ram: 2048
		  name: caas_small
		- cloud_properties:
		    cpu: 1
		    disk: 4096
		    ram: 1024
		  name: caas_small_api
		- cloud_properties:
		    cpu: 1
		    disk: 4096
		    ram: 4096
		  name: caas_medium
		- cloud_properties:
		    cpu: 2
		    disk: 8192
		    ram: 4096
		  name: service_medium
		- cloud_properties:
		    cpu: 2
		    disk: 10240
		    ram: 2048
		  name: service_medium_2G

		Succeeded


-	Deployment 파일을 서버 환경에 맞게 수정한다.

```yml
# openpaas-mongodb-shard-service 설정 파일 내용
---
name: paasta-mongodb-shard-service  # 서비스 배포이름(필수)

release:
  name: paasta-mongodb-shard  #서비스 릴리즈 이름(필수)
  version: "2.0"   #서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전

stemcells:
- alias: default
  os: ((stemcell_os))
  version: "((stemcell_version))"

update:
  canaries: 1                                          # canary 인스턴스 수(필수)
  canary_watch_time: 30000-180000                      # canary 인스턴스가 수행하기 위한 대기 시간(필수)
  max_in_flight: 6                                     # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)
  update_watch_time: 30000-180000                      # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)

instance_groups:
- name: mongodb_slave1  #작업 이름(필수): mongodb replica set의 slave 서버
  azs:
  - z5
  instances: 2  # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  persistent_disk_type: 10GB              # cloud config 에 정의한 영구 디스크 타입 
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.202
    - 10.30.107.203
  properties:
    replSetName: op1 # replicaSet1 의 이름
  templates:
  - name: mongodb_slave                # job template 이름(필수)
    release: paasta-mongodb-shard

- name: mongodb_slave2  #작업 이름(필수): mongodb replica set의 slave 서버
  azs:
  - z5
  instances: 2  # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  persistent_disk_type: 10GB              # cloud config 에 정의한 영구 디스크 타입 
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.205
    - 10.30.107.206
  properties:
    replSetName: op2 # replicaSet1 의 이름
  templates:
  - name: mongodb_slave                # job template 이름(필수)
    release: paasta-mongodb-shard

- name: mongodb_slave3  #작업 이름(필수): mongodb replica set의 slave 서버
  azs:
  - z5
  instances: 2  # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  persistent_disk_type: 10GB              # cloud config 에 정의한 영구 디스크 타입 
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.208
    - 10.30.107.209
  properties:
    replSetName: op3 # replicaSet1 의 이름
  templates:
  - name: mongodb_slave                # job template 이름(필수)
    release: paasta-mongodb-shard

- name: mongodb_master1   #작업 이름(필수): mongodb replica set의 master 서버
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  persistent_disk_type: 10GB              # cloud config 에 정의한 영구 디스크 타입 
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.201
  properties:
    replSet_hosts: ["10.30.107.201","10.30.107.202","10.30.107.203"] # 첫번째 Host는 replicaSet1의 master
    replSetName: op1 # replicaSet1 의 이름
  templates:
  - name: mongodb_master                # job template 이름(필수)
    release: paasta-mongodb-shard

- name: mongodb_master2   #작업 이름(필수): mongodb replica set의 master 서버
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  persistent_disk_type: 10GB              # cloud config 에 정의한 영구 디스크 타입 
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.204
  properties:
    replSet_hosts: ["10.30.107.204","10.30.107.205","10.30.107.206"] # 첫번째 Host는 replicaSet1의 master
    replSetName: op2 # replicaSet1 의 이름
  templates:
  - name: mongodb_master                # job template 이름(필수)
    release: paasta-mongodb-shard

- name: mongodb_master3   #작업 이름(필수): mongodb replica set의 master 서버
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  persistent_disk_type: 10GB              # cloud config 에 정의한 영구 디스크 타입 
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.207
  properties:
    replSet_hosts: ["10.30.107.207","10.30.107.208","10.30.107.209"] # 첫번째 Host는 replicaSet1의 master
    replSetName: op3 # replicaSet1 의 이름
  templates:
  - name: mongodb_master                # job template 이름(필수)
    release: paasta-mongodb-shard

- name: mongodb_config   #작업 이름(필수): mongodb  mongodb_config
  azs:
  - z5
  instances: 3  # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  persistent_disk_type: 10GB              # cloud config 에 정의한 영구 디스크 타입 
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.210
    - 10.30.107.211
    - 10.30.107.212
  templates:
  - name: mongodb_config                # job template 이름(필수)
    release: paasta-mongodb-shard

- name: mongodb_shard   #작업 이름(필수): mongodb mongodb_shard
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.213
  properties:
    bindIp: 0.0.0.0
    configsvr_hosts: # mongodb_config hosts
    - 10.30.107.210
    - 10.30.107.211
    - 10.30.107.212
    repl_name_host_list: # mongodb_master properties
    - op1/10.30.107.201 # replicaSet1 의 이름/host
    - op2/10.30.107.204 # replicaSet2 의 이름/host
    - op3/10.30.107.207 # replicaSet3 의 이름/host
  templates:
  - name: mongodb_shard                # job template 이름(필수)
    release: paasta-mongodb-shard

- name: mongodb_broker  #작업 이름(필수): mongodb 서비스 브로커
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.214
  templates:
  - name: mongodb_broker                # job template 이름(필수)
    release: paasta-mongodb-shard

- name: mongodb_broker_registrar   # 작업 이름: 서비스 브로커 등록
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  lifecycle: errand   # bosh deploy시 vm에 생성되어 설치 되지 않고 bosh errand 로 실행할때 설정, 주로 테스트 용도에 쓰임
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
  properties:
    broker:   # 서비스 브로커 설정 정보
      host: 10.30.107.214   # 서비스 브로커 IP 
      name: Mongo-DB  # CF에서 서비스 브로커를 생성시 생기는 서비스 이름 브로커에 고정되어있는 값
      password: cloudfoundry  # 브로커 접근 아이디 비밀번호(필수)
      username: admin   # 브로커 접근 아이디(필수)
      protocol: http
      port: 8080  # 브로커 포트
    cf:
      admin_password: admin   # CF 사용자의 패스워드
      admin_username: admin_test   # CF 사용자 이름
      api_url: https://api.115.68.46.189.xip.io  # CF 설치시 설정한 api uri 정보(필수)
  templates:
  - name: mongodb_broker_registrar                # job template 이름(필수)
    release: paasta-mongodb-shard

- name: mongodb_broker_deregistrar   # 작업 이름: 서비스 브로커 등록
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  lifecycle: errand   # bosh deploy시 vm에 생성되어 설치 되지 않고 bosh errand 로 실행할때 설정, 주로 >테스트 용도에 쓰임
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
  properties:
    broker:   # 서비스 브로커 설정 정보
      host: 10.30.107.214   # 서비스 브로커 IP 
      name: Mongo-DB  # CF에서 서비스 브로커를 생성시 생기는 서비스 이름 브로커에 고정되어있는 값
      password: cloudfoundry  # 브로커 접근 아이디 비밀번호(필수)
      username: admin   # 브로커 접근 아이디(필수)
      protocol: http
      port: 8080  # 브로커 포트
    cf:
      admin_password: admin   # CF 사용자의 패스워드
      admin_username: admin_test   # CF 사용자 이름
      api_url: https://api.115.68.46.189.xip.io  # CF 설치시 설정한 api uri 정보(필수)
  templates:
  - name: mongodb_broker_deregistrar                # job template 이름(필수)
    release: paasta-mongodb-shard

properties:
  mongodb:  # mongodb shard release의 여러 job에서 공통적으로 하용하는 properties
    # key는 shard를 구성할 때 mongos와 각 replicaSet의 인증을 하기위해 사용
    key: |
      +Qy+1icfeV8D2WXIfCojRjvYlryMVI2Ry+dAi8mYZ1H1Z9pDstRkOC0/oJYs0L/i
      +Dj/3PurWo8MJuqBYrWVGsRnsx31um0SVAgFZM2GQEKvHIByX5hq/MuHlulSLM0h
      GKkMT19zqDwFBFIN53jN0PLuuOnJ6FxZSb4cTLymfWM543WGpYx/31b8ehPYyeRp
      T7P2o2vUd9hecb8mQFxcjsBN7PTLwuPb5lK0BRL4Ze7rh6qeC8j7M3zimV8lX2X5
      9EtWlQP0ORYIlFpqijatZhS8Bf5AfI1EW6kZgfqwycl2ghxmSIbeleiqyQgYZNKQ
      yBXV9disuBXcKy4tsOjSFvKw7y61kjjQOn8KXElefokefdLbcrpeARP6LR9WwR1v
      ZTHcChfzWA4apHo6gJZkoqGVPjF4ArXTYxZfC+hHrsa5oe3XZjNapwV6XQfBNCuQ
      EihT3Td/B7iAUWJnGQvugFJwYKJ5EYOYubhk8QtO9QIvoZxQPDq9tgUsVgiQ6gty
      ZT83oxFAIgm3vky9l3uPwYi6jQ2FvsEJvDyiZl7gulOaC5UD/BdcM4Y5n/dxy/6Y
      qphWWuPsJwnYBXLJgwtTZ/NkYDYyX/tL9gyzXGPkpMMD7DofFjWEpJvHlVRKIxp1
      /zlxbVOMAmASgZDaqFperSQQyrfQqpuvAA8pRkWgorROyrsiRYYWlJZWWa4qHlI9
      OZ1dDp8o71l3v0SqsKbEtxINpdiUNx4OdafsMNN/KVxw9oGdrPXnDl5DomtmAoKZ
      uaCf3AQ3RsDeymgVX3j5EpLCHBhcPj+0B5tv4Yln652HAzDissOUKPyDf+PJaVRo
      OfDOkUvmuqnwl45DOoOtZ0BMw7hXGdgm6Xfv5jEmtSjJzQ1pfwHOOfiY+zZWhHAi
      ow/WNvLtUgNUhobi+OQb11bMMNNtmGWe+cZft6QzBsnd2xa/tAYTZDfAJ8OCvYQK
      e46UrHd54ZJFzdzicRZ8DeuU9G4K
    user: root  # admin 권한 사용자이름
    passwd: openpaas  # admin 권한 사용자 비밀번호
    port: 27017   # mongodb port

  mongodb_broker:
    db_name: mongodb-broker # mongodb broker 관리용 데이터베이스
    authsource: admin   # mongodb broker 관리용 데이터베이스에 접근할 때 인증정보가 있는 데이터베이스
    hosts: 10.30.107.213 # mongodb Host
```

-	deploy-mongodb-shard-bosh2.0.sh 파일을 서버 환경에 맞게 수정한다.

```sh
#!/bin/bash
# stemcell 버전은 3309 버전으로 사용하시고 https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Download_Page.md 에서 다운받아 쓰십시요.

bosh -e micro-bosh -d paasta-mongodb-shard-service deploy paasta_mongodb_shard_bosh2.0.yml \
   -v default_network_name=service_private \
   -v stemcell_os=ubuntu-trusty \
   -v stemcell_version=3309 \
   -v vm_type_small=minimal
```


-	Mongodb 서비스팩을 배포한다.

- **사용 예시**

		$ ./deploy-mongodb-shard-bosh2.0.sh
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Using deployment 'paasta-mongodb-shard-service'

		+ stemcells:
		+ - alias: default
		+   os: ubuntu-trusty
		+   version: '3309'

		+ update:
		+   canaries: 1
		+   canary_watch_time: 30000-180000
		+   max_in_flight: 6
		+   update_watch_time: 30000-180000

		+ release:
		+   name: paasta-mongodb-shard
		+   version: '2.0'

		+ instance_groups:
		+ - azs:
		+   - z5
		+   instances: 2
		+   name: mongodb_slave1
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.102
		+     - 10.30.107.103
		+   persistent_disk_type: 10GB
		+   properties:
		+     replSetName: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: mongodb_slave
		+     release: paasta-mongodb-shard
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 2
		+   name: mongodb_slave2
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.105
		+     - 10.30.107.106
		+   persistent_disk_type: 10GB
		+   properties:
		+     replSetName: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: mongodb_slave
		+     release: paasta-mongodb-shard
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 2
		+   name: mongodb_slave3
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.108
		+     - 10.30.107.109
		+   persistent_disk_type: 10GB
		+   properties:
		+     replSetName: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: mongodb_slave
		+     release: paasta-mongodb-shard
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: mongodb_master1
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.101
		+   persistent_disk_type: 10GB
		+   properties:
		+     replSet_hosts:
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+     replSetName: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: mongodb_master
		+     release: paasta-mongodb-shard
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: mongodb_master2
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.104
		+   persistent_disk_type: 10GB
		+   properties:
		+     replSet_hosts:
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+     replSetName: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: mongodb_master
		+     release: paasta-mongodb-shard
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: mongodb_master3
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.107
		+   persistent_disk_type: 10GB
		+   properties:
		+     replSet_hosts:
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+     replSetName: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: mongodb_master
		+     release: paasta-mongodb-shard
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 3
		+   name: mongodb_config
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.110
		+     - 10.30.107.111
		+     - 10.30.107.112
		+   persistent_disk_type: 10GB
		+   stemcell: default
		+   templates:
		+   - name: mongodb_config
		+     release: paasta-mongodb-shard
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: mongodb_shard
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.113
		+   properties:
		+     bindIp: "<redacted>"
		+     configsvr_hosts:
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+     repl_name_host_list:
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: mongodb_shard
		+     release: paasta-mongodb-shard
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: mongodb_broker
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.114
		+   stemcell: default
		+   templates:
		+   - name: mongodb_broker
		+     release: paasta-mongodb-shard
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   lifecycle: errand
		+   name: mongodb_broker_registrar
		+   networks:
		+   - name: service_private
		+   properties:
		+     broker:
		+       host: "<redacted>"
		+       name: "<redacted>"
		+       password: "<redacted>"
		+       port: "<redacted>"
		+       protocol: "<redacted>"
		+       username: "<redacted>"
		+     cf:
		+       admin_password: "<redacted>"
		+       admin_username: "<redacted>"
		+       api_url: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: mongodb_broker_registrar
		+     release: paasta-mongodb-shard
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   lifecycle: errand
		+   name: mongodb_broker_deregistrar
		+   networks:
		+   - name: service_private
		+   properties:
		+     broker:
		+       host: "<redacted>"
		+       name: "<redacted>"
		+       password: "<redacted>"
		+       port: "<redacted>"
		+       protocol: "<redacted>"
		+       username: "<redacted>"
		+     cf:
		+       admin_password: "<redacted>"
		+       admin_username: "<redacted>"
		+       api_url: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: mongodb_broker_deregistrar
		+     release: paasta-mongodb-shard
		+   vm_type: minimal

		+ name: paasta-mongodb-shard-service

		+ properties:
		+   mongodb:
		+     key: "<redacted>"
		+     passwd: "<redacted>"
		+     port: "<redacted>"
		+     user: "<redacted>"
		+   mongodb_broker:
		+     authsource: "<redacted>"
		+     db_name: "<redacted>"
		+     hosts: "<redacted>"

		Continue? [yN]: y

		Task 8129

		Task 8129 | 06:53:48 | Preparing deployment: Preparing deployment (00:00:02)
		Task 8129 | 06:53:52 | Preparing package compilation: Finding packages to compile (00:00:00)
		Task 8129 | 06:53:52 | Compiling packages: cli/668cd93a9dc828726341909df08f74dce57478fe
		Task 8129 | 06:53:52 | Compiling packages: java7/a6b0fd78eadc51bfbf305a5571a710128fae6fe5
		Task 8129 | 06:53:52 | Compiling packages: mongodb/f48a8e934cc387162028629a196040c17bfa3401
		Task 8129 | 06:55:49 | Compiling packages: cli/668cd93a9dc828726341909df08f74dce57478fe (00:01:57)
		Task 8129 | 06:55:59 | Compiling packages: java7/a6b0fd78eadc51bfbf305a5571a710128fae6fe5 (00:02:07)
		Task 8129 | 06:55:59 | Compiling packages: mongodb_broker/25bacb6034e74fc63b592c8bb6ba98cf512afbbc
		Task 8129 | 06:56:11 | Compiling packages: mongodb/f48a8e934cc387162028629a196040c17bfa3401 (00:02:19)
		Task 8129 | 06:56:24 | Compiling packages: mongodb_broker/25bacb6034e74fc63b592c8bb6ba98cf512afbbc (00:00:25)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_slave1/bb6275f1-4ab5-4998-ba89-ef30c36c3f67 (1)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_slave1/2710c368-dbc2-4d72-a100-1fa37d73e2ec (0)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_slave2/fed23144-9c18-42f6-9f99-213f7dc294ee (0)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_config/35ee66e6-9c25-44c2-85a4-e7c1d520641b (1)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_slave3/7cebf99b-5a79-4033-a4e8-86f8d476a709 (0)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_slave2/9671e09b-7ca1-4da2-af8a-88d20caeebfe (1)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_config/cc798fba-7840-46ea-9211-6b5646fc766f (0)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_master3/88e1aa1c-fb1f-467d-a550-b6334fdfce8d (0)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_broker/0e8933f1-1b67-4486-b37a-2b104da1351a (0)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_master1/1e8b971e-c503-4ba6-bcba-ab28dd7dd797 (0)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_slave3/ab6d22fb-d436-4c1c-a423-9e9d82c4266a (1)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_master2/7a4460e4-a9b5-4d15-9508-adba3405f387 (0)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_config/935aed3c-e7a4-4179-b397-68d0535bc1d9 (2)
		Task 8129 | 06:57:10 | Creating missing vms: mongodb_shard/1fd85812-c8d4-4ebd-98f5-c8cf637db9e5 (0)
		Task 8129 | 07:01:12 | Creating missing vms: mongodb_slave1/bb6275f1-4ab5-4998-ba89-ef30c36c3f67 (1) (00:04:02)
		Task 8129 | 07:01:26 | Creating missing vms: mongodb_master2/7a4460e4-a9b5-4d15-9508-adba3405f387 (0) (00:04:16)
		Task 8129 | 07:01:49 | Creating missing vms: mongodb_slave3/ab6d22fb-d436-4c1c-a423-9e9d82c4266a (1) (00:04:39)
		Task 8129 | 07:01:53 | Creating missing vms: mongodb_slave2/fed23144-9c18-42f6-9f99-213f7dc294ee (0) (00:04:43)
		Task 8129 | 07:02:01 | Creating missing vms: mongodb_config/935aed3c-e7a4-4179-b397-68d0535bc1d9 (2) (00:04:51)
		Task 8129 | 07:02:04 | Creating missing vms: mongodb_master3/88e1aa1c-fb1f-467d-a550-b6334fdfce8d (0) (00:04:54)
		Task 8129 | 07:02:05 | Creating missing vms: mongodb_master1/1e8b971e-c503-4ba6-bcba-ab28dd7dd797 (0) (00:04:55)
		Task 8129 | 07:02:06 | Creating missing vms: mongodb_slave1/2710c368-dbc2-4d72-a100-1fa37d73e2ec (0) (00:04:56)
		Task 8129 | 07:02:10 | Creating missing vms: mongodb_slave2/9671e09b-7ca1-4da2-af8a-88d20caeebfe (1) (00:05:00)
		Task 8129 | 07:02:12 | Creating missing vms: mongodb_config/35ee66e6-9c25-44c2-85a4-e7c1d520641b (1) (00:05:02)
		Task 8129 | 07:02:12 | Creating missing vms: mongodb_shard/1fd85812-c8d4-4ebd-98f5-c8cf637db9e5 (0) (00:05:02)
		Task 8129 | 07:02:13 | Creating missing vms: mongodb_broker/0e8933f1-1b67-4486-b37a-2b104da1351a (0) (00:05:03)
		Task 8129 | 07:02:13 | Creating missing vms: mongodb_config/cc798fba-7840-46ea-9211-6b5646fc766f (0) (00:05:03)
		Task 8129 | 07:02:14 | Creating missing vms: mongodb_slave3/7cebf99b-5a79-4033-a4e8-86f8d476a709 (0) (00:05:04)
		Task 8129 | 07:02:15 | Updating instance mongodb_slave1: mongodb_slave1/2710c368-dbc2-4d72-a100-1fa37d73e2ec (0) (canary) (00:02:03)
		Task 8129 | 07:04:18 | Updating instance mongodb_slave1: mongodb_slave1/bb6275f1-4ab5-4998-ba89-ef30c36c3f67 (1) (00:02:03)
		Task 8129 | 07:06:21 | Updating instance mongodb_slave2: mongodb_slave2/fed23144-9c18-42f6-9f99-213f7dc294ee (0) (canary) (00:02:03)
		Task 8129 | 07:08:24 | Updating instance mongodb_slave2: mongodb_slave2/9671e09b-7ca1-4da2-af8a-88d20caeebfe (1) (00:02:03)
		Task 8129 | 07:10:27 | Updating instance mongodb_slave3: mongodb_slave3/7cebf99b-5a79-4033-a4e8-86f8d476a709 (0) (canary) (00:02:04)
		Task 8129 | 07:12:31 | Updating instance mongodb_slave3: mongodb_slave3/ab6d22fb-d436-4c1c-a423-9e9d82c4266a (1) (00:02:05)
		Task 8129 | 07:14:36 | Updating instance mongodb_master1: mongodb_master1/1e8b971e-c503-4ba6-bcba-ab28dd7dd797 (0) (canary) (00:02:04)
		Task 8129 | 07:16:40 | Updating instance mongodb_master2: mongodb_master2/7a4460e4-a9b5-4d15-9508-adba3405f387 (0) (canary) (00:02:01)
		Task 8129 | 07:18:42 | Updating instance mongodb_master3: mongodb_master3/88e1aa1c-fb1f-467d-a550-b6334fdfce8d (0) (canary) (00:02:03)
		Task 8129 | 07:20:45 | Updating instance mongodb_config: mongodb_config/cc798fba-7840-46ea-9211-6b5646fc766f (0) (canary) (00:02:07)
		Task 8129 | 07:22:52 | Updating instance mongodb_config: mongodb_config/35ee66e6-9c25-44c2-85a4-e7c1d520641b (1)
		Task 8129 | 07:22:52 | Updating instance mongodb_config: mongodb_config/935aed3c-e7a4-4179-b397-68d0535bc1d9 (2)
		Task 8129 | 07:24:59 | Updating instance mongodb_config: mongodb_config/35ee66e6-9c25-44c2-85a4-e7c1d520641b (1) (00:02:07)
		Task 8129 | 07:25:01 | Updating instance mongodb_config: mongodb_config/935aed3c-e7a4-4179-b397-68d0535bc1d9 (2) (00:02:09)
		Task 8129 | 07:25:01 | Updating instance mongodb_shard: mongodb_shard/1fd85812-c8d4-4ebd-98f5-c8cf637db9e5 (0) (canary) (00:00:58)
		Task 8129 | 07:25:59 | Updating instance mongodb_broker: mongodb_broker/0e8933f1-1b67-4486-b37a-2b104da1351a (0) (canary) (00:01:01)

		Task 8129 Started  Thu Sep 13 06:53:48 UTC 2018
		Task 8129 Finished Thu Sep 13 07:27:00 UTC 2018
		Task 8129 Duration 00:33:12
		Task 8129 done

		Succeeded


-	배포된 Mongodb 서비스팩을 확인한다.

- **사용 예시**

		$bosh -e micro-bosh -d paasta-mongodb-shard-service vms
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Task 8176. Done

		Deployment 'paasta-mongodb-shard-service'

		Instance                                              Process State  AZ  IPs            VM CID                                   VM Type  Active  
		mongodb_broker/0e8933f1-1b67-4486-b37a-2b104da1351a   running        z5  10.30.107.114  vm-e0bb79c6-6482-497a-b071-f7df4bf2a059  minimal  true  
		mongodb_config/35ee66e6-9c25-44c2-85a4-e7c1d520641b   running        z5  10.30.107.111  vm-672ce5b9-4d8f-4b22-9745-43f7d9e39402  minimal  true  
		mongodb_config/935aed3c-e7a4-4179-b397-68d0535bc1d9   running        z5  10.30.107.112  vm-8069a84b-5a91-44ca-a5d8-cca37b5d8952  minimal  true  
		mongodb_config/cc798fba-7840-46ea-9211-6b5646fc766f   running        z5  10.30.107.110  vm-5a7a9d16-8de4-4adf-b504-1364716decce  minimal  true  
		mongodb_master1/1e8b971e-c503-4ba6-bcba-ab28dd7dd797  running        z5  10.30.107.101  vm-54b33ec2-582d-44ef-a4bf-6281acfbf81b  minimal  true  
		mongodb_master2/7a4460e4-a9b5-4d15-9508-adba3405f387  running        z5  10.30.107.104  vm-a388a44e-4ab9-4340-9227-b12b7bd2c410  minimal  true  
		mongodb_master3/88e1aa1c-fb1f-467d-a550-b6334fdfce8d  running        z5  10.30.107.107  vm-9c6aed35-69aa-4a7d-9b08-c5671e728e2a  minimal  true  
		mongodb_shard/1fd85812-c8d4-4ebd-98f5-c8cf637db9e5    running        z5  10.30.107.113  vm-c2628ba8-feed-4401-b1c9-be1445722d34  minimal  true  
		mongodb_slave1/2710c368-dbc2-4d72-a100-1fa37d73e2ec   running        z5  10.30.107.102  vm-048757cf-1c19-4c30-a3cd-2b0dd05c1554  minimal  true  
		mongodb_slave1/bb6275f1-4ab5-4998-ba89-ef30c36c3f67   running        z5  10.30.107.103  vm-6d0f52ef-a0b3-4c26-8e04-cb5cef30337d  minimal  true  
		mongodb_slave2/9671e09b-7ca1-4da2-af8a-88d20caeebfe   running        z5  10.30.107.106  vm-8a57713b-68df-4639-8ab3-3d12c01fd880  minimal  true  
		mongodb_slave2/fed23144-9c18-42f6-9f99-213f7dc294ee   running        z5  10.30.107.105  vm-c58e860a-8b5e-43e1-abe9-c3043cbfb16d  minimal  true  
		mongodb_slave3/7cebf99b-5a79-4033-a4e8-86f8d476a709   running        z5  10.30.107.108  vm-d34d8a3f-37fb-41a8-b995-6e8c7e8ff041  minimal  true  
		mongodb_slave3/ab6d22fb-d436-4c1c-a423-9e9d82c4266a   running        z5  10.30.107.109  vm-ca14b629-4d00-4c96-8782-1cf7a174ce1e  minimal  true  

		14 vms

		Succeeded


### <div id='24'> 2.4. Mongodb 서비스 브로커 등록

Mongodb 서비스팩 배포가 완료 되었으면 Application에서 서비스 팩을 사용하기 위해서 먼저 Mongodb 서비스 브로커를 등록해 주어야 한다.

서비스 브로커 등록시 개방형 클라우드 플랫폼에서 서비스브로커를 등록할 수 있는 사용자로 로그인이 되어있어야 한다.

##### 서비스 브로커 목록을 확인한다.


>`$ cf service-brokers`

>![mongodb_image_06]

<br>

##### Mongodb 서비스 브로커를 등록한다.

>`$ cf create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{서비스팩 URL(IP)}`
  
  **서비스팩 이름** : 서비스 팩 관리를 위해 PaaS-TA에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스팩 리스트의 명칭이다.<br>
  **서비스팩 사용자ID** / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID입니다. 서비스팩도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.<br>
  **서비스팩 URL** : 서비스팩이 제공하는 API를 사용할 수 있는 URL을 입력한다.

>`$ cf create-service-broker mongodb-shard-service-broker admin cloudfoundry http://10.30.107.114:8080`

> ![mongodb_image_07]


##### 등록된 mongodb 서비스 브로커를 확인한다.

>`$ cf service-brokers`

> ![mongodb_image_08]


##### 접근 가능한 서비스 목록을 확인한다.

>`$ cf service-access`

> ![mongodb_image_09]

>서비스 브로커 생성시 디폴트로 접근을 허용하지 않는다.


##### 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)

>`$ cf enable-service-access Mongo-DB` <br>
>`$ cf service-access`

> ![mongodb_image_10]


# <div id='3'> 3. Mongodb 연동 Sample Web App 설명

본 Sample Web App은 PaaS-TA에 배포되며 Mongodb의 서비스를 Provision과 Bind를 한 상태에서 사용이 가능하다.


### <div id='31'> 3.1. Sample App 구조

Sample Web App은 PaaS-TA에 App으로 배포가 된다. App을 배포하여 구동시 Bind 된 Mongodb 서비스 연결정보로 접속하여 초기 데이터를 생성하게 된다. 배포 완료 후 정상적으로 App 이 구동되면 브라우저나 curl로 해당 App에 접속 하여 Mongodb 환경정보(서비스 연결 정보)와 초기 적재된 데이터를 보여준다.

Sample Web App 구조는 다음과 같다.

<table>
  <tr>
    <td>이름</td>
    <td>설명</td>
  </tr>
  <tr>
    <td>src</td>
    <td>Sample 소스디렉토리</td>
  </tr>
  <tr>
    <td>manifest</td>
    <td>PaaS-TA에 app 배포시 필요한 설정을 저장하는 파일</td>
  </tr>
  <tr>
    <td>build.gradle</td>
    <td>gradle project 설정 파일</td>
  </tr>
  <tr>
    <td>build</td>
    <td>gradle 빌드시 생성되는 디렉토리(war 파일, classes 폴더 등)</td>
  </tr>
</table>


##### PaaS-TA-Sample-Apps.zip 파일 압축을 풀고 Service 폴더안에 있는 Mongodb Sample Web App인 hello-spring-mongodb를 복사한다.

>`$ ls -all`

> ![mongodb_image_11]

<br>

### <div id='32'> 3.2. PaaS-TA에서 서비스 신청

Sample Web App에서 Mongodb 서비스를 사용하기 위해서는 서비스 신청(Provision)을 해야 한다.
*참고: 서비스 신청시 개방형 클라우드 플랫폼에서 서비스를 신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.


##### 먼저 PaaS-TA Marketplace에서 서비스가 있는지 확인을 한다.

>`$ cf marketplace`

> ![mongodb_image_12]

<br>

##### Marketplace에서 원하는 서비스가 있으면 서비스 신청(Provision)을 한다.

>`$ cf create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{서비스팩 URL(IP)}`
  
  **서비스팩 이름** : 서비스 팩 관리를 위해 PaaS-TA에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스팩 리스트의 명칭이다.<br>
  **서비스팩 사용자ID** / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID입니다. 서비스팩도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.<br>
  **서비스팩 URL** : 서비스팩이 제공하는 API를 사용할 수 있는 URL을 입력한다.
  
>`$ cf create-service Mongo-DB default-plan mongodb-service-instance`

>![mongodb_image_13]

<br>

##### 생성된 Mongodb 서비스 인스턴스를 확인한다.

>`$ cf services`

>![mongodb_image_14]

<br>


3.3. <div id='33'> Sample App에 서비스 바인드 신청 및 App 확인
-------------------------------------------------
서비스 신청이 완료되었으면 Sample Web App 에서는 생성된 서비스 인스턴스를 Bind 하여 App에서 Mongodb 서비스를 이용한다.
*참고: 서비스 Bind 신청시 개방형 클라우드 플랫폼에서 서비스 Bind신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.


##### Sample Web App 디렉토리로 이동하여 manifest 파일을 확인한다.

>`$ cd hello-spring-mongodb` <br>
>`$ vi manifest.yml`

>![mongodb_image_14]

<br>

```

---
applications:
- name: hello-spring-mysql       #배포할 App 이름
  memory: 512M                # 배포시 메모리 사이즈
  instances: 1                    # 배포 인스턴스 수
path: target/hello-spring-mysql-1.0.0-BUILD-SNAPSHOT.war      #배포하는 App 파일 PATH
```

참고: target/hello-spring-mysql-1.0.0-BUILD-SNAPSHOT.war파일이 존재 하지 않을 경우 mvn 빌드를 수행 하면 파일이 생성된다.
<br>

##### --no-start 옵션으로 App을 배포한다.
- -no-start: App 배포시 구동은 하지 않는다.

>`$ cf push --no-start`

>![mongodb_image_15]

<br>

##### 배포된 Sample App을 확인하고 로그를 수행한다.

>`$ cf apps`

>![mongodb_image_16]

<br>

>`$ cf logs hello-spring-mongodb`  **// cf logs {배포된 App명}**

![mongodb_image_17]

<br>

##### Sample Web App에서 생성한 서비스 인스턴스 바인드 신청을 한다.

>`$  cf bind-service hello-spring-Mongodb Mongodb-service-instance` 

> ![mongodb_image_42]

<br>

##### 바인드가 적용되기 위해서 App을 재기동한다.

>`$  cf restart hello-spring-mongodb` 

> ![mongodb_image_18]

(참고) 바인드 후 App구동시 Mongodb 서비스 접속 에러로 App 구동이 안될 경우 보안 그룹을 추가한다.

##### rule.json 파일을 만들고 아래와 같이 내용을 넣는다.

>`$ vi rule.json` 

```
[
  {
    "protocol": "tcp",
    "destination": "10.20.0.153",
    "ports": "27017"
  }
]
```

##### 보안 그룹을 생성한다.

>`$  cf create-security-group Mongo-DB rule.json` 

> ![mongodb_image_19]

<br>


##### 모든 App에 Mongodb 서비스를 사용할 수 있도록 생성한 보안 그룹을 적용한다.


>`$ cf bind-running-security-group Mongo-DB` 

> ![mongodb_image_20]

<br>

-	App을 리부팅 한다.
```
$ cf restart hello-spring-Mongodb
```
![mongodb_image_21]


##### App이 정상적으로 Mongodb 서비스를 사용하는지 확인한다.

> curl 로 확인

>`$  curl hello-spring-Mongodb.115.68.46.30.xip.io` 

> ![mongodb_image_22]


##### 브라우에서 확인
> ![mongodb_image_23]


# <div id='4'> 4.  Mongodb Client 툴 접속

Application에 바인딩된 Mongodb 서비스 연결정보는 Private IP로 구성되어 있기 때문에 Mongodb Client 툴에서 직접 연결할수 없다. 따라서 SSH 터널, Proxy 터널 등을 제공하는 Mongodb Client 툴을 사용해서 연결하여야 한다. 본 가이드는 SSH 터널을 이용하여 연결 하는 방법을 제공하며 Mongodb Client 툴로써는 MongoChef 로 가이드한다. MongoChef 에서 접속하기 위해서 먼저 SSH 터널링 할수 있는 VM 인스턴스를 생성해야한다. 이 인스턴스는 SSH로 접속이 가능해야 하고 접속 후 PaaS-TA에 설치한 서비스팩에 Private IP 와 해당 포트로 접근이 가능하도록 시큐리티 그룹을 구성해야 한다. 이 부분은 OpenStack관리자 및 PaaS-TA 운영자에게 문의하여 구성한다.


# <div id='41'> 4.1.  MongoChef 설치 & 연결
MongoChef 프로그램은 무료로 사용할 수 있는 소프트웨어이다.


##### MongoChef을 다운로드 하기 위해 아래 URL로 이동하여 설치파일을 다운로드 한다.
[**http://3t.io/mongochef/download/platform/**](http://3t.io/mongochef/download/platform/)
> ![mongodb_image_24]


##### 다운로드한 설치파일을 실행한다.
> ![mongodb_image_25]

<br>

##### MongoChef 설치를 위한 안내사항이다. Next 버튼을 클릭한다.
>[mongodb_image_26]

<br>

##### 프로그램 라이선스에 관련된 내용이다. 동의(I accept the terms in the License Agreement)에 체크 후 Next 버튼을 클릭한다.
> ![mongodb_image_27]

<br>

##### MongoChef 을 설치할 경로를 설정 후 Next 버튼을 클릭한다. 별도의 경로 설정이 필요 없을 경우 default로 C드라이브 Program Files 폴더에 설치가 된다.
> ![mongodb_image_28]

<br>

##### Install 버튼을 클릭하여 설치를 진행한다.
> ![mongodb_image_29]

<br>

##### Finish 버튼 클릭으로 설치를 완료한다.
> ![mongodb_image_30]

<br>

##### MongoChef를 실행했을 때 처음 뜨는 화면이다. 이 화면에서 Server에 접속하기 위한 profile을 설정/저장하여 접속할 수 있다. Connect버튼을 클릭한다.
> ![mongodb_image_31]

<br>

##### 새로운 접속 정보를 작성하기 위해New Connection 버튼을 클릭한다.
> ![mongodb_image_32]

<br>

##### Server에 접속하기 위한 Connection 정보를 입력한다.
> ![mongodb_image_33]

<br>

##### 서버 정보는 Application에 바인드되어 있는 서버 정보를 입력한다. cf env <app_name> 명령어로 이용하여 확인한다.
>`$ cf env hello-spring-mongodb` 

<br>

> ![mongodb_image_34]

<br>

##### Authentication탭으로 이동하여 mongodb 의 인증정보를 입력한다.
> ![mongodb_image_35]

<br>

##### SSH 터널 탭을 클릭하고 PaaS-TA 운영 관리자에게 제공받은 SSH 터널링 가능한 서버 정보를 입력한다.
> ![mongodb_image_36]

<br>

##### 모든 정보를 입력했으면 Test Connection 버튼을 눌러 접속 테스트를 한다.
> ![mongodb_image_37]

<br>

##### 모두 OK 결과가 나오면 정상적으로 접속이 된다는 것이다. OK 버튼을 누른다.


##### Save 버튼을 눌러 작성한 접속정보를 저장한다.
> ![mongodb_image_38]

<br>

##### 방금 저장한 접속정보를 선택하고 Connect 버튼을 클릭하여 접속한다.
> ![mongodb_image_39]

<br>

##### 접속이 완료되면 좌측에 스키마 정보가 나타난다. 컬럼을 더블클릭 해보면 우측에 적재되어있는 데이터가 출력된다.
> ![mongodb_image_40]

<br>

##### 우측 화면에 쿼리 항목에 Query문을 작성한 후 실행 버튼(삼각형)을 클릭한다. Query문에 이상이 없다면 정상적으로 결과를 얻을 수 있을 것이다.
> ![mongodb_image_41]


[mongodb_image_01]:/Service-Guide/images/mongodb/mongodb_image_01.png
[mongodb_image_02]:/Service-Guide/images/mongodb/mongodb_image_02.png
[mongodb_image_03]:/Service-Guide/images/mongodb/mongodb_image_03.png
[mongodb_image_04]:/Service-Guide/images/mongodb/mongodb_image_04.png
[mongodb_image_05]:/Service-Guide/images/mongodb/mongodb_image_05.png
[mongodb_image_06]:/Service-Guide/images/mongodb/mongodb_image_06.png
[mongodb_image_07]:/Service-Guide/images/mongodb/mongodb_image_07.png
[mongodb_image_08]:/Service-Guide/images/mongodb/mongodb_image_08.png
[mongodb_image_09]:/Service-Guide/images/mongodb/mongodb_image_09.png
[mongodb_image_10]:/Service-Guide/images/mongodb/mongodb_image_10.png
[mongodb_image_11]:/Service-Guide/images/mongodb/mongodb_image_11.png
[mongodb_image_12]:/Service-Guide/images/mongodb/mongodb_image_12.png
[mongodb_image_13]:/Service-Guide/images/mongodb/mongodb_image_13.png
[mongodb_image_14]:/Service-Guide/images/mongodb/mongodb_image_14.png
[mongodb_image_15]:/Service-Guide/images/mongodb/mongodb_image_15.png
[mongodb_image_16]:/Service-Guide/images/mongodb/mongodb_image_16.png
[mongodb_image_17]:/Service-Guide/images/mongodb/mongodb_image_17.png
[mongodb_image_18]:/Service-Guide/images/mongodb/mongodb_image_18.png
[mongodb_image_19]:/Service-Guide/images/mongodb/mongodb_image_19.png
[mongodb_image_20]:/Service-Guide/images/mongodb/mongodb_image_20.png
[mongodb_image_21]:/Service-Guide/images/mongodb/mongodb_image_21.png
[mongodb_image_22]:/Service-Guide/images/mongodb/mongodb_image_22.png
[mongodb_image_23]:/Service-Guide/images/mongodb/mongodb_image_23.png
[mongodb_image_24]:/Service-Guide/images/mongodb/mongodb_image_24.png
[mongodb_image_25]:/Service-Guide/images/mongodb/mongodb_image_25.png
[mongodb_image_26]:/Service-Guide/images/mongodb/mongodb_image_26.png
[mongodb_image_27]:/Service-Guide/images/mongodb/mongodb_image_27.png
[mongodb_image_28]:/Service-Guide/images/mongodb/mongodb_image_28.png
[mongodb_image_29]:/Service-Guide/images/mongodb/mongodb_image_29.png
[mongodb_image_30]:/Service-Guide/images/mongodb/mongodb_image_30.png
[mongodb_image_31]:/Service-Guide/images/mongodb/mongodb_image_31.png
[mongodb_image_32]:/Service-Guide/images/mongodb/mongodb_image_32.png
[mongodb_image_33]:/Service-Guide/images/mongodb/mongodb_image_33.png
[mongodb_image_34]:/Service-Guide/images/mongodb/mongodb_image_34.png
[mongodb_image_35]:/Service-Guide/images/mongodb/mongodb_image_35.png
[mongodb_image_36]:/Service-Guide/images/mongodb/mongodb_image_36.png
[mongodb_image_37]:/Service-Guide/images/mongodb/mongodb_image_37.png
[mongodb_image_38]:/Service-Guide/images/mongodb/mongodb_image_38.png
[mongodb_image_39]:/Service-Guide/images/mongodb/mongodb_image_39.png
[mongodb_image_40]:/Service-Guide/images/mongodb/mongodb_image_40.png
[mongodb_image_41]:/Service-Guide/images/mongodb/mongodb_image_41.png
[mongodb_image_42]:/Service-Guide/images/mongodb/mongodb_image_42.png
