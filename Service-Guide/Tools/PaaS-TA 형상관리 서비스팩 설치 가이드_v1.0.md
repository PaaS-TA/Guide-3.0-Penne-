## Table of Contents
1. [문서 개요](#1)
  -  [1.1. 목적](#2)
  -  [1.2. 범위](#3)
  -  [1.3. 시스템 구성도](#4)
  -  [1.4. 참고 자료](#5)
2. [형상관리 서비스팩 설치](#6)
  -  [2.1. 설치 전 준비사항](#7)
  -  [2.2. 형상관리 서비스 릴리즈 업로드](#8)
  -  [2.3. 형상관리 서비스 Deployment 파일 수정 및 배포](#9)
  -  [2.4. 형상관리 서비스 브로커 등록](#10)
  -  [2.5. 형상관리 UAA Client Id 등록](#11)



# <div id='1'/> 1. 문서 개요

### <div id='2'/> 1.1 목적
본 문서(형상관리 서비스팩 설치 가이드)는 개방형 PaaS 플랫폼 고도화 및 개발자 지원 환경 기반의 Open PaaS에서 제공되는 서비스팩인 형상관리 서비스를 Bosh를 이용하여 설치하는 방법을 기술하였다.
PaaS-TA 3.5 버전부터는 Bosh2.0 기반으로 deploy를 진행하며 기존 Bosh1.0 기반으로 설치를 원할경우에는 PaaS-TA 3.1 이하 버전의 문서를 참고한다.

### <div id='3'/> 1.2 범위
설치 범위는 형상관리 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다.

### <div id='4'/> 1.3 시스템 구성도
본 문서의 설치된 시스템 구성도입니다. 형상관리 Server, 형상관리 서비스 브로커로 최소 사항을 구성하였다.
![source_controller_Service_Guide01]

### <div id='5'/> 1.4 참고 자료
> http://bosh.io/docs <br>
> http://docs.cloudfoundry.org/ 

# <div id='6'/> 2. 형상관리 서비스팩 설치

### <div id='7'/> 2.1. 설치 전 준비사항
본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다.
서비스팩 설치를 위해서는 먼저 BOSH CLI v2 가 설치 되어 있어야 하고 BOSH 에 로그인이 되어 있어야 한다.<br>
BOSH CLI v2 가 설치 되어 있지 않을 경우 먼저 BOSH2.0 설치 가이드 문서를 참고 하여 BOSH CLI v2를 설치를 하고 사용법을 숙지 해야 한다.<br>

- BOSH2.0 사용자 가이드
>BOSH2 사용자 가이드 : **<https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/v3.5/Use-Guide/Bosh/PaaS-TA_BOSH2_%EC%82%AC%EC%9A%A9%EC%9E%90_%EA%B0%80%EC%9D%B4%EB%93%9Cv1.0.md>**

>BOSH CLI V2 사용자 가이드 : **<https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/v3.5/Use-Guide/Bosh/PaaS-TA_BOSH_CLI_V2_%EC%82%AC%EC%9A%A9%EC%9E%90_%EA%B0%80%EC%9D%B4%EB%93%9Cv1.0.md>**

- PaaS-TA에서 제공하는 압축된 릴리즈 파일들을 다운받는다. (PaaSTA-Deployment.zip, PaaSTA-Sample-Apps.zip, PaaSTA-Services.zip)

- 다운로드 위치
>PaaSTA-Deployment : **<https://paas-ta.kr/data/packages/3.5/PaaSTA-Deploy.zip>**  
>PaaSTA-Sample-Apps : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Sample-Apps.zip>**


### <div id='8'/> 2.2. 형상관리 서비스 릴리즈 업로드

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

-	형상관리 서비스 릴리즈가 업로드 되어 있지 않은 것을 확인

-	형상관리 서비스 릴리즈 파일을 업로드한다.

- **사용 예시**

		$ bosh -e micro-bosh upload-release paasta-sourcecontrol-release-1.0.tgz
    		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		######################################################## 100.00% 138.05 MiB/s 1s
		Task 7827

		Task 7827 | 00:09:43 | Extracting release: Extracting release (00:00:03)
		Task 7827 | 00:09:46 | Verifying manifest: Verifying manifest (00:00:00)
		Task 7827 | 00:09:46 | Resolving package dependencies: Resolving package dependencies (00:00:00)
		Task 7827 | 00:09:46 | Creating new packages: haproxy/14b0441f6d68c89612f53ce4334a65c80d601e51 (00:00:00)
		Task 7827 | 00:09:46 | Creating new packages: bosh-helpers/104087b4a6a1d5ebe0e26b0b2e3dcd71a9b971e2 (00:00:00)
		Task 7827 | 00:09:46 | Creating new packages: java8/88ff03b20ad2f59840fbb5f74a074ca8fdc68213 (00:00:02)
		Task 7827 | 00:09:48 | Creating new packages: sourcecontrol-api/37c3b7f8c53787e1f536a68543828bb76263ef50 (00:00:02)
		Task 7827 | 00:09:50 | Creating new packages: tomcat/1ff8747498f45f21b2d43d6bc50d8e53a0ddfca7 (00:00:01)
		Task 7827 | 00:09:51 | Creating new packages: scm-server/8f9a9d9259ff144dc71e1131c25420d1b3fa056d (00:00:00)
		Task 7827 | 00:09:51 | Creating new packages: sourcecontrol-webui/70b5fb8f8fb97d196fa48d400f3c8f87bf3a36ea (00:00:02)
		Task 7827 | 00:09:53 | Creating new packages: mariadb/59a218308c6c7dcf8795b531b53aa4a1c666ce00 (00:00:23)
		Task 7827 | 00:10:16 | Creating new packages: sourcecontrol-broker/c5394441a91bddd526e232a7e7a466e8c3d90e51 (00:00:01)
		Task 7827 | 00:10:17 | Creating new jobs: haproxy/83a2a95d33ee96bb34e03e635b0d041ee39724c9 (00:00:00)
		Task 7827 | 00:10:17 | Creating new jobs: sourcecontrol-api/bfd00eb9d00b8de2d1ff7b4cf32b393ce5add3ac (00:00:00)
		Task 7827 | 00:10:17 | Creating new jobs: scm-server/5563bed2081ac657e2d857d6a4ff6a698c770175 (00:00:00)
		Task 7827 | 00:10:17 | Creating new jobs: sourcecontrol-webui/da8252527b24b150daa78bc6348aef149e6f6db7 (00:00:00)
		Task 7827 | 00:10:17 | Creating new jobs: mariadb/cf48ef607b7415b7ebe178f3ad2e71f66e7c80b4 (00:00:00)
		Task 7827 | 00:10:17 | Creating new jobs: sourcecontrol-broker/4e9d370ef1df0c400341942b327506af0568bb99 (00:00:00)
		Task 7827 | 00:10:17 | Release has been created: paasta-sourcecontrol-release/1.0 (00:00:00)

		Task 7827 Started  Thu Sep 13 00:09:43 UTC 2018
		Task 7827 Finished Thu Sep 13 00:10:17 UTC 2018
		Task 7827 Duration 00:00:34
		Task 7827 done

		Succeeded


-	업로드 된 형상관리 릴리즈를 확인한다.

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
		paasta-sourcecontrol-release      1.0       d41bda4d+  
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
		
-	형상관리 서비스 릴리즈가 업로드 되어 있는 것을 확인

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
		
>Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드를 해야 한다. (형상관리는 stemcell 3445.2 버전을 사용)


### <div id='9'/> 2.3. 형상관리 서비스 Deployment 파일 수정 및 배포
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
# paasta-sourcecontrol-service 설정 파일 내용
name: paasta-sourcecontrol-service                      # 서비스 배포이름(필수)

releases:
- name: paasta-sourcecontrol-release                    # 서비스 릴리즈 이름(필수)
  version: "1.0"                                        # 서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전

stemcells:
- alias: default
  os: ((stemcell_os))
  version: "((stemcell_version))"

update:
  canaries: 1                            # canary 인스턴스 수(필수)
  canary_watch_time: 30000-120000        # canary 인스턴스가 수행하기 위한 대기 시간(필수)
  max_in_flight: 4                       # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)
  update_watch_time: 30000-120000        # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)

instance_groups:
- name: scm-server
  azs:
  - z5
  instances: 1
  persistent_disk_type: 30GB
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.121
  templates:
  - name: scm-server
    release: paasta-sourcecontrol-release

- name: mariadb
  azs:
  - z5
  instances: 1
  persistent_disk_type: 2GB
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.122
  templates:
  - name: mariadb
    release: paasta-sourcecontrol-release

- name: haproxy
  azs:
  - z5
  instances: 1
  persistent_disk_type: 2GB
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.123
  - name: ((public_network_name))
    static_ips:
    - 115.68.47.175
  templates:
  - name: haproxy
    release: paasta-sourcecontrol-release

- name: sourcecontrol-webui
  azs:
  - z5
  instances: 1
  persistent_disk_type: 2GB
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.124
  templates:
  - name: sourcecontrol-webui
    release: paasta-sourcecontrol-release

- name: sourcecontrol-api
  azs:
  - z5
  instances: 1
  persistent_disk_type: 2GB
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.125
  templates:
  - name: sourcecontrol-api
    release: paasta-sourcecontrol-release

- name: sourcecontrol-broker
  azs:
  - z5
  instances: 1
  persistent_disk_type: 2GB
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.126
  templates:
  - name: sourcecontrol-broker
    release: paasta-sourcecontrol-release

properties:
  spring:
    datasource:
      url: jdbc:mysql://10.30.107.122:31306/PAASTA_SRC_DB?autoReconnect=true&useUnicode=true&characterEncoding=utf8
      username: sourcecontrol
      password: '!scadmin2017'
      databasename: "PAASTA_SRC_DB"
    jpa:
      database: mysql

  haproxy:                                # sourcecontrol haproxy
    host_ip: 10.30.107.123                # sourcecontrol haproxy host IP
    url: 10.30.107.123                # sourcecontrol haproxy host IP
    http_port: 8080                        # sourcecontrol haproxy host port

  sourcecontrol-webui:                  # sourcecontrol webui
    port: 8080
    haproxy:
      urls:
      - 10.30.107.124
    java_opts: '-XX:MaxMetaspaceSize=104857K -Xss349K -Xms681574K -XX:MetaspaceSize=104857K -Xmx681574K'

  sourcecontrol-api:                    # sourcecontrol api
    base:
      url: http://10.30.107.125:8082        # sourcecontrol api base url
    port: 8082                              # sourcecontrol api port
    haproxy:
      urls:
      - 10.30.107.125
    java_opts: '-XX:MaxMetaspaceSize=104857K -Xss349K -Xms681574K -XX:MetaspaceSize=104857K -Xmx681574K'

  sourcecontrol-broker:                 # sourcecontrol broker
    port: 8080                               # sourcecontrol port
    java_opts: '-XX:MaxMetaspaceSize=104857K -Xss349K -Xms681574K -XX:MetaspaceSize=104857K -Xmx681574K'

  mariadb:
    port: 31306
    datasource:
      username: root
      password: "!paas_ta202"
      databasename:  "PAASTA_SRC_DB"

  scm:
    base:
      url: http://10.30.107.121:8084/scm
      port: 8084
    haproxy:
      urls:
      - 10.30.107.121
    clone:
      url: http://115.68.47.175:8084/scm
  dashboard:
    url: http://115.68.47.175:8080/repositories/user/{instanceId}
  api:
    user: '/api/rest/users'
    repo: '/api/rest/repositories'
  cf:
    uaa:
      oauth:
        info:
          uri: https://uaa.115.68.46.189.xip.io/userinfo
        token:
          check:
            uri: https://uaa.115.68.46.189.xip.io/check_token
          access:
            uri: https://uaa.115.68.46.189.xip.io/oauth/token
        logout:
          url: https://uaa.115.68.46.189.xip.io/logout.do
        authorization:
          uri: https://uaa.115.68.46.189.xip.io/oauth/authorize
        client:
          id: scclient
          secret: clientsecret
    api:
      url: https://api.115.68.46.189.xip.io/v2/service_instances/[SUID]/permissions
```

-	deploy-mysql-bosh2.0.sh 파일을 서버 환경에 맞게 수정한다.

```sh
#!/bin/bash
# stemcell 버전은 3445.2 버전으로 사용하시고 https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Download_Page.md 에서 다운받아 쓰십시요.
# vsphere 인 경우 에는 use-public-network-vsphere.yml 사용하여 public ip를 설정 하고 그 이외의 IaaS는 use-public-network.yml 사용한다.

bosh -e micro-bosh -d paasta-sourcecontrol-service deploy paasta_sourcecontrol_bosh2.0.yml \
   -o use-public-network-vsphere.yml \
   -v default_network_name=service_private \
   -v public_network_name=service_public \
   -v stemcell_os=ubuntu-trusty \
   -v stemcell_version=3445.2 \
   -v vm_type_small=minimal
```

-	형상관리 서비스팩을 배포한다.

- **사용 예시**

		$ ./deploy-sourcecontrol-bosh2.0.sh
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Using deployment 'paasta-sourcecontrol-service'

		+ stemcells:
		+ - alias: default
		+   os: ubuntu-trusty
		+   version: '3445.2'

		+ releases:
		+ - name: paasta-sourcecontrol-release
		+   version: '1.0'

		+ update:
		+   canaries: 1
		+   canary_watch_time: 30000-120000
		+   max_in_flight: 4
		+   update_watch_time: 30000-120000

		+ instance_groups:
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: scm-server
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.121
		+   persistent_disk_type: 30GB
		+   stemcell: default
		+   templates:
		+   - name: scm-server
		+     release: paasta-sourcecontrol-release
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: mariadb
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.122
		+   persistent_disk_type: 2GB
		+   stemcell: default
		+   templates:
		+   - name: mariadb
		+     release: paasta-sourcecontrol-release
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: haproxy
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.123
		+   - default:
		+     - dns
		+     - gateway
		+     name: service_public
		+     static_ips:
		+     - 115.68.47.179
		+   persistent_disk_type: 2GB
		+   stemcell: default
		+   templates:
		+   - name: haproxy
		+     release: paasta-sourcecontrol-release
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: sourcecontrol-webui
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.124
		+   persistent_disk_type: 2GB
		+   stemcell: default
		+   templates:
		+   - name: sourcecontrol-webui
		+     release: paasta-sourcecontrol-release
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: sourcecontrol-api
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.125
		+   persistent_disk_type: 2GB
		+   stemcell: default
		+   templates:
		+   - name: sourcecontrol-api
		+     release: paasta-sourcecontrol-release
		+   vm_type: minimal
		+ - azs:
		+   - z5
		+   instances: 1
		+   name: sourcecontrol-broker
		+   networks:
		+   - name: service_private
		+     static_ips:
		+     - 10.30.107.126
		+   persistent_disk_type: 2GB
		+   stemcell: default
		+   templates:
		+   - name: sourcecontrol-broker
		+     release: paasta-sourcecontrol-release
		+   vm_type: minimal

		+ name: paasta-sourcecontrol-service

		+ properties:
		+   api:
		+     repo: "<redacted>"
		+     user: "<redacted>"
		+   cf:
		+     api:
		+       url: "<redacted>"
		+     uaa:
		+       oauth:
		+         authorization:
		+           uri: "<redacted>"
		+         client:
		+           id: "<redacted>"
		+           secret: "<redacted>"
		+         info:
		+           uri: "<redacted>"
		+         logout:
		+           url: "<redacted>"
		+         token:
		+           access:
		+             uri: "<redacted>"
		+           check:
		+             uri: "<redacted>"
		+   dashboard:
		+     url: "<redacted>"
		+   haproxy:
		+     host_ip: "<redacted>"
		+     http_port: "<redacted>"
		+     url: "<redacted>"
		+   mariadb:
		+     datasource:
		+       databasename: "<redacted>"
		+       password: "<redacted>"
		+       username: "<redacted>"
		+     port: "<redacted>"
		+   scm:
		+     base:
		+       port: "<redacted>"
		+       url: "<redacted>"
		+     clone:
		+       url: "<redacted>"
		+     haproxy:
		+       urls:
		+       - "<redacted>"
		+   sourcecontrol-api:
		+     base:
		+       url: "<redacted>"
		+     haproxy:
		+       urls:
		+       - "<redacted>"
		+     java_opts: "<redacted>"
		+     port: "<redacted>"
		+   sourcecontrol-broker:
		+     java_opts: "<redacted>"
		+     port: "<redacted>"
		+   sourcecontrol-webui:
		+     haproxy:
		+       urls:
		+       - "<redacted>"
		+     java_opts: "<redacted>"
		+     port: "<redacted>"
		+   spring:
		+     datasource:
		+       databasename: "<redacted>"
		+       password: "<redacted>"
		+       url: "<redacted>"
		+       username: "<redacted>"
		+     jpa:
		+       database: "<redacted>"

		Continue? [yN]: y

		ask 7838

		Task 7838 | 00:18:22 | Preparing deployment: Preparing deployment (00:00:02)
		Task 7838 | 00:18:25 | Preparing package compilation: Finding packages to compile (00:00:00)
		Task 7838 | 00:18:25 | Compiling packages: sourcecontrol-broker/c5394441a91bddd526e232a7e7a466e8c3d90e51
		Task 7838 | 00:18:25 | Compiling packages: sourcecontrol-api/37c3b7f8c53787e1f536a68543828bb76263ef50
		Task 7838 | 00:18:25 | Compiling packages: sourcecontrol-webui/70b5fb8f8fb97d196fa48d400f3c8f87bf3a36ea
		Task 7838 | 00:18:25 | Compiling packages: tomcat/1ff8747498f45f21b2d43d6bc50d8e53a0ddfca7
		Task 7838 | 00:18:25 | Compiling packages: haproxy/14b0441f6d68c89612f53ce4334a65c80d601e51
		Task 7838 | 00:21:13 | Compiling packages: tomcat/1ff8747498f45f21b2d43d6bc50d8e53a0ddfca7 (00:02:48)
		Task 7838 | 00:21:13 | Compiling packages: mariadb/59a218308c6c7dcf8795b531b53aa4a1c666ce00
		Task 7838 | 00:21:21 | Compiling packages: sourcecontrol-broker/c5394441a91bddd526e232a7e7a466e8c3d90e51 (00:02:56)
		Task 7838 | 00:21:21 | Compiling packages: java8/88ff03b20ad2f59840fbb5f74a074ca8fdc68213
		Task 7838 | 00:21:21 | Compiling packages: sourcecontrol-webui/70b5fb8f8fb97d196fa48d400f3c8f87bf3a36ea (00:02:56)
		Task 7838 | 00:21:21 | Compiling packages: scm-server/8f9a9d9259ff144dc71e1131c25420d1b3fa056d
		Task 7838 | 00:21:30 | Compiling packages: sourcecontrol-api/37c3b7f8c53787e1f536a68543828bb76263ef50 (00:03:05)
		Task 7838 | 00:21:59 | Compiling packages: scm-server/8f9a9d9259ff144dc71e1131c25420d1b3fa056d (00:00:38)
		Task 7838 | 00:22:05 | Compiling packages: haproxy/14b0441f6d68c89612f53ce4334a65c80d601e51 (00:03:40)
		Task 7838 | 00:22:12 | Compiling packages: java8/88ff03b20ad2f59840fbb5f74a074ca8fdc68213 (00:00:51)
		Task 7838 | 00:22:32 | Compiling packages: mariadb/59a218308c6c7dcf8795b531b53aa4a1c666ce00 (00:01:19)
		Task 7838 | 00:23:33 | Creating missing vms: scm-server/6e23addc-33c7-4bb0-af95-d66420f15c06 (0)
		Task 7838 | 00:23:33 | Creating missing vms: mariadb/90ea7861-57ed-43f7-853d-25712a67ba2a (0)
		Task 7838 | 00:23:33 | Creating missing vms: haproxy/878b393c-d817-4e71-8fe5-553ddf87d362 (0)
		Task 7838 | 00:23:33 | Creating missing vms: sourcecontrol-webui/840278e2-e1a2-4a30-b904-68538c7cd06f (0)
		Task 7838 | 00:23:33 | Creating missing vms: sourcecontrol-broker/ec83edb5-130f-4a91-9ac1-20fb622ed0a2 (0)
		Task 7838 | 00:23:33 | Creating missing vms: sourcecontrol-api/3ecb90fa-2211-4df6-82bb-5c91ed9a4310 (0)
		Task 7838 | 00:25:38 | Creating missing vms: haproxy/878b393c-d817-4e71-8fe5-553ddf87d362 (0) (00:02:05)
		Task 7838 | 00:25:40 | Creating missing vms: mariadb/90ea7861-57ed-43f7-853d-25712a67ba2a (0) (00:02:07)
		Task 7838 | 00:25:49 | Creating missing vms: sourcecontrol-api/3ecb90fa-2211-4df6-82bb-5c91ed9a4310 (0) (00:02:16)
		Task 7838 | 00:25:52 | Creating missing vms: scm-server/6e23addc-33c7-4bb0-af95-d66420f15c06 (0) (00:02:19)
		Task 7838 | 00:25:52 | Creating missing vms: sourcecontrol-webui/840278e2-e1a2-4a30-b904-68538c7cd06f (0) (00:02:19)
		Task 7838 | 00:25:52 | Creating missing vms: sourcecontrol-broker/ec83edb5-130f-4a91-9ac1-20fb622ed0a2 (0) (00:02:19)
		Task 7838 | 00:25:53 | Updating instance scm-server: scm-server/6e23addc-33c7-4bb0-af95-d66420f15c06 (0) (canary) (00:02:39)
		Task 7838 | 00:28:32 | Updating instance mariadb: mariadb/90ea7861-57ed-43f7-853d-25712a67ba2a (0) (canary) (00:02:53)
		Task 7838 | 00:31:25 | Updating instance haproxy: haproxy/878b393c-d817-4e71-8fe5-553ddf87d362 (0) (canary) (00:02:06)
		Task 7838 | 00:33:31 | Updating instance sourcecontrol-webui: sourcecontrol-webui/840278e2-e1a2-4a30-b904-68538c7cd06f (0) (canary) (00:02:08)
		Task 7838 | 00:35:39 | Updating instance sourcecontrol-api: sourcecontrol-api/3ecb90fa-2211-4df6-82bb-5c91ed9a4310 (0) (canary) (00:02:08)
		Task 7838 | 00:37:47 | Updating instance sourcecontrol-broker: sourcecontrol-broker/ec83edb5-130f-4a91-9ac1-20fb622ed0a2 (0) (canary) (00:02:10)

		Task 7838 Started  Thu Sep 13 00:18:22 UTC 2018
		Task 7838 Finished Thu Sep 13 00:39:57 UTC 2018
		Task 7838 Duration 00:21:35
		Task 7838 done

		Succeeded


-	배포된 형상관리 서비스팩을 확인한다.

- **사용 예시**

		$bosh -e micro-bosh -d paasta-sourcecontrol-service vms
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Task 7847. Done

		Deployment 'paasta-sourcecontrol-service'

		Instance                                                   Process State  AZ  IPs            VM CID                                   VM Type  Active  
		haproxy/878b393c-d817-4e71-8fe5-553ddf87d362               running        z5  115.68.47.179  vm-cc4c774d-9857-4b3a-9ffc-5f836098eb4e  minimal  true  
											      10.30.107.123                                                      
		mariadb/90ea7861-57ed-43f7-853d-25712a67ba2a               running        z5  10.30.107.122  vm-6952fb76-b4d6-4f53-86cb-b0517a76f0d0  minimal  true  
		scm-server/6e23addc-33c7-4bb0-af95-d66420f15c06            running        z5  10.30.107.121  vm-08eb6dd3-04ae-435c-9558-9b78286b730c  minimal  true  
		sourcecontrol-api/3ecb90fa-2211-4df6-82bb-5c91ed9a4310     running        z5  10.30.107.125  vm-ee4daa28-3e10-4409-9d6f-ce566d54e8a5  minimal  true  
		sourcecontrol-broker/ec83edb5-130f-4a91-9ac1-20fb622ed0a2  running        z5  10.30.107.126  vm-23d1a9fc-30d2-4a0f-8631-df8807fc8612  minimal  true  
		sourcecontrol-webui/840278e2-e1a2-4a30-b904-68538c7cd06f   running        z5  10.30.107.124  vm-0f7300dd-63b5-4399-b1fd-35aeccffac5c  minimal  true  

		6 vms

		Succeeded


### <div id='10'/> 2.4. 형상관리 서비스 브로커 등록
형상관리 서비스팩 배포가 완료되었으면 Application에서 서비스 팩을 사용하기 위해서 먼저 형상관리 서비스 브로커를 등록해 주어야 한다.
서비스 브로커 등록 시 개방형 클라우드 플랫폼에서 서비스 브로커를 등록할 수 있는 사용자로 로그인이 되어있어야 한다.

##### 서비스 브로커 목록을 확인한다.

>`$ cf service-brokers`
```
Getting service brokers as admin...

name                          url
cubrid-service-broker         http://10.30.60.22:8080
delivery-pipline              http://paas-ta-delivery-pipline-broker.115.68.46.187.xip.io
glusterfs-service             http://10.30.120.197:8080
mysql-service-broker          http://10.30.40.195:8080
paasta-redis-broker           http://10.30.60.71:12350
```

##### 형상관리 서비스 브로커를 등록한다.

>`$ cf create-service-broker {서비스팩 이름}{서비스팩 사용자ID}{서비스팩 사용자비밀번호} http://{서비스팩 URL}`

  **서비스팩 이름** : 서비스 팩 관리를 위해 PaaS-TA에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스팩 리스트의 명칭이다.<br>
  **서비스팩 사용자ID** / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID입니다. 서비스팩도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.<br>
  **서비스팩 URL** : 서비스팩이 제공하는 API를 사용할 수 있는 URL을 입력한다.

>`$ cf create-service-broker p-paasta-sourcecontrol admin cloudfoundry http://10.30.107.126`
```
inception-new:~/bosh-space/paasta-sourcecontrol-releases/deployment$ cf service-brokers
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

>`$ cf service-access`

- 서비스 브로커 생성 시 디폴트로 접근을 허용하지 않는다.

- 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)

>`$ cf enable-service-access p-paasta-sourcecontrol` <br>
>`$ cf service-access` 
```
broker : paasta-sourcecontrol-broker
service                         plan              access              orgs
p-paasta-sourcecontrol          Default           all
```

- 서비스 브로커 등록 시 최초에는 접근을 허용하지 않는다. 따라서 access는 none으로 설정된다.

### <div id='11'/> 2.5. 형상관리 UAA Client Id 등록
UAA 포털 계정 등록 절차에 대한 순서를 확인한다.

**haproxy IPs**:8080 입력 후 루트 도메인을 입력한다.**haproxy IPs**의 주소는 **실제 대시보드 URL**이다. URL 정보 다중 입력 시 ","(comma)를 사용하여 추가한다.
>`$ http://115.68.47.179:8080/repositories/user/323900a4-8333-4122-91c3-00d384466ed9` 

##### *haproxy IPs 확인*
>`$ bosh -e micro-bosh -d paasta-sourcecontrol-service vms` 
```
Deployment 'paasta-sourcecontrol-service'
 
Instance                                                   Process State  AZ  IPs            VM CID                                   VM Type  Active
haproxy/878b393c-d817-4e71-8fe5-553ddf87d362               running        z5  115.68.47.179  vm-cc4c774d-9857-4b3a-9ffc-5f836098eb4e  minimal  true
                                                                              10.30.107.123
mariadb/90ea7861-57ed-43f7-853d-25712a67ba2a               running        z5  10.30.107.122  vm-6952fb76-b4d6-4f53-86cb-b0517a76f0d0  minimal  true
scm-server/6e23addc-33c7-4bb0-af95-d66420f15c06            running        z5  10.30.107.121  vm-08eb6dd3-04ae-435c-9558-9b78286b730c  minimal  true
sourcecontrol-api/3ecb90fa-2211-4df6-82bb-5c91ed9a4310     running        z5  10.30.107.125  vm-ee4daa28-3e10-4409-9d6f-ce566d54e8a5  minimal  true
sourcecontrol-broker/ec83edb5-130f-4a91-9ac1-20fb622ed0a2  running        z5  10.30.107.126  vm-23d1a9fc-30d2-4a0f-8631-df8807fc8612  minimal  true
sourcecontrol-webui/840278e2-e1a2-4a30-b904-68538c7cd06f   running        z5  10.30.107.124  vm-0f7300dd-63b5-4399-b1fd-35aeccffac5c  minimal  true

6 vms
```

##### 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)
>`$ uaac target` 
```
Target : https://uaa.115.68.46.186.xip.io
Context : admin, from client admin
```

##### URL을 변경하고 싶을 경우 [uaac target https://uaa.115.68.46.186.xip.io] 과 같이 입력하여 변경 가능합니다. (uaac target [URL])

##### 로그인을 한다.
>`$ uaac token client gett` 

```
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

##### SC포탈 계정 생성을 한다.
>`$ uaac client add pipeclient -s clientsecret --redirect_uri "[URL]" /
    --scope "cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" /
    --authorized_grant_types "authorization_code , client_credentials , refresh_token" /
    --authorities="uaa.resource" /
    --autoapprove="openid , cloud_controller_service_permissions.read""`
```
 $ uaac client add scclient -s clientsecret --redirect_uri "http://115.68.47.179:8080 http://115.68.47.179:8080/repositories http://115.68.47.179:8080/repositories/user" \
    --scope "cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" \
    --authorized_grant_types "authorization_code , client_credentials , refresh_token" \
    --authorities="uaa.resource" \
    --autoapprove="openid , cloud_controller_service_permissions.read
```
[source_controller_Service_Guide01]:/Service-Guide/images/source_control/source_controller_Service_Guide01.PNG
