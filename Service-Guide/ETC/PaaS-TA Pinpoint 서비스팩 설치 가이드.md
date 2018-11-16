## Table of Contents
1. [문서 개요](#1)
  - 1.1. [목적](#11)
  - 1.2. [범위](#12)
  - 1.3. [시스템 구성도](#13)
  - 1.4. [참고자료](#14)
2. [Pinpoint 서비스팩 설치](#2)
  - 2.1. [설치전 준비사항](#21)
  - 2.2. [Pinpoint 서비스 릴리즈 업로드](#22)
  - 2.3. [Pinpoint 서비스 Deployment 파일 수정 및 배포](#23)
  - 2.4. [HBase 기본 데이터 실행](#24)
  - 2.5. [Pinpoint 서비스 브로커 등록](#25)
3. [Sample Web App 연동 Pinpoint 연동](#3)
  - 3.1. [Sample Web App 구조](#31)
  - 3.2. [PaaS-TA에서 서비스 신청](#32)
  - 3.3. [Sample Web App에 서비스 바인드 신청 및 App 확인](#33)

# <div id='1'> 1. 문서 개요
### <div id='11'> 1.1. 목적

본 문서(Pinpoint 서비스팩 설치 가이드)는 전자정부표준프레임워크 기반의 PaaS-TA에서 제공되는 서비스팩인 Pinpoint 서비스팩을 Bosh2.0을 이용하여 설치 하는 방법과 PaaS-TA의 SaaS 형태로 제공하는 Application 에서 Pinpoint 서비스를 사용하는 방법을 기술하였다.
PaaS-TA 3.5 버전부터는 Bosh2.0 기반으로 deploy를 진행하며 기존 Bosh1.0 기반으로 설치를 원할경우에는 PaaS-TA 3.1 이하 버전의 문서를 참고한다.

### <div id='12'> 1.2. 범위
설치 범위는 Pinpoint 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다.

### <div id='13'> 1.3. 시스템 구성도

본 문서의 설치된 시스템 구성도입니다. Pinpoint Server, HBase의 HBase Master2, HBase Slave2, Collector 2, Pinpoint 서비스 브로커, WebUI3로 최소사항을 구성하였다. 

![시스템구성도][pinpoint_image_01]

<table>
  <tr>
    <th>구분</th>
    <th>Resource Pool</th>
    <th>스펙</th>
  </tr>
  <tr>
  <td>Collector/0      </td><td>pinpoint\_medium</td><td>2vCPU / 2GB RAM / 8GB Disk</td>
  </tr>
  <tr>
  <td>Collector/1      </td><td>pinpoint\_medium</td><td>2vCPU / 2GB RAM / 8GB Disk</td>
  </tr>
  <tr>
  <td>h\_master/0      </td><td>pinpoint\_medium</td><td>2vCPU / 2GB RAM / 8GB Disk</td>
  </tr>
  <tr>
  <td>h\_secondary     </td><td>pinpoint\_ small</td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
  <tr>
  <td>h\_slave/0       </td><td>services-small </td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
  <tr>
  <td>h\_slave/1       </td><td>services-small </td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
  <tr>
  <td>haproxy\_webui/0 </td><td>services-small </td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
  <tr>
  <td>pinpoint\_broker/0</<td>services-small </td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
  <tr>
  <td>webui/0          </0><td>services-small </td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
  <tr>
  <td>webui/1          </td><td>services-small </td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
</table>

### <div id='14'> 1.4. 참고자료
[**http://bosh.io/docs**](http://bosh.io/docs)  
[**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/)

# <div id='2'> 2. Pinpoint 서비스팩 설치

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

### <div id='22'> 2.2. Pinpoint 서비스 릴리즈 업로드

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

-	Pinpoint 서비스 릴리즈가 업로드 되어 있지 않은 것을 확인

-	Pinpoint 서비스 릴리즈 파일을 업로드한다.

- **사용 예시**

		$ bosh -e micro-bosh upload-release paasta-pinpoint-2.0.tgz
        Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)
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

-	업로드 된 Pinpoint 릴리즈를 확인한다.

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
		paasta-mysql                      2.0       85e3f01e+  
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
		
-	Pinpoint 서비스 릴리즈가 업로드 되어 있는 것을 확인

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
		
>Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드를 해야 한다. (pinpoint는 stemcell 3309 버전을 사용)

### <div id='23'> 2.3. MySQL 서비스 Deployment 파일 및 deploy-mysql-bosh2.0.sh 수정 및 배포

BOSH Deployment manifest 는 components 요소 및 배포의 속성을 정의한 YAML 파일이다.
Deployment manifest 에는 sotfware를 설치 하기 위해서 어떤 Stemcell (OS, BOSH agent) 을 사용할것이며 Release (Software packages, Config templates, Scripts) 이름과 버전, VMs 용량, Jobs params 등을 정의가 되어 있다.

deployment 파일에서 사용하는 network, vm_type 등은 cloud config 를 활용하고 해당 가이드는 Bosh2.0 가이드를 참고한다.

-	cloud config 내용 조회

- **사용 예시**

		$ bosh -e micro-bosh cloud-config
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

```yaml
# paasta-pinpoint 설정 파일 내용
name: paasta-pinpoint-service                              # 서비스 배포이름(필수)

releases:
- name: paasta-pinpoint                                    # 서비스 릴리즈 이름(필수)
  version: "2.0"                                        # 서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전

stemcells:
- alias: default
  os: ((stemcell_os))                                   # 배포시 사용할 stemcell OS 이름 
  version: "((stemcell_version))"                       # 배포시 사용할 stemcell OS 버전 

update:
  canaries: 1                            # canary 인스턴스 수(필수)
  canary_watch_time: 30000-120000        # canary 인스턴스가 수행하기 위한 대기 시간(필수)
  max_in_flight: 8                       # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)
  update_watch_time: 30000-120000        # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)

instance_groups:
- name: h_slave                          #작업 이름(필수) : Hadoop slave 
  azs:                                   #설치 할 zone 이름
  - z5
  instances: 2                           # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))             # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))       # cloud config 에 정의한 network 이름
    static_ips:                          # 사용할 IP addresses
    - 10.30.107.190
    - 10.30.107.191
  templates:
  - name: h_slave                        # job template 이름(필수)
    release: paasta-pinpoint

- name: h_secondary                      #작업 이름(필수) : Hadoop secondary server
  azs:                                   #설치 할 zone 이름
  - z5
  instances: 1                           # job 인스턴스 수(필수)
  vm_type: ((vm_type_small))             # cloud config 에 정의한 vm_type
  stemcell: default
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.176
  templates:
  - name: h_secondary
    release: paasta-pinpoint

- name: h_master                         #작업 이름(필수) : Hadoop master server
  azs:
  - z5
  instances: 1
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.175
  templates:
  - name: h_master
    release: paasta-pinpoint

- name: collector                          #작업 이름(필수) : pinpoint collector server
  azs:
  - z5
  instances: 2
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.145
    - 10.30.107.146
  templates:
  - name: collector
    release: paasta-pinpoint

- name: haproxy_webui                       #작업 이름(필수) : pinpoint webui haproxy
  azs:
  - z5
  instances: 1
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.178
  - name: ((public_network_name))
    static_ips: 
    - 115.68.47.183
  templates:
  - name: haproxy_webui
    release: paasta-pinpoint

- name: webui                                 #작업 이름(필수) : pinpoint webui server
  azs:
  - z5
  instances: 2
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.179
    - 10.30.107.180
  templates:
  - name: webui
    release: paasta-pinpoint

- name: pinpoint_broker                            #작업 이름(필수) : pinpoint service broker
  azs:
  - z5
  instances: 1
  vm_type: ((vm_type_small))
  stemcell: default
  networks:
  - name: ((default_network_name))
    static_ips:
    - 10.30.107.182
  templates:
  - name: broker
    release: paasta-pinpoint

properties:                                  # Pinpoint 설정정보
  master:
    host_name: h-master                      # Pinpoint master 설정정보
    host_ip: 10.30.107.175
    port: 9000
    http_port: 50070
    replication: 2                           # Pinpoint master 복제개수
    tcp_listen_port: 29994
    udp_stat_listen_port: 29995
    udp_span_listen_port: 29996
  broker:                                    # Pinpoint broker 설정정보
    collector_ips:
    - 10.30.107.145
    - 10.30.107.146
    collector_tcp_port: 29994                           #
    collector_stat_port: 29995
    collector_span_port: 29996
    dashboard_uri: http://115.68.47.183:80/#/main       # Pinpoint dashboard url 설정정보
  secondary:                                  # Pinpoint secondary 설정정보
    host_name: h-secondary
    host_ip: 10.30.107.176
    http_port: 50090
  yarn:                                       # Pinpoint yarn 설정정보
    host_name: h-master
    host_ip: 10.30.107.175
    resource_tracker_port: 8025
    scheduler_port: 8030
    resourcemanager_port: 8040
  slave:                                      # Pinpoint slave 설정정보
    host_names:
    - h-slave0
    - h-slave1
    host_ips:
    - 10.30.107.190
    - 10.30.107.191
  collector:                              # Pinpoint collector 설정정보
    host_names:
    - h-collector0
    - h-collector1
    host_ips:
    - 10.30.107.145
    - 10.30.107.146
  haproxy:                                 # Pinpoint haproxy 설정정보
    host_name: haproxy-webui
    host_ip: 10.30.107.178
    http_port: 80
  webui:                                   # Pinpoint webui 설정정보
    host_names:
    - h-webui0
    - h-webui1
    host_ips:
    - 10.30.107.179
    - 10.30.107.180
```

-	deploy-pinpoint-bosh2.0.sh 파일을 서버 환경에 맞게 수정한다.

```sh
#!/bin/bash
# stemcell 버전은 3309 버전으로 사용하시고 https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Download_Page.md 에서 다운받아 쓰십시요.
# vsphere 인 경우 에는 use-public-network-vsphere.yml 사용하여 public ip를 설정 하고 그 이외의 IaaS는 use-public-network.yml 사용한다.

bosh -e micro-bosh -d paasta-pinpoint-service deploy paasta_pinpoint_bosh2.0.yml \
   -o use-public-network-vsphere.yml \
   -v default_network_name=service_private \
   -v public_network_name=service_public \
   -v stemcell_os=ubuntu-trusty \
   -v stemcell_version=3309 \
   -v vm_type_small=minimal
```

-	Pinpoint 서비스팩을 배포한다.

- **사용 예시**

		$ ./deploy-pinpoint-bosh2.0.sh
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Using deployment 'paasta-pinpoint-service'

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

		+ name: paasta-pinpoint-service

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

-	배포된 Pinpoint 서비스팩을 확인한다.

- **사용 예시**

		$ bosh -e micro-bosh -d paasta-pinpoint-service vms
		Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

		Task 5006. Done

		Deployment 'paasta-pinpoint-service'

		Instance                                              Process State  AZ  IPs            VM CID                                   VM Type  Active  
		collector/5f10f9cf-67ed-4c08-8c14-99d7f7a818c2        running        z5  10.30.107.145  vm-1714d225-85fa-4236-94db-7ed26baa52b3  minimal  true  
		collector/79d48854-7eb3-4f50-92aa-8b2df8380653        running        z5  10.30.107.146  vm-415e14ed-5304-40ec-9529-3f3f7c1902b4  minimal  true  
		h_master/723a34d6-0756-41e2-b11e-1530596cbb09         running        z5  10.30.107.175  vm-037e021b-3dba-4ea3-961c-6bd259d25c4b  minimal  true  
		h_secondary/78b552e3-9b38-4f92-864f-8457e6d0673d      running        z5  10.30.107.176  vm-54d4848a-8001-49e5-8be7-37fd6ca06d6c  minimal  true  
		h_slave/984ea017-9495-4402-8057-613bdee868be          running        z5  10.30.107.190  vm-9cdeec0d-d71a-4133-942b-7439ef0df359  minimal  true  
		h_slave/d32b818e-9d2d-4dbe-856d-f7027605865f          running        z5  10.30.107.191  vm-6aebc5a9-8a78-4388-b876-2c6dc192de71  minimal  true  
		haproxy_webui/22999f9c-0798-43a5-b93b-bc5d44c4d210    running        z5  10.30.107.178  vm-294e229c-aa89-4a5c-9960-4009579bbf54  minimal  true  
											 115.68.47.183                                                      
		pinpoint_broker/7a9d8423-c5f9-4e3c-be16-42e9215f8268  running        z5  10.30.107.182  vm-1b9b0891-8a02-45af-8b0c-ca21bf56e64e  minimal  true  
		webui/30f7c9cf-ab03-4f78-a9bf-96c5148a9ec1            running        z5  10.30.107.180  vm-62d5e876-2ab4-4327-ba54-2f74b53e3da9  minimal  true  
		webui/a0433ae3-2a7b-4c7f-822c-9c1d46c658ea            running        z5  10.30.107.179  vm-365edb16-1b8e-4c65-a984-b03a368d8e95  minimal  true  

		10 vms

		Succeeded

### <div id='24'> 2.4. HBase 기본 데이터 실행

-	Pinpoint 서비스팩 배포가 완료 되었으면 HBase 14개의 기본 Table이 생성되어야 Application에서 서비스 팩을 정상 사용할 수 있다.
h_master 서버에 ssh 로 접속 하여 hbase table 생성 스크립트를 구동한다.

- **사용 예시**

		$ bosh -e micro-bosh -d paasta-pinpoint-service ssh h_master
		
		h_master/fbd4ed1f-92ce-4244-a0bc-ebc089cebe12:/var/vcap/packages/hbase/bin$ sudo mount -o remount, exec /tmp && /var/vcap/packages/hbase/bin/hbase shell /var/vcap/packages/hbase/bin/hbase-create.hbase
		2018-08-28 02:21:54,165 WARN  [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
		0 row(s) in 2.0700 seconds

		0 row(s) in 4.2910 seconds

		0 row(s) in 1.2440 seconds

		0 row(s) in 1.2360 seconds

		0 row(s) in 1.2660 seconds

		0 row(s) in 2.2480 seconds

		0 row(s) in 2.2610 seconds

		0 row(s) in 2.2490 seconds

		0 row(s) in 8.2970 seconds

		0 row(s) in 2.3460 seconds

		0 row(s) in 2.2660 seconds

		0 row(s) in 2.2680 seconds

		0 row(s) in 2.3110 seconds

		0 row(s) in 2.2500 seconds

		TABLE
		AgentEvent
		AgentInfo
		AgentLifeCycle
		AgentStat
		ApiMetaData
		ApplicationIndex
		ApplicationMapStatisticsCallee_Ver2
		ApplicationMapStatisticsCaller_Ver2
		ApplicationMapStatisticsSelf_Ver2
		ApplicationTraceIndex
		HostApplicationMap_Ver2
		SqlMetaData_Ver2
		StringMetaData
		Traces
		14 row(s) in 0.0470 seconds

### <div id='25'> 2.5. Pinpoint 서비스 브로커 등록

Pinpoint 서비스팩 배포가 완료 되었으면 Application에서 서비스 팩을
사용하기 위해서 먼저 Pinpoint 서비스 브로커를 등록해 주어야 한다.

서비스 브로커 등록시 PaaS-TA에서 서비스브로커를 등록 할
수 있는 사용자로 로그인이 되어 있어야 한다.

<br>
-   서비스 브로커 목록을 확인한다.

```
$ cf service-brokers
```
```
Getting service brokers as admin...

name   url
No service brokers found
```

<br>
-   Pinpoint 서비스 브로커를 등록한다.

```
$ cf create-service-broker {서비스브로커 이름} {서비스브로커 사용자ID} {서비스브로커 사용자비밀번호} http://{서비스브로커 URL(IP)}
```
```
서비스브로커 이름 : 서비스브로커 관리를 위해 PaaS-TA에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스브로커 명칭이다.

서비스브로커 사용자ID / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID입니다. 서비스브로커도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.

서비스브로커 URL : 서비스브로커가 제공하는 API를 사용할 수 있는 URL을 입력한다.
```

```
$ cf create-service-broker pinpoint-service-broker admin cloudfoundry http://10.30.107.182:8080
```
```
Creating service broker pinpoint-service-broker as admin...
OK
```

<br>
-   등록된 Pinpoint 서비스 브로커를 확인한다.

```
$ cf service-brokers
```
```
Getting service brokers as admin...
name url
pinpoint-service-broker http:// 10.30.70.82:8080
```
<br>
-   접근 가능한 서비스 목록을 확인한다.

```
$ cf service-access
```
```
Getting service access as admin...
broker: Pinpoint-service-broker
service plan access orgs
Pinpoint Pinpoint\_standard none
```
서비스 브로커 생성시 디폴트로 접근을 허용하지 않는다.

<br>
-   특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)

```
$ cf enable-service-access Pinpoint
```

```
Enabling access to all plans of service Pinpoint for all orgs as admin...
OK
```
<br>
서비스 접근 허용을 확인한다.
```
$ cf service-access
```
```
Getting service access as admin...
broker: Pinpoint-service-broker
service plan access orgs
Pinpoint Pinpoint\_standard all
```

#  <div id='3'> 3. Sample Web App 연동 Pinpoint 연동

본 Sample Web App은 개방형 클라우드 플랫폼에 배포되며 Pinpoint의 서비스를 Provision과 Bind를 한 상태에서 사용이 가능하다.

### <div id='31'> 3.1. Sample Web App 구조

Sample Web App은 PaaS-TA에 App으로 배포가 된다. 배포된
App에 Pinpoint 서비스 Bind 를 통하여 초기 데이터를 생성하게 된다. 바인드
완료 후 연결 url을 통하여 브라우저로 해당 App에 대한 Pinpoint 서비스
모니터링을 할 수 있다.

-   Spring-music App을 이용하여 Pinpoint 모니터링을 테스트 하였다.
-   앱을 다운로드 후 –b 옵션을 주어 buildpack을 지정하여 push 해 놓는다.

```
$ cf push -b java_buildpack_pinpoint --no-start
```

```
Using manifest file /home/ubuntu/workspace/bd_test/spring-music/manifest.yml

Creating app spring-music-pinpoint in org org / space space as admin...
OK

Creating route spring-music-pinpoint.monitoring.open-paas.com...
OK

Binding spring-music-pinpoint.monitoring.open-paas.com to spring-music-pinpoint...
OK

Uploading spring-music-pinpoint...
Uploading app files from: /tmp/unzipped-app175965484
Uploading 21.2M, 126 files
Done uploading               
OK
```

```
$ cf apps
```
```
Getting apps in org org / space space as admin...
OK

name                    requested state   instances   memory   disk   urls
php-demo                started           1/1         256M     1G     php-demo.monitoring.open-paas.com
spring-music            stopped           0/1         512M     1G     spring-music.monitoring.open-paas.com
spring-music-pinpoint   stopped           0/1         512M     1G     spring-music-pinpoint.monitoring.open-paas.com
```

### <div id='32'> 3.2. PaaS-TA에서 서비스 신청

Sample Web App에서 Pinpoint 서비스를 사용하기 위해서는 서비스
신청(Provision)을 해야 한다.

\*참고: 서비스 신청시 PaaS-TA에서 서비스를 신청 할 수 있는
사용자로 로그인이 되어 있어야 한다.

-   먼저 PaaS-TA Marketplace에서 서비스가 있는지 확인을
    한다.

```
$ cf marketplace
```

```
Getting services from marketplace in org org / space space as admin...
OK

service    plans               description
Pinpoint   Pinpoint_standard   A simple pinpoint implementation 
```

<br>
-   Marketplace에서 원하는 서비스가 있으면 서비스 신청(Provision)을 하여
    서비스 인스턴스를 생성한다.

```  
$ cf create-service {서비스명} {서비스플랜} {내서비스명}
```
```
서비스명 : p-Pinpoint로 Marketplace에서 보여지는 서비스 명칭이다.
서비스플랜 : 서비스에 대한 정책으로 plans에 있는 정보 중 하나를 선택한다. Pinpoint 서비스는 10 connection, 100 connection 를 지원한다.
내서비스명 : 내 서비스에서 보여지는 명칭이다. 이 명칭을 기준으로 환경설정정보를 가져온다.

```

```
$ cf create-service Pinpoint Pinpoint_standard PS1
```
```
Creating service instance PS1 in org org / space space as admin...
OK
```

<br>
-   생성된 Pinpoint 서비스 인스턴스를 확인한다.

```
$ cf services
```
```
Getting services in org org / space space as admin...
OK

name             service         plan                bound apps               last operation
app_log_drain    user-provided
PS1              Pinpoint        Pinpoint_standard                            create succeeded
syslog_service   user-provided                       php-demo, spring-music 
```

### <div id='33'> 3.3. Sample Web App에 서비스 바인드 신청 및 App 확인
-------------------------------------------------

서비스 신청이 완료되었으면 Sample Web App 에서는 생성된 서비스
인스턴스를 Bind 하여 App에서 Pinpoint 서비스를 이용한다.

\*참고: 서비스 Bind 신청시 PaaS-TA 플랫폼에서 서비스 Bind신청 할
수 있는 사용자로 로그인이 되어 있어야 한다.

-   Sample Web App에서 생성한 서비스 인스턴스 바인드 신청을 한다.

<br>
- 서비스 인스턴스 확인
```
$ cf s
```
```
Getting services in org org / space space as admin...
OK

name             service         plan                bound apps               last operation
app_log_drain    user-provided
PS1              Pinpoint        Pinpoint_standard                            create succeeded
syslog_service   user-provided                       spring-music, php-demo

<br>
- 서비스 바인드
```
$ cf bind-service spring-music-pinpoint PS1 -c '{"application_name":"spring-music"}'
```
Binding service PS1 to app spring-music-pinpoint in org org / space space as admin...
OK
TIP: Use 'cf restage spring-music-pinpoint' to ensure your env variable changes take effect
```
  
cf cli 리눅스 버전 : 
```
cf bind-service <application이름> PS1 -c ‘{"application_name\":"<application이름>"}
```  

cf cli window 버전 : 
```
cf bind-service <application이름> PS1 -c "{\"application_name\":\"<application이름>\"}"
```
<br>
-   바인드가 적용되기 위해서 App을 restage한다.

```
$ cf restage spring-music-pinpoint
```
```

Restaging app spring-music-pinpoint in org org / space space as admin...
Downloading binary_buildpack...
Downloading go_buildpack...
Downloading staticfile_buildpack...
Downloading java_buildpack...
Downloading ruby_buildpack...
Downloading nodejs_buildpack...
Downloading python_buildpack...
Downloading php_buildpack...
Downloaded python_buildpack
Downloaded binary_buildpack
Downloaded go_buildpack
Downloaded java_buildpack
Downloaded ruby_buildpack
Downloaded nodejs_buildpack
Downloaded staticfile_buildpack
Downloaded php_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (24.5M)
Downloading build artifacts cache...
Downloaded build artifacts cache (54.1M)
Staging...
-----> Java Buildpack Version: v3.7.1 | https://github.com/cloudfoundry/java-buildpack.git#78c3d0a
-----> Downloading Open Jdk JRE 1.8.0_91-unlimited-crypto from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_91-unlimited-crypto.tar.gz (found in cache)
     Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.6s)
-----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
     Memory Settings: -XX:MaxMetaspaceSize=64M -Xss995K -Xmx382293K -Xms382293K -XX:MetaspaceSize=64M
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
-----> Downloading Tomcat Instance 8.0.39 from https://java-buildpack.cloudfoundry.org/tomcat/tomcat-8.0.39.tar.gz (found in cache)
     Expanding Tomcat Instance to .java-buildpack/tomcat (0.1s)
-----> Downloading Tomcat Lifecycle Support 2.5.0_RELEASE from https://java-buildpack.cloudfoundry.org/tomcat-lifecycle-support/tomcat-lifecycle-support-2.5.0_RELEASE.jar (found in cache)
-----> Downloading Tomcat Logging Support 2.5.0_RELEASE from https://java-buildpack.cloudfoundry.org/tomcat-logging-support/tomcat-logging-support-2.5.0_RELEASE.jar (found in cache)
-----> Downloading Tomcat Access Logging Support 2.5.0_RELEASE from https://java-buildpack.cloudfoundry.org/tomcat-access-logging-support/tomcat-access-logging-support-2.5.0_RELEASE.jar (found in cache)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading droplet...
Uploading build artifacts cache...
Uploaded build artifacts cache (54.1M)
Uploaded droplet (77.3M)
Uploading complete

0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK
```
<br>
-   App이 정상적으로 Pinpoint 서비스를 사용하는지 확인한다.

![pinpoint_image_03]

<br>
-  환경변수 확인

```
$ cf env spring-music-pinpoint
```
```
Getting env variables for app spring-music-pinpoint in org org / space space as admin...
OK

System-Provided:
{
"VCAP_SERVICES": {
"Pinpoint": [
 {
  "credentials": {
   "application_name": "spring-music",
   "collector_host": "10.244.2.30",
   "collector_span_port": 29996,
   "collector_stat_port": 29995,
   "collector_tcp_port": 29994
  },
  "label": "Pinpoint",
  "name": "PS1",
  "plan": "Pinpoint_standard",
  "provider": null,
  "syslog_drain_url": null,
  "tags": [
   "pinpoint"
  ],
  "volume_mounts": []
 }
]
}
}

{
"VCAP_APPLICATION": {
"application_id": "b010e6e9-5431-4198-81f8-7d6ba9c14f40",
"application_name": "spring-music-pinpoint",
"application_uris": [
 "spring-music-pinpoint.monitoring.open-paas.com"
],
"application_version": "9a600116-97bd-45da-a33e-3b0d5592b1d0",
"limits": {
 "disk": 1024,
 "fds": 16384,
 "mem": 512
},
"name": "spring-music-pinpoint",
"space_id": "bc70b951-d870-49ca-b57d-5c7137060e5e",
"space_name": "space",
"uris": [
 "spring-music-pinpoint.monitoring.open-paas.com"
],
"users": null,
"version": "9a600116-97bd-45da-a33e-3b0d5592b1d0"
}
}

No user-defined env variables have been set

No running env variables have been set

No staging env variables have been set
```

<br>
- App 정상 구동 확인
```
$ curl http://115.68.151.187/#/main/spring-music-pinpoint@TOMCAT
```

[pinpoint_image_01]:/Service-Guide/images/pinpoint/pinpoint-image1.png
[pinpoint_image_02]:/Service-Guide/images/pinpoint/pinpoint-image2.png
[pinpoint_image_03]:/Service-Guide/images/pinpoint/pinpoint-image3.png
