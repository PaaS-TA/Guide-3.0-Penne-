## 1.    PaaS 플랫폼 배포를 위한 Openstack 환경 설정

#### Executive Summary
본 문서는 PaaS-TA가 지원하는 IaaS 환경 중 Openstack에 대한 BOSH 설치 환경 설정하는 방법에 
대해서 설명하였다.

## 1.1   목적
본 문서에서는 Inception 서버 환경에 BOSH를 설치하여 기능을 테스트할 수 있는 환경을 구축 시 Openstack 기본 환경 설정을 하는 데 목적이 있다. 

## 1.2   범위
본 가이드에서는 Linux 환경(Ubuntu 14.04 TRUSTY) 을 설치환경 구성 기준으로 작성 하였으며, 전제 조건으로는 Openstack 계정이 있어야 한다.

## 1.3   참고자료
본 문서는 Cloud Foundry의 BOSH Document와 Openstack Document를 참고로 작성하였다.

- BOSH Document: http://bosh.io
- Openstack Document: https://docs.openstack.org/newton/

## 2.1.  OpenStack 환경설정

BOSH를 배포하기 위한 OpenStack 사전 준비사항 및 필요한 사항을 아래의 가이드를 참조하여 설정한다.

####   사전 준비 및 확인 사항

-   지원 가능한 OpenStack Release 버전: Liberty, Mitaka, Newton(권장), Ocata

-   OpenStack 서비스 사용 가능 확인

    -   Identity: 인증 및 OpenStack 서비스들의 Endpoint 조회

    -   Compute: VM 생성과 IP 할당, 볼륨 생성 및 Attach

    -   Image: 이미지 서비스를 이용한 Stemcell 저장

-   OpenStack 네트워크

    -   외부 또는 내부 네트워크의 Subnet

	-   Floating IP

-   OpenStack 프로젝트 생성


### -   Key Pair 생성

######  STEP 1

1.  오픈스택 horizon의 왼쪽 메뉴에서 [Project] -> [Compute] -> [Access & Security]를 차례대로 선택한다.

2.  Access & Security 화면의 [Key Pair] 탭을 클릭한다.

3.  Access & Security 화면 오른쪽의 [Create Key Pair] 버튼을 클릭한다

	![openstack-keypair1]

######  STEP 2

1.  Key Pair 이름을 입력 후 [Create Key Pair] 버튼을 클릭한다.

2.  Key를 생성하면 로컬에 다운로드 한다. (예제에서는 microbosh.pem 파일을 다운로드)

3.  다운로드한 Key를 로컬의 키 디렉토리(~/.ssh) 에 이동시키고 권한을 400으로 변경한다.

	![openstack-keypair2]

### -   Security Group 생성 및 구성

######  STEP1

1.  오픈스택 왼쪽 메뉴에서 [Project] -> [Compute] -> [Access & Security]를 차례대로 선택한다.

2.  Access & Security 화면의 [Security Group] 탭을 클릭한다.

3.	Access & Security 화면 오른쪽의 [Create Security Group] 버튼을 클릭한다.

	![openstack-security1]

######  STEP2

1.  생성할 Security Group의 Name, Description 입력 후 [Create Security Group] 버튼을 클릭한다.

	![openstack-security2]

######  STEP3

1.  Security Group 목록에서 생성된 Security Group의 [Manage Rules] 버튼을 클릭한다.

	![openstack-security3]

######  STEP4

1.  아래의 표와 같이 BOSH 관련 Security Group Rule을 추가한다.

	![openstack-security4]


	(예제에서는 모든 원격지에서 ssh접근을 허용하게 설정되어 있다. 설치 환경에 맞게 접근 할 수 있는 원격지를 제한 할 수 있다.) 

### -   Network 생성

######  STEP1

1.	오픈스택 DashBoard 왼쪽 메뉴에서 [Project] -> [Network] -> [Networks]를 차례대로 선택한다.

2.	Networks 화면 오른쪽의 [Create Network] 버튼을 클릭한다.

   - External과 Internal 네트워크를 구성한다.

![openstack-network1]

######  STEP2

1.  Create Network 팝업 화면의 Network 탭에서 Network 명을 입력하고 [Next] 버튼을 클릭한다.

2.	Create Network 팝업 화면의 Subnet 탭에서 Subnet 명, Network Range, Gateway를 추가한다.

	![openstack-network2]

######  STEP3

1.  Create Network 팝업 화면에서 DNS를 입력 하고 [Next] 버튼을 클릭한다.

	![openstack-network3]

### -   Router 생성

######  STEP1

1.	오픈스택 DashBoard 왼쪽 메뉴에서 [Project] -> [Network] -> [Router] 차례대로 선택한다.

2.	Routers 화면 오른쪽의 [Create Router] 버튼을 클릭한다.

	![openstack-router1]

######  STEP2

1.  Create router 팝업 화면에서 Router 명을 입력하고 [Create router] 버튼을 클릭한다.

	![openstack-router2]

### -   Network Router 연결

######  STEP1

1.  오픈스택 DashBoard 왼쪽 메뉴에서 [Project] -> [Network] -> [Routers]를 차례대로 선택한다.

2.	Routers 화면의 목록에서 Router 명을 선택한다.

	- External, Internal Subnet을 Router 에 연결한다.

	![openstack-routercon1]

######  STEP2

1.  Router Details 화면에서 [Add Interface] 버튼을 클릭한다.

	![openstack-routercon2]

######  STEP3

1.	Add Interface 팝업 화면의 Subnet 드롭 메뉴에서 생성한 Subnet을 선택하고 [Add Interface] 버튼을 클릭한다.

	![openstack-routercon3]

### -   Floating IP 생성

######  STEP1

1.	오픈스택 DashBoard 왼쪽 메뉴에서 [Project] -> [Compute] -> [Access & Security]를 차례대로 선택한다.

2.	Access & Security 화면의 [Floating IPs] 탭을 클릭한다.

3.	Access & Security 화면 오른쪽의 [Allocate IP To Project] 버튼을 클릭한다.

	![openstack-floatingip1]

######  STEP2

1.	Pool 드롭 다운 메뉴에서 External 을 선택한다.

2.	[Allocate IP] 버튼을 클릭한다.

	![openstack-floatingip2]

[openstack-keypair1]:../images/IaaS/Openstack/opstk_keypair1.png
[openstack-keypair2]:../images/IaaS/Openstack/opstk_keypair2.png
[openstack-security1]:../images/IaaS/Openstack/opstk_security1.png
[openstack-security2]:../images/IaaS/Openstack/opstk_security2.png
[openstack-security3]:../images/IaaS/Openstack/opstk_security3.png
[openstack-security4]:../images/IaaS/Openstack/opstk_security4.png
[openstack-network1]:../images/IaaS/Openstack/opstk_network1.png
[openstack-network2]:../images/IaaS/Openstack/opstk_network2.png
[openstack-network3]:../images/IaaS/Openstack/opstk_network3.png
[openstack-router1]:../images/IaaS/Openstack/opstk_router1.png
[openstack-router2]:../images/IaaS/Openstack/opstk_router2.png
[openstack-routercon1]:../images/IaaS/Openstack/opstk_routercon1.png
[openstack-routercon2]:../images/IaaS/Openstack/opstk_routercon2.png
[openstack-routercon3]:../images/IaaS/Openstack/opstk_routercon3.png
[openstack-floatingip1]:../images/IaaS/Openstack/opstk_floatingip1.png
[openstack-floatingip2]:../images/IaaS/Openstack/opstk_floatingip2.png
