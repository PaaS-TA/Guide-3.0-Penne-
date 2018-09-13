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
3. [WEB-IDE의 PaaS-TA 포털사이트 연동](#10)
  - 3.1. [WEB-IDE 대시보드 화면](#16)
4. [WEB-IDE 에서 CF CLI 사용법](#17)
  - 4.1. [WEB-IDE New Project 화면](#18)
  - 4.2. [WEB-IDE Workspace 화면](#19)
  - 4.3. [WEB-IDE Teminal에서의 CF CLI 실행](#20)


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
| paasta-web-ide1 | resource\_pools | 1vCPU / 2GB RAM / 10GB Disk |


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
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Using deployment 'paasta-mysql-service'

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


-	배포된 MySQL 서비스팩을 확인한다.

- **사용 예시**

		$bosh -e micro-bosh -d paasta-mysql-service vms
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Task 4525. Done

		Deployment 'paasta-mysql-service'

		Instance                                                       Process State  AZ  IPs            VM CID                                   VM Type  Active  
		mysql/8a830154-25b6-432a-ad39-9ff09d015760                     running        z5  10.30.107.165  vm-214663a8-fcbc-4ae4-9aae-92027b9725a9  minimal  true  
		mysql/e8c52bf2-cd48-45d0-9553-f6367942a634                     running        z5  10.30.107.164  vm-81ecdc43-03d2-44f5-9b89-c6cdaa443d8b  minimal  true  
		mysql/ea075ae6-6326-478b-a1ba-7fbb0b5b0bf5                     running        z5  10.30.107.166  vm-bee33ffa-3f65-456c-9250-1e74c7c97f64  minimal  true  
		paasta-mysql-java-broker/bb5676ca-efba-48fc-bc11-f464d0ae9c78  running        z5  10.30.107.167  vm-7c3edc00-3074-4e98-9c89-9e9ba83b47e4  minimal  true  
		proxy/023edddd-418e-46e4-8d40-db452c694e16                     running        z5  10.30.107.168  vm-e447eb75-1119-451f-adc9-71b0a6ef1a6a  minimal  true  

		5 vms

		Succeeded


# <div id='10'/> 3. WEB-IDE의 PaaS-TA 포털사이트 연동

### <div id='16'/> 3.1. WEB-IDE 대시보드 화면

-   WEB IDE 아이콘을 클릭하면 관리자에 의해 할당된 WEB IDE 대시보드 화면이 새탭으로 열리게 된다.

![](/Service-Guide/images/webide/web-ide-07.png)

<br>

# <div id='17'/> 4. WEB-IDE 에서 CF CLI 사용법

### <div id='18'/> 4.1. WEB-IDE New Project 화면

- 사용할 언어를 선택하고 Create workspace and project 로 새로운 프로젝트를 시작한다.

![](/Service-Guide/images/webide/web-ide-08.png)

<br>

- Workspace를 구성하기 위해 Docker 관련 자료를 다운로드한다.

![](/Service-Guide/images/webide/web-ide-09.png)

<br>

### <div id='19'/> 4.2. WEB-IDE Workspace 화면

- Open Project를 누르면 Workspace 화면이 열린다.

![](/Service-Guide/images/webide/web-ide-10.png)

- 실제로 소스를 개발해서 빌드하거나 GIT이나 SVN에서 IMPORT 한다.

![](/Service-Guide/images/webide/web-ide-11.png)

<br>

### <div id='20'/> 4.3. WEB-IDE Teminal에서의 CF CLI 실행

##### -cf api 명령을 이용해 endpoint를 지정한다.

> ![](/Service-Guide/images/webide/web-ide-12.png)

##### cf login 명령어로 로그인하고 조직과 공간을 선택한다.

> ![](/Service-Guide/images/webide/web-ide-13.png)

##### cf push 를 이용해 cf에 앱을 업로드한다.

> ![](/Service-Guide/images/webide/web-ide-14.png)
