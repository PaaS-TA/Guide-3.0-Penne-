## Table of Contents

1. [문서 개요](#1)
    * [1.1. 목적](#1-1)
    * [1.2. 범위](#1-2)
2. [Container 서비스 신청](#2)
    * [2.1. PaaS-TA 사용자 포탈 접속](#2-1)
    * [2.2. Container 서비스 신청](#2-2)
    * [2.3. Container 서비스 접속](#2-3)
3. [Container 서비스 사용자 메뉴얼](#3)
    * [3.1. Container 서비스 사용자 메뉴 구성](#3-1)
    * [3.2. Container 서비스 사용자 메뉴 설명](#3-2)
    * [3.2.1. Intro](#3-2-1)
    * [3.2.1.1. Overview](#3-2-1-1)
    * [3.2.1.2. Access](#3-2-1-2)
    * [3.2.1.3. Namespaces](#3-2-1-3)
    * [3.2.1.3.1. Namespace 상세 조회](#3-2-1-3-1)
    * [3.2.1.4. Nodes](#3-2-1-4)
    * [3.2.1.4.1. Node 상세 조회](#3-2-1-4-1)
    * [3.2.2. Users](#3-2-2)
    * [3.2.2.1. User 목록 조회](#3-2-2-1)
    * [3.2.2.2. User Role 변경](#3-2-2-2)
    * [3.2.2.3. User 삭제](#3-2-2-3)
    * [3.2.3. Roles](#3-2-3)
    * [3.2.4. Workloads](#3-2-4)
    * [3.2.4.1. Deployments](#3-2-4-1)
    * [3.2.4.1.1. Deployment 목록 조회](#3-2-4-1-1)
    * [3.2.4.1.2. Deployment 상세 조회](#3-2-4-1-2)
    * [3.2.4.2. Pods](#3-2-4-2)
    * [3.2.4.2.1. Pod 목록 조회](#3-2-4-2-1)
    * [3.2.4.2.2. Pod 상세 조회](#3-2-4-2-2)
    * [3.2.4.3. Replica Sets](#3-2-4-3)
    * [3.2.4.3.1. Replica Set 목록 조회](#3-2-4-3-1)
    * [3.2.4.3.2. Replica Set 상세 조회](#3-2-4-3-2)
    * [3.2.5. Services](#3-2-5)
    * [3.2.5.1. Service 목록 조회](#3-2-5-1)
    * [3.2.5.2. Service 상세 조회](#3-2-5-2)

----

# <div id='1'/> 1. 문서 개요

## <div id='1-1'/> 1.1. 목적
본 문서는 Container 서비스를 사용할 사용자의 사용 방법에 대해 기술하였다.

## <div id='1-2'/> 1.2. 범위
본 문서는 Windows 환경을 기준으로 Container 서비스를 사용할 사용자의 사용 방법에 대해 작성되었다.

# <div id='2'/> 2. Container 서비스 신청

### <div id='2-1'/> 2.1. PaaS-TA 사용자 포탈 접속
1. PaaS-TA 사용자 포탈에 접속하여 "로그인" 버튼을 클릭한다.

![IMG_002]

2. 사용할 이메일 계정과 비밀번호를 입력하고, "SING IN" 버튼을 클릭하여 PaaS-TA 사용자 포탈에 로그인한다.

![IMG_003]

3. 로그인 한 후 카탈로그 페이지로 이동한다.

![IMG_011]

### <div id='2-2'/> 2.2. Container 서비스 신청
- Container 서비스는 조직 별 한 개의 서비스를 생성할 수 있다.

1. 카탈로그 페이지에서 Container 서비스를 선택한다.

![IMG_012]

2. 서비스를 생성하려는 지역, 조직, 공간, 서비스 이름, 서비스 이용사양을 입력하여 Container 서비스 인스턴스를 생성한다.

![IMG_013]


### <div id='2-3'/> 2.3. Container 서비스 접속
1. PaaS-TA 사용자 포탈의 대시보드 페이지에서 서비스 탭을 클릭해 생성한 Container 서비스를 확인한다.

![IMG_014]

2. 생성한 Container 서비스의 "대시보드" 버튼을 클릭하여 Container 서비스 대시보드로 이동한다.

![IMG_016]


## <div id='3'/> 3. Container 서비스 사용자 메뉴얼


## <div id='3-1'/> 3.1. Container 서비스 사용자 메뉴 구성
| <center>메뉴</center> | <center>분류</center> | <center>설명</center> |
| :--- | :--- | :--- |
| Intro | Overview | Container 서비스 대시보드 |
|| Access | Container 서비스 CLI 사용을 위한 환경 설정 정보 관리 |
| Workloads | Overview | Workloads 대시보드 |
|| Deployments | Deployments 정보 관리 |
|| Pods | Pods 정보 관리 |
|| Replica Sets | Replica Sets 정보 관리 |
| Services | Services | Services 대시보드 |
| Users | Users | 사용자 관리 |
| Roles | Roles | Role 조회 |


## <div id='3-2'/> 3.2. Container 서비스 사용자 메뉴 설명
본 장에서는 Container 서비스의 5개 메뉴에 대한 설명을 기술한다.

### <div id='3-2-1'/> 3.2.1. Intro

#### <div id='3-2-1-1'/> 3.2.1.1. Overview
- Namespace 정보, Container 서비스 Plan 정보, Resource Quota 정보를 조회한다.

1. Intro의 Overview 탭을 클릭하여 Overview 페이지로 이동한다.

![IMG_016]


#### <div id='3-2-1-2'/> 3.2.1.2. Access
- Container 서비스의 CLI 사용을 위한 환경 설정 정보를 조회한다.

1. Intro의 Access 탭을 클릭하여 Access 페이지로 이동한다.

![IMG_017]


#### <div id='3-2-1-3'/> 3.2.1.3. Namespaces
##### <div id='3-2-1-3-1'/> 3.2.1.3.1. Namespace 상세 조회

- Namespace 상세 페이지는 Details, Events 탭으로 구성된다.

1. Deployments, Pods, Replica Sets 목록에서 Namespace 명을 클릭하여 Namespace 상세 페이지로 이동한다.

![IMG_047]
![IMG_048]
![IMG_049]


#### <div id='3-2-1-4'/> 3.2.1.4. Nodes
##### <div id='3-2-1-4-1'/> 3.2.1.4.1. Node 상세 조회

- Node 상세 페이지는 Summary, Details, Events 탭으로 구성된다.

1. Pods 목록에서 Node 명을 클릭하여 Node 상세 페이지로 이동한다.

![IMG_050]
![IMG_051]
![IMG_052]


### <div id='3-2-2'/> 3.2.2. Users
- 본 장에서는 Container 서비스를 사용하는 사용자들의 Role 관리 및 정보 조회에 대한 설명을 기술한다.
- 서비스 신청자는 Administrator Role, 그 외 신규 사용자는 Init User Role을 부여 받는다.
- User 삭제 및 Role 변경은 Administrator Role을 가진 User만이 가능하다.


#### <div id='3-2-2-1'/> 3.2.2.1. User 목록 조회
1. Container 서비스를 사용하는 User 목록을 조회한다.

![IMG_059]


#### <div id='3-2-2-2'/> 3.2.2.2. User Role 변경
1. User Role을 변경하여 저장[①] 버튼을 클릭한다.

![IMG_062]

2. 팝업창의 "확인" 버튼을 클릭하여 해당 User Role을 변경한다.

![IMG_063]

3. User 목록에서 Role이 변경되었음을 확인한다.

![IMG_064]


#### <div id='3-2-2-3'/> 3.2.2.3. User 삭제
1. 삭제할 User의 삭제[①] 버튼을 클릭한다.

![IMG_065]

2. 팝업창의 "확인" 버튼을 클릭하여 User를 삭제한다.

![IMG_066]

3. User 목록에서 해당 User가 삭제되었음을 확인한다.

![IMG_067]


### <div id='3-2-3'/> 3.2.3. Roles
1. Container 서비스의 사용 가능한 권한을 조회한다.

![IMG_069]


### <div id='3-2-4'/> 3.2.4. Workloads
-  Overview, Deployments, Pods, Replica Sets 목록을 조회한다.

![IMG_020]

#### <div id='3-2-4-1'/> 3.2.4.1. Deployments

##### <div id='3-2-4-1-1'/> 3.2.4.1.1. Deployment 목록 조회
1. Workloads의 Deployment 탭을 클릭하여 Deployment 목록 페이지로 이동한다.

![IMG_028]

##### <div id='3-2-4-1-2'/> 3.2.4.1.2. Deployment 상세 조회
- Deployment 상세 페이지는 Details, Events, YAML 탭으로 구성된다.


1. Deployment 목록에서 Deployment 명을 클릭하여 Deployment 상세 페이지로 이동한다.

![IMG_034]
![IMG_036]
![IMG_037]


#### <div id='3-2-4-2'/> 3.2.4.2. Pods

##### <div id='3-2-4-2-1'/> 3.2.4.2.1. Pod 목록 조회
1. Workloads의 Pods 탭을 클릭하여 Pod 목록 페이지로 이동한다.

![IMG_029]

##### <div id='3-2-4-2-2'/> 3.2.4.2.2. Pod 상세 조회
- Pod 상세 페이지는 Details, Events, YAML 탭으로 구성된다.

1. Pod 목록에서 Pod 명을 클릭하여 Pod 상세 페이지로 이동한다.

![IMG_043]
![IMG_045]
![IMG_046]


#### <div id='3-2-4-3'/> 3.2.4.3. Replica Sets

##### <div id='3-2-4-3-1'/> 3.2.4.3.1. Replica Set 목록 조회
1. Workloads의 Replica Sets 탭을 클릭하여 Replica Set 목록 페이지로 이동한다.

![IMG_030]

##### <div id='3-2-4-3-2'/> 3.2.4.3.2. Replica Set 상세 조회
- Replica Set 상세 페이지는 Details, Events, YAML 탭으로 구성된다.

1. Replica Set 목록에서 Replica Set 명을 클릭하여 Replica Set 상세 페이지로 이동한다.

![IMG_039]
![IMG_041]
![IMG_042]


### <div id='3-2-5'/> 3.2.5. Services 메뉴

#### <div id='3-2-5-1'/> 3.2.5.1. Service 목록 조회
1. Services 메뉴를 클릭하여 Service 목록 페이지로 이동한다.

![IMG_053]


#### <div id='3-2-5-2'/> 3.2.5.2. Service 내용 조회
- Service 상세 페이지는 Details, Events, YAML 탭으로 구성된다.

1. Service 목록에서 Service 명을 클릭하여 Service 상세 페이지로 이동한다.

![IMG_054]
![IMG_056]
![IMG_057]


----

[IMG_002]:/Use-Guide/images/caas/CAAS-002.jpg
[IMG_003]:/Use-Guide/images/caas/CAAS-003.png
[IMG_011]:/Use-Guide/images/caas/CAAS-011.png
[IMG_012]:/Use-Guide/images/caas/CAAS-012.png
[IMG_013]:/Use-Guide/images/caas/CAAS-013.png
[IMG_014]:/Use-Guide/images/caas/CAAS-014.png
[IMG_016]:/Use-Guide/images/caas/CAAS-016.png
[IMG_017]:/Use-Guide/images/caas/CAAS-017.png
[IMG_020]:/Use-Guide/images/caas/CAAS-020.png
[IMG_028]:/Use-Guide/images/caas/CAAS-028.png
[IMG_029]:/Use-Guide/images/caas/CAAS-029.png
[IMG_030]:/Use-Guide/images/caas/CAAS-030.png
[IMG_034]:/Use-Guide/images/caas/CAAS-034.png
[IMG_036]:/Use-Guide/images/caas/CAAS-036.png
[IMG_037]:/Use-Guide/images/caas/CAAS-037.png
[IMG_039]:/Use-Guide/images/caas/CAAS-039.png
[IMG_041]:/Use-Guide/images/caas/CAAS-041.png
[IMG_042]:/Use-Guide/images/caas/CAAS-042.png
[IMG_043]:/Use-Guide/images/caas/CAAS-043.png
[IMG_045]:/Use-Guide/images/caas/CAAS-045.png
[IMG_046]:/Use-Guide/images/caas/CAAS-046.png
[IMG_047]:/Use-Guide/images/caas/CAAS-047.png
[IMG_048]:/Use-Guide/images/caas/CAAS-048.png
[IMG_049]:/Use-Guide/images/caas/CAAS-049.png
[IMG_050]:/Use-Guide/images/caas/CAAS-050.png
[IMG_051]:/Use-Guide/images/caas/CAAS-051.png
[IMG_052]:/Use-Guide/images/caas/CAAS-052.png
[IMG_053]:/Use-Guide/images/caas/CAAS-053.png
[IMG_054]:/Use-Guide/images/caas/CAAS-054.png
[IMG_056]:/Use-Guide/images/caas/CAAS-056.png
[IMG_057]:/Use-Guide/images/caas/CAAS-057.png
[IMG_059]:/Use-Guide/images/caas/CAAS-059.png
[IMG_062]:/Use-Guide/images/caas/CAAS-062.png
[IMG_063]:/Use-Guide/images/caas/CAAS-063.png
[IMG_064]:/Use-Guide/images/caas/CAAS-064.png
[IMG_065]:/Use-Guide/images/caas/CAAS-065.png
[IMG_066]:/Use-Guide/images/caas/CAAS-066.png
[IMG_067]:/Use-Guide/images/caas/CAAS-067.png
[IMG_069]:/Use-Guide/images/caas/CAAS-069.png
