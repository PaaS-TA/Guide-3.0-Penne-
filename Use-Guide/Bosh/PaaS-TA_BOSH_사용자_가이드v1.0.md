## Table of Contents

1. [개요](#1)
  * [목적](#2)
  * [범위](#3)
  * [참고자료](#4)
2. [Basic BOSH Workflow](#5)
	* [BOSH](#6)
	* [BOSH Component 구성](#7)
3. [BOSH 설치 환경 구성](#8)
	* [BOSH 설치 절차](#9)
    * [Inception 서버 구성](#10)
    	*  [Install BOSH-INIT](#11)
    	*  [Install Ruby](#12)
    	*  [Install BOSH-CLI](#13)
    *  [IaaS 환경 설정](#14)
    *  [스템셀 다운로드](#15)
    *  [릴리즈 다운로드](#16)
    *  [배포 Manifest 파일 작성](#17)
    *  [BOSH 배포](#18)

## Executive Summary

본 문서는 BOSH의 설명 및 설치하는 가이드 문서로 BOSH를 실행할 수 있는 환경을 구성하여 실행하고 사용하는 방법에 대해서 설명하였다.

# <div id='1'/>1.  문서 개요 

## <div id='2'/>1.1.  목적
클라우드 환경에 서비스 시스템을 배포할 수 있는 BOSH는 릴리즈 엔지니어링, 개발, 소프트웨어 라이프사이클 관리를 통합한 오픈소스 프로젝트로 본 문서에서는 Inception 서버 환경에 BOSH를 설치하여 기능을 테스트할 수 있는 환경을 구축하는 데 목적이 있다. 

## <div id='3'/>1.2.  범위
본 가이드에서는 Linux 환경(Ubuntu 14.04)을 기준으로 BOSH 설치를 위한 패키지 및 라이브러리를 설치, 구성을 하고 이를 이용하여 BOSH를 설치하는 것을 기준으로 작성 하였다.

## <div id='4'/>1.3.  참고자료

본 문서는 Cloud Foundry의 BOSH Document와 Cloud Foundry Document를 참고로 작성하였다.
BOSH Document: [http://bosh.io](http://bosh.io)

Cloud Foundry Document: [https://docs.cloudfoundry.org/](https://docs.cloudfoundry.org/)


# <div id='5'/>2. Basic BOSH Workflow
본 문서의 목적에서도 언급했듯이 BOSH는 클라우드 환경에 서비스를 배포 관리하는 소프트웨어로 BOSH 자체도 클라우드에 배포 되어야 하는 서비스로 설치 환경이 구성되어 있는 Inception 서버로부터 BOSH를 설치하고 BOSH의 Director를 통해 PaaS-TA 및 다양한 소프트웨어 설치, 실제 앱 환경 구성을 할 수 있다. 

![PaaSTa_BOSH_Use_Guide_Image1]


## <div id='6'/>2.1. BOSH

BOSH는 초기에 Cloud Foundry PaaS를 위해 개발 되었지만 현재는 jenkins, Hadoop 등 Yaml 파일 형식으로 소프트웨어를 쉽게 배포할 수 있으며 수 백 가지의 VM을 설치 할 수 있고 각 각의 VM에 대해 모니터링, 장애복구 등 라이프 사이클을 관리 할 수 있는 통합 프로젝트이다.

BOSH가 지원이 되는 IaaS는 vSphere, Google Cloud Platform, AWS, OpenStack, Azure, vCloud, RackHD, SoftLayer가 있다.
현재 PaaS-TA에서 지원 되는 IaaS 항목은 vSphere, AWS, Openstack, Google Cloud Platform이 지원이 되고 향 후 Azure가 추가 될 예정이다.

## <div id='7'/>2.2. BOSH Component 구성

BOSH의 컴포넌트 구성은 다음과 같다.

![PaaSTa_BOSH_Use_Guide_Image2]

1.	Command Line Interface(CLI): Director와 상호작용을 위한 Command Line Interface
2.	Director: Director가 VM을 생성 또는 수정할 때 설정 정보를 레지스트리에 저장한다. 저장된 레지스트리 정보는 VM의 bootstrapping stage에서 이용된다.
3.	Database: Director가 사용하는 Postgres 데이터베이스로 Deployment시에 필요로하는 Stemcell / Release / Deployment의 메타 정보들을 저장한다.
4.	Message Bus(Nats): Message Bus는 Director와 Agent간 통신하기 위한 publish-subscribe 방식의 Message System으로 VM 모니터링과 특정 명령을 수행하기 위해 사용된다.
5.	Registry: VM 생성을 위한 설정 정보 저장
6.	Health Monitor: Health Monitor는 BOSH Agent로부터 클라우드의 상태정보들을 수집한다. 클라우드로부터 특정 Alert이 발생하면 Resurrector를 하거나 Notification Plug-in을 통해 Alert Message를 전송할 수도 있다.
7.	Agent: Agent는 클라우드에 배포되는 모든 VM에 설치되고 Director로부터 특정 명령을 받고 수행하는 역할을 하게된다. Agent는 Director로부터 수신 받은 Job Specification(설치할 패키지 및 구성 방법)정보로부터 해당 VM에 Director의 지시대로 지정된 패키지를 설치하고, 필요한 구성정보를 설정하게 된다.

# <div id='8'/>3. BOSH 설치 환경 구성

## <div id='9'/>3.1. BOSH 설치 절차

![PaaSTa_BOSH_Use_Guide_Image3]

## <div id='10'/>3.2. Inception 서버 구성
Inception 서버는 BOSH 설치와 BOSH의 Director를 설정하여 PaaS-TA를 설치 하기 위해 필요한 패키지 및 라이브러리, Manifest 파일 등의 환경을 가지고 있는 배포 작업 및 실행 서버이다. 환경 구성에 있어서 전제조건으로 Inception 서버는 외부와 통신이 가능해야 한다.

BOSH 및 PaaS-TA 설치를 위해 Inception 서버에 구성해야 할 환경은 다음과 같다.

-	BOSH-INIT (version 0.0.103 이상)
-	RUBY (verson 1.9.3 이상)
-	BOSH-CLI (version 1)
	
### <div id='11'/>3.2.1 Install BOSH-INIT

Bosh-init은 BOSH의 디렉터 VM과 각 컴포넌트들을 생성하고 업데이트 하는데 사용하는 도구이다.

Bosh-init을 설치하는 절차는 다음과 같다.
Bosh_init 설치
※ 다음의 웹사이트에서 설치환경에 맞는 bosh-init 바이너리 파일을 다운로드한다.

[https://bosh.io/docs/install-bosh-init.html](https://bosh.io/docs/install-bosh-init.html)

![PaaSTa_BOSH_Use_Guide_Image4]

※ 다운로드 받은 bosh-init에 실행 권한을 부여하고 설치한다. (~/Download 디렉토리에 파일을 다운로드 받았다고 가정)

	$ chmod +x ~/Downloads/bosh-init-*
	$ sudo mv ~/Downloads/bosh-init-* /usr/local/bin/bosh-init

※ bosh-init 설치 확인
	
	$ bosh-init -v

### <div id='12'/>3.2.2 Install Ruby
BOSH는 Ruby언어로 구현되어 있으므로 Ruby개발 환경 설치 후 BOSH CLI를 설치해야 한다. BOSH CLI설치 가능한 Ruby버전은 1.9.3 이상이다.

※ BOSH Dependency 및 Ruby 설치

	$ sudo apt-get update
	$ sudo apt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt-dev libxml2-dev libssl-dev libreadline6 libreadline6-dev libyaml-dev libsqlite3-dev sqlite3 libxslt1-dev libpq-dev libmysqlclient-dev unzip

※ Ruby 설치 확인

	$ ruby -v

### <div id='13'/>3.2.2 Install BOSH-CLI
BOSH-CLI는 BOSH의 Director와 상호 작용하는데 사용 되는 명령 행 인터페이스이다.
Ruby언어로 구현이 되어 있으며 BOSH-CLI를 통하여 Director를 로그인 및 설정하고 VM을 설치 및 운영 관리 할 수 있다.

※ BOSH-CLI 설치

	$ sudo gem install bosh_cli --no-ri --no-rdoc

※ BOSH-CLI 설치 확인

	$ bosh

※ 기타 BOSH-CLI(Command Line Interface) 사용 방법은 아래 링크에서 참고 한다.

[BOSH-CLI](https://github.com/PaaS-TA/Guide-1.0-Spaghetti-/blob/master/Use-Guide/OpenPaaS_PaaSTA_BOSH_CLI_guide.md )

## <div id='14'/>3.3. IaaS 환경 설정
다음은 BOSH를 설치 하기 위해 IaaS(AWS, Openstack, Google) 환경에 네트워크 설정, Key Pair 설정, 보안 그룹 설정 등을 설정하는 방법 이다. 설치 할 환경에 따라 가이드 문서를 확인한다.

AWS: [AWS 환경 설정 가이드](../IaaS/PaaS-TA_AWS_환경설정_가이드v1.0.md)

Openstack:  [Openstack 환경 설정 가이드](../IaaS/PaaS-TA_Openstack_환경설정_가이드v1.0.md)

Google Cloud Platform: [Google 환경 설정 가이드](../IaaS/PaaS-TA_GCP_환경설정_가이드v1.0.md)

## <div id='15'/>3.4. 스템셀 다운로드

스템셀은 IaaS의 특정 패키징으로 포장 된 버전 관리 OS 이미지로 기본 OS 이미지의 구성을 가지고 있다. 스템셀에는 BOSH Agent를 포함하고 있으며 Agent를 통해 해당 bosh의 Blobstore로부터 Job을 배포 한다. 

BOSH 설치를 위해 다운로드 할 스템셀은 다음과 같다. 설치 할 인프라 환경에 따라 스템셀을 선택하여 다운로드 한다.

<table>
<tr>
<th>인프라 환경</th>
<th>참조 사이트</th>
<th>버전</th>
</tr>

<tr>
<td>AWS</td>
<td>http://bosh.io/stemcells/bosh-aws-xen-hvm-ubuntu-trusty-go_agent</td>
<td>3445.2</td>
</tr>

<tr>
<td>Openstack</td>
<td>http://bosh.io/stemcells/bosh-openstack-kvm-ubuntu-trusty-go_agent</td>
<td>3445.2</td>
</tr>

<tr>
<td>vSphere</td>
<td>http://bosh.io/stemcells/bosh-vsphere-esxi-ubuntu-trusty-go_agent</td>
<td>3445.2</td>
</tr>

 <tr>
<td>Google</td>
<td>http://bosh.io/stemcells/bosh-vsphere-esxi-ubuntu-trusty-go_agent</td>
<td>3312</td>
</tr>
</table>

※ Command Line 스템셀 이미지 다운로드 방법

	# Openstack
	$ wget --content-disposition https://bosh.io/d/stemcells/bosh-openstack-kvm-ubuntu-trusty-go_agent?v=3445.2

	# AWS
	$ wget --content-disposition https://s3.amazonaws.com/bosh-aws-light-stemcells/light-bosh-stemcell-3445.2-aws-xen-hvm-ubuntu-trusty-go_agent.tgz

	# vSphere
	$ wget --content-disposition https://bosh.io/d/stemcells/bosh-vsphere-esxi-ubuntu-trusty-go_agent?v=3445.2

	# GCP
	$ wget https://s3.amazonaws.com/bosh-gce-light-stemcells/light-bosh-stemcell-3312-google-kvm-ubuntu-trusty-go_agent.tgz

## <div id='16'/>3.5. 릴리즈 다운로드
릴리즈는 해당 소프트웨어나 패키지의 구성 정보와 시작과 중지 스크립트 등이 존재하며 소프트웨어를 빌드하고 실행 시키는데 필요한 Package와 Job의 모음이다.

-	Packages에는 Package와 package 간 의존 관계 및 설치 스크립트 등이 존재 한다.
-	Job에는 VM을 배포 시 적용 되는 설정 파일과 프로세스 시작/종료에 관한 스크립트 파일 등이 존재 한다.

BOSH를 설치 하기 위해서는 다음과 같은 릴리즈가 필요 하다.

※ BOSH-RELEASE

1.	BOSH Release는 BOSH의 주요 컴포넌트를 설치 할 때 필요 한 패키지와 패키지의 의존성 시작/중지 스크립트, WAS의 설정 파일 등이 존재하는 릴리즈 파일이다.

※ BOSH-CPI-RELEASE

1.	BOSH CPI Release는 BOSH에서 IaaS의 리소스 제어를 위해 정의한 추상화 Interface이다.
2.	BOSH CPI Release는 IaaS의 API와의 연동 및 제어를 구현한 추상화 Interface이다.
3.	CPI의 주요 구성
	-	Stemcell의 관리
	-	VM의 관리
	-	Disk의 관리
	-	스냅샷의 관리

BOSH 설치를 위해 다운로드 할 릴리즈는 다음과 같다. BOSH-Release는 공통으로 쓰이는 릴리즈이고 설치 할 인프라 환경에 따라 BOSH-CPI-Release를 선택하여 다운로드 한다.

<table>
<tr>
<th>릴리즈 명</th>
<th>참조 사이트</th>
<th>버전</th>
</tr>

<tr>
<td>BOSH</td>
<td>http://bosh.io/releases/github.com/cloudfoundry/bosh?all=1</td>
<td>261</td>
</tr>

<tr>
<td>BOSH-AWS-CPI</td>
<td>http://bosh.io/releases/github.com/cloudfoundry-incubator/bosh-aws-cpi-release?all=1</td>
<td>50</td>
</tr>

<tr>
<td>BOSH-Openstack-CPI</td>
<td>http://bosh.io/releases/github.com/cloudfoundry-incubator/bosh-openstack-cpi-release?all=1</td>
<td>30</td>
</tr>

 <tr>
<td>vSphere</td>
<td>http://bosh.io/releases/github.com/cloudfoundry-incubator/bosh-vsphere-cpi-release?all=1</td>
<td>27</td>
</tr>


 <tr>
<td>Google</td>
<td>http://bosh.io/releases/github.com/cloudfoundry-incubator/bosh-google-cpi-release?all=1</td>
<td>25.0.0</td>
</tr>

</table>

※ Command Line 릴리즈 다운로드 방법

	# BOSH-Release
	$ wget http://bosh.io/d/github.com/cloudfoundry/bosh?v=261

	# BOSH-AWS-CPI-Release
	$ wget http://bosh.io/d/github.com/cloudfoundry-incubator/bosh-aws-cpi-release?v=50

	# BOSH-Openstack-CPI-Release
	$ wget http://bosh.io/d/github.com/cloudfoundry-incubator/bosh-openstack-cpi-release?v=30

	# BOSH-vSphere-CPI-Release
	$ wget http://bosh.io/d/github.com/cloudfoundry-incubator/bosh-vsphere-cpi-release?v=27

	# BOSH-Google-CPI-Release
	$ wget http://bosh.io/d/github.com/cloudfoundry-incubator/bosh-google-cpi-release?v=25.0.0

## <div id='17'/>3.6. 배포 Manifest 파일 작성

배포는 IaaS 환경 별 스템셀 이미지를 기반으로 다 수의 VM 생성 하는 것이며 특정 릴리즈의 Package, Job과 배포 Manifest 파일에 명시 되어 있는 Resource(VM의 Disk Size, 소프트웨어 정보, VM의 네트워크 정보 등)를 BOSH의 Director에 업로드를하고 Director를 통해 리소스를 할당 하여 IaaS에 VM 및 을 생성 한다.

BOSH 설치 Manifest 파일은 배포에 필요한 컴포넌트 및 속성 정보를 YAML 파일 형식으로 구성이 되어 있으며 크게 Release 정보를 설정 하는 Block, Network 정보를 설정하는 Block, Resource Pools 스템셀과 CPI 정보를 설정하는 Block, Disk Pool 사이즈를 설정하는 Block, Job 정보를 설정하는 Bloc, Property를설정하는 Block 등이 존재한다.

※ 다음은 BOSH 설치 Manifest 파일의 Block 설명 이다.

1.	Deployment Identification: 배포 명과 Director의 고유 UUID를 설정하는 Block
2.	Releases Block: 배포 시 사용할 Release 명과 버전을 설정하는 Block
3.	Networks Block: 배포 시 할당 할 IaaS의 네트워크 정보를 설정하는 Block
4.	Resource Pools Block: BOSH가 설치 하고 관리 하는 VM의 속성 Block
5.	Disk Pools Block: BOSH가 작성하고 관리하는 Disk Pool의 등록 정보를 설정하는 Block
6.	Compilation Block: VM의 컴파일 속성을 설정하는 Block
7.	Update Block: 배포 중에 BOSH가 작업 인스턴스를 업데이트하는 방법을 정의하는 Block
8.	Jobs Block: Job에 대한 구성 및 자원을 설정하는 Block
9.	Properties Block: 전역 속성과 일반화 된 구성 정보(config)를 설정 하는 Block

위에서 언급 했듯이 현재 PaaS-TA에서 지원 되는 IaaS 항목은 vSphere, AWS, Openstack, Google Cloud Platform이 지원이 되고 향 후 Azure가 추가 될 예정이다.

※ 설치 할 BOSH 설치 Manifest 파일을 아래의 링크에서 다운로드 한다.

 [다운로드](http://115.68.46.186:8080/data/packages/3.0/PaaSTA-Deployment.zip)

설치 할 Manifest 파일의 정보를 IaaS 환경에 맞게 편집 후 저장 한다.

<table>
<tr>
<th>IaaS 환경</th>
<th>Manifest 파일 명</th>

</tr>

<tr>
<td>AWS</td>
<td>httppaasta-microbosh-aws-3.0.yml</td>

</tr>

<tr>
<td>Openstack</td>
<td>httppaasta-microbosh-openstack-3.0.yml</td>
</tr>
   
<tr>
<td>vSphere</td>
<td>paasta-microbosh-vsphere-3.0.yml</td>
</tr>

 <tr>
<td>GCP</td>
<td>paasta-microbosh-gcp-3.0.yml</td>
</tr>
</table>

## <div id='18'/>3.7. BOSH 배포
Bosh-init을 이용해 MicroBOSH를 배포한다.

※ 터미널 상에 bosh-init의 deploy를 실행한다.

	$ bosh-init deploy <작성한 manifest>

Deploy를 실행하면 manifest파일 있는 디렉토리에 <manifest명>-state.json 파일이 생긴다. 해당 Json 파일은 BOSH 설치 시 VM의 아이디, DISK 아이디, 스템셀 아이디, 릴리즈 아이디 등 정보가 등록 되어 있다.
BOSH를 삭제할 경우 필요한 파일 이므로 삭제하지 않는다. 만약 해당 Json 파일이 존재 하지 않을 경우에 삭제가 실행 되지 않거나 오류가 발생 한다.


※ 터미널 상에 bosh-init의 deploy 배포 메시지

	Deployment manifest: '/home/min/.bosh_plugin/deployment/microbosh.yml'
	Deployment state: '/home/min/.bosh_plugin/deployment/microbosh-state.json'

	Started validating
  	Validating release 'bosh'... Finished (00:00:02)
  	Validating release 'bosh-cpi'... Finished (00:00:00)
  	Validating cpi release... Finished (00:00:00)
  	Validating deployment manifest... Finished (00:00:00)
  	Validating stemcell... Finished (00:00:07)
	Finished validating (00:00:10)

	Started installing CPI
  	Compiling package 'ruby_cpi/9485b5753d4609e92e1491ff991cb28fbde81445'... Finished (00:01:43)
  	Compiling package 'bosh_cpi/d75668418cb713fbffdd94fd80072c0f09e70e3d'... Finished (00:00:01)
  	Installing packages... Finished (00:00:00)
  	Rendering job templates... Finished (00:00:00)
  	Installing job bosh_cpi'... Finished (00:00:00)
	Finished installing CPI (00:01:46)

	Starting registry... Finished (00:00:00)
	Uploading stemcell 'bosh-kvm-ubuntu-trusty-go_agent/3445.2'... Finished (00:00:45)

	Started deploying
	Creating VM for instance 'bosh/0' from stemcell 'dcca3765-c056-41a8-bb0a-345325c7b3c3'... Finished (00:00:53)
	…………………………………………………………(이하 생략)


※ 배포가 성공하면 다음과 같이 Target 설정하면서 기본 계정인 admin/admin으로 로그인한다.

	$ bosh target <외부 IP>
	$ name: admin
	$ password: admin

※ BOSH의 상태 정보를 확인 한다.

$ bosh status

	Config
	             /home/inception/.bosh_config

	Director
	RSA 1024 bit CA certificates are loaded due to old openssl compatibility
	  Name       bosh
	  URL        https://172.16.100.62:25555
	  Version    261.0.0 (00000000)
	  User       admin
	  UUID       006984d5-b52b-4214-8d41-dbaeed951a81
	  CPI        iaas_cpi
	  dns        disabled
	  compiled_package_cache disabled
	  snapshots  disabled
	  config_server disabled

※ BOSH를 삭제 할 경우에는 이전에 언급했듯이 state.json 파일이 필요하며 CPI 릴리즈가 로컬 환경에 반드시 존재 해야 한다.

	$ bosh-init delete <작성한 manifest>

※ bosh-init 배포에 대한 상세한 로그를 출력하고 싶은 경우, 다음과 같이 설정하고 bosh-init을 실행한다. 
참고 문서: [BOSH-INIT-LOG-LEVEL](https://bosh.io/docs/using-bosh-init.html#logging) 

	#bosh-init 로그출력 레벨 설정
	# 설정 가능한 로그 레벨은 debug, info, warn, error, none로 none이 기본 설정되어 있다.
	$ export BOSH_INIT_LOG_LEVEL=<로그 레벨>


[PaaSTa_BOSH_Use_Guide_Image1]:../images/Bosh/boshworkFlow.png
[PaaSTa_BOSH_Use_Guide_Image2]:../images/Bosh/Component.png
[PaaSTa_BOSH_Use_Guide_Image3]:../images/Bosh/InstrallProcess.png
[PaaSTa_BOSH_Use_Guide_Image4]:../images/Bosh/bosh-init.png
