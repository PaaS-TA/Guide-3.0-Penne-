## Table of Contents
1. [문서 개요](#1-문서-개요)
    *  [1.1 목적](#11-목적)
    *  [1.2 범위](#12-범위)
    *  [1.3 시스템 구성도](#13-시스템-구성도)
    *  [1.4 참고자료](#14-참고자료)
2. [PaaS-TA Portal 설치](#2-paas-ta-portal-설치)
    *  [2.1 설치전 준비사항](#21-설치전-준비사항)
    *  [2.2 PaaS-TA Portal 릴리즈 업로드](#22-paas-ta-portal-릴리즈-업로드)
    *  [2.3 PaaS-TA Portal Deployment 배포](#23-paas-ta-portal-deployment-배포)
    *  [2.4 사용자의 조직 생성 Flag 활성화](#24-사용자의-조직-생성-flag-활성화)
3. [PaaS-TA Portal 운영](#3-paas-ta-portal-운영)
    *  [3.1 DB Migration](#31-db-migration)
    *  [3.2 Log](#32-log)
    *  [3.3 카탈로그 적용](#33-카탈로그-적용)

# 1. 문서 개요
### 1.1. 목적

본 문서(PaaS-TA Portal Release 설치 가이드)는 전자정부표준프레임워크 기반의 PaaS-TA에서 제공되는 PaaS-TA Portal Release를 Bosh2.0을 이용하여 설치 하는 방법을 기술하였다.
PaaS-TA 3.5 버전부터는 Bosh2.0 기반으로 deploy를 진행하며 기존 Bosh1.0 기반으로 설치를 원할경우에는 PaaS-TA 3.1 이하 버전의 문서를 참고한다.

### 1.2. 범위
설치 범위는 PaaS-TA Portal Release를 검증하기 위한 기본 설치를 기준으로 작성하였다.

### 1.3. 시스템 구성도
본 문서의 설치된 시스템 구성도이다. Binary Storage, Mariadb, Proxy, Gateway Api, Registration Api, Portal Api, Common Api, Log Api, Storage Api, Webadmin, Webuser로 최소사항을 구성하였다.

![시스템구성도][paas-ta-portal-01]
* Paas-TA Portal 설치할때 cloud config에 추가적으로 정의한 VM_Tpye명과 스펙 

| VM_Type | 스펙 |
|--------|-------|
|portal_tiny| 1vCPU / 256MB RAM / 4GB Disk|
|portal_medium| 1vCPU / 1GB RAM / 4GB Disk|
|portal_small| 1vCPU / 512MB RAM / 4GB Disk|


* Paas-TA Portal각 Instance의 Resource Pool과 스펙

| 구분 | Resource Pool | 스펙 |
|--------|-------|-------|
| binary_storage | portal_small | 1vCPU / 512MB RAM / 4GB Disk 10GB(영구적 Disk) |
| haproxy |portal_small| 1vCPU / 512MB RAM / 4GB Disk|
| mariadb | portal_small | 1vCPU / 512MB RAM / 4GB Disk +10GB(영구적 Disk) |
| paas-ta-portal-registration | portal_small | 1vCPU / 512MB RAM / 4GB Disk |
| paas-ta-portal-gateway | portal_small | 1vCPU / 512MB RAM / 4GB Disk |
| paas-ta-portal-api | portal_medium | 1vCPU / 1GB RAM / 4GB Disk |
| paas-ta-portal-common-api | portal_small | 1vCPU / 512MB RAM / 4GB Disk |
| paas-ta-portal-log-api | portal_small | 1vCPU / 512MB RAM / 4GB Disk |
| paas-ta-portal-storage-api | portal_small | 1vCPU / 512MB RAM / 4GB Disk |
| paas-ta-portal-webadmin | portal_small | 1vCPU / 512MB RAM / 4GB Disk |
| paas-ta-portal-webuser |portal_small| 1vCPU / 512MB RAM / 4GB Disk|

### 1.4. 참고자료
[**http://bosh.io/docs**](http://bosh.io/docs)  
[**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/)

# 2. PaaS-TA Portal 설치

### 2.1. 설치전 준비사항

본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다.
서비스팩 설치를 위해서는 먼저 BOSH CLI v2 가 설치 되어 있어야 하고 BOSH 에 로그인이 되어 있어야 한다.<br>
BOSH CLI v2 가 설치 되어 있지 않을 경우 먼저 BOSH2.0 설치 가이드 문서를 참고 하여 BOSH CLI v2를 설치를 하고 사용법을 숙지 해야 한다.<br>

- BOSH2.0 사용자 가이드
>BOSH2 사용자 가이드 : **<https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/v3.5/Use-Guide/Bosh/PaaS-TA_BOSH2_%EC%82%AC%EC%9A%A9%EC%9E%90_%EA%B0%80%EC%9D%B4%EB%93%9Cv1.0.md>**

>BOSH CLI V2 사용자 가이드 : **<https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/v3.5/Use-Guide/Bosh/PaaS-TA_BOSH_CLI_V2_%EC%82%AC%EC%9A%A9%EC%9E%90_%EA%B0%80%EC%9D%B4%EB%93%9Cv1.0.md>**

- PaaS-TA에서 제공하는 압축된 릴리즈 파일들을 다운받는다. (PAASTA-PORTAL.zip)

- 다운로드 위치
>PaaSTA-Portal-Release : **<https://paas-ta.kr/data/packages/3.5/PaaSTA-Releases-Portal.zip>**


### 2.2. PaaS-TA Portal 릴리즈 업로드

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

	    	30 releases

	    	Succeeded

- PaaS-TA Portal 릴리즈가 업로드 되어 있지 않은 것을 확인

- PaaS-TA Portal 릴리즈 파일을 업로드한다.

        릴리즈 파일 위치 : PAAS-TA-PORTAL-RELEASE/releases/paas-ta-portal.tgz
            
- **사용 예시**

		$ bosh -e micro-bosh upload-release paas-ta-portal.tgz
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		######################################################## 100.00% 153.81 MiB/s 3s
        Task 4687
        
        Task 4687 | 02:09:08 | Extracting release: Extracting release (00:00:05)
        Task 4687 | 02:09:13 | Verifying manifest: Verifying manifest (00:00:00)
        Task 4687 | 02:09:14 | Resolving package dependencies: Resolving package dependencies (00:00:00)
        Task 4687 | 02:09:14 | Creating new packages: paas-ta-portal-webadmin/a74db51496832a2b1e1e947c424d08b33fb46c83 (00:00:01)
        Task 4687 | 02:09:15 | Creating new packages: paas-ta-portal-log-api/0aff609e64bd4f3ea2cef82d470c0881933b5bbf (00:00:01)
        Task 4687 | 02:09:16 | Creating new packages: paas-ta-portal-api/02e6c62924be62a2672a662f80bbf8d77c0afdb1 (00:00:01)
        Task 4687 | 02:09:17 | Creating new packages: paas-ta-portal-storage-api/6cd6d9b3dd2376509ab79ed944c10c3782617db7 (00:00:01)
        Task 4687 | 02:09:18 | Creating new packages: apache2/ced88bee5e97185ff8d299e2892975bd2062fa01 (00:00:00)
        Task 4687 | 02:09:18 | Creating new packages: python/4e255efa754d91b825476b57e111345f200944e1 (00:00:02)
        Task 4687 | 02:09:20 | Creating new packages: haproxy/14b0441f6d68c89612f53ce4334a65c80d601e51 (00:00:00)
        Task 4687 | 02:09:20 | Creating new packages: paas-ta-portal-registration/66ed72c68e86717bf53af02f07e85fa19d7da874 (00:00:01)
        Task 4687 | 02:09:21 | Creating new packages: sshpass/1b75eb8cd625da33e67fe33deba7b675796adb83 (00:00:00)
        Task 4687 | 02:09:21 | Creating new packages: paas-ta-portal-config-server/8064c9e7488f2f688bc259d8cb8b15f0894faa94 (00:00:00)
        Task 4687 | 02:09:21 | Creating new packages: paas-ta-portal-infra-admin/69c651f70ca9ecfa305d60ef6c2cec36ae6d8fb6 (00:00:01)
        Task 4687 | 02:09:22 | Creating new packages: tomcat/1ff8747498f45f21b2d43d6bc50d8e53a0ddfca7 (00:00:01)
        Task 4687 | 02:09:23 | Creating new packages: swift-all-in-one/af76e98d69570ca8c886b3c7fcb23c3a1ccbbe40 (00:00:05)
        Task 4687 | 02:09:28 | Creating new packages: paas-ta-portal-webuser/e3901a2d9e9cd349a06fee226bc3230e1c91b430 (00:00:02)
        Task 4687 | 02:09:30 | Creating new packages: paas-ta-portal-common-api/8d7f3eb353ec32adf744d4514e95a936c2ea1ce3 (00:00:02)
        Task 4687 | 02:09:32 | Creating new packages: java/86d8b8f8115418addf836753c1735abe547d4105 (00:00:03)
        Task 4687 | 02:09:35 | Creating new packages: mariadb/59a218308c6c7dcf8795b531b53aa4a1c666ce00 (00:00:23)
        Task 4687 | 02:09:58 | Creating new packages: paas-ta-portal-gateway/1fc98dd004de2dd036ed8c69fe9643efa4d74f5a (00:00:00)
        Task 4687 | 02:09:58 | Creating new jobs: paas-ta-portal-webadmin/233eb71833ed12faef66b4ef1b298bee0f6f10d2 (00:00:01)
        Task 4687 | 02:09:59 | Creating new jobs: paas-ta-portal-log-api/3a5acb369f0491a48a3c7f8c2fdcddf469a0c2e8 (00:00:00)
        Task 4687 | 02:09:59 | Creating new jobs: paas-ta-portal-api/d047be88ad2335b7981b7bf3e8a1396176ec04eb (00:00:00)
        Task 4687 | 02:09:59 | Creating new jobs: paas-ta-portal-storage-api/46783767f597931024cd2ae3e9d4f71f08a5a71b (00:00:00)
        Task 4687 | 02:09:59 | Creating new jobs: haproxy/6fb2bf3eefc2ec935bbb6a1d05ed92ba66ea7988 (00:00:00)
        Task 4687 | 02:09:59 | Creating new jobs: paas-ta-portal-registration/dc5256799b47b1e0abab0d55dbbed8dabbd4997b (00:00:00)
        Task 4687 | 02:09:59 | Creating new jobs: paas-ta-portal-infra-admin/6d09e520e3cb89e443c13b0a77f700cceb46b37a (00:00:00)
        Task 4687 | 02:09:59 | Creating new jobs: paas-ta-portal-webuser/e866e3a0fb36020dd8f207a0db5f7bd22be18d2b (00:00:00)
        Task 4687 | 02:09:59 | Creating new jobs: binary_storage/8315c60cb8259e61fb0df4742e72cb2093dd32e4 (00:00:00)
        Task 4687 | 02:09:59 | Creating new jobs: paas-ta-portal-common-api/e9ce476a7e453bb54a1581fbfaa09f669dbe39f0 (00:00:00)
        Task 4687 | 02:09:59 | Creating new jobs: mariadb/3810edf74d908d7fb1cd284ce69a172b5fb51225 (00:00:01)
        Task 4687 | 02:10:00 | Creating new jobs: paas-ta-portal-gateway/5950348e71002c4e7e511e8218e645f35cc887be (00:00:00)
        Task 4687 | 02:10:00 | Release has been created: paas-ta-portal-release/2.0 (00:00:00)
        
        Task 4687 Started  Mon Sep  3 02:09:08 UTC 2018
        Task 4687 Finished Mon Sep  3 02:10:00 UTC 2018
        Task 4687 Duration 00:00:52
        Task 4687 done
        
        Succeeded

-	PaaS-TA Portal 릴리즈를 확인한다.

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
		paas-ta-portal-release            2.0*      00000000  
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
		
-	PaaS-TA Portal 릴리즈가 업로드 되어 있는 것을 확인

-	Deploy시 사용할 Stemcell을 확인한다.

- **사용 예시**

		$ bosh -e micro-bosh stemcells
		Name                                      Version   OS             CPI  CID  
		bosh-vsphere-esxi-ubuntu-trusty-go_agent  3586.26*  ubuntu-trusty  -    sc-109fbdb0-f663-49e8-9c30-8dbdd2e5b9b9
		~                                         3468.51*  ubuntu-trusty  -    sc-bd8fadf2-92f2-495b-b172-d571a388abf5  
		~                                         3445.2*   ubuntu-trusty  -    sc-025c70b5-7d6e-4ba3-a12b-7e71c33dad24
		~                                         3309*     ubuntu-trusty  -    sc-22429dba-e5cc-4469-ab3a-882091573277  

		(*) Currently deployed

		4 stemcells

		Succeeded
		
>Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드를 해야 한다. (Paas-TA Portal 은 stemcell 3468.51 버전을 사용)

### 2.3. PaaS-TA Portal Deployment 배포

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
        - cloud_properties:
            cpu: 1
            disk: 4096
            ram: 512
          name: portal_small
        - cloud_properties:
            cpu: 1
            disk: 4096
            ram: 1024
          name: portal_medium
        - cloud_properties:
            cpu: 1
            disk: 4096
            ram: 2048
          name: portal_large


		Succeeded


-  Deployment 파일을 서버 환경에 맞게 수정한다.
-  azs의 경우 z5 ~ z6 로 설정한다.
-  "(())" 구문은 bosh deploy 할 때 변수로 받아서 처리하는 구문이므로 이 부분의 수정 방법은 아래의 deploy-portal-bosh2.0.sh 참고 예) os : ((stemcell_os))
```diff
-  Object Storage 경우 stemcell 버전을 3468.51이상의 버전에선 지원하지 않음.
```
 
```yml
# paas-ta-portal-bosh2.0.yml 설정 파일 내용
---
name: paasta-portal                      # 서비스 배포이름(필수) bosh deployments 로 확인 가능한 이름

stemcells:
- alias: ((stemcell_alias))
  os: ((stemcell_os))
  version: ((stemcell_version))

releases:
- name: paas-ta-portal-release                   # 서비스 릴리즈 이름(필수) bosh releases로 확인 가능
  version: ((release_version))                                              # 서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전

update:
  canaries: 1                                               # canary 인스턴스 수(필수)
  canary_watch_time: 5000-120000                            # canary 인스턴스가 수행하기 위한 대기 시간(필수)
  update_watch_time: 5000-120000                            # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)
  max_in_flight: 1                                          # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)
  serial: false

instance_groups:
########## INFRA ##########
- name: mariadb
  azs:
  - z6
  instances: 1
  vm_type: portal_large
  stemcell: "((stemcell_alias))"
  persistent_disk_type: "((mariadb_disk_type))"
  networks:
  - name: ((internal_networks_name))
    static_ips:
    - "((mariadb_ips))"
  templates:
  - name: mariadb
    release: paas-ta-portal-release
  syslog_aggregator: null

- name: binary_storage
  azs:
  - z5
  instances: 1
  persistent_disk_type: "((binary_storage_disk_type))"
  vm_type: portal_large
  stemcell: "((stemcell_alias))"
  networks:
  - name: ((internal_networks_name))
    static_ips:
    - "((binary_storage_ips))"
  templates:
  - name: binary_storage
    release: paas-ta-portal-release
  syslog_aggregator: null


- name: haproxy
  azs:
  - z5
  instances: 1
  vm_type: portal_large
  stemcell: "((stemcell_alias))"
  networks:
  - name: ((internal_networks_name))
    static_ips:
    - "((haproxy_ips))"
  - name: ((external_networks_name))
    default: [dns, gateway]
    static_ips: "((haproxy_static_ips))"
  templates:
  - name: haproxy
    release: paas-ta-portal-release
  syslog_aggregator: null

######## WEB SERVICE ########

- name: paas-ta-portal-gateway
  azs:
  - z3
  instances: 1
  vm_type: portal_medium
  stemcell: "((stemcell_alias))"
  networks:
  - name: ((internal_networks_name))
    static_ips:
    - "((portal_gateway_ips))"
  templates:
  - name: paas-ta-portal-gateway
    release: paas-ta-portal-release
  syslog_aggregator: null
  properties:
    eureka:
      client:
        serviceUrl:
          defaultZone: http://((portal_registration_ips))

- name: paas-ta-portal-registration
  azs:
  - z5
  instances: 1
  vm_type: portal_small
  stemcell: "((stemcell_alias))"
  networks:
  - name: ((internal_networks_name))
    static_ips:
    - "((portal_registration_ips))"
  templates:
  - name: paas-ta-portal-registration
    release: paas-ta-portal-release
  syslog_aggregator: null
  properties:
    java_opts: "-Xms256m -Xmx512m -XX:ReservedCodeCacheSize=240m -XX:+UseCompressedOops -Dfile.encoding=UTF-8 -XX:+UseConcMarkSweepGC -XX:SoftRefLRUPolicyMSPerMB=50 -Dsun.io.useCanonCaches=false -Djava.net.preferIPv4Stack=true -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Xverify:none -XX:ErrorFile=/var/vcap/sys/log/java_error_in_idea_%p.log -XX:HeapDumpPath=/var/vcap/sys/log/java_error_in_idea.hprof"
    infra:
      admin:
        enable: false
    server:
      port: 2221

- name: paas-ta-portal-api
  azs:
  - z5
  instances: 1
  vm_type: portal_large
  stemcell: "((stemcell_alias))"
  networks:
  - name: ((internal_networks_name))
    static_ips:
    - "((portal_api_ips))"
  templates:
  - name: paas-ta-portal-api
    release: paas-ta-portal-release
  syslog_aggregator: null
  properties:
    cloudfoundry:
      cc:
        api:
          url: ((cf_api_url))
          uaaUrl: ((cf_uaa_url))
          sslSkipValidation: true
      user: # CloudFoundry Login information
        admin:
          username: admin
          password: "((cf_admin_password))"
        uaaClient:
          clientId: admin
          clientSecret: ((cf_uaa_admin_client_secret))
          adminClientId: admin
          adminClientSecret: ((cf_uaa_admin_client_secret))
          loginClientId: admin
          loginClientSecret: ((cf_uaa_admin_client_secret))
          skipSSLValidation: true
      authorization: cf-Authorization
    abacus:
      url: ((abacus_url))
    monitoring:
      api:
        url: ((monitoring_api_url))
    paasta:
      api:
        portal:
          zuul:
            url: http://((portal_gateway_ips))
    eureka:
      client:
        serviceUrl:
          defaultZone: http://((portal_registration_ips))

- name: paas-ta-portal-log-api
  azs:
  - z5
  instances: 1
  vm_type: portal_medium
  stemcell: "((stemcell_alias))"
  networks:
  - name: ((internal_networks_name))
    static_ips:
    - "((portal_log_ips))"
  templates:
  - name: paas-ta-portal-log-api
    release: paas-ta-portal-release
  syslog_aggregator: null
  properties:
    cloudfoundry:
      cc:
        api:
          url: ((cf_api_url))
          uaaUrl: ((cf_uaa_url))
          sslSkipValidation: true
      user: # CloudFoundry Login information
        admin:
          username: admin
          password: "((cf_admin_password))"
        uaaClient:
          clientId: login
          clientSecret: login-secret
          adminClientId: admin
          adminClientSecret: "((cf_uaa_admin_client_secret))"
          loginClientId: login
          loginClientSecret: login-secret
          skipSSLValidation: true
      authorization: cf-Authorization
    paasta:
      api:
        portal:
          zuul:
            url: http://((portal_gateway_ips))
    eureka:
      client:
        serviceUrl:
          defaultZone: http://((portal_registration_ips))

- name: paas-ta-portal-common-api
  azs:
  - z5
  instances: 1
  vm_type: portal_medium
  stemcell: "((stemcell_alias))"
  networks:
  - name: ((internal_networks_name))
    static_ips:
    - ((portal_common_ips))
  templates:
  - name: paas-ta-portal-common-api
    release: paas-ta-portal-release
  syslog_aggregator: null
  properties:
    datasource:
      cc:
        driver-class-name: org.postgresql.Driver
        url: jdbc:postgresql://((cf_db_ips)):((cf_db_port))/cloud_controller
        username: "((cc_db_id))"
        password: "((cc_db_password))"
      portal:
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://((mariadb_ips)):((mariadb_port))/portaldb
        username: root
        password: "((mariadb_user_password))"
      uaa:
        driver-class-name: org.postgresql.Driver
        url: jdbc:postgresql://((cf_db_ips)):((cf_db_port))/uaa
        username: "((uaa_db_id))"
        password: "((uaa_db_password))"
    mail:
      smtp:
        host: ((mail_smtp_host))
        port: ((mail_smtp_port))
        username: ((mail_smtp_username))
        password: "((mail_smtp_password))"
        useremail: ((mail_smtp_useremail))
        properties:
          auth: ((mail_smtp_properties_auth))
          starttls:
            enable: ((mail_smtp_properties_starttls_enable))
            required: ((mail_smtp_properties_starttls_required))
          maximumTotalQps: 90
          authUrl: ((paas_ta_web_user_url))
          charset: UTF-8
          subject: "((mail_smtp_properties_subject))"
          createUrl: authcreate
          expiredUrl: authreset
          inviteUrl: inviteorg
    paasta:
      api:
        portal:
          zuul:
            url: http://((portal_gateway_ips))
    eureka:
      client:
        serviceUrl:
          defaultZone: http://((portal_registration_ips))

- name: paas-ta-portal-storage-api
  azs:
  - z5
  instances: 1
  vm_type: portal_medium
  stemcell: "((stemcell_alias))"
  networks:
  - name: ((internal_networks_name))
    static_ips:
    - "((portal_storage_api_ips))"
  templates:
  - name: paas-ta-portal-storage-api
    release: paas-ta-portal-release
  syslog_aggregator: null
  properties:
    objectStorage:
      swift:
        tenantName: ((binary_storage_tenantname))
        username: ((binary_storage_username))
        password: ((binary_storage_password))
        authUrl: http://((binary_storage_ips)):5000/v2.0/tokens
        authMethod: keystone
        preferredRegion: Public
        container: portal-container
    eureka:
      client:
        serviceUrl:
          defaultZone: http://((portal_registration_ips))

- name: paas-ta-portal-webadmin
  azs:
  - z5
  instances: 1
  vm_type: portal_medium
  stemcell: "((stemcell_alias))"
  networks:
  - name: ((internal_networks_name))
    static_ips:
    - "((portal_webadmin_ips))"
  templates:
  - name: paas-ta-portal-webadmin
    release: paas-ta-portal-release
  syslog_aggregator: null
  properties:
    paasta:
      api:
        portal:
          zuul:
            url: http://((portal_gateway_ips))
    eureka:
      client:
        serviceUrl:
          defaultZone: http://((portal_registration_ips))

- name: paas-ta-portal-webuser
  azs:
  - z5
  instances: 1
  vm_type: portal_medium
  stemcell: "((stemcell_alias))"
  networks:
  - name: ((internal_networks_name))
    static_ips:
    - "((portal_webuser_ips))"
  templates:
  - name: paas-ta-portal-webuser
    release: paas-ta-portal-release
  syslog_aggregator: null
  properties:
    gatewayserver:
      ip: http://((portal_gateway_ips))
    portallogapi:
      ip: ws://((portal_log_ips))
    logPath: "/var/vcap/sys/log/paas-ta-portal-webuser"   # WEBUSER는 아파치를 사용함, APACHE 로그 위치
    webDir: "/var/vcap/packages/apache2/htdocs"           # WEBUSER는 아파치를 사용함, APACHE 웹 디렉토리 설정
    cf:
      uaa:
        url: ((cf_uaa_url))
        clientsecret: ((portal_client_secret))
        logouturl: ((cf_uaa_logouturl))
    monitoring: ((portal_webuser_monitoring))
    quantity: ((portal_webuser_quantity))
    automaticApproval: ((portal_webuser_automaticapproval))

######### COMMON PROPERTIES ##########
properties:
  mariadb:                                                # MARIA DB SERVER 설정 정보
    port: ((mariadb_port))                                            # MARIA DB PORT 번호
    admin_user:
      password: '((mariadb_user_password))'                             # MARIA DB ROOT 계정 비밀번호
    host: ((mariadb_ips))                                     # MARIA DB IP 주소
    host_names:
    - mariadb0
    host_ips:
    - ((mariadb_ips))                                   # MARIA DB IP 주소
    haproxy:                                              # Haproxy를 이용하여, 외부에서 접근하기 위해서는 설정해야함
      urls:
      - ((mariadb_ips))

  binary_storage:                                         # BINARY STORAGE SERVER 설정 정보
    proxy_ip: ((binary_storage_ips))                    # 프록시 서버 IP(swift-keystone job의 static_ip, Object Storage 접속 IP)
    proxy_port: 10008                                     # 프록시 서버 Port(Object Storage 접속 Port)
    auth_port: 5000
    username:                                             # 최초 생성되는 유저이름(Object Storage 접속 유저이름)
    - ((binary_storage_username))
    password:                                             # 최초 생성되는 유저 비밀번호(Object Storage 접속 유저 비밀번호)
    - ((binary_storage_password))
    tenantname:                                           # 최초 생성되는 테넌트 이름(Object Storage 접속 테넌트 이름)
    - ((binary_storage_tenantname))
    email:                                                # 최소 생성되는 유저의 이메일
    - ((binary_storage_email))
    container:                                            # 최초 생성되는 컨테이너 이름
    - portal-container
    binary_desc:                                          # 최초 생성되는 컨테이너에 대한 설명
    - "portal binary_storage"


  infradmin: # Haproxy를 이용하여, 외부에서 접근하기 위해서는 설정해야함
    ip: "((portal_infra_admin_ips))"
  eurekaserver: # Haproxy를 이용하여, 외부에서 접근하기 위해서는 설정해야함
    ip: "((portal_registration_ips))"
  gatewayserver: # Haproxy를 이용하여, 외부에서 접근하기 위해서는 설정해야함
    ip: "((portal_gateway_ips))"
  webadmin: # Haproxy를 이용하여, 외부에서 접근하기 위해서는 설정해야함
    ip: "((portal_webadmin_ips))"
  webuser: # Haproxy를 이용하여, 외부에서 접근하기 위해서는 설정해야함
    ip: "((portal_webuser_ips))"
```
>현재 기본으로 제공된 release는 infra-admin은 비활성화 상태다. 활성화 하려면 instance_group의 infra-admin 설정부분앞의 #을 제거하고 paas-ta-portal-registration의 infra admin enable을 true로 바꿔야한다.

-	deploy-portal-bosh2.0.sh 파일을 서버 환경에 맞게 수정한다.
        - bosh 명령문 후에 주석(#)을 사용할경우 오류가 발생한다. 
```sh
기본 명령어 : bosh -e micro-bosh -d [deployment name] [deploy.yml]

#!/bin/bash
# stemcell 버전은 3468.51 버전으로 사용하십시요.
# vsphere 인 경우 에는 use-public-network-vsphere.yml 사용하여 public ip를 설정 하고 그 이외의 IaaS는 use-public-network.yml 사용한다.

bosh -e micro-bosh -d paas-ta-portal-v2 deploy paasta-portal-bosh2.0.yml \ 
   -v stemcell_os="ubuntu-trusty"\                                          Stemcell_os
   -v stemcell_version="3468.51"\                                           Stemcell version
   -v stemcell_alias="default"\                                             Stemcell_alias
   -v vm_type_small="portal_small"\                                         small vm type (512MB)
   -v vm_type_medium="portal_medium"\                                       medium vm type (1GB)
   -v vm_type_large="portal_large"\                                         large vm type (2GB)
   -v internal_networks_name="service_private"\                             Private ip 네트워크 이름(cloud config 참고) 
   -v external_networks_name="portal_service_public"\                       Public ip 네트워크 이름(cloud config 참고)
   -v mariadb_ips="10.30.107.211"\                                          Mariadb ip
   -v mariadb_disk_type="10GB"\                                             Mariadb disk
   -v mariadb_port="3306"\                                                  Mariadb port
   -v mariadb_user_password="xxxxxxxxxxx"\                                  Mariadb paasword, id는 root로 통일
   -v binary_storage_ips="10.30.107.212"\                                   Binary_storage ip
   -v binary_storage_disk_type="10GB"\                                      Binary_storage disk
   -v binary_storage_username="paasta-portal"\                              Binary_storage user name
   -v binary_storage_password="paasta"\                                     Binary_storage password
   -v binary_storage_tenantname="paasta-portal"\                            Binary_storage tenantname
   -v binary_storage_email="paasta@paasta.com"\                             Binary_storage email
   -v haproxy_ips="10.30.107.213"\                                          Proxy private ip
   -v haproxy_static_ips="115.68.46.214"\                                   Proxy public ip
   -v portal_gateway_ips="10.30.107.214"\                                   Portal_gateway ip
   -v portal_registration_ips="10.30.107.215"\                              Portal_registration ip
   -v portal_infra_admin_ips="10.30.107.216"\                               Infra_admin ip
   -v portal_api_ips="10.30.107.217"\                                       Portal_api ip
   -v portal_log_ips="10.30.107.218"\                                       Portal_log ip
   -v portal_common_ips="10.30.107.219"\                                    Portal_common ip
   -v portal_storage_api_ips="10.30.107.220"\                               Portal_storage ip
   -v portal_webadmin_ips="10.30.107.221"\                                  Portal_webadmin ip
   -v portal_webuser_ips="10.30.107.222"\                                   Portal_webuser ip
   -v cf_db_ips="10.30.112.4"\                                              CF_database ip
   -v cf_db_port="5524"\                                                    CF_database port
   -v cc_db_id="cloud_controller"\                                          CF_database id
   -v cc_db_password="cc_admin"\                                            CF_database password
   -v uaa_db_id="uaa"\                                                      UAA_database id
   -v uaa_db_password="uaa_admin"\                                          UAA_database password
   -v cf_uaa_url="https://uaa.115.68.46.189.xip.io"\                        UAA_페이지 URL
   -v cf_uaa_logouturl="logout.do"\                                         UAA logout URL
   -v cf_api_url="https://api.115.68.46.189.xip.io"\                        CF_api url
   -v cf_admin_password="admin_test"\                                       CF_admin password, id는 admin으로 통일
   -v cf_uaa_admin_client_secret="admin-secret"\                            UAA admin client secret
   -v portal_client_secret="portalclient"\                                  Portal client secret
   -v paas_ta_web_user_url="http://portal-web-user.115.68.46.214.xip.io"\   User_portal url 설정
   -v abacus_url="http://monitoring.115.68.46.214"\                         Abacus_url
   -v portal_webuser_quantity=false\                                        사용량 조회 사용 여부
   -v monitoring_api_url="http://abacus.115.68.46.214"\                     Monitoring_api url
   -v portal_webuser_monitoring=false\                                      Monitoring 사용 여부
   -v mail_smtp_host="smtp.gmail.com"\                                      Mail_smtp host
   -v mail_smtp_port="465"\                                                 Mail_smtp port
   -v mail_smtp_username="PaaS-TA"\                                         Mail_smtp username
   -v mail_smtp_password="xxxxxxxxxxx"\                                     Mail_smtp password
   -v mail_smtp_useremail="xxxxxxxxx@gmail.com"\                            Mail_smtp user email
   -v mail_smtp_properties_auth="true"\                                     Mail_smtp auth
   -v mail_smtp_properties_starttls_enable="true"\                          Mail_smtp host enable
   -v mail_smtp_properties_starttls_required="true"\                        Mail_smtp starttls required
   -v mail_smtp_properties_subject="PaaS-TA User Potal"\                    Mail_smtp subject(메일 제목)
   -v portal_webuser_automaticapproval=false                                자동 가입 승인 여부
```
> release_version : 릴리즈 버전을 입력한다. $bosh releases 명령문으로 확인가능
 
    - $ bosh releases
      Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)
      Name                              Version   Commit Hash  
      paas-ta-portal-release             2.0*      00000000  

> stemcell_os : 스템셀 OS를 입력한다. $bosh stemcells 명령문으로 확인가능\
> stemcell_version : 스템셀 버전을 입력한다. $bosh stemcells 명령문으로 확인가능(3468.51 verion 사용)\
> stemcell_alias : bosh deploy시 사용할 스템셀 명칭을 정한다.

    - $ bosh stemcells
      Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)
      Name                                      Version   OS             CPI  CID  
      bosh-vsphere-esxi-ubuntu-trusty-go_agent  3586.26*  ubuntu-trusty  -    sc-109fbdb0-f663-49e8-9c30-8dbdd2e5b9b9
      ~                                         3468.51*  ubuntu-trusty  -    sc-bd8fadf2-92f2-495b-b172-d571a388abf5  
      ~                                         3445.2*   ubuntu-trusty  -    sc-025c70b5-7d6e-4ba3-a12b-7e71c33dad24  
      ~                                         3309*     ubuntu-trusty  -    sc-22429dba-e5cc-4469-ab3a-882091573277  
      (*) Currently deployed
      4 stemcells
      
> internal_networks_name : 내부 ip 할당할 network name $ bosh -e micro-bosh cloud-config로 확인가능\
> external_networks_name : 외부 ip 할당할 network name $ bosh -e micro-bosh cloud-config로 확인가능
      
      $ bosh -e micro-bosh cloud-config
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

> mariadb_ips: Mariadb의 ip 할당, internal_networks_name에 할당된 ip를 사용해야한다.\
  mariadb_disk_type: Mariadb의 persistent_disk용량을 정한다.\
  mariadb_port: Mariadb의 port를 정한다.\
  mariadb_user_password: Mariadb의 비밀번호를 정한다. id: root\
  binary_storage_ips: Binary Storage의 ip 할당, internal_networks_name에 할당된 ip를 사용해야한다.\
  binary_storage_disk_type: Binary Storage의 persistent_disk용량을 정한다.\
  binary_storage_username: Binary Storage의 접속 유저 계정을 정한다.\
  binary_storage_password: Binary Storage의 접속 유저의 비밀번호를 정한다.\
  binary_storage_tenantname: Binary Storage의 접속 테넌트 계정을 정한다.\
  binary_storage_email: Binary Storage 생성되는 유저의 이메일을 정한다.\
  haproxy_ips: Haproxy의 ip 할당, internal_networks_name에 할당된 ip를 사용해야한다.\
  haproxy_static_ips: Haproxy의 ip 할당, external_networks_name에 할당된 ip를 사용해야한다.\
  portal_gateway_ips: Gateway Api의 ip 할당, internal_networks_name에 할당된 ip를 사용해야한다.\
  portal_registration_ips: Registration Api의 ip 할당, internal_networks_name에 할당된 ip를 사용해야한다.\
  portal_infra_admin_ips: Infra Admin의 ip 할당, internal_networks_name에 할당된 ip를 사용해야한다.\
  portal_api_ips: Portal Api의 ip 할당, internal_networks_name에 할당된 ip를 사용해야한다.\
  portal_log_ips: Haproxy의 ip 할당, internal_networks_name에 할당된 ip를 사용해야한다.\
  portal_common_ips: Haproxy의 ip 할당, internal_networks_name에 할당된 ip를 사용해야한다.\
  portal_storage_api_ips: Haproxy의 ip 할당, internal_networks_name에 할당된 ip를 사용해야한다.\
  portal_webadmin_ips: Haproxy의 ip 할당, internal_networks_name에 할당된 ip를 사용해야한다.\
  portal_webuser_ips: Haproxy의 ip 할당, internal_networks_name에 할당된 ip를 사용해야한다.\


>cf_db_ips: CF Database의 ip를 입력한다.\
 cf_db_port: CF Database의 port를 입력한다.\
 cc_db_id: CF Database의 계정을 입력한다.\
 cc_db_password: CF Database의 비밀번호를 입력한다.\
 uaa_db_id: UAA Database의 계정을 입력한다.\
 uaa_db_password: UAA Database의 비밀번호를 입력한다.
![paas-ta-portal-09]
>>cf_db_ips: api, uaa가 포함된 deployment내의 database ip주소
 
> cf_uaa_url: "https://uaa.[uaa가 포함된 public ip].xip.io"\
  cf_api_url: "https://api.[api가 포함된 public ip].xip.io"
![paas-ta-portal-07]
>>예) cf_uaa_url="https://uaa.115.68.46.189.xip.io"

> cf_admin_password: CF 관리자 계정 비밀번호를 입력한다.

>cf_uaa_admin_client_secret: uaac admin client의 secret를 입력한다.\
 portal_client_secret: uaac portalclient의 secret를 입력한다.\
 
>paas_ta_web_user_url: Portal Webuser의 Url을 입력한다.
 abacus_url= Abacus Url을 입력한다.
 monitoring_api_url: Monitoring Api의 Url을 입력한다.
 
>mail_smtp_host: smtp의 host를 설정한다.\
 mail_smtp_port: smtp의 port를 설정한다.\
 mail_smtp_username: smtp의 계정을 설정한다.\
 mail_smtp_password: smtp의 비밀번호을 설정한다.\
 mail_smtp_useremail: smtp의 email을 설정한다.\
 mail_smtp_properties_auth=: smtp의 auth값을 설정한다.\
 mail_smtp_properties_starttls_enable: smtp의 starttls_enable값을 설정한다.\
 mail_smtp_properties_starttls_required: smtp의 starttls_required값을 설정한다.\
 mail_smtp_properties_subject: 메일제목을 설정한다.
 
 -	PaaS-TA Portal을 배포한다.

- **사용 예시**

		$ ./deploy-vsphere.sh
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)
        
        Using deployment 'paas-ta-portal-v2'
        
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
        +   version: '3468.51'
          
        + releases:
        + - name: paas-ta-portal-release
        +   version: '2.0'
          
        + update:
        +   canaries: 1
        +   canary_watch_time: 5000-120000
        +   max_in_flight: 1
        +   serial: false
        +   update_watch_time: 5000-120000
          
        + instance_groups:
        + - azs:
        +   - z3
        +   instances: 1
        +   name: mariadb
        +   networks:
        +   - name: service_private
        +     static_ips:
        +     - 10.30.107.211
        +   persistent_disk_type: 10GB
        +   stemcell: default
        +   syslog_aggregator: 
        +   templates:
        +   - name: mariadb
        +     release: paas-ta-portal-release
        +   vm_type: portal_large
        + - azs:
        +   - z5
        +   instances: 1
        +   name: binary_storage
        +   networks:
        +   - name: service_private
        +     static_ips:
        +     - 10.30.107.212
        +   persistent_disk_type: 10GB
        +   stemcell: default
        +   syslog_aggregator: 
        +   templates:
        +   - name: binary_storage
        +     release: paas-ta-portal-release
        +   vm_type: portal_large
        + - azs:
        +   - z5
        +   instances: 1
        +   name: haproxy
        +   networks:
        +   - name: service_private
        +     static_ips:
        +     - 10.30.107.213
        +   - default:
        +     - dns
        +     - gateway
        +     name: portal_service_public
        +     static_ips: 115.68.46.214
        +   stemcell: default
        +   syslog_aggregator: 
        +   templates:
        +   - name: haproxy
        +     release: paas-ta-portal-release
        +   vm_type: portal_large
        + - azs:
        +   - z5
        +   instances: 1
        +   name: paas-ta-portal-gateway
        +   networks:
        +   - name: service_private
        +     static_ips:
        +     - 10.30.107.214
        +   properties:
        +     eureka:
        +       client:
        +         serviceUrl:
        +           defaultZone: "<redacted>"
        +   stemcell: default
        +   syslog_aggregator: 
        +   templates:
        +   - name: paas-ta-portal-gateway
        +     release: paas-ta-portal-release
        +   vm_type: portal_medium
        + - azs:
        +   - z5
        +   instances: 1
        +   name: paas-ta-portal-registration
        +   networks:
        +   - name: service_private
        +     static_ips:
        +     - 10.30.107.215
        +   properties:
        +     infra:
        +       admin:
        +         enable: "<redacted>"
        +     java_opts: "<redacted>"
        +     server:
        +       port: "<redacted>"
        +   stemcell: default
        +   syslog_aggregator: 
        +   templates:
        +   - name: paas-ta-portal-registration
        +     release: paas-ta-portal-release
        +   vm_type: portal_small
        + - azs:
        +   - z5
        +   instances: 1
        +   name: paas-ta-portal-api
        +   networks:
        +   - name: service_private
        +     static_ips:
        +     - 10.30.107.217
        +   properties:
        +     abacus:
        +       url: "<redacted>"
        +     cloudfoundry:
        +       authorization: "<redacted>"
        +       cc:
        +         api:
        +           sslSkipValidation: "<redacted>"
        +           uaaUrl: "<redacted>"
        +           url: "<redacted>"
        +       user:
        +         admin:
        +           password: "<redacted>"
        +           username: "<redacted>"
        +         uaaClient:
        +           adminClientId: "<redacted>"
        +           adminClientSecret: "<redacted>"
        +           clientId: "<redacted>"
        +           clientSecret: "<redacted>"
        +           loginClientId: "<redacted>"
        +           loginClientSecret: "<redacted>"
        +           skipSSLValidation: "<redacted>"
        +     eureka:
        +       client:
        +         serviceUrl:
        +           defaultZone: "<redacted>"
        +     monitoring:
        +       api:
        +         url: "<redacted>"
        +     paasta:
        +       api:
        +         portal:
        +           zuul:
        +             url: "<redacted>"
        +   stemcell: default
        +   syslog_aggregator: 
        +   templates:
        +   - name: paas-ta-portal-api
        +     release: paas-ta-portal-release
        +   vm_type: portal_large
        + - azs:
        +   - z5
        +   instances: 1
        +   name: paas-ta-portal-log-api
        +   networks:
        +   - name: service_private
        +     static_ips:
        +     - 10.30.107.218
        +   properties:
        +     cloudfoundry:
        +       authorization: "<redacted>"
        +       cc:
        +         api:
        +           sslSkipValidation: "<redacted>"
        +           uaaUrl: "<redacted>"
        +           url: "<redacted>"
        +       user:
        +         admin:
        +           password: "<redacted>"
        +           username: "<redacted>"
        +         uaaClient:
        +           adminClientId: "<redacted>"
        +           adminClientSecret: "<redacted>"
        +           clientId: "<redacted>"
        +           clientSecret: "<redacted>"
        +           loginClientId: "<redacted>"
        +           loginClientSecret: "<redacted>"
        +           skipSSLValidation: "<redacted>"
        +     eureka:
        +       client:
        +         serviceUrl:
        +           defaultZone: "<redacted>"
        +     paasta:
        +       api:
        +         portal:
        +           zuul:
        +             url: "<redacted>"
        +   stemcell: default
        +   syslog_aggregator: 
        +   templates:
        +   - name: paas-ta-portal-log-api
        +     release: paas-ta-portal-release
        +   vm_type: portal_medium
        + - azs:
        +   - z5
        +   instances: 1
        +   name: paas-ta-portal-common-api
        +   networks:
        +   - name: service_private
        +     static_ips:
        +     - 10.30.107.219
        +   properties:
        +     datasource:
        +       cc:
        +         driver-class-name: "<redacted>"
        +         password: "<redacted>"
        +         url: "<redacted>"
        +         username: "<redacted>"
        +       portal:
        +         driver-class-name: "<redacted>"
        +         password: "<redacted>"
        +         url: "<redacted>"
        +         username: "<redacted>"
        +       uaa:
        +         driver-class-name: "<redacted>"
        +         password: "<redacted>"
        +         url: "<redacted>"
        +         username: "<redacted>"
        +     eureka:
        +       client:
        +         serviceUrl:
        +           defaultZone: "<redacted>"
        +     mail:
        +       smtp:
        +         host: "<redacted>"
        +         password: "<redacted>"
        +         port: "<redacted>"
        +         properties:
        +           auth: "<redacted>"
        +           authUrl: "<redacted>"
        +           charset: "<redacted>"
        +           createUrl: "<redacted>"
        +           expiredUrl: "<redacted>"
        +           inviteUrl: "<redacted>"
        +           maximumTotalQps: "<redacted>"
        +           starttls:
        +             enable: "<redacted>"
        +             required: "<redacted>"
        +           subject: "<redacted>"
        +         useremail: "<redacted>"
        +         username: "<redacted>"
        +     paasta:
        +       api:
        +         portal:
        +           zuul:
        +             url: "<redacted>"
        +   stemcell: default
        +   syslog_aggregator: 
        +   templates:
        +   - name: paas-ta-portal-common-api
        +     release: paas-ta-portal-release
        +   vm_type: portal_medium
        + - azs:
        +   - z5
        +   instances: 1
        +   name: paas-ta-portal-storage-api
        +   networks:
        +   - name: service_private
        +     static_ips:
        +     - 10.30.107.220
        +   properties:
        +     eureka:
        +       client:
        +         serviceUrl:
        +           defaultZone: "<redacted>"
        +     objectStorage:
        +       swift:
        +         authMethod: "<redacted>"
        +         authUrl: "<redacted>"
        +         container: "<redacted>"
        +         password: "<redacted>"
        +         preferredRegion: "<redacted>"
        +         tenantName: "<redacted>"
        +         username: "<redacted>"
        +   stemcell: default
        +   syslog_aggregator: 
        +   templates:
        +   - name: paas-ta-portal-storage-api
        +     release: paas-ta-portal-release
        +   vm_type: portal_medium
        + - azs:
        +   - z5
        +   instances: 1
        +   name: paas-ta-portal-webadmin
        +   networks:
        +   - name: service_private
        +     static_ips:
        +     - 10.30.107.221
        +   properties:
        +     eureka:
        +       client:
        +         serviceUrl:
        +           defaultZone: "<redacted>"
        +     paasta:
        +       api:
        +         portal:
        +           zuul:
        +             url: "<redacted>"
        +   stemcell: default
        +   syslog_aggregator: 
        +   templates:
        +   - name: paas-ta-portal-webadmin
        +     release: paas-ta-portal-release
        +   vm_type: portal_medium
        + - azs:
        +   - z5
        +   instances: 1
        +   name: paas-ta-portal-webuser
        +   networks:
        +   - name: service_private
        +     static_ips:
        +     - 10.30.107.222
        +   properties:
        +     cf:
        +       uaa:
        +         clientsecret: "<redacted>"
        +         url: "<redacted>"
        +     gatewayserver:
        +       ip: "<redacted>"
        +     logPath: "<redacted>"
        +     portallogapi:
        +       ip: "<redacted>"
        +     webDir: "<redacted>"
        +   stemcell: default
        +   syslog_aggregator: 
        +   templates:
        +   - name: paas-ta-portal-webuser
        +     release: paas-ta-portal-release
        +   vm_type: portal_medium
          
        + name: paas-ta-portal-v2
          
        + properties:
        +   binary_storage:
        +     auth_port: "<redacted>"
        +     binary_desc:
        +     - "<redacted>"
        +     container:
        +     - "<redacted>"
        +     email:
        +     - "<redacted>"
        +     password:
        +     - "<redacted>"
        +     proxy_ip: "<redacted>"
        +     proxy_port: "<redacted>"
        +     tenantname:
        +     - "<redacted>"
        +     username:
        +     - "<redacted>"
        +   eurekaserver:
        +     ip: "<redacted>"
        +   gatewayserver:
        +     ip: "<redacted>"
        +   infradmin:
        +     ip: "<redacted>"
        +   mariadb:
        +     admin_user:
        +       password: "<redacted>"
        +     haproxy:
        +       urls:
        +       - "<redacted>"
        +     host: "<redacted>"
        +     host_ips:
        +     - "<redacted>"
        +     host_names:
        +     - "<redacted>"
        +     port: "<redacted>"
        +   webadmin:
        +     ip: "<redacted>"
        +   webuser:
        +     ip: "<redacted>"

		Continue? [yN]: y
        
        Task 4773
        
        Task 4773 | 06:55:36 | Preparing deployment: Preparing deployment (00:00:03)
        Task 4773 | 06:55:43 | Preparing package compilation: Finding packages to compile (00:00:00)
        Task 4773 | 06:55:43 | Compiling packages: paas-ta-portal-storage-api/6cd6d9b3dd2376509ab79ed944c10c3782617db7
        Task 4773 | 06:55:43 | Compiling packages: apache2/ced88bee5e97185ff8d299e2892975bd2062fa01
        Task 4773 | 06:55:43 | Compiling packages: paas-ta-portal-webuser/e3901a2d9e9cd349a06fee226bc3230e1c91b430
        Task 4773 | 06:55:43 | Compiling packages: paas-ta-portal-webadmin/a74db51496832a2b1e1e947c424d08b33fb46c83
        Task 4773 | 06:55:43 | Compiling packages: paas-ta-portal-common-api/8d7f3eb353ec32adf744d4514e95a936c2ea1ce3 (00:02:47)
        Task 4773 | 06:58:30 | Compiling packages: paas-ta-portal-log-api/0aff609e64bd4f3ea2cef82d470c0881933b5bbf
        Task 4773 | 06:58:39 | Compiling packages: paas-ta-portal-webadmin/a74db51496832a2b1e1e947c424d08b33fb46c83 (00:02:56)
        Task 4773 | 06:58:39 | Compiling packages: paas-ta-portal-api/02e6c62924be62a2672a662f80bbf8d77c0afdb1
        Task 4773 | 06:58:40 | Compiling packages: paas-ta-portal-storage-api/6cd6d9b3dd2376509ab79ed944c10c3782617db7 (00:02:57)
        Task 4773 | 06:58:40 | Compiling packages: paas-ta-portal-registration/66ed72c68e86717bf53af02f07e85fa19d7da874
        Task 4773 | 06:58:59 | Compiling packages: paas-ta-portal-webuser/e3901a2d9e9cd349a06fee226bc3230e1c91b430 (00:03:16)
        Task 4773 | 06:58:59 | Compiling packages: paas-ta-portal-gateway/1fc98dd004de2dd036ed8c69fe9643efa4d74f5a
        Task 4773 | 06:59:03 | Compiling packages: paas-ta-portal-log-api/0aff609e64bd4f3ea2cef82d470c0881933b5bbf (00:00:33)
        Task 4773 | 06:59:03 | Compiling packages: haproxy/14b0441f6d68c89612f53ce4334a65c80d601e51
        Task 4773 | 06:59:18 | Compiling packages: paas-ta-portal-registration/66ed72c68e86717bf53af02f07e85fa19d7da874 (00:00:38)
        Task 4773 | 06:59:18 | Compiling packages: java/86d8b8f8115418addf836753c1735abe547d4105
        Task 4773 | 06:59:20 | Compiling packages: paas-ta-portal-api/02e6c62924be62a2672a662f80bbf8d77c0afdb1 (00:00:41)
        Task 4773 | 06:59:20 | Compiling packages: swift-all-in-one/af76e98d69570ca8c886b3c7fcb23c3a1ccbbe40
        Task 4773 | 06:59:31 | Compiling packages: paas-ta-portal-gateway/1fc98dd004de2dd036ed8c69fe9643efa4d74f5a (00:00:32)
        Task 4773 | 06:59:31 | Compiling packages: python/4e255efa754d91b825476b57e111345f200944e1
        Task 4773 | 07:00:07 | Compiling packages: swift-all-in-one/af76e98d69570ca8c886b3c7fcb23c3a1ccbbe40 (00:00:47)
        Task 4773 | 07:00:07 | Compiling packages: mariadb/59a218308c6c7dcf8795b531b53aa4a1c666ce00
        Task 4773 | 07:00:10 | Compiling packages: java/86d8b8f8115418addf836753c1735abe547d4105 (00:00:52)
        Task 4773 | 07:00:17 | Compiling packages: haproxy/14b0441f6d68c89612f53ce4334a65c80d601e51 (00:01:14)
        Task 4773 | 07:01:16 | Compiling packages: mariadb/59a218308c6c7dcf8795b531b53aa4a1c666ce00 (00:01:09)
        Task 4773 | 07:02:22 | Compiling packages: apache2/ced88bee5e97185ff8d299e2892975bd2062fa01 (00:06:39)
        Task 4773 | 07:04:08 | Compiling packages: python/4e255efa754d91b825476b57e111345f200944e1 (00:04:37)
        Task 4773 | 07:05:04 | Creating missing vms: binary_storage/b405f486-e29b-48f1-9595-e624a92aa90f (0)
        Task 4773 | 07:05:04 | Creating missing vms: mariadb/ecb93167-602a-45d7-bcbc-502a3802c1f1 (0)
        Task 4773 | 07:05:04 | Creating missing vms: haproxy/d579d961-90bc-437c-96b8-6c23db2884ca (0)
        Task 4773 | 07:05:04 | Creating missing vms: paas-ta-portal-registration/31137bcb-551c-4261-b03c-e9a12da030c9 (0)
        Task 4773 | 07:05:04 | Creating missing vms: paas-ta-portal-gateway/50671fc0-0e63-4a8a-a3ae-9180da92708f (0)
        Task 4773 | 07:05:04 | Creating missing vms: paas-ta-portal-common-api/8be2c6ae-b9f3-4759-82aa-4b3e677bd421 (0)
        Task 4773 | 07:05:04 | Creating missing vms: paas-ta-portal-storage-api/6b5fc5a9-9066-42e1-839f-e441af45e622 (0)
        Task 4773 | 07:05:04 | Creating missing vms: paas-ta-portal-api/3ece24a3-6355-4a8d-8ef6-5ae0c1fa13ef (0)
        Task 4773 | 07:05:04 | Creating missing vms: paas-ta-portal-log-api/3df3861e-5652-4939-956c-019a7895611a (0)
        Task 4773 | 07:05:04 | Creating missing vms: paas-ta-portal-webadmin/969fcecc-943a-4f50-a00d-3d5f1bc65609 (0)
        Task 4773 | 07:05:04 | Creating missing vms: paas-ta-portal-webuser/07db9f77-faa8-4490-afdf-3287a82a497d (0) (00:03:32)
        Task 4773 | 07:08:36 | Creating missing vms: paas-ta-portal-storage-api/6b5fc5a9-9066-42e1-839f-e441af45e622 (0) (00:03:32)
        Task 4773 | 07:08:38 | Creating missing vms: mariadb/ecb93167-602a-45d7-bcbc-502a3802c1f1 (0) (00:03:34)
        Task 4773 | 07:08:47 | Creating missing vms: paas-ta-portal-common-api/8be2c6ae-b9f3-4759-82aa-4b3e677bd421 (0) (00:03:43)
        Task 4773 | 07:08:54 | Creating missing vms: paas-ta-portal-api/3ece24a3-6355-4a8d-8ef6-5ae0c1fa13ef (0) (00:03:50)
        Task 4773 | 07:08:54 | Creating missing vms: paas-ta-portal-webadmin/969fcecc-943a-4f50-a00d-3d5f1bc65609 (0) (00:03:50)
        Task 4773 | 07:08:55 | Creating missing vms: binary_storage/b405f486-e29b-48f1-9595-e624a92aa90f (0) (00:03:51)
        Task 4773 | 07:08:55 | Creating missing vms: paas-ta-portal-registration/31137bcb-551c-4261-b03c-e9a12da030c9 (0) (00:03:51)
        Task 4773 | 07:08:55 | Creating missing vms: haproxy/d579d961-90bc-437c-96b8-6c23db2884ca (0) (00:03:51)
        Task 4773 | 07:08:56 | Creating missing vms: paas-ta-portal-log-api/3df3861e-5652-4939-956c-019a7895611a (0) (00:03:52)
        Task 4773 | 07:08:57 | Creating missing vms: paas-ta-portal-gateway/50671fc0-0e63-4a8a-a3ae-9180da92708f (0) (00:03:53)
        Task 4773 | 07:09:02 | Updating instance mariadb: mariadb/ecb93167-602a-45d7-bcbc-502a3802c1f1 (0) (canary)
        Task 4773 | 07:09:02 | Updating instance paas-ta-portal-log-api: paas-ta-portal-log-api/3df3861e-5652-4939-956c-019a7895611a (0) (canary)
        Task 4773 | 07:09:02 | Updating instance paas-ta-portal-webuser: paas-ta-portal-webuser/07db9f77-faa8-4490-afdf-3287a82a497d (0) (canary)
        Task 4773 | 07:09:02 | Updating instance paas-ta-portal-api: paas-ta-portal-api/3ece24a3-6355-4a8d-8ef6-5ae0c1fa13ef (0) (canary)
        Task 4773 | 07:09:02 | Updating instance haproxy: haproxy/d579d961-90bc-437c-96b8-6c23db2884ca (0) (canary)
        Task 4773 | 07:09:02 | Updating instance paas-ta-portal-gateway: paas-ta-portal-gateway/50671fc0-0e63-4a8a-a3ae-9180da92708f (0) (canary)
        Task 4773 | 07:09:02 | Updating instance binary_storage: binary_storage/b405f486-e29b-48f1-9595-e624a92aa90f (0) (canary)
        Task 4773 | 07:09:02 | Updating instance paas-ta-portal-common-api: paas-ta-portal-common-api/8be2c6ae-b9f3-4759-82aa-4b3e677bd421 (0) (canary)
        Task 4773 | 07:09:02 | Updating instance paas-ta-portal-storage-api: paas-ta-portal-storage-api/6b5fc5a9-9066-42e1-839f-e441af45e622 (0) (canary)
        Task 4773 | 07:09:02 | Updating instance paas-ta-portal-registration: paas-ta-portal-registration/31137bcb-551c-4261-b03c-e9a12da030c9 (0) (canary)
        Task 4773 | 07:09:02 | Updating instance paas-ta-portal-webadmin: paas-ta-portal-webadmin/969fcecc-943a-4f50-a00d-3d5f1bc65609 (0) (canary)
        Task 4773 | 07:09:53 | Updating instance paas-ta-portal-webuser: paas-ta-portal-webuser/07db9f77-faa8-4490-afdf-3287a82a497d (0) (canary) (00:00:51)
        Task 4773 | 07:09:56 | Updating instance haproxy: haproxy/d579d961-90bc-437c-96b8-6c23db2884ca (0) (canary) (00:00:54)
        Task 4773 | 07:10:00 | Updating instance paas-ta-portal-log-api: paas-ta-portal-log-api/3df3861e-5652-4939-956c-019a7895611a (0) (canary) (00:00:58)
        Task 4773 | 07:10:02 | Updating instance paas-ta-portal-gateway: paas-ta-portal-gateway/50671fc0-0e63-4a8a-a3ae-9180da92708f (0) (canary) (00:01:00)
        Task 4773 | 07:10:02 | Updating instance paas-ta-portal-storage-api: paas-ta-portal-storage-api/6b5fc5a9-9066-42e1-839f-e441af45e622 (0) (canary) (00:01:00)
        Task 4773 | 07:10:03 | Updating instance paas-ta-portal-registration: paas-ta-portal-registration/31137bcb-551c-4261-b03c-e9a12da030c9 (0) (canary) (00:01:01)
        Task 4773 | 07:10:05 | Updating instance paas-ta-portal-common-api: paas-ta-portal-common-api/8be2c6ae-b9f3-4759-82aa-4b3e677bd421 (0) (canary) (00:01:03)
        Task 4773 | 07:10:12 | Updating instance paas-ta-portal-webadmin: paas-ta-portal-webadmin/969fcecc-943a-4f50-a00d-3d5f1bc65609 (0) (canary) (00:01:10)
        Task 4773 | 07:10:16 | Updating instance paas-ta-portal-api: paas-ta-portal-api/3ece24a3-6355-4a8d-8ef6-5ae0c1fa13ef (0) (canary) (00:01:14)
        Task 4773 | 07:12:09 | Updating instance mariadb: mariadb/ecb93167-602a-45d7-bcbc-502a3802c1f1 (0) (canary) (00:03:07)
        Task 4773 | 07:13:13 | Updating instance binary_storage: binary_storage/b405f486-e29b-48f1-9595-e624a92aa90f (0) (canary) (00:04:11)
        
        Task 4773 Started  Mon Sep  3 06:55:36 UTC 2018
        Task 4773 Finished Mon Sep  3 07:13:13 UTC 2018
        Task 4773 Duration 00:17:37
        Task 4773 done
        
        Succeeded



-	배포된 PaaS-TA Portal을 확인한다.

- **사용 예시**

		bosh -e micro-bosh -d paasta-portal vms
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Task 4823. Done
        
        Deployment 'paasta-portal'
        
        Instance                                                          Process State  AZ  IPs            VM CID                                   VM Type        Active  
        binary_storage/9f58a9b7-2a3d-4ee9-8975-7b04b99c0a21               running        z5  10.30.107.212  vm-e65ad396-ce65-4ef0-962d-5c54fa411769  portal_large   true  
        haproxy/8cc2d633-2b43-4f3d-a2e8-72f5279c11d5                      running        z5  10.30.107.213  vm-315bfa1b-9829-46de-a19d-3bd65e9f9ad4  portal_large   true  
                                                                                             115.68.46.214                                                            
        mariadb/117cbf05-b223-4133-bf61-e15f16494e21                      running        z5  10.30.107.211  vm-bc5ae334-12d4-41d4-8411-d9315a96a305  portal_large   true  
        paas-ta-portal-api/48fa0c5a-52eb-4ae8-a7b9-91275615318c           running        z5  10.30.107.217  vm-9d2a1929-0157-4c77-af5e-707ec496ed87  portal_medium  true  
        paas-ta-portal-common-api/060320fa-7f26-4032-a1d9-6a7a41a044a8    running        z5  10.30.107.219  vm-f35e9838-74cf-40e0-9f97-894b53a68d1f  portal_medium  true  
        paas-ta-portal-gateway/6baba810-9a4a-479d-98b2-97e5ba651784       running        z5  10.30.107.214  vm-7ec75160-bf34-442e-b755-778ae7dd3fec  portal_medium  true  
        paas-ta-portal-log-api/a4460008-42b5-4ba0-84ee-fff49fe6c1bd       running        z5  10.30.107.218  vm-9ec0a1b0-09f6-415b-8e23-53af91fd94b8  portal_medium  true  
        paas-ta-portal-registration/3728ed73-451e-4b93-ab9b-c610826c3135  running        z5  10.30.107.215  vm-c4020514-c458-41c6-bcbc-7e0ee1bc6f42  portal_small   true  
        paas-ta-portal-storage-api/2940366a-8294-4509-a9c0-811c8140663a   running        z5  10.30.107.220  vm-79ad6ee1-1bb5-4308-8b71-9ed30418e2c1  portal_medium  true  
        paas-ta-portal-webadmin/8047fcbd-9a98-4b61-b161-0cbb277fa643      running        z5  10.30.107.221  vm-188250fd-e918-4aab-9cbe-7d368852ea8a  portal_medium  true  
        paas-ta-portal-webuser/cb206717-81c9-49ed-a0a8-e6c3b957cb66       running        z5  10.30.107.222  vm-822f68a5-91c8-453a-b9b3-c1bbb388e377  portal_medium  true
        
        11 vms
        
        Succeeded

### 2.4. 사용자의 조직 생성 Flag 활성화
PaaS-TA는 기본적으로 일반 사용자는 조직을 생성할 수 없도록 설정되어 있다. 포털 배포를 위해 조직 및 공간을 생성해야 하고 또 테스트를 구동하기 위해서도 필요하므로 사용자가 조직을 생성할 수 있도록 user_org_creation FLAG를 활성화 한다. FLAG 활성화를 위해서는 PaaS-TA 운영자 계정으로 로그인이 필요하다.

```
$ cf enable-feature-flag user_org_creation
```
```
Setting status of user_org_creation as admin...
OK

Feature user_org_creation Enabled.
```

# 3. PaaS-TA Portal 운영
이전버전에서 사용한 Portal DB를 PaasTA 3.5 Portal DB에 마이그레이션 하는 방법을 설명한다.
### 3.1. DB Migration

##### 1. DB tool을 이용해 기존에 사용한 DB와 Paas-TA 3.5 Portal DB를 연결한다. 
 * 가이드의 DB tool을 이용한 마이그레이션 설명은 navicat을 기준으로 한다.
##### 2. 마이그레이션할 table의 레코드 데이터를 전부 삭제한다.
>![paas-ta-portal-25]
##### 3. Tools - Data Transfer를 클릭해서 마이그레이션 설정창을 띄운다.
>![paas-ta-portal-21]
##### 4. 마이그레이션할 source DB(기존 DB), target DB(Paas-TA 3.5 Portal DB)를 설정한다.
>![paas-ta-portal-20]
##### 5. Option에 들어가 Table Options의 Create tables 옵션에 체크를 해제, Orther Options의 Contiune on error를 체크한 후 next를 누른다.
>![paas-ta-portal-24]
##### 6. 데이터를 이동할 테이블을 설정 후 next를 누른다.
>![paas-ta-portal-22]
##### 7-1. 마이그레이션이 정상적으로 완료된 모습
>![paas-ta-portal-23]
##### 7-2. 마이그레이션 오류난 모습
>![paas-ta-portal-26]
##### 7-3. 기존 DB에 오류난 Paas-TA Portal table의 Design에 맞춰 수정후에 다시 마이그레이션을 진행한다.
  
### 3.2. Log
Paas-TA Portal 각각 Instance의 log를 확인 할 수 있다.
1. 로그를 확인할 Instance에 접근한다.
    > bosh ssh -d [deployment name] [instance name]
       
       Instance                                                          Process State  AZ  IPs            VM CID                                   VM Type        Active  
       binary_storage/9f58a9b7-2a3d-4ee9-8975-7b04b99c0a21               running        z5  10.30.107.212  vm-e65ad396-ce65-4ef0-962d-5c54fa411769  portal_large   true  
       haproxy/8cc2d633-2b43-4f3d-a2e8-72f5279c11d5                      running        z5  10.30.107.213  vm-315bfa1b-9829-46de-a19d-3bd65e9f9ad4  portal_large   true  
                                                                                            115.68.46.214                                                            
       mariadb/117cbf05-b223-4133-bf61-e15f16494e21                      running        z5  10.30.107.211  vm-bc5ae334-12d4-41d4-8411-d9315a96a305  portal_large   true  
       paas-ta-portal-api/48fa0c5a-52eb-4ae8-a7b9-91275615318c           running        z5  10.30.107.217  vm-9d2a1929-0157-4c77-af5e-707ec496ed87  portal_medium  true  
       paas-ta-portal-common-api/060320fa-7f26-4032-a1d9-6a7a41a044a8    running        z5  10.30.107.219  vm-f35e9838-74cf-40e0-9f97-894b53a68d1f  portal_medium  true  
       paas-ta-portal-gateway/6baba810-9a4a-479d-98b2-97e5ba651784       running        z5  10.30.107.214  vm-7ec75160-bf34-442e-b755-778ae7dd3fec  portal_medium  true  
       paas-ta-portal-log-api/a4460008-42b5-4ba0-84ee-fff49fe6c1bd       running        z5  10.30.107.218  vm-9ec0a1b0-09f6-415b-8e23-53af91fd94b8  portal_medium  true  
       paas-ta-portal-registration/3728ed73-451e-4b93-ab9b-c610826c3135  running        z5  10.30.107.215  vm-c4020514-c458-41c6-bcbc-7e0ee1bc6f42  portal_small   true  
       paas-ta-portal-storage-api/2940366a-8294-4509-a9c0-811c8140663a   running        z5  10.30.107.220  vm-79ad6ee1-1bb5-4308-8b71-9ed30418e2c1  portal_medium  true  
       paas-ta-portal-webadmin/8047fcbd-9a98-4b61-b161-0cbb277fa643      running        z5  10.30.107.221  vm-188250fd-e918-4aab-9cbe-7d368852ea8a  portal_medium  true  
       paas-ta-portal-webuser/cb206717-81c9-49ed-a0a8-e6c3b957cb66       running        z5  10.30.107.222  vm-822f68a5-91c8-453a-b9b3-c1bbb388e377  portal_medium  true  
       
       11 vms
       
       Succeeded
       inception@inception:~$ bosh ssh -d paas-ta-portal-v2 paas-ta-portal-api  << instance 접근(bosh ssh) 명령어 입력
       Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)
       
       Using deployment 'paas-ta-portal-v2'
       
       Task 5195. Done
       Unauthorized use is strictly prohibited. All access and activity
       is subject to logging and monitoring.
       Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-92-generic x86_64)
       
        * Documentation:  https://help.ubuntu.com/
       
       The programs included with the Ubuntu system are free software;
       the exact distribution terms for each program are described in the
       individual files in /usr/share/doc/*/copyright.
       
       Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
       applicable law.
       
       Last login: Tue Sep  4 07:11:42 2018 from 10.30.20.28
       To run a command as administrator (user "root"), use "sudo <command>".
       See "man sudo_root" for details.
       
       paas-ta-portal-api/48fa0c5a-52eb-4ae8-a7b9-91275615318c:~$ 

2. 로그파일이 있는 폴더로 이동한다.
    > 위치 : /var/vcap/sys/log/[job name]/
    
         paas-ta-portal-api/48fa0c5a-52eb-4ae8-a7b9-91275615318c:~$ cd /var/vcap/sys/log/paas-ta-portal-api/
         paas-ta-portal-api/48fa0c5a-52eb-4ae8-a7b9-91275615318c:/var/vcap/sys/log/paas-ta-portal-api$ ls
         paas-ta-portal-api.stderr.log  paas-ta-portal-api.stdout.log

3. 로그파일을 열어 내용을 확인한다.
    > vim [job name].stdout.log
        
        예)
        vim paas-ta-portal-api.stdout.log
        2018-09-04 02:08:42.447 ERROR 7268 --- [nio-2222-exec-1] p.p.a.e.GlobalControllerExceptionHandler : Error message : Response : org.springframework.security.web.firewall.FirewalledResponse@298a1dc2
        Occured an exception : 403 Access token denied.
        Caused by...
        org.cloudfoundry.client.lib.CloudFoundryException: 403 Access token denied. (error="access_denied", error_description="Access token denied.")
                at org.cloudfoundry.client.lib.oauth2.OauthClient.createToken(OauthClient.java:114)
                at org.cloudfoundry.client.lib.oauth2.OauthClient.init(OauthClient.java:70)
                at org.cloudfoundry.client.lib.rest.CloudControllerClientImpl.initialize(CloudControllerClientImpl.java:187)
                at org.cloudfoundry.client.lib.rest.CloudControllerClientImpl.<init>(CloudControllerClientImpl.java:163)
                at org.cloudfoundry.client.lib.rest.CloudControllerClientFactory.newCloudController(CloudControllerClientFactory.java:69)
                at org.cloudfoundry.client.lib.CloudFoundryClient.<init>(CloudFoundryClient.java:138)
                at org.cloudfoundry.client.lib.CloudFoundryClient.<init>(CloudFoundryClient.java:102)
                at org.openpaas.paasta.portal.api.service.LoginService.login(LoginService.java:47)
                at org.openpaas.paasta.portal.api.controller.LoginController.login(LoginController.java:51)
                at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
                at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
                at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
                at java.lang.reflect.Method.invoke(Method.java:498)
                at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:205)
                at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:133)
                at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:97)
                at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:827)
                at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:738)
                at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:85)
                at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:967)
                at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:901)
                at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:970)
                at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:872)
                at javax.servlet.http.HttpServlet.service(HttpServlet.java:661)
                at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:846)
                at javax.servlet.http.HttpServlet.service(HttpServlet.java:742)
                at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)
                at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
                at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
                at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
                at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)

### 3.3. 카탈로그 적용
##### 1. Catalog 빌드팩, 서비스팩 추가
Paas-TA Portal 설치 후에 관리자 포탈에서 빌드팩, 서비스팩을 등록해야 사용자 포탈에서 사용이 가능하다.
 
 1. 관리자 포탈에 접속한다.(portal-web-admin.[public ip].xip.io)
    >![paas-ta-portal-15]
 2. 운영관리를 누른다.
    >![paas-ta-portal-16]
 2. 카탈로그 페이지에 들어간다.
    >![paas-ta-portal-17]
 3. 빌드팩, 서비스팩 상세화면에 들어가서 각 항목란에 값을 입력후에 저장을 누른다.
    >![paas-ta-portal-18]
 4. 사용자포탈에서 변경된값이 적용되어있는지 확인한다.
    >![paas-ta-portal-19]    
    
[paas-ta-portal-01]:../../Install-Guide/Portal/images/Paas-TA-Portal_01.png
[paas-ta-portal-02]:../../Install-Guide/Portal/images/Paas-TA-Portal_02.png
[paas-ta-portal-03]:../../Install-Guide/Portal/images/Paas-TA-Portal_03.png
[paas-ta-portal-04]:../../Install-Guide/Portal/images/Paas-TA-Portal_04.png
[paas-ta-portal-05]:../../Install-Guide/Portal/images/Paas-TA-Portal_05.png
[paas-ta-portal-06]:../../Install-Guide/Portal/images/Paas-TA-Portal_06.png
[paas-ta-portal-07]:../../Install-Guide/Portal/images/Paas-TA-Portal_07.png
[paas-ta-portal-08]:../../Install-Guide/Portal/images/Paas-TA-Portal_08.png
[paas-ta-portal-09]:../../Install-Guide/Portal/images/Paas-TA-Portal_09.png
[paas-ta-portal-10]:../../Install-Guide/Portal/images/Paas-TA-Portal_10.png
[paas-ta-portal-11]:../../Install-Guide/Portal/images/Paas-TA-Portal_11.png
[paas-ta-portal-12]:../../Install-Guide/Portal/images/Paas-TA-Portal_12.png
[paas-ta-portal-13]:../../Install-Guide/Portal/images/Paas-TA-Portal_13.png
[paas-ta-portal-14]:../../Install-Guide/Portal/images/Paas-TA-Portal_14.png
[paas-ta-portal-15]:../../Install-Guide/Portal/images/Paas-TA-Portal_15.png
[paas-ta-portal-16]:../../Install-Guide/Portal/images/Paas-TA-Portal_16.png
[paas-ta-portal-17]:../../Install-Guide/Portal/images/Paas-TA-Portal_17.png
[paas-ta-portal-18]:../../Install-Guide/Portal/images/Paas-TA-Portal_18.png
[paas-ta-portal-19]:../../Install-Guide/Portal/images/Paas-TA-Portal_19.png
[paas-ta-portal-20]:../../Install-Guide/Portal/images/Paas-TA-Portal_20.png
[paas-ta-portal-21]:../../Install-Guide/Portal/images/Paas-TA-Portal_21.png
[paas-ta-portal-22]:../../Install-Guide/Portal/images/Paas-TA-Portal_22.png
[paas-ta-portal-23]:../../Install-Guide/Portal/images/Paas-TA-Portal_23.png
[paas-ta-portal-24]:../../Install-Guide/Portal/images/Paas-TA-Portal_24.png
[paas-ta-portal-25]:../../Install-Guide/Portal/images/Paas-TA-Portal_25.png
[paas-ta-portal-26]:../../Install-Guide/Portal/images/Paas-TA-Portal_26.png
