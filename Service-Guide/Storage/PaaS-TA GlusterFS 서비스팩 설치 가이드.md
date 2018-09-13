## Table of Contents
1. [문서 개요](#1)
  - [1.1. 목적](#2)
  - [1.2. 범위](#3)
  - [1.3. 시스템 구성도](#4)
  - [1.4. 참고자료](#5)
2. [GlusterFS 서비스팩 설치](#6)
  -  [2.1. 설치전 준비사항](#7)
  -  [2.2. GlusterFS 서비스 릴리즈 업로드](#8)
  -  [2.3. GlusterFS 서비스 Deployment 파일 수정 및 배포](#9)
  -  [2.4. GlusterFS 서비스 브로커 등록](#10)
3. [GlusterFS 연동 Sample App 설명](#11)
  - [3.1. Sample App 구조](#12)
  - [3.2. PaaS-TA에서 서비스 신청](#13)
  - [3.3. Sample App에 서비스 바인드 신청 및 App 확인](#14)


# 1. 문서 개요

### <div id='2'>1.1. 목적</div>
본 문서(GlusterFS 서비스팩 설치 가이드)는 전자정부 표준 프레임워크 기반의 PaaS-TA에서 제공되는 서비스팩인 GlusterFS 서비스팩을 Bosh를 이용하여 설치 하는 방법과 PaaS-TA의 SaaS 형태로 제공하는 Application 에서GlusterFS 서비스를 사용하는 방법을 기술하였다.
PaaS-TA 3.5 버전부터는 Bosh2.0 기반으로 deploy를 진행하며 기존 Bosh1.0 기반으로 설치를 원할경우에는 PaaS-TA 3.1 이하 버전의 문서를 참고한다.

### <div id='3'> 1.2. 범위 </div>
설치 범위는 GlusterFS 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다.

### <div id='4'>1.3. 시스템 구성도</div>
본 문서의 설치된 시스템 구성도이다. Mysql Server, GlusterFS 서비스 브로커로 최소사항을 구성하였고 서비스 백엔드는 외부에 구성되어 있다.
![시스템 구성도][glusterfs_image_01]

* 설치할때 cloud config에서 사용하는 VM_Tpye명과 스펙 

| VM_Type | 스펙 |
|--------|-------|
|minimal| 1vCPU / 1GB RAM / 8GB Disk|

* 각 Instance의 Resource Pool과 스펙

| 구분 | Resource Pool | 스펙 |
|--------|-------|-------|
| paasta-glusterfs-broke | minimal | 1vCPU / 1GB RAM / 8GB Disk |
| mysql | minimal | 1vCPU / 1GB RAM / 8GB Disk |

<br>
<div id='5'></div>

### 1.4. 참고자료
[**http://bosh.io/docs**](http://bosh.io/docs) <br>
[**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/)

# <div id='6'>2. GlusterFS 서비스팩 설치</div>

### <div id='7'> 2.1. 설치전 준비사항</div>
본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다.
서비스팩 설치를 위해서는 먼저 BOSH CLI v2 가 설치 되어 있어야 하고 BOSH 에 로그인이 되어 있어야 한다.<br>
BOSH CLI v2 가 설치 되어 있지 않을 경우 먼저 BOSH2.0 설치 가이드 문서를 참고 하여 BOSH CLI v2를 설치를 하고 사용법을 숙지 해야 한다.<br>

- PaaS-TA에서 제공하는 압축된 릴리즈 파일들을 다운받는다. (PaaSTA-Deployment.zip, PaaSTA-Sample-Apps.zip, PaaSTA-Services.zip)

- 다운로드 위치
>PaaSTA-Services : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Services.zip>**  
>PaaSTA-Deployment : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Deployment.zip>**  
>PaaSTA-Sample-Apps : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Sample-Apps.zip>**


### <div id='8'>2.2. GlusterFS 서비스 릴리즈 업로드</div>

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
		
-	GlusterFS 서비스 릴리즈가 업로드 되어 있지 않은 것을 확인

-	GlusterFS 서비스 릴리즈 파일을 업로드한다.

- **사용 예시**

		$ bosh -e micro-bosh upload-release paasta-glusterfs-2.0.tgz
    		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		######################################################## 100.00% 144.14 MiB/s 2s
		Task 4460

		Task 4460 | 04:31:41 | Extracting release: Extracting release (00:00:04)
		Task 4460 | 04:31:45 | Verifying manifest: Verifying manifest (00:00:00)
		Task 4460 | 04:31:45 | Resolving package dependencies: Resolving package dependencies (00:00:00)
		Task 4460 | 04:31:45 | Creating new packages: gra-log-purger/f02fa5774ab54dbb1b1c3702d03cb929b85d60e6 (00:00:00)
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
		Task 4460 | 04:31:56 | Creating new packages: golang/f57ddbc8d55d7a0f08775bf76bb6a27dc98c7ea7 (00:00:01)
		Task 4460 | 04:31:57 | Creating new jobs: acceptance-tests/48c00c36ec5210cbdd3b125ae6a72cfdf6eaf4e2 (00:00:00)
		Task 4460 | 04:31:57 | Creating new jobs: broker-deregistrar/b5f6f776d46eb1ac561ab1e8f58d8ddedb97f86e (00:00:00)
		Task 4460 | 04:31:57 | Creating new jobs: proxy/7907d8759aa11dfcbbe79220dc945c96b5562ac1 (00:00:00)
		Task 4460 | 04:31:57 | Creating new jobs: mysql/078561f02f2516212ed59c48e1dd45360f93871c (00:00:00)
		Task 4460 | 04:31:57 | Creating new jobs: broker-registrar/e1f5e30b87e70e916ea74ea8eb63a7b6ff6ff643 (00:00:00)
		Task 4460 | 04:31:57 | Release has been created: paasta-mysql/2.0 (00:00:00)

		Task 4460 Started  Fri Aug 31 04:31:41 UTC 2018
		Task 4460 Finished Fri Aug 31 04:31:57 UTC 2018
		Task 4460 Duration 00:00:16
		Task 4460 done

		Succeeded
          


-    업로드 된 GlusterFS 릴리즈를 확인한다.

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
		paasta-glusterfs                  2.0       85e3f01e+  
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
		
>Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드를 해야 한다. (glusterfs 는  stemcell 3309 버전을 사용)

###   <div id='9'>2.3. glusterfs 서비스 Deployment 파일 수정 및 배포</div>
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
# paasta-swift-object-service 설정 파일 내용
name: paasta-swift-object-service                       # 서비스 배포이름(필수)

releases:
- name: paasta-glusterfs             # 서비스 릴리즈 이름(필수)
  version: "2.0"                       # 서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전

stemcells:
- alias: default
  os: ((stemcell_os))
  version: "((stemcell_version))"

update:
  canaries: 1                        # canary 인스턴스 수(필수)
  canary_watch_time: 30000-600000    # canary 인스턴스가 수행하기 위한 대기 시간(필수)
  max_in_flight: 1                   # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)
  update_watch_time: 30000-600000    # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)

instance_groups:
- name: mysql                        #작업 이름(필수): mysql 서버
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.196
  properties:
    admin_username: root          # MySQL 어드민 계정
    admin_password: admin         # MySQL 어드민 패스워드
  templates:
  - name: mysql                # job template 이름(필수)
    release: paasta-glusterfs

- name: paasta-glusterfs-broker           #작업 이름(필수): broker 서버
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
    static_ips:                           # 사용할 IP addresses
    - 10.30.107.197
  properties:
    jdbc_ip: 10.30.107.196             # Mysql IP(필수)
    jdbc_pwd: admin                    # Mysql password(필수)
    jdbc_port: 3306                    # Mysql Port
    log_dir: paasta-glusterfs-broker   # Broker Log 저장 디렉토리 명
    log_file: paasta-glusterfs-broker  # Broker Log 저장 파일 명
    log_level: INFO                    # Broker Log 단계
    glusterfs_url: 54.211.7.235        # Glusterfs 서비스 주소
    glusterfs_tenantname: service      # Glusterfs 서비스 테넌트 이름
    glusterfs_username: swift          # Glusterfs 서비스 계정 아이디
    glusterfs_password: password       # Glusterfs 서비스 암호
  templates:
  - name: op-glusterfs-java-broker                # job template 이름(필수)
    release: paasta-glusterfs

- name: broker-registrar           #작업 이름(필수): broker-registrar 
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  lifecycle: errand                    # bosh deploy시 vm에 생성되어 설치 되지 않고 bosh errand 로실행>할때 설정, 주로 테스트 용도에 쓰임
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
  properties:
    broker:
      host: 10.30.107.197          # Service Broker IP
      name: glusterfs-service      # Service Broker Name
      password: cloudfoundry       # Service Broker Auth Password
      username: admin              # Service Broker Auth Id
      protocol: http               # Service Broker Http Protocol
      port: 8080                   # Service Broker port
    cf:
      admin_password: admin                      # CF Paasword
      admin_username: admin_test                      # CF Id
      api_url: https://api.115.68.46.189.xip.io   # CF Target Url
      skip_ssl_validation: true                  # CF SSL 설정
  templates:
  - name: broker-registrar                # job template 이름(필수)
    release: paasta-glusterfs

- name: broker-deregistrar           #작업 이름(필수): broker-deregistrar 
  azs:
  - z5
  instances: 1  # job 인스턴스 수(필수)
  lifecycle: errand                    # bosh deploy시 vm에 생성되어 설치 되지 않고 bosh errand 로실행>할때 설정, 주로 테스트 용도에 쓰임
  vm_type: ((vm_type_small))              # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))        # cloud config 에 정의한 network 이름
  properties:
    broker:
      name: glusterfs-service
    cf:
      admin_password: admin                      # CF Paasword
      admin_username: admin_test                      # CF Id
      api_url: https://api.115.68.46.189.xip.io   # CF Target Url
      skip_ssl_validation: true                  # CF SSL 설정
  templates:
  - name: broker-deregistrar                # job template 이름(필수)
    release: paasta-glusterfs
```

-	deploy-mysql-bosh2.0.sh 파일을 서버 환경에 맞게 수정한다.

```sh
#!/bin/bash
# stemcell 버전은 3309 버전으로 사용하시고 https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Download_Page.md 에서 다운받아 쓰십시요.

bosh -e micro-bosh -d paasta-swift-object-service deploy paasta_swift_object_bosh2.0.yml \
   -v default_network_name=service_private \
   -v stemcell_os=ubuntu-trusty \
   -v stemcell_version=3309 \
   -v vm_type_small=minimal
```


-	Swift Object 서비스팩을 배포한다.

- **사용 예시**

		$ ./deploy-swift-object-bosh2.0.sh
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Using deployment 'paasta-swift-object-service'

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

		+ name: paasta-swift-object-service

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


-	배포된 swift object 서비스팩을 확인한다.

- **사용 예시**

		$ bosh -e micro-bosh -d paasta-swift-object-service vms
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Task 6445. Done

		Deployment 'paasta-swift-object-service'

		Instance                                                      Process State  AZ  IPs            VM CID                                   VM Type  Active  
		mysql/a217e0a4-a2a8-4b5e-9fb5-771ac6e6d99d                    running        z5  10.30.107.196  vm-b02e1b45-3d13-45db-9617-360feb8ad2e2  minimal  true  
		paasta-glusterfs-broker/c924a4e0-07ae-4371-a129-22b172e72a83  running        z5  10.30.107.197  vm-b2247547-ca01-4e12-831e-286998fb36b1  minimal  true  

		2 vms

		Succeeded

<div id='10'> </div>

### 2.4. GlusterFS 서비스 브로커 등록
GlusterFS 서비스팩 배포가 완료 되었으면 Application에서 서비스 팩을 사용하기 위해서 먼저 GlusterFS 서비스 브로커를 등록해 주어야 한다.
서비스 브로커 등록시에는 PaaS-TA에서 서비스 브로커를 등록할 수 있는 사용자로 로그인 하여야 한다


##### 서비스 브로커 목록을 확인한다.

>`$ $ cf service-brokers`

> ![glusterfs_image_02]

<br>

##### GlusterFS 서비스 브로커를 등록한다.

>`$ cf create-service {서비스명} {서비스 플랜} {내 서비스명}`
- **서비스명** : p-rabbitmq로 Marketplace에서 보여지는 서비스 명칭이다.
- **서비스플랜** : 서비스에 대한 정책으로 plans에 있는 정보 중 하나를 선택한다. RabbitMQ 서비스는 standard plan만 지원한다.
- **내 서비스명** : 내 서비스에서 보여지는 명칭이다. 이 명칭을 기준으로 환경 설정 정보를 가져온다.

>`$ cf create-service-broker glusterfs-service admin cloudfoundry http://10.30.107.197:8080`

> ![glusterfs_image_03]

<br>

##### 등록된 GlusterFS 서비스 브로커를 확인한다.

>`$ cf service-brokers`

> ![glusterfs_image_04]

<br>

##### 접근 가능한 서비스 목록을 확인한다.

>`$ cf service-access`

>![glusterfs_image_05]

>서비스 브로커 등록시 최초에는 접근을 허용하지 않는다. 따라서 access는 none으로 설정된다.

<br>

##### 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)


>`$ cf enable-service-access glusterfs` <br>

>`$ cf service-access`

>![glusterfs_image_06]

<br>

<div id='11'></div>

#   3. GlusterFS 연동 Sample App 설명
본 Sample Web App은 PaaS-TA에 배포되며 GlusterFS의 서비스를 Provision과 Bind를 한 상태에서 사용이 가능하다.


<div id='12'></div>

### 3.1. Sample App 구조
Sample Web App은 PaaS-TA에 App으로 배포가 된다. 배포 완료 후 정상적으로 App 이 구동되면 브라우저나 curl로 해당 App에 접속 하여 GlusterFS 환경정보(서비스 연결 정보)와파일 업로드하고 확인하는 기능을 제공한다.

Sample App 구조는 다음과 같다.
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
    <td>pom.xml</td>
    <td>maven project 설정 파일</td>
  </tr>
  <tr>
    <td>target</td>
    <td>maven build시 생성되는 디렉토리(war 파일, classes 폴더 등)</td>
  </tr>
</table>

<br>

##### PaaSTA-Sample-Apps.zip 파일 압축을 풀고 Service폴더안에 있는 GlusterFSSample Web App인 hello-spring-glusterfs를 복사한다.

>`$ ls -all`

>![glusterfs_image_07]

<br>

<div id='13'></div>

### 3.2. PaaS-TA에서 서비스 신청
Sample App에서 GlusterFS 서비스를 사용하기 위해서는 서비스 신청(Provision)을 해야 한다.
*참고: 서비스 신청시 PaaS-TA에서 서비스를 신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.

##### 먼저 PaaS-TA Marketplace에서 서비스가 있는지 확인을 한다.

>`$ cf marketplace`

>![glusterfs_image_08]

<br>

##### Marketplace에서 원하는 서비스가 있으면 서비스 신청(Provision)을 한다.

>`$ cf create-service {서비스명} {서비스 플랜} {내 서비스명}`
- **서비스명** : p-rabbitmq로 Marketplace에서 보여지는 서비스 명칭이다.
- **서비스플랜** : 서비스에 대한 정책으로 plans에 있는 정보 중 하나를 선택한다. RabbitMQ 서비스는 standard plan만 지원한다.
- **내 서비스명** : 내 서비스에서 보여지는 명칭이다. 이 명칭을 기준으로 환경 설정 정보를 가져온다.

>`$ cf create-service glusterfs glusterfs-1000Mb glusterfs-service-instance`

>![glusterfs_image_09]

<br>


##### 생성된 GlusterFS 서비스 인스턴스를 확인한다.

>`$ cf services`

>![glusterfs_image_10]

<br>

<div id='14'></div>

### 3.3. Sample App에 서비스 바인드 신청 및 App 확인
서비스 신청이 완료되었으면 Sample App 에서는 생성된 서비스 인스턴스를 Bind 하여 App에서 GlusterFS 서비스를 이용한다.
* 참고: 서비스 Bind 신청시 PaaS-TA에서 서비스 Bind 신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.


##### Sample App 디렉토리로 이동하여 manifest 파일을 확인한다.

>`$ cd hello-spring-glusterfs`

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

>![glusterfs_image_11]

<br>

##### 배포된 Sample App을 확인하고 로그를 수행한다.

>`$ cf apps`

>![glusterfs_image_12]

<br>


>`$ cf logs hello-spring-glusterfs`  **//cf logs {배포된 App명} **

>![glusterfs_image_13]

<br>

##### Sample App에서 생성한 서비스 인스턴스 바인드 신청을 한다. 

>`$ cf bind-service hello-spring-glusterfs glusterfs-service-instance` 

> ![glusterfs_image_14]

<br>

##### 바인드가 적용되기 위해서 App을 재기동한다.

>`$ cf restart hello-spring-glusterfs` 

> ![glusterfs_image_15]

<br>

##### App이 정상적으로 GlusterFS  서비스를 사용하는지 확인한다.


##### curl 로 확인

>`$ curl --form attchFile=@../../../Desert.jpg --form press=OK hello-spring-glusterfs.115.68.46.30.xip.io/swifttest` 

> ![glusterfs_image_16]

<br>

##### 브라우에서 이미지 확인

> ![glusterfs_image_17]

[glusterfs_image_01]:/Service-Guide/images/glusterfs/glusterfs_image_01.png
[glusterfs_image_02]:/Service-Guide/images/glusterfs/glusterfs_image_02.png
[glusterfs_image_03]:/Service-Guide/images/glusterfs/glusterfs_image_03.png
[glusterfs_image_04]:/Service-Guide/images/glusterfs/glusterfs_image_04.png
[glusterfs_image_05]:/Service-Guide/images/glusterfs/glusterfs_image_05.png
[glusterfs_image_06]:/Service-Guide/images/glusterfs/glusterfs_image_06.png
[glusterfs_image_07]:/Service-Guide/images/glusterfs/glusterfs_image_07.png
[glusterfs_image_08]:/Service-Guide/images/glusterfs/glusterfs_image_08.png
[glusterfs_image_09]:/Service-Guide/images/glusterfs/glusterfs_image_09.png
[glusterfs_image_10]:/Service-Guide/images/glusterfs/glusterfs_image_10.png
[glusterfs_image_11]:/Service-Guide/images/glusterfs/glusterfs_image_11.png
[glusterfs_image_12]:/Service-Guide/images/glusterfs/glusterfs_image_12.png
[glusterfs_image_13]:/Service-Guide/images/glusterfs/glusterfs_image_13.png
[glusterfs_image_14]:/Service-Guide/images/glusterfs/glusterfs_image_14.png
[glusterfs_image_15]:/Service-Guide/images/glusterfs/glusterfs_image_15.png
[glusterfs_image_16]:/Service-Guide/images/glusterfs/glusterfs_image_16.png
[glusterfs_image_17]:/Service-Guide/images/glusterfs/glusterfs_image_17.jpeg

