## Table of Contents
1. [문서 개요](#1)
  - 1.1. [목적](#2)
  - 1.2. [범위](#3)
  - 1.3. [시스템 구성도](#4)
  - 1.4. [참고자료](#5)
2. [WEB IDE 설치](#6)
  - 2.1. [설치전 준비사항](#7)
  - 2.2. [WEB-IDE 릴리즈 업로드](#8)
  - 2.3. [WEB-IDE Deployment 파일 수정 및 배포](#9)
3. [WEB IDE 브로커 설치](#10)
  - 3.1. [설치전 준비사항](#11)
  - 3.2. [WEB-IDE 브로커 릴리즈 업로드](#12)
  - 3.3. [WEB-IDE 브로커 Deployment 파일 수정 및 배포](#13)
  - 3.4. [WEB-IDE 브로커 서비스 등록](#14)
4. [WEB-IDE의 PaaS-TA 포털사이트 연동](#15)
  - 4.1. [WEB-IDE 대시보드 화면](#16)
5. [WEB-IDE 에서 CF CLI 사용법](#17)
  - 5.1. [WEB-IDE New Project 화면](#18)
  - 5.2. [WEB-IDE Workspace 화면](#19)
  - 5.3. [WEB-IDE Teminal에서의 CF CLI 실행](#20)


# 1. 문서 개요

### <div id='2'/>1.1. 목적

본 문서(WEB-IDE 설치 가이드)는 PaaS-TA에서 사용할 수 있는 WEB-IDE의 설치를 Bosh를 이용하여 설치 하는 방법과 PaaS-TA 포털에서 WEB-IDE 서비스를 사용하는 방법을 기술하였다.
PaaS-TA 3.5 버전부터는 Bosh2.0 기반으로 deploy를 진행하며 기존 Bosh1.0 기반으로 설치를 원할경우에는 PaaS-TA 3.1 이하 버전의 문서를 참고한다.

### <div id='3'/> 1.2. 범위
설치 범위는 WEB-IDE 사용을 검증하기 위한 기본 설치를 기준으로 작성하였다.

### <div id='4'/> 1.3. 시스템 구성도
본 문서의 설치된 시스템 구성도입니다. Browser(paasta Portal), WEB IDE
Server, Workspace, Desktop IDE로 최소사항을 구성하였다.

![](/Service-Guide/images/webide/web-ide-01.png)

| 구분 | Resource Pool | 스펙 |
|--------|-------|-------|
| paasta-web-ide1 | resource\_pools | 1vCPU / 2GB RAM / 10GB Disk |
| paasta-web-ide2 | resource\_pools | 1vCPU / 2GB RAM / 10GB Disk |


### <div id='5'/>1.4. 참고자료

> [**http://bosh.io/docs**](http://bosh.io/docs) <br>
> [**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/) <br>
> [**https://www.eclipse.org/che/technology/**](https://www.eclipse.org/che/technology/) <br>


# <div id='6'/> 2.WEB IDE 설치

### <div id='7'/> 2.1. 설치전 준비사항

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

### <div id='8'/> 2.2. WEB-IDE 릴리즈 업로드

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

-	WEB-IDE 서비스 릴리즈가 업로드 되어 있지 않은 것을 확인

-	WEB-IDE 서비스 릴리즈 파일을 업로드한다.

- **사용 예시**

		$ bosh -e micro-bosh upload-release paasta-web-ide-2.0.tgz
    		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		######################################################## 100.00% 160.45 MiB/s 1s
		Task 7866

		Task 7866 | 02:21:08 | Extracting release: Extracting release (00:00:03)
		Task 7866 | 02:21:11 | Verifying manifest: Verifying manifest (00:00:00)
		Task 7866 | 02:21:11 | Resolving package dependencies: Resolving package dependencies (00:00:00)
		Task 7866 | 02:21:11 | Creating new packages: eclipse-che/eff6040fd5ed2a30190955140bb58f892ff830ec (00:00:03)
		Task 7866 | 02:21:14 | Creating new packages: bosh-helpers/2b45cec940a80e582427f61c460269c6ccb031c8 (00:00:01)
		Task 7866 | 02:21:15 | Creating new packages: docker/8da016ec9d1b172b779d5ff0a9fbbfc4973ea734 (00:00:00)
		Task 7866 | 02:21:15 | Creating new packages: java/b74e140053eddb6a3a958568d66f801686d09e04 (00:00:02)
		Task 7866 | 02:21:17 | Creating new jobs: eclipse-che/2f368c268ee821488f04f4b05a25eba963cda484 (00:00:00)
		Task 7866 | 02:21:17 | Release has been created: paasta-web-ide/2.0 (00:00:00)

		Task 7866 Started  Thu Sep 13 02:21:08 UTC 2018
		Task 7866 Finished Thu Sep 13 02:21:17 UTC 2018
		Task 7866 Duration 00:00:09
		Task 7866 done

		Succeeded

-	업로드 된 WEB-IDE 릴리즈를 확인한다.

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
		paasta-web-ide                    2.0       00000000  
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
		
-	WEB-IDE 서비스 릴리즈가 업로드 되어 있는 것을 확인

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
		
>Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드를 해야 한다. (WEB-IDE 은 stemcell 3309 버전을 사용)
		 

### <div id='9'/> 2.3.WEB-IDE Deployment 파일 수정 및 배포

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
# paasta-web-ide-service 설정 파일 내용
---
name: paasta-web-ide-service  # 서비스 배포이름(필수)

release:
  name: paasta-web-ide  #서비스 릴리즈 이름(필수)
  version: "2.0"   #서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전

stemcells:
- alias: default
  os: ((stemcell_os))
  version: "((stemcell_version))"

update:
  canaries: 1                                          # canary 인스턴스 수(필수)
  canary_watch_time: 30000-180000                      # canary 인스턴스가 수행하기 위한 대기 시간(필수)
  max_in_flight: 1                                      # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)
  update_watch_time: 30000-180000                      # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)

instance_groups:
- name: paasta-web-ide1 #작업 이름(필수)
  azs:
  - z5
  instances: 1          # job 인스턴스 수(필수)
  vm_type: ((vm_type_medium))            # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))       # cloud config 에 정의한 network 이름
  - name: ((public_network_name))
    static_ips: 115.68.47.181
  properties:
    che:
      ip: 115.68.47.181
      port: 8080
  templates:
  - name: eclipse-che                # job template 이름(필수)
    release: paasta-web-ide

- name: paasta-web-ide2 #작업 이름(필수)
  azs:
  - z5
  instances: 1          # job 인스턴스 수(필수)
  vm_type: ((vm_type_medium))            # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))       # cloud config 에 정의한 network 이름
  - name: ((public_network_name))
    static_ips: 115.68.47.182
  properties:
    che:
      ip: 115.68.47.182
      port: 8080
  templates:
  - name: eclipse-che                # job template 이름(필수)
    release: paasta-web-ide

```

-	deploy-web-ide-bosh2.0.sh 파일을 서버 환경에 맞게 수정한다.

```sh
#!/bin/bash
# stemcell 버전은 3309 버전으로 사용하시고 https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Download_Page.md 에서 다운받아 쓰십시요.
# vsphere 이이외 iaas 사용시 use-public-network-vsphere.yml 대신 use-public-network.yml 을 사용

bosh -e micro-bosh -d paasta-web-ide-service deploy paasta_web_ide_bosh2.0.yml \
   -o use-public-network-vsphere.yml \
   -v default_network_name=service_private \
   -v public_network_name=service_public \
   -v stemcell_os=ubuntu-trusty \
   -v stemcell_version=3309 \
   -v vm_type_medium=service_medium_2G
```


-	WEB-IDE 서비스팩을 배포한다.

- **사용 예시**

		$ ./deploy-web-ide-bosh2.0.sh
		# paasta-web-ide-service 설정 파일 내용
		---
		name: paasta-web-ide-service  # 서비스 배포이름(필수)

		release:
		  name: paasta-web-ide  #서비스 릴리즈 이름(필수)
		  version: "2.0"   #서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전

		stemcells:
		- alias: default
		  os: ((stemcell_os))
		  version: "((stemcell_version))"

		update:
		  canaries: 1                                          # canary 인스턴스 수(필수)
		  canary_watch_time: 30000-180000                      # canary 인스턴스가 수행하기 위한 대기 시간(필수)
		  max_in_flight: 1                                      # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)
		  update_watch_time: 30000-180000                      # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)

		instance_groups:
		- name: paasta-web-ide1 #작업 이름(필수)
		  azs:
		  - z5
		  instances: 1          # job 인스턴스 수(필수)
		  vm_type: ((vm_type_medium))            # cloud config 에 정의한 vm_type
		  stemcell: default
		  networks:
		  - name: ((default_network_name))       # cloud config 에 정의한 network 이름
		  - name: ((public_network_name))
		    static_ips: 115.68.47.181
		  properties:
		    che:
		      ip: 115.68.47.181
		      port: 8080
		  templates:
		  - name: eclipse-che                # job template 이름(필수)
		    release: paasta-web-ide

		- name: paasta-web-ide2 #작업 이름(필수)
		  azs:
		  - z5
		  instances: 1          # job 인스턴스 수(필수)
		  vm_type: ((vm_type_medium))            # cloud config 에 정의한 vm_type
		  stemcell: default
		  networks:
		  - name: ((default_network_name))       # cloud config 에 정의한 network 이름
		  - name: ((public_network_name))
		    static_ips: 115.68.47.182
		  properties:
		    che:
		      ip: 115.68.47.182
		      port: 8080
		  templates:
		  - name: eclipse-che                # job template 이름(필수)
		    release: paasta-web-ide
		inception@inception:~/workspace/servicepacks/paasta-web-ide-2.0$ 
		inception@inception:~/workspace/servicepacks/paasta-web-ide-2.0$ 
		inception@inception:~/workspace/servicepacks/paasta-web-ide-2.0$ ./deploy-web-ide-bosh2.0.sh 
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Using deployment 'paasta-web-ide-service'

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
		+   canary_watch_time: 30000-180000
		+   max_in_flight: 1
		+   update_watch_time: 30000-180000

		+ release:
		+   name: paasta-web-ide
		+   version: '2.0'

		+ instance_groups:
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: paasta-web-ide1
		+   networks:
		+   - name: service_private
		+   - default:
		+     - dns
		+     - gateway
		+     name: service_public
		+     static_ips: 115.68.47.181
		+   properties:
		+     che:
		+       ip: "<redacted>"
		+       port: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: eclipse-che
		+     release: paasta-web-ide
		+   vm_type: service_medium_2G
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: paasta-web-ide2
		+   networks:
		+   - name: service_private
		+   - default:
		+     - dns
		+     - gateway
		+     name: service_public
		+     static_ips: 115.68.47.182
		+   properties:
		+     che:
		+       ip: "<redacted>"
		+       port: "<redacted>"
		+   stemcell: default
		+   templates:
		+   - name: eclipse-che
		+     release: paasta-web-ide
		+   vm_type: service_medium_2G

		+ name: paasta-web-ide-service

		Continue? [yN]: y

		Task 7867

		Task 7867 | 02:29:25 | Preparing deployment: Preparing deployment (00:00:02)
		Task 7867 | 02:29:27 | Preparing package compilation: Finding packages to compile (00:00:00)
		Task 7867 | 02:29:27 | Compiling packages: bosh-helpers/2b45cec940a80e582427f61c460269c6ccb031c8
		Task 7867 | 02:29:27 | Compiling packages: docker/8da016ec9d1b172b779d5ff0a9fbbfc4973ea734
		Task 7867 | 02:29:27 | Compiling packages: java/b74e140053eddb6a3a958568d66f801686d09e04
		Task 7867 | 02:31:36 | Compiling packages: bosh-helpers/2b45cec940a80e582427f61c460269c6ccb031c8 (00:02:09)
		Task 7867 | 02:31:38 | Compiling packages: docker/8da016ec9d1b172b779d5ff0a9fbbfc4973ea734 (00:02:11)
		Task 7867 | 02:31:59 | Compiling packages: java/b74e140053eddb6a3a958568d66f801686d09e04 (00:02:32)
		Task 7867 | 02:31:59 | Compiling packages: eclipse-che/eff6040fd5ed2a30190955140bb58f892ff830ec (00:00:55)
		Task 7867 | 02:33:27 | Creating missing vms: paasta-web-ide1/dfa63633-f846-48a4-9ea8-c23291fe0ea0 (0)
		Task 7867 | 02:33:27 | Creating missing vms: paasta-web-ide2/9a1e6f85-a8d5-41c0-96f7-56ddb8bce657 (0) (00:01:18)
		Task 7867 | 02:34:46 | Creating missing vms: paasta-web-ide1/dfa63633-f846-48a4-9ea8-c23291fe0ea0 (0) (00:01:19)
		Task 7867 | 02:34:47 | Updating instance paasta-web-ide1: paasta-web-ide1/dfa63633-f846-48a4-9ea8-c23291fe0ea0 (0) (canary) (00:01:29)
		Task 7867 | 02:36:16 | Updating instance paasta-web-ide2: paasta-web-ide2/9a1e6f85-a8d5-41c0-96f7-56ddb8bce657 (0) (canary) (00:01:30)

		Task 7867 Started  Thu Sep 13 02:29:25 UTC 2018
		Task 7867 Finished Thu Sep 13 02:37:46 UTC 2018
		Task 7867 Duration 00:08:21
		Task 7867 done

		Succeeded

-	배포된 WEB-IDE 서비스팩을 확인한다.

- **사용 예시**

		$bosh -e micro-bosh -d paasta-web-ide-service vms
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Task 7872. Done

		Deployment 'paasta-web-ide-service'

		Instance                                              Process State  AZ  IPs            VM CID                                   VM Type            Active  
		paasta-web-ide1/dfa63633-f846-48a4-9ea8-c23291fe0ea0  running        z5  10.30.107.0    vm-4811a409-e48b-44c1-976c-268fe0af0697  service_medium_2G  true  
											 115.68.47.181                                                                
		paasta-web-ide2/9a1e6f85-a8d5-41c0-96f7-56ddb8bce657  running        z5  10.30.108.0    vm-ba125e95-804a-4272-8cda-e3bfa38be98f  service_medium_2G  true  
											 115.68.47.182                                                                

		2 vms

		Succeeded

# <div id='10'/> 3.WEB IDE 브로커 설치

### <div id='11'/> 3.1. 설치전 준비사항

- PaaS-TA WEB IDE 브로커에서 제공하는 압축된 릴리즈 파일들을 다운받는다. (webide-broker-release.zip)

- 다운로드 위치
>WEBIDE-BROKER-RELEASE : **<http://45.248.73.44/index.php/s/yHWRW44Zdk9CxEH/download>**  


### <div id='12'/> 3.2. WEB-IDE 브로커 릴리즈 업로드

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

-	WEB-IDE 브로커 서비스 릴리즈가 업로드 되어 있지 않은 것을 확인

-	WEB-IDE 브로커 서비스 릴리즈 파일을 업로드한다.

- **사용 예시**

		$ bosh -e micro-bosh upload-release webide-broker-release.tgz
    		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		######################################################## 100.00% 160.45 MiB/s 1s
		Task 7866

		Task 7866 | 02:21:08 | Extracting release: Extracting release (00:00:03)
		Task 7866 | 02:21:11 | Verifying manifest: Verifying manifest (00:00:00)
		Task 7866 | 02:21:11 | Resolving package dependencies: Resolving package dependencies (00:00:00)
		Task 7866 | 02:21:11 | Creating new packages: eclipse-che/eff6040fd5ed2a30190955140bb58f892ff830ec (00:00:03)
		Task 7866 | 02:21:14 | Creating new packages: bosh-helpers/2b45cec940a80e582427f61c460269c6ccb031c8 (00:00:01)
		Task 7866 | 02:21:15 | Creating new packages: docker/8da016ec9d1b172b779d5ff0a9fbbfc4973ea734 (00:00:00)
		Task 7866 | 02:21:15 | Creating new packages: java/b74e140053eddb6a3a958568d66f801686d09e04 (00:00:02)
		Task 7866 | 02:21:17 | Creating new jobs: eclipse-che/2f368c268ee821488f04f4b05a25eba963cda484 (00:00:00)
		Task 7866 | 02:21:17 | Release has been created: webide-broker-release (00:00:00)

		Task 7866 Started  Thu Sep 13 02:21:08 UTC 2018
		Task 7866 Finished Thu Sep 13 02:21:17 UTC 2018
		Task 7866 Duration 00:00:09
		Task 7866 done

		Succeeded

-	업로드 된 WEB-IDE 릴리즈를 확인한다.

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
		webide-broker-release             1.1       non-git  

	    (*) Currently deployed
	    (+) Uncommitted changes

	    32 releases

	    Succeeded
		
-	WEB-IDE 서비스 릴리즈가 업로드 되어 있는 것을 확인

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
		
>Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드를 해야 한다. (WEB-IDE 브로커는 stemcell 3445.2 버전을 사용)
		 

### <div id='13'/> 3.3.WEB-IDE Deployment 파일 수정 및 배포

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
            - 115.68.47.175 - 115.68.47.190
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
            - 115.68.46.217 - 115.68.46.222
            static:
            - 115.68.46.214 - 115.68.46.216
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
            cpu: 1
            disk: 4096
            ram: 256
          name: service_tiny
        - cloud_properties:
            cpu: 1
            disk: 4096
            ram: 512
          name: service_small
        - cloud_properties:
            cpu: 1
            disk: 4096
            ram: 1024
          name: service_medium
        - cloud_properties:
            cpu: 1
            disk: 4096
            ram: 2048
          name: service_medium_1CPU_2G
        - cloud_properties:
            cpu: 2
            disk: 8192
            ram: 4096
          name: service_medium_4G
        - cloud_properties:
            cpu: 2
            disk: 10240
            ram: 2048
          name: service_medium_2G
        - cloud_properties:
            cpu: 1
            disk: 4096
            ram: 256
          name: portal_tiny
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


-	Deployment 파일을 서버 환경에 맞게 수정한다.

```yml
# webide-broker-service 설정 파일 내용
---
name: webide-broker-service       #서비스 배포이름(필수) bosh deployments 로 확인 가능한 이름
# director_uuid: <%= `bosh status --uuid` %>                  # Director UUID을 입력(필수) bosh status 명령으로 확인 가능

stemcells:
- alias: default
  os: ((stemcell_os))
  version: "((stemcell_version))"

releases:
- name: webide-broker-release                  # 서비스 릴리즈 이름(필수) bosh releases로 확인 가능
  version: "1.1"                                              # 서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전

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
  - z3
  instances: 1
  vm_type: small
  stemcell: "((stemcell_alias))"
  persistent_disk_type: "((mariadb_disk_type))"
  networks:
  - name: ((internal_networks_name))
  templates:
  - name: mariadb
    release: webide-broker-release
  syslog_aggregator: null


######## BROKER ########

- name: webide-broker
  azs:
  - z3
  instances: 1
  vm_type: medium
  stemcell: "((stemcell_alias))"
  networks:
  - name: ((internal_networks_name))
  templates:
  - name: web-ide-broker
    release: webide-broker-release
  syslog_aggregator: null
  properties:
    server:
      port: ((server_port))
    datasource:
      password: "((mariadb_user_password))"
    webide:
      servers: ((webide_servers))
    serviceDefinition:
      id: ((serviceDefinition_id))
      plan1:
        id: ((serviceDefinition_plan1_id))

######### COMMON PROPERTIES ##########
properties:
  mariadb:                                                # MARIA DB SERVER 설정 정보
    port: ((mariadb_port))                                            # MARIA DB PORT 번호
    admin_user:
      password: '((mariadb_user_password))'                             # MARIA DB ROOT 계정 비밀번호
    host_names:
    - mariadb0

```

-	deploy-vsphere.sh 파일을 서버 환경에 맞게 수정한다.

```sh
#!/bin/bash

# vsphere 이이외 iaas 사용시 use-public-network-vsphere.yml 대신 use-public-network.yml 을 사용

bosh -e micro-bosh -d webide-broker-service deploy paasta_web_ide_vsphere_2.0.yml \
   -v stemcell_os="ubuntu-trusty"\
   -v stemcell_version="3445.2"\
   -v stemcell_alias="default"\
   -v internal_networks_name="service_private"\
   -v external_networks_name="portal_service_public"\
   -v mariadb_disk_type="10GB"\
   -v mariadb_port="3306"\
   -v mariadb_user_password="Paasta@2018"\
   -v server_port="8080"\
   -v webide_servers=["http://115.68.47.184:8080/dashboard","http://115.68.47.185:8080/dashboard"]\
   -v serviceDefinition_id="af86588c-6212-11e7-907b-b6006ad3webide0"\
   -v serviceDefinition_plan1_id="a5930564-6212-11e7-907b-b6006ad3webide1"\
```

-	WEB-IDE 브로커 서비스팩을 배포한다.

- **사용 예시**

		$ ./deploy-vsphere.sh

		+ name: webide-broker-service

		Continue? [yN]: y

		Task 66439

		Task 66439 | 01:29:25 | Preparing deployment: Preparing deployment (00:00:02)
		Task 66439 | 01:29:27 | Preparing package compilation: Finding packages to compile (00:00:00)
		Task 66439 | 01:29:27 | Compiling packages: bosh-helpers/2b45cec940a80e582427f61c460269c6ccb031c8
		Task 66439 | 01:29:27 | Compiling packages: docker/8da016ec9d1b172b779d5ff0a9fbbfc4973ea734
		Task 66439 | 01:29:27 | Compiling packages: java/b74e140053eddb6a3a958568d66f801686d09e04
		Task 66439 | 01:31:36 | Compiling packages: bosh-helpers/2b45cec940a80e582427f61c460269c6ccb031c8 (00:02:09)
		Task 66439 | 01:31:38 | Compiling packages: docker/8da016ec9d1b172b779d5ff0a9fbbfc4973ea734 (00:02:11)
		Task 66439 | 01:31:59 | Compiling packages: java/b74e140053eddb6a3a958568d66f801686d09e04 (00:02:32)
		Task 66439 | 01:31:59 | Compiling packages: eclipse-che/eff6040fd5ed2a30190955140bb58f892ff830ec (00:00:55)
		Task 66439 | 01:33:27 | Creating missing vms: webide-broker-release/1.1 (0)
		Task 66439 | 01:34:46 | Updating instance webide-broker-release1: webide-broker-release/1.1 (0) (canary) (00:01:19)
	

		Task 66439 Started  Wed Nov 21 01:22:54 UTC 2018
        Task 66439 Finished Wed Nov 21 01:22:54 UTC 2018

		Task 66439 Duration 00:05:02
		Task 66439 done

		Succeeded

-	배포된 WEB-IDE 브로커 서비스팩을 확인한다.

- **사용 예시**

		$ bosh -e micro-bosh -d webide-broker-service vms
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Task 66437. Done

		Deployment 'webide-broker-service'

		Instance                                              Process State  AZ  IPs            VM CID                                   VM Type            Active  
		mariadb/a6a4c5fd-e4e4-409a-99ac-51a9e972b3d2          running        z3  10.30.107.0    vm-05cf73a3-3d6c-4229-b201-0c1da2516bc3  small              true  								                                                                 
		webide-broker/c2f9a7c2-8697-4edb-8ec5-6a1126509160    running        z3  10.30.108.0    vm-f663014e-53d0-4ff4-ae6d-396f8b78b077  medium             true  
										   									                                                             
		2 vms

		Succeeded

### <div id='14'/> 3.4.WEB-IDE 브로커 서비스 등록

-	CF 로그인을 한다.

- **사용 예시**

		$ cf login
        API 엔드포인트: https://api.115.68.46.189.xip.io
        
        Email> admin
        
        Password>
        인증 중...
        확인
        
        조직 선택(또는 Enter를 눌러 건너뜀):
        1. system
        2. test
        3. org

        Org> 1

-	정상적인 WEB IDE 브로커 서비스 유뮤를 확인한다.

- **사용 예시**

		$ curl -H "X-Broker-API-Version: 2.8" http://admin:cloudfoundry@10.30.108.0:8080/v2/catalog
        {"services":[{"planUpdatable":false,"id":"af86588c-6212-11e7-907b-b6006ad3webide0","name":"webide","description":"A paasta web ide service for application development.provision parameters","bindable":false,"plan_updateable":false,"plans":[{"id":"a5930564-6212-11e7-907b-b6006ad3webide1","name":"webide-shared","description":"This is service plan. All services are created equally.","metadata":{"costs":[{"amount":{"usd":0.0},"unit":"MONTHLY"}],"bullets":["WEB-IDE shared build server use","WEB-IDE build service using a shared server"]},"free":false}],"tags":["webide-shared"],"metadata":{"longDescription":"Paas-TA Web ide","documentationUrl":"https://paas-ta.kr","providerDisplayName":"PaaS-TA","displayName":"Web-Ide","imageUrl":"","supportUrl":"https://paas-ta.kr"},"requires":[],"dashboard_client":null}]}
		
-	WEB IDE 브로커 서비스를 등록한다.

- **사용 예시**

		$ cf create-service-broker webide-broker admin cloudfoundry http://10.30.108.0:8080
        admin(으)로 서비스 브로커 webide-broker 작성 중...
        성공
        	
-	WEB IDE 브로커 서비스 등록 여부 및 활성화 여부를 확인한다.

- **사용 예시**

		$ cf service-access
        admin(으)로 서비스 액세스를 가져오는 중...
        브로커: caas-service-broker
           서비스            플랜       액세스   조직
           caas-kubernetes   Micro      모두
           caas-kubernetes   Small      모두
           caas-kubernetes   Advanced   모두
        
        브로커: paasta-redis-broker
           서비스   플랜           액세스   조직
           redis    shared-vm      모두
           redis    dedicated-vm   모두
        
        브로커: paasta-glusterfs-broker
           서비스      플랜               액세스   조직
           glusterfs   glusterfs-100Mb    모두
           glusterfs   glusterfs-1000Mb   모두     system
        
        브로커: oracle-service-broker
           서비스   플랜           액세스   조직
           oracle   oracle-plan1   모두     system
           oracle   oracle-plan2   모두
        
        브로커: mysql-service-broker
           서비스     플랜                 액세스   조직
           Mysql-DB   Mysql-Plan1-10con    모두
           Mysql-DB   Mysql-Plan2-100con   모두
        
        브로커: delivery-pipeline-service-broker
           서비스                 플랜                          액세스   조직
           delivery-pipeline-v2   delivery-pipeline-shared      모두
           delivery-pipeline-v2   delivery-pipeline-dedicated   모두
        
        브로커: webide-broker
           서비스   플랜            액세스   조직
           webide   webide-shared   모두
        

-	WEB IDE 브로커 서비스를 활성화 한다.

- **사용 예시**

		$ cf enable-service-access webide -p webide-shared
        admin(으)로 webide 서비스의 webide-shared 플랜 액세스 사용 설정 중...
        확인
        
-	조직에서 WEB IDE 브로커 서비스를 확인한다.

- **사용 예시**

		$ cf m
        admin(으)로 system 조직/dev 영역에서 서비스를 가져오는 중...
        확인
        
        서비스                 플랜                                                      설명
        Mysql-DB               Mysql-Plan1-10con, Mysql-Plan2-100con*                    A simple mysql implementation
        caas-kubernetes        Micro, Small, Advanced                                    for CaaS Plans, You can choose plan about CPU, Memory, disk.
        delivery-pipeline-v2   delivery-pipeline-shared*, delivery-pipeline-dedicated*   A paasta source control service for application development.provision parameters : parameters {owner : owner}
        glusterfs              glusterfs-100Mb*, glusterfs-1000Mb*                       A simple glusterfs implementation
        oracle                 oracle-plan1, oracle-plan2                                A Oracle Database Service for application development
        redis                  shared-vm, dedicated-vm                                   Redis service to provide a key-value store
        webide                 webide-shared*                                            A paasta web ide service for application development.provision parameters
        
        * 해당 서비스 플랜에 연관된 비용이 있습니다. 서비스 인스턴스를 작성하면 이 비용이 발생합니다.
 <br>

# <div id='15'/> 4. WEB-IDE 브로커 PaaS-TA 포털사이트 연동

### <div id='16'/> 4.1. WEB-IDE 브로커 대시보드 화면

-   카탈로그 서비스 목록의 [ 앱 서비스 ]를 클릭한다.
> ① 앱 서비스에 등록 할 이름을 입력한다.<br>
  ② 분류는 "개발 지원 도구"로 지정한다.<br>
  ③ 서비스는 등록 된 WEB IDE 브로커이름 [ webide ]를 선택한다.<br>
  ④ 썸네일은 사용자가 등록하고자 하는 이미지를 지정하여 선택한다.<br>
  ⑤ 문서 URL을 입력한다. (예) https://github.com/PaaS-TA/PAAS-TA-WEB-IDE-BROKER<br>
  ⑥ 서비스 생성 파라미터 사용은 서비스 생성 시 필요한 파라미터 값을 설정한다.<br>
  ⑦ 앱 바인드 사용은 [ N ]으로 선택한다.<br>
  ⑧ 공개 [ Y ] 를 설정 시 활성화 된다. 공개 [ N ] 설정 시 공개 여부가 비 활성화 된다.<br>
  ⑨ 대시보드 [ Y ] 를 필수 설정한다. 대시보드 [ N ] 설정 시 대시보드 버튼이 비 활성화 된다.<br>
  ⑩ 태그는 생성시 필요한 태그 값을 설정한다.<br>
  ⑪ 요약에 등록 할 간략한 정보를 입력한다.<br>
  ⑫ 설명에 등록 할 자세한 정보를 입력한다.<br>
  
 ![](/Service-Guide/images/webide/web-ide-07.png) <br>
  
-   참고 : https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/v3.5/Use-Guide/portal/PaaS-TA%20%EC%9A%B4%EC%98%81%EC%9E%90%20%ED%8F%AC%ED%83%88%20%EA%B0%80%EC%9D%B4%EB%93%9C_v1.1.md#--433-%EC%B9%B4%ED%83%88%EB%A1%9C%EA%B7%B8-%EA%B4%80%EB%A6%AC-%EC%84%9C%EB%B9%84%EC%8A%A4
<br>

# <div id='17'/> 5. WEB-IDE 에서 CF CLI 사용법

### <div id='18'/> 5.1. WEB-IDE New Project 화면

- 서비스 생성에 필요한 항목을 입력 후, "생성" 버튼을 클릭하여 서비스를 생성한다.
> ① 조직 목록에서 조직을 선택한다.<br>
  ② 공간 목록에서 공간을 선택한다.<br>
  ③ 서비스 이름을 입력한다.<br>
  
 ![](/Service-Guide/images/webide/web-ide-08.png)

<br>

- 서비스 목록에서 해당 WEB-IDE 서비스의 "대시보드" 버튼을 클릭한다. 

 ![](/Service-Guide/images/webide/web-ide-15.png)
 
<br>

- Workspace를 구성하기 위해 Docker 관련 자료를 다운로드한다.

 ![](/Service-Guide/images/webide/web-ide-09.png)

<br>

### <div id='19'/> 5.2. WEB-IDE Workspace 화면

- Open Project를 누르면 Workspace 화면이 열린다.

![](/Service-Guide/images/webide/web-ide-10.png)

- 실제로 소스를 개발해서 빌드하거나 GIT이나 SVN에서 IMPORT 한다.

![](/Service-Guide/images/webide/web-ide-11.png)

<br>

### <div id='20'/> 5.3. WEB-IDE Teminal에서의 CF CLI 실행

##### -cf api 명령을 이용해 endpoint를 지정한다.

> ![](/Service-Guide/images/webide/web-ide-12.png)

##### cf login 명령어로 로그인하고 조직과 공간을 선택한다.

> ![](/Service-Guide/images/webide/web-ide-13.png)

##### cf push 를 이용해 cf에 앱을 업로드한다.

> ![](/Service-Guide/images/webide/web-ide-14.png)
