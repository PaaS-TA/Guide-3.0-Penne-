## Table of Contents
1. [문서 개요](#1)
  - [1.1. 목적](#11)
  - [1.2. 범위](#12)
  - [1.3. 시스템 구성도](#13)
  - [1.4. 참고자료](#14)
2. [Redis 서비스팩 설치](#2)
  - [2.1. 설치 전 준비 사항](#21)
  - [2.2. Redis 서비스 릴리즈 업로드](#22)
  - [2.3. Redis 서비스 Deployment 파일 수정 및 배포](#23)
  - [2.4. Redis 서비스 브로커 등록](#24)
3. [Redis 연동 Sample App 설명](#3)
  - [3.1. Sample App 구조](#31)
  - [3.2. PaaS-TA에서 서비스 신청](#32)
  - [3.3. Sample App에 서비스 바인드 신청 및 App 확인](#33)
4. [Redis Client 툴 접속](#4)
  - [4.1. Redis Desktop Manager 설치 및 연결](#41)




# <div id='1'> 1. 문서 개요

### <div id='11'> 1.1. 목적
본 문서(Redis 서비스팩 설치 가이드)는 전자정부표준프레임워크 기반의 PaaS-TA에서 제공되는 서비스팩인 Redis 서비스팩을 Bosh를 이용하여 설치하는 방법과 PaaS-TA의 SaaS 형태로 제공하는 Application에서 Redis 서비스를 사용하는 방법을 기술하였다.

### <div id='12'> 1.2. 범위
설치 범위는 Redis서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다.



### <div id='13'> 1.3. 시스템 구성도
본 문서의 설치된 시스템 구성도입니다. Redis dedicated-node(2대), Redis 서비스 브로커로 최소사항을 구성하였다.
![시스템 구성도][redis_image_01]

<table>
  <tr>
    <td>구분</td>
    <td>스펙</td>
  </tr>
  <tr>
    <td>paasta-redis-broker</td>
    <td>1vCPU / 1GB RAM / 8GB Disk+4GB(영구적 Disk)</td>
  </tr>
  <tr>
    <td>dedicated-node</td>
    <td>1vCPU / 1GB RAM / 8GB Disk+4GB(영구적 Disk)</td>
  </tr>
</table>



### <div id='14'> 1.4. 참고자료
[**http://bosh.io/docs**](http://bosh.io/docs) <br>
[**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/)


# <div id='2'>  2. Redis 서비스팩 설치

### <div id='21'> 2.1. 설치 전 준비 사항
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


###  <div id='22'> 2.2. Redis 서비스 릴리즈 업로드
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

-	Redis 서비스 릴리즈가 업로드 되어 있지 않은 것을 확인

-	Redis 서비스 릴리즈 파일을 업로드한다.

- **사용 예시**

		$ bosh -e micro-bosh upload-release paasta-redis-2.0.tgz
    		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)
		
		######################################################## 100.00% 144.14 MiB/s 2s
		Task 4460

		Task 4460 | 04:31:41 | Extracting release: Extracting release (00:00:04)
		Task 4460 | 04:31:45 | Verifying manifest: Verifying manifest (00:00:00)
		Task 4460 | 04:31:45 | Resolving package dependencies: Resolving package dependencies (00:00:00)
		Task 4460 | 04:31:45 | Creating new packages: gra-log-purger/f02fa5774ab54dbb1b1c3702d03cb929b85d60e6 (00:00:00)
		Task 4460 | 04:31:45 | Creating new packages: cf-mysql-broker/250c6466bdaff96677e501ed5219d92ce4e61bd8 (00:00:00)
		Task 4460 | 04:31:45 | Creating new packages: mysqlclient/ce95f8ac566f76b650992987d5282ee473356e43 (00:00:00)
		Task 4460 | 04:31:45 | Creating new packages: acceptance-tests/1cb3ce7e20f5a8395b43fc6f0e3f2e92b0dc27bd (00:00:00)
		Task 4460 | 04:31:45 | Creating new packages: galera/d15a1d2d15e5e7417278d4aa1b908566022b9623 (00:00:01)
		Task 4460 | 04:31:46 | Creating new packages: galera-healthcheck/3da4dedbcd7d9f404a19e7720e226fd472002266 (00:00:00)
		Task 4460 | 04:31:46 | Creating new packages: quota-enforcer/e2c4c9e7d7bbbe4bfdc0866962461b00e654cca3 (00:00:00)
		Task 4460 | 04:31:46 | Creating new packages: python/4e255efa754d91b825476b57e111345f200944e1 (00:00:01)
		Task 4460 | 04:31:47 | Creating new packages: ruby/ff79c965224b4160c1526bd704b3b21e4ad7c362 (00:00:00)
		Task 4460 | 04:31:47 | Creating new packages: route-registrar/f3fdfb8c940e7227a96c06e413ae6827aba8eeda (00:00:00)
		Task 4460 | 04:31:47 | Creating new packages: check/d6811f25e9d56428a9b942631c27c9b24f5064dc (00:00:01)
		Task 4460 | 04:31:48 | Creating new packages: cli/24305e50a638ece2cace4ef4803746c0c9fe4bb0 (00:00:00)
		Task 4460 | 04:31:48 | Creating new packages: mariadb/43aa3547bc5a01dd51f1501e6b93c215dd7255e9 (00:00:01)
		Task 4460 | 04:31:49 | Creating new packages: openjdk-1.8.0_45/57e0ee876ea9d90f5470e3784ae1171bccee850a (00:00:02)
		Task 4460 | 04:31:51 | Creating new packages: mariadb_ctrl/7658290da98e2cad209456f174d3b9fa143c87fc (00:00:01)
		Task 4460 | 04:31:52 | Creating new packages: scons/11e7ad3b28b43a96de3df7aa41afddde582fcc38 (00:00:00)
		Task 4460 | 04:31:52 | Creating new packages: syslog_aggregator/078da6dcb999c1e6f5398a6eb739182ccb4aba25 (00:00:00)
		Task 4460 | 04:31:52 | Creating new packages: xtrabackup/2e701e7a9e4241b28052d984733de36aae152275 (00:00:01)
		Task 4460 | 04:31:53 | Creating new packages: boost/3eb8bdb1abb7eff5b63c4c5bdb41c0a778925c31 (00:00:01)
		Task 4460 | 04:31:54 | Creating new packages: common/ba480a46c4b2aa9484fb24ed01a8649453573e6f (00:00:00)
		Task 4460 | 04:31:54 | Creating new packages: switchboard/fad565dadbb37470771801952001c7071e55a364 (00:00:01)
		Task 4460 | 04:31:55 | Creating new packages: op-mysql-java-broker/3bf47851b2c0d3bea63a0c58452df58c14a15482 (00:00:01)
		Task 4460 | 04:31:56 | Creating new packages: golang/f57ddbc8d55d7a0f08775bf76bb6a27dc98c7ea7 (00:00:01)
		Task 4460 | 04:31:57 | Creating new jobs: cf-mysql-broker/9828ead15eabdc33b2c27fe275b463735edb115d (00:00:00)
		Task 4460 | 04:31:57 | Creating new jobs: acceptance-tests/48c00c36ec5210cbdd3b125ae6a72cfdf6eaf4e2 (00:00:00)
		Task 4460 | 04:31:57 | Creating new jobs: broker-deregistrar/b5f6f776d46eb1ac561ab1e8f58d8ddedb97f86e (00:00:00)
		Task 4460 | 04:31:57 | Creating new jobs: proxy/7907d8759aa11dfcbbe79220dc945c96b5562ac1 (00:00:00)
		Task 4460 | 04:31:57 | Creating new jobs: mysql/078561f02f2516212ed59c48e1dd45360f93871c (00:00:00)
		Task 4460 | 04:31:57 | Creating new jobs: op-mysql-java-broker/6e47c9ea6fbe0867d4a476af5abf157830c03024 (00:00:00)
		Task 4460 | 04:31:57 | Creating new jobs: broker-registrar/e1f5e30b87e70e916ea74ea8eb63a7b6ff6ff643 (00:00:00)
		Task 4460 | 04:31:57 | Release has been created: paasta-mysql/2.0 (00:00:00)

		Task 4460 Started  Fri Aug 31 04:31:41 UTC 2018
		Task 4460 Finished Fri Aug 31 04:31:57 UTC 2018
		Task 4460 Duration 00:00:16
		Task 4460 done

		Succeeded


-	업로드 된 Redis 릴리즈를 확인한다.

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
		paasta-redis                      2.0       85e3f01e+  
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
		
-	Redis 서비스 릴리즈가 업로드 되어 있는 것을 확인

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
		
>Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드를 해야 한다. (redis는 stemcell 3309 버전을 사용)
		
		
### <div id='23'>  2.3. Redis 서비스 Deployment 파일 수정 및 배포
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
# paasta-redis-service 설정 파일 내용

name: paasta-redis-service                             # 서비스 배포이름(필수)

releases:
- name: paasta-redis                                   #서비스 릴리즈 이름(필수)
  version: "2.0"                                         #서비스 릴리즈 버전(필수): latest 시 업로드된 서비스 릴리즈 최신버전

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
- name: paasta-redis-broker
  azs:
  - z5
  instances: 1
  vm_type: ((vm_type_small))
  stemcell: default
  persistent_disk_type: 4GB
  update:
    max_in_flight: 1
    serial: true
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.171
  properties:
    nats:                                  # paas-ta nats 정보
      machines:
      - 10.30.112.2
      password: "((nats_password))"
      port: 4222
      user: nats
  templates:
  - name: cf-redis-broker
    release: paasta-redis
  - name: syslog-configurator
    release: paasta-redis

- name: dedicated-node
  azs:
  - z5
  instances: 3
  vm_type: ((vm_type_small))
  stemcell: default
  persistent_disk_type: 4GB
  update:
    max_in_flight: 1
    serial: true
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.172
    - 10.30.107.173
    - 10.30.107.174
  templates:
  - name: dedicated-node
    release: paasta-redis
  - name: syslog-configurator
    release: paasta-redis

- name: broker-registrar
  azs:
  - z5
  instances: 1
  lifecycle: errand
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
  properties:
    broker:
      host: 10.30.107.171
      name: paasta-redis-service
      password: admin
      username: admin
  templates:
  - name: broker-registrar
    release: paasta-redis

- name: broker-deregistrar
  azs:
  - z5
  instances: 1
  lifecycle: errand
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
  properties:
    broker:
      host: 10.30.107.171
      name: paasta-redis-service
      password: admin
      username: admin
  templates:
  - name: broker-deregistrar
    release: paasta-redis

- name: smoke-tests
  azs:
  - z5
  instances: 1
  lifecycle: errand
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
  templates:
  - name: smoke-tests
    release: paasta-redis

meta:
  apps_domain: 115.68.46.189.xip.io
  broker:                                                      # broker 정보 : 디폴트 포트 12350
    host: paasta-redis-broker.115.68.46.189.xip.io
    name: redis
    password: admin
    username: admin
  cf:                                                          # paas-ta 정보
    admin_password: admin
    admin_username: admin_test
    api_url: https://api.115.68.46.189.xip.io
    apps_domain: 115.68.46.189.xip.io
    skip_ssl_validation: false
    system_domain: 115.68.46.189.xip.io
  deployment_name: paasta-redis-service
  director_uuid: 484eb672-5b91-4c0d-bb16-bad80f0f3e61          # uuid 정보 bosh status
  external_domain: 115.68.46.189.xip.io
  nats:                                                        # nats 정보
    machines:
    - 10.30.112.2
    password: "((nats_password))"
    port: 4222
    user: nats
  redis:
    bg_save_command: yetanotherrandomstring
    broker:                                                       # broker 정보
      dedicated_vm_plan_id: 48b35349-d3de-4e19-bc4a-66996ae07766
      name: redis
      service_id: 7aba7e52-f61b-4263-9de1-14e9d11fb67d
      shared_vm_plan_id: 78bf886c-bc50-4f31-a03c-cb786a158286
      subdomain: paasta-redis-broker
    config_command: configalias
    dedicated_plan:
      instance_count: 3                                        # 전용노드 수
    save_command: anotherrandomstring
    shared_plan:
      instance_limit: 10                                       # 공유 노드 수
      max_memory: 262144000
  release_name: paasta-redis
  route_name: paasta-redis-broker
  service_name: redis
  syslog_aggregator: null

properties:
  broker:                                                       # broker 정보
    host: paasta-redis-broker.115.68.46.189.xip.io
    name: redis
    password: admin
    username: admin
  cf:                                                           # paas-ta 정보 
    admin_password: admin
    admin_username: admin_test
    api_url: https://api.115.68.46.189.xip.io
    apps_domain: 115.68.46.189.xip.io
    skip_ssl_validation: false
    system_domain: 115.68.46.189.xip.io
  redis:
    agent:
      backend_port: 54321
    bg_save_command: yetanotherrandomstring
    broker:
      auth:
        password: admin
        username: admin
      backend_host: 10.30.107.171
      backend_port: 12345
      dedicated_nodes:
      - 10.30.107.172
      - 10.30.107.173
      - 10.30.107.174
      dedicated_vm_plan_id: 48b35349-d3de-4e19-bc4a-66996ae07766
      enable_service_access: true
      host: 10.30.107.171
      name: redis
      network: default
      route_name: paasta-redis-broker
      service_id: 7aba7e52-f61b-4263-9de1-14e9d11fb67d
      service_instance_limit: 20                                    # 최대 서비스 인스턴스 개수
      service_name: redis
      shared_vm_plan_id: 78bf886c-bc50-4f31-a03c-cb786a158286
      subdomain: redis-broker
    config_command: configalias
    host: 10.30.107.171
    maxmemory: 262144000
    save_command: anotherrandomstring
  syslog_aggregator: null
```

-	deploy-redis-bosh2.0.sh 파일을 서버 환경에 맞게 수정한다.

```sh
#!/bin/bash
# stemcell 버전은 3309 버전으로 사용하시고 https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Download_Page.md 에서 다운받아 쓰십시요.

bosh -e micro-bosh -d paasta-redis-service deploy paasta_redis_bosh2.0.yml \
   -v default_network_name=service_private \
   -v stemcell_os=ubuntu-trusty \
   -v stemcell_version=3309 \
   -v nats_password=fxaqRErYZ1TD8296u9HdMg8ol8dJ0G \
   -v vm_type_small=minimal
```

-	Redis 서비스팩을 배포한다.

- **사용 예시**

		$ ./deploy-redis-bosh2.0.sh
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Using deployment 'paasta-redis-service'

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
		+ - name: paasta-mysql
		+   version: '2.0'

		+ update:
		+   canaries: 1
		+   canary_watch_time: 30000-600000
		+   max_in_flight: 1
		+   update_watch_time: 30000-600000

		+ instance_groups:
		+ - azs:
		+   - z5
		+   instances: 3
		+   name: mysql
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.166
		+     - 10.30.107.165
		+     - 10.30.107.164
		+   persistent_disk_type: 8GB
		+   properties:
		+     admin_password: "<redacted>"
		+     character_set_server: "<redacted>"
		+     cluster_ips:
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+     collation_server: "<redacted>"
		+     network_name: "<redacted>"
		+     seeded_databases: "<redacted>"
		+     syslog_aggregator: "<redacted>"
		+   release: paasta-mysql
		+   stemcell: default
		+   template: mysql
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: proxy
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.168
		+   properties:
		+     cluster_ips:
		+     - "<redacted>"
		+     - "<redacted>"
		+     - "<redacted>"
		+     external_host: "<redacted>"
		+     nats:
		+       machines:
		+       - "<redacted>"
		+       password: "<redacted>"
		+       port: "<redacted>"
		+       user: "<redacted>"
		+     network_name: "<redacted>"
		+     proxy:
		+       api_force_https: "<redacted>"
		+       api_password: "<redacted>"
		+       api_username: "<redacted>"
		+     syslog_aggregator: "<redacted>"
		+   release: paasta-mysql
		+   stemcell: default
		+   template: proxy
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: paasta-mysql-java-broker
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.167
		+   properties:
		+     jdbc_ip: "<redacted>"
		+     jdbc_port: "<redacted>"
		+     jdbc_pwd: "<redacted>"
		+     log_dir: "<redacted>"
		+     log_file: "<redacted>"
		+     log_level: "<redacted>"
		+   release: paasta-mysql
		+   stemcell: default
		+   template: op-mysql-java-broker
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
		+   release: paasta-mysql
		+   stemcell: default
		+   template: broker-registrar
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
		+   release: paasta-mysql
		+   stemcell: default
		+   template: broker-deregistrar
		+   vm_type: minimal

		+ meta:
		+   apps_domain: 115.68.46.189.xip.io
		+   environment: 
		+   external_domain: 115.68.46.189.xip.io
		+   nats:
		+     machines:
		+     - 10.30.112.2
		+     password: fxaqRErYZ1TD8296u9HdMg8ol8dJ0G
		+     port: 4222
		+     user: nats
		+   syslog_aggregator: 

		+ name: paasta-mysql-service

		Continue? [yN]: y

		Task 4506

		Task 4506 | 06:04:10 | Preparing deployment: Preparing deployment (00:00:01)
		Task 4506 | 06:04:12 | Preparing package compilation: Finding packages to compile (00:00:00)
		Task 4506 | 06:04:12 | Compiling packages: cli/24305e50a638ece2cace4ef4803746c0c9fe4bb0
		Task 4506 | 06:04:12 | Compiling packages: openjdk-1.8.0_45/57e0ee876ea9d90f5470e3784ae1171bccee850a
		Task 4506 | 06:04:12 | Compiling packages: op-mysql-java-broker/3bf47851b2c0d3bea63a0c58452df58c14a15482
		Task 4506 | 06:04:12 | Compiling packages: syslog_aggregator/078da6dcb999c1e6f5398a6eb739182ccb4aba25
		Task 4506 | 06:04:12 | Compiling packages: common/ba480a46c4b2aa9484fb24ed01a8649453573e6f
		Task 4506 | 06:06:53 | Compiling packages: syslog_aggregator/078da6dcb999c1e6f5398a6eb739182ccb4aba25 (00:02:41)
		Task 4506 | 06:06:53 | Compiling packages: golang/f57ddbc8d55d7a0f08775bf76bb6a27dc98c7ea7
		Task 4506 | 06:06:55 | Compiling packages: common/ba480a46c4b2aa9484fb24ed01a8649453573e6f (00:02:43)
		Task 4506 | 06:06:55 | Compiling packages: python/4e255efa754d91b825476b57e111345f200944e1
		Task 4506 | 06:06:55 | Compiling packages: cli/24305e50a638ece2cace4ef4803746c0c9fe4bb0 (00:02:43)
		Task 4506 | 06:06:55 | Compiling packages: check/d6811f25e9d56428a9b942631c27c9b24f5064dc
		Task 4506 | 06:07:05 | Compiling packages: op-mysql-java-broker/3bf47851b2c0d3bea63a0c58452df58c14a15482 (00:02:53)
		Task 4506 | 06:07:05 | Compiling packages: boost/3eb8bdb1abb7eff5b63c4c5bdb41c0a778925c31
		Task 4506 | 06:07:10 | Compiling packages: openjdk-1.8.0_45/57e0ee876ea9d90f5470e3784ae1171bccee850a (00:02:58)
		Task 4506 | 06:07:53 | Compiling packages: golang/f57ddbc8d55d7a0f08775bf76bb6a27dc98c7ea7 (00:01:00)
		Task 4506 | 06:07:53 | Compiling packages: switchboard/fad565dadbb37470771801952001c7071e55a364
		Task 4506 | 06:07:53 | Compiling packages: route-registrar/f3fdfb8c940e7227a96c06e413ae6827aba8eeda
		Task 4506 | 06:07:55 | Compiling packages: check/d6811f25e9d56428a9b942631c27c9b24f5064dc (00:01:00)
		Task 4506 | 06:07:55 | Compiling packages: gra-log-purger/f02fa5774ab54dbb1b1c3702d03cb929b85d60e6
		Task 4506 | 06:08:30 | Compiling packages: route-registrar/f3fdfb8c940e7227a96c06e413ae6827aba8eeda (00:00:37)
		Task 4506 | 06:08:30 | Compiling packages: galera-healthcheck/3da4dedbcd7d9f404a19e7720e226fd472002266
		Task 4506 | 06:08:31 | Compiling packages: gra-log-purger/f02fa5774ab54dbb1b1c3702d03cb929b85d60e6 (00:00:36)
		Task 4506 | 06:08:31 | Compiling packages: mariadb_ctrl/7658290da98e2cad209456f174d3b9fa143c87fc
		Task 4506 | 06:08:32 | Compiling packages: switchboard/fad565dadbb37470771801952001c7071e55a364 (00:00:39)
		Task 4506 | 06:08:58 | Compiling packages: galera-healthcheck/3da4dedbcd7d9f404a19e7720e226fd472002266 (00:00:28)
		Task 4506 | 06:08:59 | Compiling packages: mariadb_ctrl/7658290da98e2cad209456f174d3b9fa143c87fc (00:00:28)
		Task 4506 | 06:09:42 | Compiling packages: boost/3eb8bdb1abb7eff5b63c4c5bdb41c0a778925c31 (00:02:37)
		Task 4506 | 06:11:27 | Compiling packages: python/4e255efa754d91b825476b57e111345f200944e1 (00:04:32)
		Task 4506 | 06:11:27 | Compiling packages: scons/11e7ad3b28b43a96de3df7aa41afddde582fcc38 (00:00:22)
		Task 4506 | 06:11:49 | Compiling packages: galera/d15a1d2d15e5e7417278d4aa1b908566022b9623 (00:13:18)
		Task 4506 | 06:25:07 | Compiling packages: mariadb/43aa3547bc5a01dd51f1501e6b93c215dd7255e9 (00:18:49)
		Task 4506 | 06:43:56 | Compiling packages: xtrabackup/2e701e7a9e4241b28052d984733de36aae152275 (00:10:26)
		Task 4506 | 06:55:22 | Creating missing vms: mysql/ea075ae6-6326-478b-a1ba-7fbb0b5b0bf5 (0)
		Task 4506 | 06:55:22 | Creating missing vms: mysql/e8c52bf2-cd48-45d0-9553-f6367942a634 (2)
		Task 4506 | 06:55:22 | Creating missing vms: proxy/023edddd-418e-46e4-8d40-db452c694e16 (0)
		Task 4506 | 06:55:22 | Creating missing vms: mysql/8a830154-25b6-432a-ad39-9ff09d015760 (1)
		Task 4506 | 06:55:22 | Creating missing vms: paasta-mysql-java-broker/bb5676ca-efba-48fc-bc11-f464d0ae9c78 (0)
		Task 4506 | 06:57:18 | Creating missing vms: mysql/ea075ae6-6326-478b-a1ba-7fbb0b5b0bf5 (0) (00:01:56)
		Task 4506 | 06:57:23 | Creating missing vms: proxy/023edddd-418e-46e4-8d40-db452c694e16 (0) (00:02:01)
		Task 4506 | 06:57:23 | Creating missing vms: mysql/e8c52bf2-cd48-45d0-9553-f6367942a634 (2) (00:02:01)
		Task 4506 | 06:57:23 | Creating missing vms: paasta-mysql-java-broker/bb5676ca-efba-48fc-bc11-f464d0ae9c78 (0) (00:02:01)
		Task 4506 | 06:57:23 | Creating missing vms: mysql/8a830154-25b6-432a-ad39-9ff09d015760 (1) (00:02:01)
		Task 4506 | 06:57:24 | Updating instance mysql: mysql/ea075ae6-6326-478b-a1ba-7fbb0b5b0bf5 (0) (canary) (00:02:32)
		Task 4506 | 06:59:56 | Updating instance mysql: mysql/8a830154-25b6-432a-ad39-9ff09d015760 (1) (00:03:03)
		Task 4506 | 07:02:59 | Updating instance mysql: mysql/e8c52bf2-cd48-45d0-9553-f6367942a634 (2) (00:03:04)
		Task 4506 | 07:06:03 | Updating instance proxy: proxy/023edddd-418e-46e4-8d40-db452c694e16 (0) (canary) (00:01:01)
		Task 4506 | 07:07:04 | Updating instance paasta-mysql-java-broker: paasta-mysql-java-broker/bb5676ca-efba-48fc-bc11-f464d0ae9c78 (0) (canary) (00:01:02)

		Task 4506 Started  Fri Aug 31 06:04:10 UTC 2018
		Task 4506 Finished Fri Aug 31 07:08:06 UTC 2018
		Task 4506 Duration 01:03:56
		Task 4506 done

		Succeeded


-	배포된 Redis 서비스팩을 확인한다.

- **사용 예시**

		$bosh -e micro-bosh -d paasta-redis-service vms
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Task 8105. Done

		Deployment 'paasta-redis-service'

		Instance                                                  Process State  AZ  IPs            VM CID                                   VM Type  Active  
		dedicated-node/609c24b3-7a05-4060-b6c3-e6af2c93dd01       running        z5  10.30.107.174  vm-49ab7b30-dcc7-44c7-ba95-3fe450744a61  minimal  true  
		dedicated-node/7790fa96-7830-4e5a-ad1b-e9f3962b1002       running        z5  10.30.107.173  vm-c0d3ae90-3315-4840-80e5-c671107c5719  minimal  true  
		dedicated-node/831598c5-48eb-4654-b2cc-df03a0f7a1db       running        z5  10.30.107.172  vm-6cc60d6d-9a3a-41c0-a2eb-ee9919256fe4  minimal  true  
		paasta-redis-broker/7031a6f9-2f7a-4080-8bad-5753bfce1acf  running        z5  10.30.107.171  vm-c124e2b1-e06a-452b-bd85-a75fc1dee4aa  minimal  true  

		4 vms

		Succeeded


### <div id='24'> 2.4. Redis 서비스 브로커 등록
Redis 서비스팩 배포가 완료 되었으면 Application에서 서비스 팩을 사용하기 위해서 먼저 Redis 서비스 브로커를 등록해 주어야 한다.
서비스 브로커 등록시에는 PaaS-TA에서 서비스 브로커를 등록할 수 있는 사용자로 로그인하여야 한다


##### 서비스 브로커 목록을 확인한다.

>`$ cf service-brokers`
```
Getting service brokers as admin...

name                     url
paasta-pinpoint-broker  http://10.30.70.82:8080
```


##### Redis 서비스 브로커를 등록한다.
>`$ cf create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{서비스팩 URL(IP)}`
  
  **서비스팩 이름** : 서비스 팩 관리를 위해 PaaS-TA에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스팩 리스트의 명칭이다.<br>
  **서비스팩 사용자ID** / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID입니다. 서비스팩도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.<br>
  **서비스팩 URL** : 서비스팩이 제공하는 API를 사용할 수 있는 URL을 입력한다.

>`$ cf create-service-broker redis-service-broker admin admin http://10.30.107.171:12350`

```
Creating service broker paasta-redis-brokeras admin...
OK
```

##### 등록된 Redis 서비스 브로커를 확인한다.

>`$ cf service-brokers`
```
Getting service brokers as admin...

name                     url
paasta-pinpoint-broker  http://10.30.70.82:8080
paasta-redis-broker     http://10.30.60.71:12350
```


##### 접근 가능한 서비스 목록을 확인한다.

>`$ cf service-access`
```
Getting service access as admin...
broker: paasta-redis-broker
  service   plan           access   orgs
  redis     shared-vm      none
  redis     dedicated-vm   none

broker: paasta-pinpoint-broker
  service    plan                access   orgs
  Pinpoint   Pinpoint_standard   none
```
서비스 브로커 등록시 최초에는 접근을 허용하지 않는다. 따라서 access는 none으로 설정된다.


##### 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)

>`$ cf enable-service-access redis` <br>
>`$ ccf service-access`
```
Getting service access as admin...
broker: paasta-redis-broker
  service   plan           access   orgs
  redis     shared-vm      all
  redis     dedicated-vm   all

broker: paasta-pinpoint-broker
  service    plan                access   orgs
  Pinpoint   Pinpoint_standard   none
```

# <div id='3'>  3. Redis연동 Sample App 설명
본 Sample App은 PaaS-TA에 배포되며 Redis의 서비스를 Provision과 Bind를 한 상태에서 사용이 가능하다.

###<div id='31'> 3.1. Sample App 구조
Sample App은 PaaS-TA에 App으로 배포가 된다. 배포 완료 후 정상적으로 App이 구동되면 curl 명령어로 배포된 도메인 정보를 이용하여 Redis에 특정 key 에 값을 저장/조회/삭제를 한다.

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


##### PaaSTA-Sample-Apps.zip 파일의 압축을 풀고 Service 폴더 안에 있는 Redis Sample Web App인 redis-example-app을 복사한다.

>`$ ls -all`

> ![redis_image_02]

<br>

### <div id='32'> 3.2. PaaS-TA에서 서비스 신청
Sample App에서 Redis 서비스를 사용하기 위해서는 서비스 신청(Provision)을 해야 한다.
*참고: 서비스 신청시 PaaS-TA에서 서비스를 신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.

##### 먼저 PaaS-TA Marketplace에서 서비스가 있는지 확인을 한다.

>`$  cf marketplace`

> ![redis_image_03]

<br>

##### Marketplace에서 원하는 서비스가 있으면 서비스 신청(Provision)을 한다.

>`$ cf create-service {서비스명} {서비스 플랜} {내 서비스명}`
- **서비스명** : p-rabbitmq로 Marketplace에서 보여지는 서비스 명칭이다.
- **서비스플랜** : 서비스에 대한 정책으로 plans에 있는 정보 중 하나를 선택한다. RabbitMQ 서비스는 standard plan만 지원한다.
- **내 서비스명** : 내 서비스에서 보여지는 명칭이다. 이 명칭을 기준으로 환경 설정 정보를 가져온다.


>`$  cf create-service redis shared-vm redis-service-instance`

>![redis_image_04]

<br>

##### 생성된 Redis 서비스 인스턴스를 확인한다.

>`$ cf services`

> ![redis_image_05]

<br>

### <div id='33'> 3.3. Sample App에 서비스 바인드 신청 및 App 확인
서비스 신청이 완료되었으면 Sample App 에서는 생성된 서비스 인스턴스를 Bind 하여 App에서 Redis 서비스를 이용한다.
*참고: 서비스 Bind 신청시 PaaS-TA에서 서비스 Bind신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.


##### Sample App 디렉토리로 이동하여 manifest 파일을 확인한다.

>`$ cd redis-example-app`

>`$ vi manifest.yml`
```

---
applications:
- name: hello- spring-glustefs          # 배포할 App 이름
  memory: 512M                          # 배포시 메모리 사이즈
  instances: 1                          # 배포 인스턴스 수
path: target/hello-spring-glusterfs.war # 배포하는 App 파일 PATH
```


##### --no-start 옵션으로 App을 배포한다.
- -no-start: App 배포시 구동은 하지 않는다.

>`$ cf push --no-start`

> ![redis_image_06]

<br>

##### 배포된 Sample App을 확인하고 로그를 수행한다.

>`$ cf apps`

> ![redis_image_07]

<br>

>`$ cf logs redis-example-app`

> ![redis_image_08]

<br>

##### Sample App에서 생성한 서비스 인스턴스 바인드 신청을 한다.

>`$ cf bind-service redis-example-app redis-service-instancep`

> ![redis_image_09]

##### 바인드를 적용하기 위해서 App을 재기동한다.

>`$ cf restart redis-example-app`

> ![redis_image_10]

<br>

##### App이 정상적으로 Redis 서비스를 사용하는지 확인한다.

##### curl 로 확인

>`$  export APP=redis-example-app.controller.open-paas.com`

>`$ curl -X PUT $APP/foo -d 'data=bar`

> ![redis_image_11]

<br>

##### Key 값을 이용하여 Redis 에 저장되어 있는 Value 값 Get

>`$curl -X GET $APP/foo`

![redis_image_12]

<br>

##### Key 값을 이용하여 Redis 에 저장되어 있는 Value 삭제

>`$  curl -X DELETE $APP/foo`

![redis_image_13]

<br>


# <div id='4'> 4. Redis Client 툴 접속
Application에 바인딩 된 Redis 서비스 연결정보는 Private IP로 구성되어 있기 때문에 Redis Client 툴에서 직접 연결할 수 없다. 따라서 Redis Client 툴에서 SSH 터널, Proxy 터널 등을 제공하는 툴을 사용해서 연결하여야 한다. 본 가이드는 SSH 터널을 이용하여 연결 하는 방법을 제공하며 Redis Client 툴로써는 오픈 소스인 Redis Desktop Manager로 가이드한다. Redis Desktop Manager 에서 접속하기 위해서 먼저 SSH 터널링할수 있는 VM 인스턴스를 생성해야한다. 이 인스턴스는 SSH로 접속이 가능해야 하고 접속 후 PaaS-TA에 설치한 서비스팩에 Private IP 와 해당 포트로 접근이 가능하도록 시큐리티 그룹을 구성해야 한다. 이 부분은 OpenStack 관리자 및 PaaS-TA 운영자에게 문의하여 구성한다. vsphere 에서 구성한 인스턴스는 공개키(.pem) 로 접속을 해야 하므로 공개키는 운영 담당자에게 문의하여 제공받는다. 참고) 개인키(.ppk)로는 접속이 되지 않는다.


### <div id='41'> 4.1. Redis Desktop Manager 설치 및 연결
Redis Desktop Manager 프로그램은 무료로 사용할 수 있는 오픈소스 소프트웨어이다.

##### Redis Desktop Manager를 다운로드 하기 위해 아래 URL로 이동하여 설치파일을 다운로드 한다.
[**http://redisdesktop.com/download**](http://redisdesktop.com/download)
![redis_image_14]

##### 다운로드한 설치파일을 실행한다.
> ![redis_image_15]

##### Redis Desktop Manager 설치를 위한 안내사항이다. Next 버튼을 클릭한다.
> ![redis_image_16]

##### 프로그램 라이선스에 관련된 내용이다. I Agree 버튼을 클릭한다.
> ![redis_image_17]

##### Redis Desktop Manager를 설치할 경로를 설정 후 Install 버튼을 클릭한다.
별도의 경로 설정이 필요 없을 경우 default로 C드라이브 Program Files 폴더에 설치가 된다.
> ![redis_image_18]

##### 설치 완료 후 Next 버튼을 클릭하여 다음 과정을 진행한다.
> ![redis_image_19]

##### Finish 버튼 클릭으로 설치를 완료한다.
> ![redis_image_20]

##### Redis Desktop Manager를 실행했을 때 처음 뜨는 화면이다. 이 화면에서 Server에 접속하기 위한 profile을 설정/저장하여 접속할 수 있다. Connect to Redis Server 버튼을 클릭한다.
> ![redis_image_21]

##### Connection 탭에서 아래 붉은색 영역에 접속하려는 서버 정보를 모두 입력한다.
> ![redis_image_22]

##### 서버 정보는 Application에 바인드 되어 있는 서버 정보를 입력한다. cfenv<app_name> 명령어로 이용하여 확인한다.
예) $ cfenvredis-example-app
> ![redis_image_23]

##### SSH Tunnel탭을 클릭하고 PaaS-TA 운영 관리자에게 제공받은 SSH 터널링 가능한 서버 정보를 입력하고 공개키(.pem) 파일을 불러온다. Test Connection 버튼을 클릭하면 Redis 서버에 접속이 되는지 테스트 하고 OK 버튼을 눌러 Redis 서버에 접속한다.
(참고) 만일 공개키 없이 ID/Password로 접속이 가능한 경우에는 공개키 대신 사용자 이름과 암호를 입력한다.
> ![redis_image_24]

##### 접속이 완료되고 좌측 서버 정보를 더블 클릭하면 좌측에 스키마 정보가 나타난다.
> ![redis_image_25]

##### 신규 키 등록후 확인
> ![redis_image_26]

[redis_image_01]:/Service-Guide/images/redis/redis_image_01.png
[redis_image_02]:/Service-Guide/images/redis/redis_image_02.png
[redis_image_03]:/Service-Guide/images/redis/redis_image_03.png
[redis_image_04]:/Service-Guide/images/redis/redis_image_04.png
[redis_image_05]:/Service-Guide/images/redis/redis_image_05.png
[redis_image_06]:/Service-Guide/images/redis/redis_image_06.png
[redis_image_07]:/Service-Guide/images/redis/redis_image_07.png
[redis_image_08]:/Service-Guide/images/redis/redis_image_08.png
[redis_image_09]:/Service-Guide/images/redis/redis_image_09.png
[redis_image_10]:/Service-Guide/images/redis/redis_image_10.png
[redis_image_11]:/Service-Guide/images/redis/redis_image_11.png
[redis_image_12]:/Service-Guide/images/redis/redis_image_12.png
[redis_image_13]:/Service-Guide/images/redis/redis_image_13.png
[redis_image_14]:/Service-Guide/images/redis/redis_image_14.png
[redis_image_15]:/Service-Guide/images/redis/redis_image_15.png
[redis_image_16]:/Service-Guide/images/redis/redis_image_16.png
[redis_image_17]:/Service-Guide/images/redis/redis_image_17.png
[redis_image_18]:/Service-Guide/images/redis/redis_image_18.png
[redis_image_19]:/Service-Guide/images/redis/redis_image_19.png
[redis_image_20]:/Service-Guide/images/redis/redis_image_20.png
[redis_image_21]:/Service-Guide/images/redis/redis_image_21.png
[redis_image_22]:/Service-Guide/images/redis/redis_image_22.png
[redis_image_23]:/Service-Guide/images/redis/redis_image_23.png
[redis_image_24]:/Service-Guide/images/redis/redis_image_24.png
[redis_image_25]:/Service-Guide/images/redis/redis_image_25.png
[redis_image_26]:/Service-Guide/images/redis/redis_image_26.png
