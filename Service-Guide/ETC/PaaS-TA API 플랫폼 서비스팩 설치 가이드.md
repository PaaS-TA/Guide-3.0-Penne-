## Table of Contents
1. [문서 개요](#1)
  - 1.1. [목적](#11)
  - 1.2. [범위](#12)
  - 1.3. [시스템 구성도](#13)
  - 1.4. [참고자료](#14)
2. [API 플랫폼 서비스팩 설치](#2)
  - 2.1. [설치전 준비사항](#21)
  - 2.2. [API 플랫폼 서비스 릴리즈 업로드](#22)
  - 2.3. [API 플랫폼 서비스 Deployment 파일 수정 및 배포](#23)
  - 2.4. [API 매니저에서 API 생성 및 배포](#24)
  - 2.5. [API 플랫폼 서비스 브로커 등록](#25)
3. [API 플랫폼 연동](#3)
  - 3.1. [Sample Web App에 서비스 바인드 신청 및 App 확인](#31)
  - 3.2. [서비스 바인드 확인](#32)


# <div id='1'> 1. 문서 개요

### <div id='11'> 1.1. 목적
      
본 문서(API 플랫폼 서비스팩 설치 가이드)는 전자정부표준프레임워크 기반의 PaaS-TA에서 제공되는 서비스팩인 API 플랫폼 서비스팩을 Bosh를 이용하여 설치 하는 방법과 PaaS-TA의 SaaS 형태로 제공하는 Application 에서 API 플랫폼 서비스를 사용하는 방법을 기술하였다.
PaaS-TA 3.5 버전부터는 Bosh2.0 기반으로 deploy를 진행하며 기존 Bosh1.0 기반으로 설치를 원할경우에는 PaaS-TA 3.1 이하 버전의 문서를 참고한다.

### <div id='12'> 1.2. 범위 

설치 범위는 API 플랫폼 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다. 


### <div id='13'> 1.3. 시스템 구성도
본 문서의 설치된 시스템 구성도입니다.  API 플랫폼(api manager), API 플랫폼 서비스 브로커, Business Activity Monitor(bam), Maria DB로 최소사항을 구성하였다.

![apiplatform_image_01]

* 설치할때 cloud config에서 사용하는 VM_Tpye명과 스펙 

| VM_Type | 스펙 |
|--------|-------|
|minimal| 1vCPU / 1GB RAM / 8GB Disk|
|service_medium| 2vCPU / 4GB RAM / 8GB Disk|

* 각 Instance의 Resource Pool과 스펙

| 구분 | Resource Pool | 스펙 |
|--------|-------|-------|
| apiplatform-broker | minimal | 1vCPU / 1GB RAM / 8GB Disk |
| apimanager | service_medium | 2vCPU / 4GB RAM / 8GB Disk |
| bam | service_medium | 2vCPU / 4GB RAM / 8GB Disk |
| mariadb | minimal | 1vCPU / 1GB RAM / 8GB Disk |

### <div id='14'> 1.4. 참고자료
[**http://bosh.io/docs**](http://bosh.io/docs)  
[**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/)  
[**https://docs.wso2.com/display/AM180/Quick+Start+Guide**](https://docs.wso2.com/display/AM180/Quick+Start+Guide/)


# <div id='2'> 2. API 플랫폼 서비스팩 설치

### <div id='21'> 2.1. 설치전 준비사항
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


### <div id='22'> 2.2. API 플랫폼 서비스 릴리즈 업로드

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

-	API 플랫폼 서비스 릴리즈가 업로드 되어 있지 않은 것을 확인

-	API 플랫폼 서비스 릴리즈 파일을 업로드한다.

- **사용 예시**

		$ bosh -e micro-bosh upload-release paasta-apiplatform-2.0.tgz
    		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		######################################################## 100.00% 165.27 MiB/s 5s
		Task 7608

		Task 7608 | 07:51:51 | Extracting release: Extracting release (00:00:14)
		Task 7608 | 07:52:05 | Verifying manifest: Verifying manifest (00:00:00)
		Task 7608 | 07:52:05 | Resolving package dependencies: Resolving package dependencies (00:00:00)
		Task 7608 | 07:52:06 | Creating new packages: apiplatform-broker/b9356e568d495e53294ba465c28b2cac8b923e1f (00:00:03)
		Task 7608 | 07:52:09 | Creating new packages: bam/c72a2d571666da622c7a61b638289dc5477d4d25 (00:00:05)
		Task 7608 | 07:52:14 | Creating new packages: java7/5e1e7d676990295c095b632bddc5c42c0da1fccd (00:00:04)
		Task 7608 | 07:52:18 | Creating new packages: mariadb/76d00089f1c7ee1122f6b584d26d21a14254e1f0 (00:00:05)
		Task 7608 | 07:52:23 | Creating new packages: jre7/856cd96de84744fa28c222e7a0d8e4a357203e74 (00:00:01)
		Task 7608 | 07:52:24 | Creating new packages: mysql_connector_java/b900fe25d061ec118fbd0cead39366de046d9a68 (00:00:01)
		Task 7608 | 07:52:25 | Creating new packages: apimanager/f6fdd14e4f28d302661aae793f82c6949515036d (00:00:09)
		Task 7608 | 07:52:34 | Creating new packages: cf-cli/99238e9bbf54ff4ebc80aef311127c2568e5497d (00:00:00)
		Task 7608 | 07:52:34 | Creating new jobs: apiplatform-broker/ea5d45d4d3a32c1aa4749b06ccf2810db5d4623c (00:00:01)
		Task 7608 | 07:52:35 | Creating new jobs: bam/ff28a9aaf2d72880c7f5416722c17fa3cc98d384 (00:00:00)
		Task 7608 | 07:52:35 | Creating new jobs: mariadb/2daf54b9238968ef5084148a4917ca9188ab4945 (00:00:00)
		Task 7608 | 07:52:35 | Creating new jobs: broker-deregistrar/40ceeab0a3394ba8327d7c29dcef90f3bc8b8928 (00:00:00)
		Task 7608 | 07:52:35 | Creating new jobs: apimanager/0a6694f5b75baf21b7554109ab9dc2d2d6a0caea (00:00:00)
		Task 7608 | 07:52:35 | Creating new jobs: broker-registrar/e12e1c97975cce90a9b7199f414587fd6715d170 (00:00:00)
		Task 7608 | 07:52:35 | Release has been created: paasta-apiplatform/2.0 (00:00:00)

		Task 7608 Started  Wed Sep 12 07:51:51 UTC 2018
		Task 7608 Finished Wed Sep 12 07:52:35 UTC 2018
		Task 7608 Duration 00:00:44
		Task 7608 done

		Succeeded


-	업로드 된 MySQL 릴리즈를 확인한다.

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
		paasta-apiplatform                2.0       85e3f01e+  
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
		
-	API 플랫폼 서비스 릴리즈가 업로드 되어 있는 것을 확인

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
		
>Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드를 해야 한다. (API 플랫폼 서비스는 stemcell 3309 버전을 사용)


### <div id='23'> 2.3. API 플랫폼 서비스 릴리즈 Deployment 파일 수정 및 배포

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
# paasta-apiplatform-service 설정 파일 내용
---
name: paasta-apiplatform-service                     # 서비스 배포이름(필수)

release:
  name: paasta-apiplatform  #서비스 릴리즈 이름(필수)
  version: "2.0"   #서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전

stemcells:
- alias: default
  os: ((stemcell_os))
  version: "((stemcell_version))"

update:
  canaries: 1                          # canary 인스턴스 수(필수)
  canary_watch_time: 30000-600000      # canary 인스턴스가 수행하기 위한 대기 시간(필수)
  max_in_flight: 1                     # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)
  update_watch_time: 30000-600000      # 컴파일 시 필요한 가상머신의 속성(필수)

instance_groups:
- name: mariadb                               # 작업 이름(필수): DB 서버
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.210
  properties:                                 # job에 대한 속성을 지정(필수)
    admin_username: root                      # DB 어드민 유저이름 
    admin_password: openpaas                  # DB 어드민 패스워드
    apiplatform_username: wso2                # API Platform(API Manager)의 DB 접속 유저이름
    apiplatform_password: openpaas            # API Platform(API Manager)의 DB 접속 패스워드
    apiplatform_broker_username: apiplatform  # API Platform Service Broker의 DB 접속 유저이름
    apiplatform_broker_password: openpaas     # API Platform Service Broker의 DB 접속 패스워드
  templates:
  - name: mariadb                # job template 이름(필수)
    release: paasta-apiplatform

- name: bam                            # 작업 이름(필수): BAM
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  vm_type: ((vm_type_medium))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.203
  templates:
  - name: bam                # job template 이름(필수)
    release: paasta-apiplatform

- name: apimanager                    # 작업 이름(필수): API Platform(API Manager)
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  vm_type: ((vm_type_medium))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.201
  templates:
  - name: apimanager                # job template 이름(필수)
    release: paasta-apiplatform

- name: apiplatform-broker            # 작업 이름(필수): API Platform Service Broker
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.200
  properties:                                  # job에 대한 속성을 지정(필수)
    catalog_login_id: admin                    # 카탈로그 API를 사용하기 위한 API플랫폼 유저 아이디
    catalog_login_password: admin              # 카탈로그 API를 사용하기 위한 API플랫폼 유저 패스워드
    apimanager_url: http://10.30.107.201        # API Platform(API Manager) URL
    database_ip: 10.30.107.210                  # DB 접속 URL
    apiplatform_broker_username: apiplatform   # API Platform Service Broker의 DB 접속 유저이름
    apiplatform_broker_password: openpaas      # API Platform Service Broker의 DB 접속 패스워드
  templates:
  - name: apiplatform-broker                # job template 이름(필수)
    release: paasta-apiplatform

- name: broker-registrar                        # 작업 이름(필수): 서비스 브로커 등록
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  lifecycle: errand                             # bosh deploy시 vm에 생성되어 설치 되지 않고 bosh errand 로 실행할 때 설정, 주로 테스트 용도에 쓰임
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
  properties:                                   # job에 대한 속성을 지정(필수)
    cf:                                         # PaaS-TA 접속 정보
      api_url: http://api.115.68.46.189.xip.io   # PaaS-TA 접속 URL
      admin_username: "admin"                   # PaaS-TA 접속 유저이름
      admin_password: "admin_test"                   # PaaS-TA 접속 패스워드
      skip_ssl_validation: "true"               # PaaS-TA 접속 옵션의 일부
    broker:                                     # API Platform Service Broker 설정정보
      protocol: http                            # API Platform Service Broker 접속 프로토콜
      host: 10.30.107.200                        # API Platform Service Broker 접속 URL
      port: 8080                                # API Platform Service Broker 접속 포트
      name: apiplatform-serivce-broker          # API Platform Service Broker 생성명
      username: "admin"                         # API Platform Service Broker auth 유저이름
      password: "cloudfoundry"                  # API Platform Service Broker auth 패스워드
  templates:
  - name: broker-registrar                # job template 이름(필수)
    release: paasta-apiplatform

- name: broker-deregistrar                        # 작업 이름(필수): 서비스 브로커 등록
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  lifecycle: errand                             # bosh deploy시 vm에 생성되어 설치 되지 않고 bosh errand 로 실행할 때 설정, 주로 테스트 용도에 쓰임
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
  properties:                                   # job에 대한 속성을 지정(필수)
    cf:                                         # PaaS-TA 접속 정보
      api_url: http://api.115.68.46.189.xip.io   # PaaS-TA 접속 URL
      admin_username: "admin"                   # PaaS-TA 접속 유저이름
      admin_password: "admin_test"                   # PaaS-TA 접속 패스워드
      skip_ssl_validation: "true"               # PaaS-TA 접속 옵션의 일부
    broker:                                     # API Platform Service Broker 설정정보
      name: apiplatform-service-broker         # API Platform Service Broker 생성명
  templates:
  - name: broker-deregistrar                # job template 이름(필수)
    release: paasta-apiplatform
```

-	deploy-apiplatform-bosh2.0.sh 파일을 서버 환경에 맞게 수정한다.

```sh
#!/bin/bash
# stemcell 버전은 3309 버전으로 사용하시고 https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Download_Page.md 에서 다운받아 쓰십시요.

bosh -e micro-bosh -d paasta-apiplatform-service deploy paasta_apiplatform_bosh2.0.yml \
   -v default_network_name=service_private \
   -v stemcell_os=ubuntu-trusty \
   -v stemcell_version=3309 \
   -v vm_type_medium=service_medium \
   -v vm_type_small=minimal
```

-	API 플랫폼 서비스팩을 배포한다.

- **사용 예시**

		$ ./deploy-apiplatform-bosh2.0.sh
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Using deployment 'paasta-apiplatform-service'

		+ azs:
		+ - cloud_properties:
		+     datacenters:
		+     - clusters:
		+       - BD-HA:
		+           resource_pool: CF_BOSH2_Pool
		+       name: BD-HA
		+   name: z1
		+ - cloud_properties:
		+     datacenters:
		+     - clusters:
		+       - BD-HA:
		+           resource_pool: CF_BOSH2_Pool
		+       name: BD-HA
		+   name: z2
		+ - cloud_properties:
		+     datacenters:
		+     - clusters:
		+       - BD-HA:
		+           resource_pool: CF_BOSH2_Pool
		+       name: BD-HA
		+   name: z3
		+ - cloud_properties:
		+     datacenters:
		+     - clusters:
		+       - BD-HA:
		+           resource_pool: CF_BOSH2_Pool
		+       name: BD-HA
		+   name: z4
		+ - cloud_properties:
		+     datacenters:
		+     - clusters:
		+       - BD-HA:
		+           resource_pool: CF_BOSH2_Pool
		+       name: BD-HA
		+   name: z5
		+ - cloud_properties:
		+     datacenters:
		+     - clusters:
		+       - BD-HA:
		+           resource_pool: CF_BOSH2_Pool
		+       name: BD-HA
		+   name: z6

		+ vm_types:
		+ - cloud_properties:
		+     cpu: 1
		+     disk: 8192
		+     ram: 1024
		+   name: minimal
		+ - cloud_properties:
		+     cpu: 1
		+     disk: 10240
		+     ram: 2048
		+   name: default
		+ - cloud_properties:
		+     cpu: 1
		+     disk: 30720
		+     ram: 4096
		+   name: small
		+ - cloud_properties:
		+     cpu: 2
		+     disk: 20480
		+     ram: 4096
		+   name: medium
		+ - cloud_properties:
		+     cpu: 2
		+     disk: 20480
		+     ram: 8192
		+   name: medium-memory-8GB
		+ - cloud_properties:
		+     cpu: 4
		+     disk: 20480
		+     ram: 8192
		+   name: large
		+ - cloud_properties:
		+     cpu: 8
		+     disk: 20480
		+     ram: 16384
		+   name: xlarge
		+ - cloud_properties:
		+     cpu: 2
		+     disk: 51200
		+     ram: 4096
		+   name: small-50GB
		+ - cloud_properties:
		+     cpu: 2
		+     disk: 51200
		+     ram: 4096
		+   name: small-50GB-ephemeral-disk
		+ - cloud_properties:
		+     cpu: 4
		+     disk: 102400
		+     ram: 8192
		+   name: small-100GB-ephemeral-disk
		+ - cloud_properties:
		+     cpu: 4
		+     disk: 102400
		+     ram: 8192
		+   name: small-highmem-100GB-ephemeral-disk
		+ - cloud_properties:
		+     cpu: 8
		+     disk: 20480
		+     ram: 16384
		+   name: small-highmem-16GB
		+ - cloud_properties:
		+     cpu: 1
		+     disk: 4096
		+     ram: 2048
		+   name: caas_small
		+ - cloud_properties:
		+     cpu: 1
		+     disk: 4096
		+     ram: 1024
		+   name: caas_small_api
		+ - cloud_properties:
		+     cpu: 1
		+     disk: 4096
		+     ram: 4096
		+   name: caas_medium
		+ - cloud_properties:
		+     cpu: 2
		+     disk: 8192
		+     ram: 4096
		+   name: service_medium
		+ - cloud_properties:
		+     cpu: 2
		+     disk: 10240
		+     ram: 2048
		+   name: service_medium_2G
		+ - cloud_properties:
		+     cpu: 1
		+     disk: 4096
		+     ram: 512
		+   name: portal_small
		+ - cloud_properties:
		+     cpu: 1
		+     disk: 4096
		+     ram: 1024
		+   name: portal_medium
		+ - cloud_properties:
		+     cpu: 1
		+     disk: 4096
		+     ram: 2048
		+   name: portal_large

		+ vm_extensions:
		+ - cloud_properties:
		+     ports:
		+     - host: 3306
		+   name: mysql-proxy-lb
		+ - name: cf-router-network-properties
		+ - name: cf-tcp-router-network-properties
		+ - name: diego-ssh-proxy-network-properties
		+ - name: cf-haproxy-network-properties
		+ - cloud_properties:
		+     disk: 51200
		+   name: small-50GB
		+ - cloud_properties:
		+     disk: 102400
		+   name: small-highmem-100GB

		+ compilation:
		+   az: z1
		+   network: default
		+   reuse_compilation_vms: true
		+   vm_type: large
		+   workers: 5

		+ networks:
		+ - name: default
		+   subnets:
		+   - azs:
		+     - z1
		+     - z2
		+     - z3
		+     - z4
		+     - z5
		+     - z6
		+     cloud_properties:
		+       name: Internal
		+     dns:
		+     - 8.8.8.8
		+     gateway: 10.30.20.23
		+     range: 10.30.0.0/16
		+     reserved:
		+     - 10.30.0.0 - 10.30.111.40
		+ - name: public
		+   subnets:
		+   - azs:
		+     - z1
		+     - z2
		+     - z3
		+     - z4
		+     - z5
		+     - z6
		+     cloud_properties:
		+       name: External
		+     dns:
		+     - 8.8.8.8
		+     gateway: 115.68.46.177
		+     range: 115.68.46.176/28
		+     reserved:
		+     - 115.68.46.176 - 115.68.46.188
		+     static:
		+     - 115.68.46.189 - 115.68.46.190
		+   type: manual
		+ - name: service_private
		+   subnets:
		+   - azs:
		+     - z1
		+     - z2
		+     - z3
		+     - z4
		+     - z5
		+     - z6
		+     cloud_properties:
		+       name: Internal
		+     dns:
		+     - 8.8.8.8
		+     gateway: 10.30.20.23
		+     range: 10.30.0.0/16
		+     reserved:
		+     - 10.30.0.0 - 10.30.106.255
		+     static:
		+     - 10.30.107.1 - 10.30.107.255
		+ - name: service_public
		+   subnets:
		+   - azs:
		+     - z1
		+     - z2
		+     - z3
		+     - z4
		+     - z5
		+     - z6
		+     cloud_properties:
		+       name: External
		+     dns:
		+     - 8.8.8.8
		+     gateway: 115.68.47.161
		+     range: 115.68.47.160/24
		+     reserved:
		+     - 115.68.47.161 - 115.68.47.174
		+     static:
		+     - 115.68.47.175 - 115.68.47.185
		+   type: manual
		+ - name: portal_service_public
		+   subnets:
		+   - azs:
		+     - z1
		+     - z2
		+     - z3
		+     - z4
		+     - z5
		+     - z6
		+     cloud_properties:
		+       name: External
		+     dns:
		+     - 8.8.8.8
		+     gateway: 115.68.46.209
		+     range: 115.68.46.208/28
		+     reserved:
		+     - 115.68.46.216 - 115.68.46.222
		+     static:
		+     - 115.68.46.214
		+   type: manual

		+ disk_types:
		+ - disk_size: 1024
		+   name: default
		+ - disk_size: 1024
		+   name: 1GB
		+ - disk_size: 2048
		+   name: 2GB
		+ - disk_size: 4096
		+   name: 4GB
		+ - disk_size: 5120
		+   name: 5GB
		+ - disk_size: 8192
		+   name: 8GB
		+ - disk_size: 10240
		+   name: 10GB
		+ - disk_size: 20480
		+   name: 20GB
		+ - disk_size: 30720
		+   name: 30GB
		+ - disk_size: 51200
		+   name: 50GB
		+ - disk_size: 102400
		+   name: 100GB
		+ - disk_size: 1048576
		+   name: 1TB

		+ stemcells:
		+ - alias: default
		+   os: ubuntu-trusty
		+   version: '3309'

		+ update:
		+   canaries: 1
		+   canary_watch_time: 30000-600000
		+   max_in_flight: 1
		+   update_watch_time: 30000-600000

		+ release:
		+   name: paasta-apiplatform
		+   version: '2.0'

		+ instance_groups:
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: mariadb
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.210
		+   properties:
		+     admin_password: "<redacted>"
		+     admin_username: "<redacted>"
		+     apiplatform_broker_password: "<redacted>"
		+     apiplatform_broker_username: "<redacted>"
		+     apiplatform_password: "<redacted>"
		+     apiplatform_username: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: mariadb
		+     release: paasta-apiplatform
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: bam
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.203
		+   stemcell: default
		+   templates:
		+   - name: bam
		+     release: paasta-apiplatform
		+   vm_type: service_medium
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: apimanager
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.201
		+   stemcell: default
		+   templates:
		+   - name: apimanager
		+     release: paasta-apiplatform
		+   vm_type: service_medium
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: apiplatform-broker
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.200
		+   properties:
		+     apimanager_url: "<redacted>"
		+     apiplatform_broker_password: "<redacted>"
		+     apiplatform_broker_username: "<redacted>"
		+     catalog_login_id: "<redacted>"
		+     catalog_login_password: "<redacted>"
		+     database_ip: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: apiplatform-broker
		+     release: paasta-apiplatform
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   lifecycle: errand
		+   name: broker-registrar
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
		+       skip_ssl_validation: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: broker-registrar
		+     release: paasta-apiplatform
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   lifecycle: errand
		+   name: broker-deregistrar
		+   networks:
		+   - name: service_private
		+   properties:
		+     broker:
		+       name: "<redacted>"
		+     cf:
		+       admin_password: "<redacted>"
		+       admin_username: "<redacted>"
		+       api_url: "<redacted>"
		+       skip_ssl_validation: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: broker-deregistrar
		+     release: paasta-apiplatform
		+   vm_type: minimal

		+ name: paasta-apiplatform-service

		Continue? [yN]: y

		Task 7609

		Task 7609 | 07:55:31 | Preparing deployment: Preparing deployment (00:00:02)
		Task 7609 | 07:55:35 | Preparing package compilation: Finding packages to compile (00:00:00)
		Task 7609 | 07:55:35 | Compiling packages: cf-cli/99238e9bbf54ff4ebc80aef311127c2568e5497d
		Task 7609 | 07:55:35 | Compiling packages: jre7/856cd96de84744fa28c222e7a0d8e4a357203e74
		Task 7609 | 07:55:35 | Compiling packages: bam/c72a2d571666da622c7a61b638289dc5477d4d25
		Task 7609 | 07:55:35 | Compiling packages: mysql_connector_java/b900fe25d061ec118fbd0cead39366de046d9a68
		Task 7609 | 07:55:35 | Compiling packages: java7/5e1e7d676990295c095b632bddc5c42c0da1fccd
		Task 7609 | 07:58:07 | Compiling packages: mysql_connector_java/b900fe25d061ec118fbd0cead39366de046d9a68 (00:02:32)
		Task 7609 | 07:58:07 | Compiling packages: mariadb/76d00089f1c7ee1122f6b584d26d21a14254e1f0
		Task 7609 | 07:58:08 | Compiling packages: cf-cli/99238e9bbf54ff4ebc80aef311127c2568e5497d (00:02:33)
		Task 7609 | 07:58:08 | Compiling packages: apimanager/f6fdd14e4f28d302661aae793f82c6949515036d
		Task 7609 | 07:58:47 | Compiling packages: jre7/856cd96de84744fa28c222e7a0d8e4a357203e74 (00:03:12)
		Task 7609 | 07:58:47 | Compiling packages: apiplatform-broker/b9356e568d495e53294ba465c28b2cac8b923e1f
		Task 7609 | 07:59:15 | Compiling packages: java7/5e1e7d676990295c095b632bddc5c42c0da1fccd (00:03:40)
		Task 7609 | 07:59:16 | Compiling packages: apiplatform-broker/b9356e568d495e53294ba465c28b2cac8b923e1f (00:00:29)
		Task 7609 | 07:59:22 | Compiling packages: bam/c72a2d571666da622c7a61b638289dc5477d4d25 (00:03:47)
		Task 7609 | 07:59:55 | Compiling packages: apimanager/f6fdd14e4f28d302661aae793f82c6949515036d (00:01:47)
		Task 7609 | 08:40:27 | Compiling packages: mariadb/76d00089f1c7ee1122f6b584d26d21a14254e1f0 (00:42:20)
		Task 7609 | 08:41:27 | Creating missing vms: mariadb/95ea8fa0-81ff-4900-b331-9c8ba0b19e30 (0)
		Task 7609 | 08:41:27 | Creating missing vms: bam/9fcf3305-3536-48d1-a212-64da93694fcc (0)
		Task 7609 | 08:41:27 | Creating missing vms: apimanager/fad601f4-01fe-4626-b3d0-07340b35cd24 (0)
		Task 7609 | 08:41:27 | Creating missing vms: apiplatform-broker/d00e29cb-c5dd-4d7d-bb4d-179a5901fcc6 (0) (00:01:41)
		Task 7609 | 08:43:08 | Creating missing vms: bam/9fcf3305-3536-48d1-a212-64da93694fcc (0) (00:01:41)
		Task 7609 | 08:43:08 | Creating missing vms: mariadb/95ea8fa0-81ff-4900-b331-9c8ba0b19e30 (0) (00:01:41)
		Task 7609 | 08:43:09 | Creating missing vms: apimanager/fad601f4-01fe-4626-b3d0-07340b35cd24 (0) (00:01:42)
		Task 7609 | 08:43:11 | Updating instance mariadb: mariadb/95ea8fa0-81ff-4900-b331-9c8ba0b19e30 (0) (canary) (00:01:11)
		Task 7609 | 08:44:23 | Updating instance bam: bam/9fcf3305-3536-48d1-a212-64da93694fcc (0) (canary) (00:01:25)
		Task 7609 | 08:45:48 | Updating instance apimanager: apimanager/fad601f4-01fe-4626-b3d0-07340b35cd24 (0) (canary) (00:01:32)
		Task 7609 | 08:47:20 | Updating instance apiplatform-broker: apiplatform-broker/d00e29cb-c5dd-4d7d-bb4d-179a5901fcc6 (0) (canary) (00:00:59)

		Task 7609 Started  Wed Sep 12 07:55:31 UTC 2018
		Task 7609 Finished Wed Sep 12 08:48:19 UTC 2018
		Task 7609 Duration 00:52:48
		Task 7609 done

		Succeeded


-	배포된 API 플랫폼 서비스팩을 확인한다.

- **사용 예시**

		$bosh -e micro-bosh -d paasta-apiplatform-service vms
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Task 7696. Done

		Deployment 'paasta-apiplatform-service'

		Instance                                                 Process State  AZ  IPs            VM CID                                   VM Type         Active  
		apimanager/fad601f4-01fe-4626-b3d0-07340b35cd24          running        z5  10.30.107.201  vm-b698177a-0448-48b8-a90e-ebf56ae2bc9a  service_medium  true  
		apiplatform-broker/d00e29cb-c5dd-4d7d-bb4d-179a5901fcc6  running        z5  10.30.107.200  vm-9677ce51-3807-43ce-bf0d-814362fc48f2  minimal         true  
		bam/9fcf3305-3536-48d1-a212-64da93694fcc                 running        z5  10.30.107.203  vm-233fa349-a3cc-44dc-8dfb-f86d2555a158  service_medium  true  
		mariadb/95ea8fa0-81ff-4900-b331-9c8ba0b19e30             running        z5  10.30.107.210  vm-14687d6d-edc9-4c15-8410-a09efce70c71  minimal         true  

		4 vms

		Succeeded


### <div id='24'> 2.4. API 매니저에서 API 생성 및 배포
API 플랫폼 서비스 팩에는 API 매니저(API 플랫폼) 서비스 브로커가 포함되어 있다. API 매니저에 등록된 API 서비스를 PaaS-TA에서 사용하기 위해서는 PaaS-TA에서 이 서비스 브로커를 등록하여야 한다. 이때, API 매니저에 API 서비스가 존재하지 않으면, PaaS-TA에서 API 매니저 서비스 브로커를 등록할 수 없기 때문에 서비스 브로커를 등록하기 전에 API 매니저에서 API 서비스를 등록한다. 
※ 본 문서에서 등록하는 API 서비스는 WSO2 API Manager의 공식 문서에서 가이드 하는 샘플 API이다.


##### 2.4.1. 터널링 설정
API 플랫폼 서비스팩으로 배포한 API 매니저에는 Public IP가 할당되지 않기 때문에 사용자가 웹 브라우저를 통해 API 매니저에 접속할 수 있도록 터널링 설정이 필요하다. 터널링은 다양한 방법을 사용할 수 있지만, 본 문서에서는 SSH 클라이언트인 Putty와 웹 브라우저 Firefox를 이용한 터널링 방법에 대해서 안내한다.

>※ Putty 다운로드: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html

>※ Firefox 다운로드: https://www.mozilla.org/ko/firefox/new/

※ 설치는 별도로 안내하지 않는다.


###### 1. Putty 설정
(1)다운로드한 Putty를 실행하고 Connection 메뉴를 열어 SSH 메뉴에서 Tunnels를 메뉴를 연다.

![apiplatform_image_02]


(2) 터널링 정보를 입력한다.<br>
① Source port에 사용하지 않는 임의의 포트를 입력하고 ②번 Dynamic을 선택한 후, ③ Add 버튼을 눌러 추가한다. ④번 위치에서 D{입력한 포트}의 형태로 추가된 것을 확인한다.

![apiplatform_image_03]


(3) Putty 접속 정보를 입력한다.<br>
① Session 메뉴를 클릭하여 접속정보 설정 화면으로 이동한다. ② 배포한 API 매니저와 내부망으로 연결되어 있는 머신(설치 환경에 따라 상이함)의 Public IP를 입력한다.

![apiplatform_image_04]


(4) Putty 접속 및 로그인<br>
Open 버튼을 클릭하여 해당 머신에 연결한다

![apiplatform_image_05]


로그인 화면에서 로그인을 완료한 채로 접속을 유지해둔다.

![apiplatform_image_06]


###### 2. Firefox 설정
(1) Mozila Firefox 브라우저를 실행하여 첫 화면 하단의 옵션 버튼을 클릭한다.

![apiplatform_image_07]


(2) 설정 창을 연다.<br>
① 고급 탭-  ② 네트워크 탭- ③ 설정 버튼을 차례로 클릭하여 설정 창을 연다.

![apiplatform_image_08]


(3) 연결 설정 정보를 입력하고 저장한다.<br>
① 프록시 수동설정에 체크하고 ② 화면과 같이 설정을 입력한다. 포트는 Putty 터널링 설정에서 입력한 Source Port와 동일한 포트를 입력한다. ③ 설정을 완료하였으면 확인 버튼을 눌러 저장한다.

![apiplatform_image_09]


###### 3. 터널링 설정 확인
터널링 설정이 되어있는 Putty 접속을 유지한 채로, Mozila Firefox 브라우저를 이용하여 API 매니저 관리자 화면에 접속해본다. 하단의 화면이 확인된다면 API 매니저 배포 및 터널링 설정이 정상적으로 이루어진 것이다.

<div id=APICreatePublish></div>

##### 2.4.2. API 생성 및 배포
터널링 설정이 완료 되었다면, API 플랫폼 서비스팩을 통해 배포된 API 매니저에 접속하여 API를 생성하고 배포(Publish)한다.

```
{API매니저 URL}:9443/carbon
예) https://10.0.0.201:9443/carbon
```

>![apiplatform_image_10]

###### 1. API 매니저 접속 및 로그인
① API 플랫폼 서비스팩을 통해 배포된 API 매니저의 publisher 대시보드에 접속한다.

```
{API매니저 URL}:{API매니저 포트}/publisher
예) https://10.30.60.201:9443/publisher
```
※ http가 아닌 https임에 주의한다.


② 관리자 계정으로 로그인한다. 관리자 계정의 Username과 Password는 admin/admin이다.
※ API 매니저 관리자 대시보드({API매니저 URL}:{API매니저 포트}/carbon)에서 계정을 추가하고 권한을 설정하여 사용할 수도 있지만, 그에 대한 설명은 본 문서에서는 기술하지 않는다.

![apiplatform_image_11]


###### 2. API 생성
① 로그인이 완료되면 다음과 같은 화면을 확인할 수 있다. 최초 배포가 완료되면 API가 생성되지 않은 상태이므로 ②번의 New API 버튼이 화면에 보여진다. New API 버튼을 클릭하여 API 생성화면으로 이동한다.

![apiplatform_image_12]


<div id=DefineGeneralDetails></div>

###### 3. General Details 정의
① API 생성 화면으로 이동하였다.<br>
② API 생성의 첫 단계인 Design 단계이다.<br>
③ General Details에 하단의 화면과 동일하게 다음과 같이 값을 입력한다.<br>

```
Name: Phoneverification
Context: /phoneverify
Version: 1.0.0
```

※ ④번 Edit Swagger Definition 버튼을 클릭하여 다음의 [[4. Resources 정의]](#DefineResources) 과정을 생략 할 수 있다. 이에 대한 설명은 [[4. Resources 정의]](#DefineResources) 하단에 [[Swagger 정의]](#DefineSwagger)로 첨부한다.

![apiplatform_image_13]

<div id=DefineResources></div>

###### 4. Resources 정의
① General Details 하단에 Resources 입력란이 있다. URL Pattern에 대소문자 구분에 유의하여 CheckPhoneNumber 값을 입력하고 GET, POST, OPTIONS 메소드를 선택한다. Resource Name의 값은 URL Pattern을 입력하면 같은 값이 자동으로 입력되는데 사용자 필요에 따라 변경할 수 있다.<br>
② 입력이 완료되었다면, Add New Resource 버튼을 클릭하여 Resource를 추가한다.<br>

![apiplatform_image_14]


리소스를 추가하면 하단의 그림처럼 추가된 Resource가 화면에 나타난다. 그 중 GET 메소드의 리소스를 클릭하여 파라미터 세부사항을 정의한다.

![apiplatform_image_15]

리소스를 클릭하면 파라미터 세부사항 입력란이 노출된다.<br>
① 추가하고자 하는 파라미터 명을 입력한다.<br>
② Add Parameter 버튼을 클릭하여 파라미터를 추가 한다. Prameter Name 항목에서 입력한 파라미터명을 확인할 수 있다.<br>
③ 파라미터의 세부사항을 정의하기 위해서 각각의 파라미터의 값(값이 정의 되어있지 않다면 [+Empty]로 표시됨)을 클릭하여 다음과 같이 파라미터를 정의한다.<br>

| Parameter Name | Description | Parameter Type | Data Type | Required |
|--------|-------|-------|-------|-------|
| PhoneNumber | Give the phone number to be validated | query | string | True |
| LicenseKey | Give the license key. If you don't have any, enter 0 | query | string | True |

④ 하단의 버튼 중, 좌측의 Save버튼을 클릭하여 저장하고 가운데 Implement 버튼을 클릭하여 다음단계인 Implement 단계로 진행한다.<br>

![apiplatform_image_16]


<div id=DefineSwagger></div>

※ Swagger 정의<br>
상단의 [[3. General Details 정의]](#DefineGeneralDetails)에서 푸른색 ④번 박스로 표시된 Edit Swagger Definition 버튼을 클릭하면 Swagger 수정이 가능하다. 다음과 같이 수정하고 Save버튼을 클릭하여 Resources를 정의한다. 이 방법을 통해 상단에 기술된 [[4. Resources 정의]](#DefineResources) 절차를 생략할 수 있다.


```yml
apiVersion: 1.0.0
swaggerVersion: "1.2"
apis:
  - path: /checkphonenumber
    description: ""
    file:
      apiVersion: 1.0.0
      swaggerVersion: "1.2"
      basePath: "http://localhost:8280/phoneverify/1.0.0"
      resourcePath: /checkphonenumber
      apis:
        - path: /CheckPhoneNumber
          operations:
            - method: GET
              parameters:
                - description: Give the phone number to be validated
                  name: PhoneNumber
                  type: string
                  required: "True"
                  paramType: query
                - description: "Give the license key. If you don't have any, enter 0"
                  name: LicenseKey
                  type: string
                  required: "True"
                  paramType: query
              nickname: get_CheckPhoneNumber
            - method: POST
              parameters:
                - name: body
                  description: Request Body
                  allowMultiple: false
                  required: true
                  paramType: body
                  type: string
              nickname: post_CheckPhoneNumber
            - method: OPTIONS
              parameters:
                - name: body
                  description: Request Body
                  allowMultiple: false
                  required: true
                  paramType: body
                  type: string
              nickname: options_CheckPhoneNumber
info:
  title: PhoneVerification
  description: ""
  termsOfServiceUrl: ""
  contact: ""
  license: ""
  licenseUrl: ""
authorizations:
  oauth2:
    type: oauth2
    scopes: []
```


###### 5. Endpoint 입력
① Endpoint를 입력하고 Endpoint 설정과 보안 설정 등을 입력하는 Implement 화면이다.<br>
② Production Endpoint 입력란과 Sandbox Endpoint 입력란에 다음과 같이 Endpoint를 입력한다.<br>
>Production Endpoint: http://ws.cdyne.com/phoneverify/phoneverify.asmx<br>
>Sandbox Endpoint: http://ws.cdyne.com/phoneverify/phoneverify.asmx

③ 추가된 사항을 저장한다.<br>
④ 다음 단계인 Manage 화면으로 이동한다.<br>

![apiplatform_image_17]


<div id=ChooseTier></div>

###### 6. Tier 선택
① API의 Tier 및 추가적인 설정을 입력하는 Manage화면이다.<br>
② Tier Availability는 해당 API 서비스의 호출 횟수를 제한하는 설정으로 PaaS-TA에서는 플랜으로 표시된다. API 매니저에 설정된 4가지 기본 Tier 중, 사용자 필요에 맞게 선택할 수 있고 API 매니저 관리자 대시보드({API매니저 URL}:{API매니저 포트}/carbon)에서 Tier명과 호출 횟수 제한을 변경할 수 있다. 다만, 현재는 API 서비스 브로커 설계상 API Tier는 Unlimited Tier를 선택하여야 한다. 따라서 Unlimited Tier만을 선택한다. Unlimited Tier만 선택하여 API 서비스를 배포하더라도 PaaS-TA에서는 Unlimited와 Bronze의 두 가지 플랜을 선택할 수 있다.<br>
③ Save & Publish 버튼을 클릭하여 저장하고 API를 배포한다.<br>

![apiplatform_image_18]


###### 7. 배포 확인
① API 매니저의 Store 대시보드에 접속한다.

```
{API매니저 URL}:{API매니저 포트}/store
예) https://10.30.60.201:9443/store
```
※ http가 아닌 https임에 주의한다.

② 생성한 API가 배포되어 있음을 확인한다. 배포되지 않은 API는 Store 대시보드에 노출되지 않는다.

![apiplatform_image_19]

### <div id='25'> 2.5. 플랫폼 서비스 브로커 등록
API 매니저(API 플랫폼)에 API 서비스가 정상적으로 등록 및 배포가 완료되었다면, 등록된 API 서비스를 PaaS-TA의 서비스 형태로 제공하기 위해 API 플랫폼 서비스 브로커를 등록해 주어야 한다. 서비스 브로커 등록 시, PaaS-TA에서 서비스 브로커를 등록할 수 있는 사용자로 로그인이 되어 있어야 한다.

- 서비스 브로커 목록을 확인한다.

```
$ cf service-brokers
```
![apiplatform_image_20]

- API 플랫폼 서비스 브로커를 등록한다.

```
$ cf create-service-broker {서비스 브로커 이름} {서비스 브로커 사용자ID} {서비스 브로커 사용자 비밀번호} {서비스 브로커 URL}
서비스 브로커 이름 : 서비스 팩 관리를 위해 PaaS-TA에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스팩 리스트의 명칭이다.
서비스 브로커 사용자ID / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID입니다. 서비스 팩도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.
서비스브로커 URL : 서비스 팩이 제공하는 API를 사용할 수 있는 URL을 입력한다.
```

```
$ cf create-service-broker apiplatform-service-broker admin cloudfoundry http://10.30.107.200:8080
```
![apiplatform_image_21]

- 등록된 API 플랫폼 서비스 브로커를 확인한다.

```
$ cf service-brokers
```
![apiplatform_image_22]

- 접근 가능한 서비스 목록을 확인한다.

```
$ cf service-access
```
![apiplatform_image_23]
서비스 브로커 생성시 기본적으로 접근을 허용하지 않는다. access가 none인 것을 확인할 수 있다.

※ [[2.4 API 매니저에서 API 생성 및 배포]](#APICreatePublish) 장의 [[6. Tier 선택]](#ChooseTier)을 보면 tier는 Unlimited 한 가지만 선택하도록 되어있다. 그럼에도 서비스 목록에 Unlimited와 Bronze 두 개의 플랜으로 서비스가 생기는 것은 서비스 브로커가 두 개의 플랜으로 서비스를 생성하도록 설계되어 있기 때문이다. 두개의 플랜 중 선택에 따라 API 호출 횟수가 제한된다.

<br>
- 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)

```
$ cf enable-service-access Phoneverification
```
```
$ cf service-access
```

![apiplatform_image_24]

<br>
- 서비스가 Marketplace에 등록된 것을 확인한다.

```
$ cf marketplace
```
![apiplatform_image_25]


# <div id='3'> 3. API 플랫폼 연동
[[2.4 API 매니저에서 API 생성 및 배포]](#APICreatePublish)에서 생성한 API 서비스를 샘플 App에 바인드하여, Vcap 환경설정 정보를 정상적으로 획득할 수 있는지를 확인함으로써 연동여부를 확인한다. 서비스 바인드만 진행하기 때문에 샘플 App은 어떤 App을 사용해도 무방하다.


### <div id='31'> 3.1. Sample Web App에 서비스 바인드 신청 및 App 확인
샘플 App을 배포하고 API 서비스와 바인드 신청을 한다. 먼저 샘플 App을 배포한다.
※ 참고: 서비스 Bind 신청시 PaaS-TA에서 서비스 Bind를 할 수 있는 사용자로 로그인이 되어 있어야 한다.

- Sample Web App 디렉토리로 이동하여 manifest 파일을 확인한다.

```
$ cd {샘플앱 경로}
```
```
$ vi manifest.yml
```

```yml
# manifest.yml
---
applications:
- name: hello-servlet-api         # 배포할 App 이름
  memory: 512M                    # 배포시 메모리 사이즈
  instances: 1                    # 배포 인스턴스 수
path: target/hello-java-1.0.war   # 배포하는 App의 war 파일 경로
```


- '--no-start' 옵션으로 App을 배포한다. '--no-start'는 App 배포시 구동을 하지 않도록 하는 옵션이다.

```
$ cf push --no-start
```
![apiplatform_image_26]


- 배포된 Sample App을 확인하고 로그를 수행한다.

```
$ cf apps
```
![apiplatform_image_27]

```
$ cf logs hello-servlet-api
```
![apiplatform_image_28]


- 바인드할 서비스의 서비스 인스턴스를 생성한다.

```
$ cf create-service {서비스명} {서비스플랜} {내서비스명}

서비스명 : p-mysql로 Marketplace에서 보여지는 서비스 명칭이다.
서비스플랜 : 서비스에 대한 정책으로 plans에 있는 정보 중 하나를 선택한다. MySQL 서비스는 100mb, 1gb를 지원한다.
내 서비스명 : 내 서비스에서 보여지는 명칭이다. 이 명칭을 기준으로 환경설정 정보를 가져온다.
```

```
$ cf create-service PhoneVerification Unlimited phoneverification-instance-unlimited
```
![apiplatform_image_29]

<br>
- 생성된 API 플랫폼 서비스 인스턴스를 확인한다.

```
$ cf services
```
![apiplatform_image_30]


- 샘플앱과 생성한 서비스 인스턴스를 바인드 한다.
```
$ cf bind-service hello-servlet-api phoneverification-instance-unlimited
```
![apiplatform_image_31]


### <div id='32'> 3.2. 서비스 바인드 확인
샘플 앱에 대한 VCAP_SERVICES 정보를 확인하여 샘플 앱과 서비스의 정상적인 바인드 여부를 판단한다. VCAP_SERVICES 정보를 확인하는 명령어는 다음과 같다. VCAP_SERVICES에서 앱에서 API를 사용하는데 필요한 정보를 확인할 수 있고 따라서 앱은 이 값을 읽을 수 있도록 작성되어야 한다.

```
$ cf env hello-servlet-api
```

[apiplatform_image_01]:/Service-Guide/images/apiplatform/apiplatform_image_01.png
[apiplatform_image_02]:/Service-Guide/images/apiplatform/apiplatform_image_02.png 
[apiplatform_image_03]:/Service-Guide/images/apiplatform/apiplatform_image_03.png
[apiplatform_image_04]:/Service-Guide/images/apiplatform/apiplatform_image_04.png
[apiplatform_image_05]:/Service-Guide/images/apiplatform/apiplatform_image_05.png
[apiplatform_image_06]:/Service-Guide/images/apiplatform/apiplatform_image_06.png
[apiplatform_image_07]:/Service-Guide/images/apiplatform/apiplatform_image_07.jpeg
[apiplatform_image_08]:/Service-Guide/images/apiplatform/apiplatform_image_08.png
[apiplatform_image_09]:/Service-Guide/images/apiplatform/apiplatform_image_09.png
[apiplatform_image_10]:/Service-Guide/images/apiplatform/apiplatform_image_10.png
[apiplatform_image_11]:/Service-Guide/images/apiplatform/apiplatform_image_11.png
[apiplatform_image_12]:/Service-Guide/images/apiplatform/apiplatform_image_12.png
[apiplatform_image_13]:/Service-Guide/images/apiplatform/apiplatform_image_13.png
[apiplatform_image_14]:/Service-Guide/images/apiplatform/apiplatform_image_14.png
[apiplatform_image_15]:/Service-Guide/images/apiplatform/apiplatform_image_15.png
[apiplatform_image_16]:/Service-Guide/images/apiplatform/apiplatform_image_16.png
[apiplatform_image_17]:/Service-Guide/images/apiplatform/apiplatform_image_17.png
[apiplatform_image_18]:/Service-Guide/images/apiplatform/apiplatform_image_18.png
[apiplatform_image_19]:/Service-Guide/images/apiplatform/apiplatform_image_19.png
[apiplatform_image_20]:/Service-Guide/images/apiplatform/apiplatform_image_20.png
[apiplatform_image_21]:/Service-Guide/images/apiplatform/apiplatform_image_21.png
[apiplatform_image_22]:/Service-Guide/images/apiplatform/apiplatform_image_22.png
[apiplatform_image_23]:/Service-Guide/images/apiplatform/apiplatform_image_23.png
[apiplatform_image_24]:/Service-Guide/images/apiplatform/apiplatform_image_24.png
[apiplatform_image_25]:/Service-Guide/images/apiplatform/apiplatform_image_25.png
[apiplatform_image_26]:/Service-Guide/images/apiplatform/apiplatform_image_26.png
[apiplatform_image_27]:/Service-Guide/images/apiplatform/apiplatform_image_27.png
[apiplatform_image_28]:/Service-Guide/images/apiplatform/apiplatform_image_28.png
[apiplatform_image_29]:/Service-Guide/images/apiplatform/apiplatform_image_29.png
[apiplatform_image_30]:/Service-Guide/images/apiplatform/apiplatform_image_30.png
[apiplatform_image_31]:/Service-Guide/images/apiplatform/apiplatform_image_31.png
