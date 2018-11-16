## Table of Contents
1. [문서 개요](#1)
  - 1.1. [목적](#11)
  - 1.2. [범위](#12)
  - 1.3. [시스템 구성도](#13)
  - 1.4. [참고자료](#14)
2. [RabbitMQ 서비스팩 설치](#2)
  - 2.1. [설치전 준비사항](#21)
  - 2.2. [RabbitMQ 서비스 릴리즈 업로드](#22)
  - 2.3. [2.3. RabbitMQ 서비스 Deployment 파일 수정 및 배포](#23)
  - 2.4. [RabbitMQ 서비스 브로커 등록](#24)
3. [RabbitMQ 연동 Sample App 설명](#3)
  - 3.1. [Sample App 구조](#31)
  - 3.2. [PaaS-TA에서 서비스 신청](#32)
  - 3.3. [Sample App에 서비스 바인드 신청 및 App 확인](#33)
     
# <div id='1'> 1. 문서 개요
### <div id='11'> 1.1. 목적
본 문서(RabbitMQ서비스팩 설치 가이드)는 전자정부표준프레임워크 기반의 PaaS-TA에서 제공되는 서비스팩인 RabbitMQ 서비스팩을 Bosh를 이용하여 설치 하는 방법과 PaaS-TA의 SaaS 형태로 제공하는 Application에서 RabbitMQ 서비스를 사용하는 방법을 기술하였다.
PaaS-TA 3.5 버전부터는 Bosh2.0 기반으로 deploy를 진행하며 기존 Bosh1.0 기반으로 설치를 원할경우에는 PaaS-TA 3.1 이하 버전의 문서를 참고한다.

### <div id='12'> 1.2. 범위 
설치 범위는 RabbitMQ 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다. 


### <div id='13'> 1.3. 시스템 구성도
본 문서의 설치된 시스템 구성도이다. RabbitMQ(1대), RabbitMQ 서비스 브로커, haproxy로 최소사항을 구성하였다.

![시스템 구성도][rabbitmq_image_01]

<table>
  <tr>
    <td>구분</td>
    <td>스펙</td>
  </tr>
  <tr>
    <td>paasta-rmq-broker</td>
    <td>1vCPU / 1GB RAM / 8GB Disk</td>
  </tr>
  <tr>
    <td>haproxy</td>
    <td>1vCPU / 1GB RAM / 8GB Disk</td>
  </tr>
  <tr>
    <td>rmq</td>
    <td>1vCPU / 1GB RAM / 8GB Disk</td>
  </tr>
</table>


### <div id='14'> 1.4. 참고자료
[**http://bosh.io/docs**](http://bosh.io/docs)  
[**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/)


# <div id='2'> 2. RabbitMQ 서비스팩 설치

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


### <div id='22'> 2.2. RabbitMQ 서비스 릴리즈 업로드

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

-	RabbitMQ 서비스 릴리즈가 업로드 되어 있지 않은 것을 확인

-	RabbitMQ 서비스 릴리즈 파일을 업로드한다.

- **사용 예시**

		$ bosh -e micro-bosh upload-release paasta-rabbitmq-2.0.tgz
    		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		######################################################## 100.00% 152.97 MiB/s 1s
		Task 8066

		Task 8066 | 06:14:13 | Extracting release: Extracting release (00:00:03)
		Task 8066 | 06:14:16 | Verifying manifest: Verifying manifest (00:00:00)
		Task 8066 | 06:14:16 | Resolving package dependencies: Resolving package dependencies (00:00:00)
		Task 8066 | 06:14:16 | Creating new packages: cf-cli/68cbd378bc91c97fd0847075d71528c3eb1f7928 (00:00:00)
		Task 8066 | 06:14:16 | Creating new packages: cf-rabbitmq-smoke-tests/b674b591adfffa72e6ddbe9eaa202f3dfae45b80 (00:00:01)
		Task 8066 | 06:14:17 | Creating new packages: erlang/d689b1d899513421b211ded55d614407a1b51ad6 (00:00:01)
		Task 8066 | 06:14:18 | Creating new packages: golang-1.6.1/7c83e83f822259c6324742e3dfc5d4aaae25e9e6 (00:00:02)
		Task 8066 | 06:14:20 | Creating new packages: haproxy/f0f2335dc5e7cab5c8a7232fd4aa9d92cbc8dba8 (00:00:00)
		Task 8066 | 06:14:20 | Creating new packages: java/4cc1119c49707a07d69b3863914b6f1307e611b5 (00:00:01)
		Task 8066 | 06:14:21 | Creating new packages: rabbitmq-broker/66260363d9b0ee0c671adc4a9b5e38453652fc06 (00:00:02)
		Task 8066 | 06:14:23 | Creating new packages: rabbitmq-cluster-migration-tool/aa2db9b54d0d87e67b4b7ef7ae1854ebe6cef867 (00:00:00)
		Task 8066 | 06:14:23 | Creating new packages: rabbitmq-common/a1ceb4166602035b5e8783f2d90a2048a654875f (00:00:00)
		Task 8066 | 06:14:23 | Creating new packages: rabbitmq-server/c93764197128c6e731899062f1cc33aef9cef053 (00:00:00)
		Task 8066 | 06:14:23 | Creating new packages: rabbitmq-syslog-aggregator/a1662acacf8b2ef2ff54707681b125370544c824 (00:00:00)
		Task 8066 | 06:14:23 | Creating new packages: rabbitmq-upgrade-preparation/3ef7571fb3896b0c2260fd8063e47dfd4802a81c (00:00:00)
		Task 8066 | 06:14:23 | Creating new jobs: broker-deregistrar/57c1e5bf6e59618fa4c5d13efd7f8420cabeba2a (00:00:00)
		Task 8066 | 06:14:23 | Creating new jobs: broker-registrar/e354968e46093de4e61d825a875bb33bffef79e3 (00:00:01)
		Task 8066 | 06:14:24 | Creating new jobs: rabbitmq-broker/4d344fde68bcf2712b60c3369137a970d2c6116c (00:00:00)
		Task 8066 | 06:14:24 | Creating new jobs: rabbitmq-haproxy/a7c6a8a87a15f3fb40382a282d6075a4baf0f24f (00:00:00)
		Task 8066 | 06:14:24 | Creating new jobs: rabbitmq-server/b7871ae1cce45589a135f6bab459d2a5c4dd3bc9 (00:00:00)
		Task 8066 | 06:14:24 | Creating new jobs: smoke-tests/a9230370cc5954574231962a0f7e2233daabf1e4 (00:00:00)
		Task 8066 | 06:14:24 | Release has been created: paasta-rabbitmq/2.0 (00:00:00)

		Task 8066 Started  Thu Sep 13 06:14:13 UTC 2018
		Task 8066 Finished Thu Sep 13 06:14:24 UTC 2018
		Task 8066 Duration 00:00:11
		Task 8066 done

		Succeeded


-	업로드 된 RabbitMQ 릴리즈를 확인한다.

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
		paasta-rabbitmq                   2.0       21516d49+  
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
		
-	RabbitMQ 서비스 릴리즈가 업로드 되어 있는 것을 확인

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
		
>Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드를 해야 한다. (RabbitMQ는 stemcell 3309 버전을 사용)
	

### <div id='23'> 2.3. RabbitMQ 서비스 Deployment 파일 수정 및 배포
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
# paasta-rabbitmq-service 설정 파일 내용
---
name: paasta-rabbitmq-service                          # 서비스 배포이름(필수)

releases:
- name: paasta-rabbitmq                                # 서비스 릴리즈 이름(필수)
  version: "2.0"                                         # 서비스 릴리즈 버전(필수): latest 시 업로드된 서비스 릴리즈 최신버전

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
- name: rmq                              # 작업 이름(필수): rabbitmq 서버
  azs:
  - z5
  instances: 1                           # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))             # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))       # cloud config 에 정의한 network 이름
    static_ips:
    - 10.30.107.193                      # 사용할 IP addresses 정의(필수): rabbitmq-server IP
  templates:
  - name: rabbitmq-server                # job template 이름(필수)
    release: paasta-rabbitmq             # 릴리즈 이름(필수)
  syslog_aggregator: null

- name: haproxy                          # 작업 이름(필수): rabbitmq haproxy
  azs:
  - z5
  instances: 1                           # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))             # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))       # cloud config 에 정의한 network 이름
    static_ips:
    - 10.30.107.192                      # 사용할 IP addresses 정의(필수): rabbitmq haproxy IP
  templates:
  - name: rabbitmq-haproxy                # job template 이름(필수)
    release: paasta-rabbitmq             # 릴리즈 이름(필수)
  syslog_aggregator: null

- name: paasta-rmq-broker                # 작업 이름(필수): rabbit mq broker
  azs:
  - z5
  instances: 1                           # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))             # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))       # cloud config 에 정의한 network 이름
    static_ips:
    - 10.30.107.191                      # 사용할 IP addresses 정의(필수): rabbitmq broker
  templates:
  - name: rabbitmq-broker                # job template 이름(필수)
    release: paasta-rabbitmq             # 릴리즈 이름(필수)
  syslog_aggregator: null

- name: broker-registrar
  lifecycle: errand                      # bosh deploy시 vm에 생성되어 설치 되지 않고 bosh errand 로 실할때 설정, 주로 테스트 용도에 쓰임
  azs:
  - z5
  instances: 1                           # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))             # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))       # cloud config 에 정의한 network 이름
  properties:
    broker:
      host: 10.30.107.191
      name: rabbitmq-sb
      password: admin
      username: admin
      protocol: http
      port: 4567
  templates:
  - name: broker-registrar
    release: paasta-rabbitmq

- name: broker-deregistrar
  lifecycle: errand                      # bosh deploy시 vm에 생성되어 설치 되지 않고 bosh errand 로실행할때 설정, 주로 테스트 용도에 쓰임
  azs:
  - z5
  instances: 1                           # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))             # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))       # cloud config 에 정의한 network 이름
  properties:
    broker:
      host: 10.30.107.191
      name: rabbitmq-sb
      password: admin
      username: admin
      protocol: http
      port: 4567
  templates:
  - name: broker-deregistrar
    release: paasta-rabbitmq

properties:
  cf:                                          # CF 환경 정보
    admin_password: admin                      # CF 어드민 아이디 비밀번호(필수)
    admin_username: admin_test                      # CF 어드민 아이디 (필수)
    api_url: http://api.115.68.46.189.xip.io    # CF API url(필수)
    domain: 115.68.46.189.xip.io                # CF 도메인(필수)
  rabbitmq-server:
    restart_statsdb_cron_schedule: "42 */4 * * *"
    plugins:                                  # rabbitmq 플러그인 정보(필수)
    - rabbitmq_management
    - rabbitmq_mqtt
    - rabbitmq_stomp
    ports:                                    # rabbitmq haproxy 에서 허용하는 포트 목록(필수)
    - 5672
    - 5671
    - 1883
    - 8883
    - 61613
    - 61614
    - 15672
    - 15674
    administrators:
      management:                            # rabbitmq management 사용자 및 패스워드 정보
        username: admin
        password: admin
      broker:                                # rabbitmq service broker 아이디 및 패스워드 정보
        username: admin          
        password: admin
  rabbitmq-haproxy:                          # rabbitmq service broker 아이디 및 패스워드 정보
    stats:
      username: admin
      password: admin
  broker:                                    # rabbitmq service broker 아이디 및 패스워드 정보
    host: 10.30.107.181 
    protocol: http
    name: p-rabbitmq
    username: "admin"
    password: "admin"
  rabbitmq-broker:
    route: paasta-rabbitmq-broker
    cc_endpoint: http://api.115.68.46.189.xip.io
    service:                                 # 서비스 이름및 uuid 플랜 Id 정보
      username: "admin"
      password: "admin"
      name: p-rabbitmq
      uuid: 163b47c6-a2f3-43b1-97f7-b83b37ecabcd
      plan_uuid: 4e816145-4e71-4e24-a402-0c686b868e2d
    logging:
      level: debug
      print_stack_traces: false
    rabbitmq:
      operator_set_policy:
        enabled: true
        policy_name: "operator_set_policy"
        policy_definition: "{\"ha-mode\":\"exactly\",\"ha-params\":2,\"ha-sync-mode\":\"automatic\"}"
        policy_priority: 50
      management_domain: 115.68.46.189.xip.io
      ssl: |
        -----BEGIN CERTIFICATE-----
        MIIC+zCCAeOgAwIBAgIBAjANBgkqhkiG9w0BAQUFADAnMRUwEwYDVQQDEwxNeVRl
        c3RSb290Q0ExDjAMBgNVBAcTBTY0MTAzMB4XDTE0MDkwNDA3MjIwOFoXDTI0MDkw
        MTA3MjIwOFowKjEXMBUGA1UEAxMObWVyY3VyaW8ubG9jYWwxDzANBgNVBAoTBnNl
        cnZlcjCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANxxSzf958VIm8lp
        qQ4BHSmz1z8yU/KEKSbuEfIqpGwpVx6TZ+ZYiXa0cMV2pE7UKR4OyJiuvtvv9kzu
        6g+HTXmJo2cqVonGCAMp6d9TkCAMaMF76IrbLyGmvXQDcjOmWarvsGHW/w6gJpw9
        svDP9EXyXTBUfaJq3T8+9UQBfMsL4dHwAt79YgvSQLsYiIi2rzQixK/4PHFeHf3G
        I/UDgjG0YG9iCWp2g1Sc3Z6hYB/0pOCBxE7LCrSGS6/M/7c2569yK7NqSSNN7Lqz
        ZoQSF1NYE6KRd2MK2A0QaKrn9v8K5/Lp0fk70bvwtLxTWtp3wq3bYQg8UdqY/6R8
        UATS/aMCAwEAAaMvMC0wCQYDVR0TBAIwADALBgNVHQ8EBAMCBSAwEwYDVR0lBAww
        CgYIKwYBBQUHAwEwDQYJKoZIhvcNAQEFBQADggEBAHC89mK1HJgDqwxjsGpa3V7t
        Nuqe/XxEIUUN3Lm4gBLKq4wed4c6z4csv16f3uL9cypyHPSrQmMPV7CDgWLX4F7g
        YN9PGaVfIp/rGNsDWJEVNU2rfIEDIUfcL+o844jE8CtmzZ4bGVrCHqKW5pAraai1
        o5h3JaU4yDLo49rqPeRft2n/gj+5E3gi/1TsnuLuzB7kK1gaTTOrV3GASiGokCEN
        4v1ZjaqMSGMcwA/esaLv2N6UYJgd5lyJ7PEL4ddE8QCTo2EPhYyltLxRqOjrxa+5
        KONA94PDj14gOSSsoXkoj7gWQsuHT2RXmurYXk4/PkS+k1j0+ZCzKi/ZxF5jt50=
        -----END CERTIFICATE-----
      administrator:
        username: admin
        password: admin
```

-	deploy-rabbitmq-bosh2.0.sh 파일을 서버 환경에 맞게 수정한다.

```sh
#!/bin/bash
# stemcell 버전은 3309 버전으로 사용하시고 https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Download_Page.md 에서 다운받아 쓰십시요.

bosh -e micro-bosh -d paasta-rabbitmq-service deploy paasta_rabbitmq_bosh2.0.yml \
   -v default_network_name=service_private \
   -v stemcell_os=ubuntu-trusty \
   -v stemcell_version=3309 \
   -v vm_type_small=minimal
```

-	RabbitMQ 서비스팩을 배포한다.

- **사용 예시**

		$ ./deploy-rabbitmq-bosh2.0.sh
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Using deployment 'paasta-rabbitmq-service'

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

		+ releases:
		+ - name: paasta-rabbitmq
		+   version: '2.0'

		+ update:
		+   canaries: 1
		+   canary_watch_time: 30000-180000
		+   max_in_flight: 6
		+   update_watch_time: 30000-180000

		+ instance_groups:
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: rmq
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.193
		+   stemcell: default
		+   syslog_aggregator: 
		+   templates:
		+   - name: rabbitmq-server
		+     release: paasta-rabbitmq
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: haproxy
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.192
		+   stemcell: default
		+   syslog_aggregator: 
		+   templates:
		+   - name: rabbitmq-haproxy
		+     release: paasta-rabbitmq
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: paasta-rmq-broker
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.191
		+   stemcell: default
		+   syslog_aggregator: 
		+   templates:
		+   - name: rabbitmq-broker
		+     release: paasta-rabbitmq
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
		+   stemcell: default
		+   templates:
		+   - name: broker-registrar
		+     release: paasta-rabbitmq
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
		+       host: "<redacted>"
		+       name: "<redacted>"
		+       password: "<redacted>"
		+       port: "<redacted>"
		+       protocol: "<redacted>"
		+       username: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: broker-deregistrar
		+     release: paasta-rabbitmq
		+   vm_type: minimal

		+ name: paasta-rabbitmq-service

		+ properties:
		+   broker:
		+     host: "<redacted>"
		+     name: "<redacted>"
		+     password: "<redacted>"
		+     protocol: "<redacted>"
		+     username: "<redacted>"
		+   cf:
		+     admin_password: "<redacted>"
		+     admin_username: "<redacted>"
		+     api_url: "<redacted>"
		+     domain: "<redacted>"
		+   rabbitmq-broker:
		+     cc_endpoint: "<redacted>"
		+     logging:
		+       level: "<redacted>"
		+       print_stack_traces: "<redacted>"
		+     rabbitmq:
		+       administrator:
		+         password: "<redacted>"
		+         username: "<redacted>"
		+       management_domain: "<redacted>"
		+       operator_set_policy:
		+         enabled: "<redacted>"
		+         policy_definition: "<redacted>"
		+         policy_name: "<redacted>"
		+         policy_priority: "<redacted>"
		+       ssl: "<redacted>"
		+     route: "<redacted>"
		+     service:
		+       name: "<redacted>"
		+       password: "<redacted>"
		+       plan_uuid: "<redacted>"
		+       username: "<redacted>"
		+       uuid: "<redacted>"
		+   rabbitmq-haproxy:
		+     stats:
		+       password: "<redacted>"
		+       username: "<redacted>"
		+   rabbitmq-server:
		+     administrators:
		+       broker:
		+         password: "<redacted>"
		+         username: "<redacted>"
		+       management:
		+         password: "<redacted>"
		+         username: "<redacted>"
		+     plugins:
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+     ports:
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+     restart_statsdb_cron_schedule: "<redacted>"

		Continue? [yN]: y

		Task 8070

		Task 8070 | 06:16:40 | Preparing deployment: Preparing deployment (00:00:02)
		Task 8070 | 06:16:43 | Preparing package compilation: Finding packages to compile (00:00:00)
		Task 8070 | 06:16:43 | Compiling packages: cf-cli/68cbd378bc91c97fd0847075d71528c3eb1f7928
		Task 8070 | 06:16:43 | Compiling packages: java/4cc1119c49707a07d69b3863914b6f1307e611b5
		Task 8070 | 06:16:43 | Compiling packages: rabbitmq-common/a1ceb4166602035b5e8783f2d90a2048a654875f
		Task 8070 | 06:16:43 | Compiling packages: rabbitmq-broker/66260363d9b0ee0c671adc4a9b5e38453652fc06
		Task 8070 | 06:16:43 | Compiling packages: haproxy/f0f2335dc5e7cab5c8a7232fd4aa9d92cbc8dba8
		Task 8070 | 06:19:23 | Compiling packages: cf-cli/68cbd378bc91c97fd0847075d71528c3eb1f7928 (00:02:40)
		Task 8070 | 06:19:23 | Compiling packages: rabbitmq-syslog-aggregator/a1662acacf8b2ef2ff54707681b125370544c824
		Task 8070 | 06:19:25 | Compiling packages: rabbitmq-common/a1ceb4166602035b5e8783f2d90a2048a654875f (00:02:42)
		Task 8070 | 06:19:25 | Compiling packages: golang-1.6.1/7c83e83f822259c6324742e3dfc5d4aaae25e9e6
		Task 8070 | 06:19:34 | Compiling packages: java/4cc1119c49707a07d69b3863914b6f1307e611b5 (00:02:51)
		Task 8070 | 06:19:34 | Compiling packages: rabbitmq-server/c93764197128c6e731899062f1cc33aef9cef053
		Task 8070 | 06:19:35 | Compiling packages: rabbitmq-broker/66260363d9b0ee0c671adc4a9b5e38453652fc06 (00:02:52)
		Task 8070 | 06:19:35 | Compiling packages: erlang/d689b1d899513421b211ded55d614407a1b51ad6
		Task 8070 | 06:19:51 | Compiling packages: rabbitmq-syslog-aggregator/a1662acacf8b2ef2ff54707681b125370544c824 (00:00:28)
		Task 8070 | 06:20:04 | Compiling packages: rabbitmq-server/c93764197128c6e731899062f1cc33aef9cef053 (00:00:30)
		Task 8070 | 06:20:20 | Compiling packages: haproxy/f0f2335dc5e7cab5c8a7232fd4aa9d92cbc8dba8 (00:03:37)
		Task 8070 | 06:20:28 | Compiling packages: golang-1.6.1/7c83e83f822259c6324742e3dfc5d4aaae25e9e6 (00:01:03)
		Task 8070 | 06:20:28 | Compiling packages: rabbitmq-cluster-migration-tool/aa2db9b54d0d87e67b4b7ef7ae1854ebe6cef867
		Task 8070 | 06:20:28 | Compiling packages: rabbitmq-upgrade-preparation/3ef7571fb3896b0c2260fd8063e47dfd4802a81c (00:00:27)
		Task 8070 | 06:20:57 | Compiling packages: rabbitmq-cluster-migration-tool/aa2db9b54d0d87e67b4b7ef7ae1854ebe6cef867 (00:00:29)
		Task 8070 | 06:29:01 | Compiling packages: erlang/d689b1d899513421b211ded55d614407a1b51ad6 (00:09:26)
		Task 8070 | 06:30:08 | Creating missing vms: rmq/a4ef4c7e-4776-411d-8317-b2b059e416dd (0)
		Task 8070 | 06:30:08 | Creating missing vms: haproxy/a30fb543-000d-4f74-b62d-7418da0e6101 (0)
		Task 8070 | 06:30:08 | Creating missing vms: paasta-rmq-broker/52629ddb-32c9-4097-b9f6-e5dc0aff55ce (0) (00:01:28)
		Task 8070 | 06:31:38 | Creating missing vms: rmq/a4ef4c7e-4776-411d-8317-b2b059e416dd (0) (00:01:30)
		Task 8070 | 06:31:39 | Creating missing vms: haproxy/a30fb543-000d-4f74-b62d-7418da0e6101 (0) (00:01:31)
		Task 8070 | 06:31:40 | Updating instance rmq: rmq/a4ef4c7e-4776-411d-8317-b2b059e416dd (0) (canary) (00:01:00)
		Task 8070 | 06:32:40 | Updating instance haproxy: haproxy/a30fb543-000d-4f74-b62d-7418da0e6101 (0) (canary) (00:00:55)
		Task 8070 | 06:33:35 | Updating instance paasta-rmq-broker: paasta-rmq-broker/52629ddb-32c9-4097-b9f6-e5dc0aff55ce (0) (canary) (00:01:07)

		Task 8070 Started  Thu Sep 13 06:16:40 UTC 2018
		Task 8070 Finished Thu Sep 13 06:34:42 UTC 2018
		Task 8070 Duration 00:18:02
		Task 8070 done

		Succeeded


-	배포된 RabbitMQ 서비스팩을 확인한다.

- **사용 예시**

		$bosh -e micro-bosh -d paasta-rabbitmq-service vms
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Task 8077. Done

		Deployment 'paasta-rabbitmq-service'

		Instance                                                Process State  AZ  IPs            VM CID                                   VM Type  Active  
		haproxy/a30fb543-000d-4f74-b62d-7418da0e6101            running        z5  10.30.107.192  vm-fbd4a04a-5346-4e00-b793-17c327f90aa7  minimal  true  
		paasta-rmq-broker/52629ddb-32c9-4097-b9f6-e5dc0aff55ce  running        z5  10.30.107.191  vm-5238f05b-ec4f-449c-ab1d-a1a5b932d76e  minimal  true  
		rmq/a4ef4c7e-4776-411d-8317-b2b059e416dd                running        z5  10.30.107.193  vm-f8d8a62d-bfc4-442e-8306-9f133ebfc518  minimal  true  

		3 vms

		Succeeded

### <div id='24'> 2.4. RabbitMQ 서비스 브로커 등록
RabbitMQ 서비스팩 배포가 완료 되었으면 Application에서 서비스 팩을 사용하기 위해서 먼저 RabbitMQ 서비스 브로커를 등록해 주어야 한다.
서비스 브로커 등록시에는 PaaS-TA에서 서비스 브로커를 등록할 수 있는 사용자로 로그인 하여야 한다

##### 서비스 브로커 목록을 확인한다.

>`$ cf service-brokers`

>![rabbitmq_image_02]

<br>

##### rabbitmq 서비스 브로커를 등록한다.

>`$ cf create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{서비스팩 URL(IP)}`
  
  **서비스팩 이름** : 서비스 팩 관리를 위해 PaaS-TA에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스팩 리스트의 명칭이다.<br>
  **서비스팩 사용자ID** / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID입니다. 서비스팩도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.<br>
  **서비스팩 URL** : 서비스팩이 제공하는 API를 사용할 수 있는 URL을 입력한다.

>`$ cf create-service-broker rabbitmq-service-broker admin admin http://10.30.107.191:4567}`

> ![rabbitmq_image_03]

<br>

##### 등록된 RabbitMQ 서비스 브로커를 확인한다.
>`$ cf service-brokers`

>![rabbitmq_image_04]

<br>

#### 접근 가능한 서비스 목록을 확인한다.
>`$ cf service-access`

>![rabbitmq_image_05]

<br>

- 서비스 브로커 등록시 최초에는 접근을 허용하지 않는다. 따라서 access는 none으로 설정된다.

#### 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)
>`$ cf enable-service-access p-rabbitmq`<br>
>`$ cf service-access`

>![rabbitmq_image_06]

<br>

<div id='11'></div>

# <div id='3'> 3. RabbitMQ 연동 Sample App 설명
본 Sample App은 PaaS-TA에 배포되며 RabbitMQ의 서비스를 Provision과 Bind를 한 상태에서 사용이 가능하다.


<div id='12'></div>

### <div id='31'> 3.1. Sample App 구조
Sample App은 PaaS-TA에 App으로 배포가 된다. 배포 완료 후 정상적으로 App이 구동되면 curl 명령어로 App url를 입력하면 RabbitMQ 환경정보(서비스 연결 정보)를 보여주는 url 페이지를 제공한다.

Sample App 구조는 다음과 같다.
<table>
  <tr>
    <td>이름</td>
    <td>설명</td>
  </tr>
  <tr>
    <td>lib</td>
    <td>Sample 소스디렉토리</td>
  </tr>
  <tr>
    <td>manifest.yml</td>
    <td>PaaS-TA에 app 배포시 필요한 설정을 저장하는 파일</td>
  </tr>
  <tr>
    <td>Gemfile</td>
    <td>Sample App 구동시 필요한 ruby gem 설정 파일</td>
  </tr>
  <tr>
    <td>config.ru</td>
    <td>Sample App 구동 파일</td>
  </tr>
</table>


#### PaaSTA-Sample-Apps.zip 파일 압축을 풀고 Service 폴더안에 있는 RabbitMQ Sample Web App인 rabbit-labrat을 복사한다.

>`$ ls -all`

>![rabbitmq_image_07]

<br>

<div id='13'></div>

### <div id='32'> 3.2. PaaS-TA에서 서비스 신청
Sample App에서 RabbitMQ 서비스를 사용하기 위해서는 서비스 신청(Provision)을 해야 한다.
*참고: 서비스 신청시 PaaS-TA에서 서비스를 신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.

#### 먼저 PaaS-TA Marketplace에서 서비스가 있는지 확인을 한다.

>`$ cf marketplace`

>![rabbitmq_image_08]

<br>

#### Marketplace에서 원하는 서비스가 있으면 서비스 신청(Provision)을 한다.

>`$ cf create-service {서비스명} {서비스 플랜} {내 서비스명}`
- **서비스명** : p-rabbitmq로 Marketplace에서 보여지는 서비스 명칭이다.
- **서비스플랜** : 서비스에 대한 정책으로 plans에 있는 정보 중 하나를 선택한다. RabbitMQ 서비스는 standard plan만 지원한다.
- **내 서비스명** : 내 서비스에서 보여지는 명칭이다. 이 명칭을 기준으로 환경 설정 정보를 가져온다.


>`$ cf create-service p-rabbitmq standard rabbitmq-service-instance`

>![rabbitmq_image_09]

<br>

#### 생성된 rabbitmq 서비스 인스턴스를 확인한다.

>`$ cf services`

>![rabbitmq_image_10]

<br>

<div id='14'></div>

### <div id='33'> 3.3. Sample App에 서비스 바인드 신청 및 App 확인
서비스 신청이 완료되었으면 Sample App 에서는 생성된 서비스 인스턴스를 Bind 하여 App에서 RabbitMQ 서비스를 이용한다.
* 참고: 서비스 Bind 신청시 PaaS-TA에서 서비스 Bind 신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.

#### Sample App 디렉토리로 이동하여 manifest 파일을 확인한다.

>`$ cd rabbit-labrat`<br>

>`$ vi manifest.yml`

**applications:**
- name: lab-rat         # 배포할 App 이름 <br>
- command: puma           # 배포시 명령어

<br>

#### --no-start 옵션으로 App을 배포한다. 
--no-start: App 배포시 구동은 하지 않는다.

>`$cf push --no-start`<br>

>![rabbitmq_image_11]

#### 배포된 Sample App을 확인하고 로그를 수행한다.

>`$cf apps`<br>

>![rabbitmq_image_12]

<br>

>`$ cf logs lab-rat}`  **// cf logs {배포된 App명}}**

>![rabbitmq_image_13]

<br>

#### Sample App에서 생성한 서비스 인스턴스 바인드 신청을 한다. 

>`cf bind-service lab-rat rabbitmq-service-instance`<br>

>![rabbitmq_image_14]

<br>

#### 바인드가 적용되기 위해서 App을 재기동한다.

>`cf restart lab-rat`

>![rabbitmq_image_15]

>![rabbitmq_image_16]

<br>

####  App이 정상적으로 RabbitMQ 서비스를 사용하는지 확인한다.


>`curl lab-rat.115.68.46.186.xip.io`  **//- curl 로 확인**
>![rabbitmq_image_17]

>![rabbitmq_image_18]

>![rabbitmq_image_19]


[rabbitmq_image_01]:/Service-Guide/images/rabbitmq/rabbitmq_image_01.png
[rabbitmq_image_02]:/Service-Guide/images/rabbitmq/rabbitmq_image_02.png
[rabbitmq_image_03]:/Service-Guide/images/rabbitmq/rabbitmq_image_03.png
[rabbitmq_image_04]:/Service-Guide/images/rabbitmq/rabbitmq_image_04.png
[rabbitmq_image_05]:/Service-Guide/images/rabbitmq/rabbitmq_image_05.png
[rabbitmq_image_06]:/Service-Guide/images/rabbitmq/rabbitmq_image_06.png
[rabbitmq_image_07]:/Service-Guide/images/rabbitmq/rabbitmq_image_07.png
[rabbitmq_image_08]:/Service-Guide/images/rabbitmq/rabbitmq_image_08.png
[rabbitmq_image_09]:/Service-Guide/images/rabbitmq/rabbitmq_image_09.png
[rabbitmq_image_10]:/Service-Guide/images/rabbitmq/rabbitmq_image_10.png
[rabbitmq_image_11]:/Service-Guide/images/rabbitmq/rabbitmq_image_11.png
[rabbitmq_image_12]:/Service-Guide/images/rabbitmq/rabbitmq_image_12.png
[rabbitmq_image_13]:/Service-Guide/images/rabbitmq/rabbitmq_image_13.png
[rabbitmq_image_14]:/Service-Guide/images/rabbitmq/rabbitmq_image_14.png
[rabbitmq_image_15]:/Service-Guide/images/rabbitmq/rabbitmq_image_15.png
[rabbitmq_image_16]:/Service-Guide/images/rabbitmq/rabbitmq_image_16.png
[rabbitmq_image_17]:/Service-Guide/images/rabbitmq/rabbitmq_image_17.png
[rabbitmq_image_18]:/Service-Guide/images/rabbitmq/rabbitmq_image_18.png
[rabbitmq_image_19]:/Service-Guide/images/rabbitmq/rabbitmq_image_19.png
