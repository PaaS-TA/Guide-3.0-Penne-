## Table of Contents
1. [문서 개요](#1)
    * [1.1. 목적](#1.1)
    * [1.2. 범위](#1.2)
    * [1.3. 운영자 포탈 설치](#1.3)
2. [PaaS-TA 운영자 포탈 시작](#2)
    * [2.1. PaaS-TA 운영자 포탈 로그인](#2.1)
    * [2.2. PaaS-TA 운영자 포탈 로그](#2.2)
3. [PaaS-TA 운영자 포탈 대시보드](#3)
    * [3.1. 대시보드 정보 조회](#3.1)
4. [운영 관리 메뉴](#4)
    * [4.1. 조직 및 공간 조회](#4.1)    
    * [4.2. 클라이언트](#4.2)    
    * [4.3. 카탈로그](#4.3)    
    * [4.3.1 카탈로그 관리_앱 템플릿](#4.3.1)
    * [4.3.2. 카탈로그 관리_앱 개발환경](#4.3.2)
    * [4.3.3. 카탈로그 관리_서비스](#4.3.3)
    * [4.4. 사용자 관리](#4.4)    
    * [4.5. 권한 관리](#4.5)    
    * [4.6. 코드 관리](#4.6)   
    * [4.7. 설정 정보](#4.7)
    * [4.8. 할당량 관리](#4.8)      
    * [4.9. 분할 관리](#4.9)  
5. [서비스 관리](#5)
    * [5.1. 빌드팩](#5.1)
    * [5.2. 서비스 브로커](#5.2)
    * [5.3. 서비스 제어](#5.3)
6. [보안 관리](#6)
    * [6.1. 시큐리티 그룹](#6.1)


## <a name="1"/>1.  문서 개요

### <a name="1.1"/>   1.1. 목적

본 문서는 JAVA로 작성된 PaaS-TA 운영자 포탈 웹 사이트를 사용하고 관리하는 방법에 대해 설명한다.

### <a name="1.2"/>   1.2. 범위

이미 설치된 PaaS-TA 운영자 포탈 웹 사이트 사용하는 방법에 대해 기술하였다.

### <a name="1.3"/>  1.3. 운영자 포탈 설치

본 사용 가이드는 설치에 대한 내용은 기술하지 않았다.

 운영자 포탈 설치는 [PaaS-TA Portal 배포 가이드 문서](https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/v3.5/Install-Guide/Portal/PaaS-TA%20Portal%20%EC%84%A4%EC%B9%98%EA%B0%80%EC%9D%B4%EB%93%9C.md)를 참고한다.

## <a name="2"/>2.  PaaS-TA 운영자 포탈 시작

PaaS-TA 운영자 포탈 웹 사이트에 접속한다.

### <a name="2.1"/>  2.1. PaaS-TA 운영자 포탈 로그인

1. PaaS-TA 운영자 포탈에 접속하면 인증을 위한 로그인 화면이 나타난다.
![01]

2. 아이디와 비밀번호를 입력후 "로그인" 버튼을 클릭한다.
![02]

### <a name="2.2"/>  2.2. PaaS-TA 운영자 포탈 로그아웃

1. 로그인 후 메인화면 ① "로그아웃" 버튼을 클릭하여 로그아웃을 한다.
![03]

## <a name="3"/>3.  PaaS-TA 운영자 포탈 대시보드

PaaS-TA에 등록된 조직, 공간, APP, 사용자 통계 정보를 보여준다.

### <a name="3.1"/>  3.1. 대시보드 정보 조회

1. 조직의 ①공간, ②APP, ③사용자 정보를 보여준다.<br>
![04] 

2. ①조직명을 클릭하면 조직의 영역 상세 정보를 확인 할 수 있다.<br>
![05]

## <a name="4"/>4.  운영 관리 메뉴

PaaS-TA 포탈의 운영에 필요한 관리를 수행하기 위한 메뉴이다.<br>

### <a name="4.1"/>  4.1. 조직 및 공간 조회

1. ①조직과 ②공간을 조회한다. ③검색어 조회도 가능하다.<br>
![06]

2. 조직이름 옆의 ①상세정보 버튼을 클릭한다. ②조직 상세 정보를 조회한다.<br>
![07]

3. 공간이름 옆의 ①상세정보 버튼을 클릭한다. ②공간 영역 상세 정보를 조회한다.<br>
![08]


### <a name="4.2"/>  4.2. 클라이언트

1. 클라이언트 목록을 조회한다.<br>    
![09]

2. **클라이언트 ID**를 클릭 후 클라이언트 상세화면을 이동한다. 클라이언트 상세화면에서 "삭제" 버튼을 클릭하면 클라이언트 ID 삭제가 가능하다.<br>
![10]

3. ①"클라이언트 등록" 버튼을 클릭한다. 클라이언트 아이디, 패스워드, 옵션, 옵션 가이드를 입력하고 ②"등록" 버튼을 클릭하여 등록한다.<br>
![11]

### <a name="4.3"/>  4.3. 카탈로그
PaaS-TA 포탈의 관리에 필요한 기능을 수행하기 위한 메뉴이다.
![12]

### <a name="4.3.1"/>  4.3.1 카탈로그 관리 앱 템플릿
1. 카탈로그 앱 템플릿 목록을 조회한다. 이름, 요약, 분류, 공개여부를 조회할 수 있다.    
검색창에서 검색유형, 검색항목, 공개여부, 검색어로 검색이 가능하다.
![13]

2. 앱 템플릿 이름을 클릭하면 상세화면이 나온다. 수정할 내역이 있을 경우 수정 한 후 ①"저장"버튼을 클릭하여 저장한다.
![14]

3. ②"삭제" 버튼을 클릭하여 앱 템플릿을 삭제한다.
![15]

4. 앱 템플릿을 등록 시, ①"앱 템플릿 등록" 버튼을 클릭한다. 앱 템플릿 등록 팝업에 이름, 분류, 앱 개발환경, 서비스, 썸네일, 공개여부, 요약, 설명을 입력한다. ②"등록" 버튼을 누르면 새로운 앱 템플릿이 등록된다.
![16]

### <a name="4.3.2"/>  4.3.2 카탈로그 관리 앱 개발환경
1. 카탈로그 앱 개발환경 목록을 조회한다. 이름, 요약, 분류, 공개여부를 조회할 수 있다.
검색창에서 검색유형, 검색항목, 공개여부, 검색어로 검색이 가능하다.
![17]

2. 앱 개발환경 이름을 클릭하면 상세화면이 나온다. 수정 내역 완료 후 ①"저장"버튼을 클릭하여 저장(수정)한다.
![18]

3. ②"삭제" 버튼을 클릭하여 앱 개발환경을 삭제한다.
![19]

4. 카탈로그 화면에서 앱 개발환경을 등록하기 위해 ①"앱 개발환경 등록" 버튼을 클릭한다. 앱 개발환경 등록 팝업에 이름, 분류, 앱 개발환경, 서비스, 썸네일, 공개여부, 요약, 설명을 입력한다. ②"등록" 버튼을 누르면 새로운 앱 개발환경이 등록된다.
![20]

### <a name="4.3.3"/>  4.3.3 카탈로그 관리 서비스
1. 카탈로그 서비스 목록을 조회한다. 이름, 요약, 분류, 공개여부를 조회할 수 있다. 검색창에서 검색유형, 검색항목, 공개여부, 검색어로 검색이 가능하다.<br>
![21]

2. 서비스 이름을 클릭하면 상세화면이 나온다. 수정 내역 완료 후 ①"저장"버튼을 클릭하여 저장한다.<br>
![22]

3. ②"삭제" 버튼을 클릭하여 서비스를 삭제한다.<br>
![23]

4. 카탈로그 화면에서 앱 서비스를 등록 시,  ①"앱 서비스 등록버튼"을 클릭한다. 
이름, 분류, 서비스, 썸네일, 파라미터, 공개여부, 요약, 설명을 입력한다. ②"등록" 버튼을 누르면 새로운 서비스가 등록된다. <br>
![24]

※ **delivery-pipeline** 앱 서비스 등록 방법 예시
1. 앱 서비스 등록 폼 화면에 앱 서비스 등록에 필요한 내용을 입력한다.
2. 앱 서비스에 등록할 이름을 입력한다. 
3. 분류는 "개발 지원 도구"로 지정한다.
4. 서비스는 "delivery-pipeline-v2" 로 지정한다.
5. 썸네일는 사용자가 등록하고자 하는 이미지를 지정하여 선택한다. 
     > **썸네일 적용 시 'PaaS-TA 사용자'에서 카탈로그 서비스 리스트 및 서비스 등록 시 이미지를 확인 할 수 있다.**<br>
6. 문서 URL을 넣어준다. 
     > (예) https://github.com/PaaS-TA/DELIVERY-PIPELINE-UI <br>
     > **문서 URL 적용 시 'PaaS-TA 사용자'카탈로그 서비스 등록 '문서보기' 버튼 활성화 된다.**
     > ![pipeline-app-service-create]

7. 서비스 생성시 필요한 파라미터값을 설정한다.
    (예) owner : '자동입력' /  org_name : '자동입력'
8. 앱 바인드 사용 은 "N"으로 선택한다.
9. 대시보드 사용 시 "Y" 를 필수 설정한다. 대시보드 "N" 설정시 대시보드 버튼이 비 활성화 된다.
     > **대시보드 적용 시 'PaaS-TA 사용자'에서 카탈로그 앱 서비스 "대시보드" 버튼 활성화 된다**
     > ![pipeline-app-dashboard]
10. 그 외 공개여부, 요약, 설명을 입력한다. "저장" 버튼을 클릭하여 서비스 등록을 완료한다.
     > ![pipeline]

     
※ **p-sourcecontrol** 앱 서비스 등록 방법 예시
1. 앱 서비스 등록 폼 화면에 앱 서비스 등록에 필요한 내용을 입력한다.
2. 앱 서비스에 등록할 이름을 입력한다. 
3. 분류는 "개발 지원 도구"로 지정한다.
4. 서비스는 "p-paasta-sourcecontrol" 로 지정한다.
5. 썸네일는 사용자가 등록하고자 하는 이미지를 지정하여 선택한다.
6. 문서 URL을 넣어준다.
     > (예) https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/master/Service-Guide/Tools/PaaS-TA%20%ED%98%95%EC%83%81%EA%B4%80%EB%A6%AC%20%EC%84%9C%EB%B9%84%EC%8A%A4%ED%8C%A9%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C_v1.0.md <br>
     > **문서 URL 적용 시 'PaaS-TA 사용자'카탈로그 서비스 등록 '문서보기' 버튼 활성화 된다.**<br>
7. 서비스 생성시 필요한 파라미터값을 설정한다.
    (예) owner : '자동입력' /  org_name : '자동입력'
8. 앱 바인드 사용 은 "N"으로 선택한다.
9. 대시보드 사용 시 "Y" 를 필수 설정한다. 대시보드 "N" 설정 시 대시보드 버튼이 비 활성화 된다.
     > **대시보드 적용 시 'PaaS-TA 사용자'에서 카탈로그 앱 서비스 "대시보드" 버튼 활성화 된다**
10. 그 외 공개여부, 요약, 설명을 입력한다. "저장" 버튼을 클릭하여 서비스 등록을 완료한다.
     > ![sourcecontroller]

### <a name="4.4"/>  4.4. 사용자 관리

1. 사용자 리스트를 조회한다. 사용자 계정, 이름, 전화번호, 관리자유무, 승인상태를 조회 할 수 있다. 검색창에서 검색어로 검색할 수 있다.<br>
![25]

2. ①"관리" 버튼 클릭 후 "비밀번호 초기화"를 클릭하여 비밀번호를 초기화 한다.<br>
![26]

3. ①"관리" 버튼 클릭 후 "운영자권한 삭제"를 클릭하여 운영권한 삭제를 한다.<br>
![28]
![27]
     > "실행" 버튼 클릭 후 사용자 관리 리스트 확인 시[관리자] **Y**에서 **N** 으로 변한다. 사용자 관리 리스트 [관리자] **N** 경우 ①"관리" 버튼 클릭 후 "운영자권한 부여"를 클릭한다. <br>
     > ![27-1]
4. ①"관리" 버튼 클릭 후 "가입 해제"를 클릭하여 가입을 해제한다.<br>
![30]
![31]
     > "수정" 버튼 클릭 후 사용자 관리 리스트를 확인 시 [상태] **승인**에서 **비승인**으로 변한다. <br>
     > ![31-1]
5. ①"관리" 버튼 클릭 후 "계정 삭제"를 클릭하여 해당 계정을 삭제한다.<br>
![32]
![33]

### <a name="4.5"/>  4.5. 권한 관리

1. 권한 그룹과 사용자를 조회한다.<br>
![34]

2. ①"그룹 생성" 버튼을 클릭 후 그룹 이름을 입력한다. ②"저장" 버튼 클릭하여 권한그룹을 생성한다.   
![35]

3. ① 권한 그룹에서 삭제할 그룹을 선택한다. 확인 폼 레이어가 팝업되면 ②"삭제"를 클릭하여 권한 그룹을 삭제한다.
![36]

4. ① 권한 그룹 리스트에서 해당 그룹을 클릭하여 사용자를 확인한다. ②"유저 등록" 버튼을 클릭한다. ③ 리스트에서 사용자를 클릭한다. ④"저장"버튼을 클릭하여 유저를 등록한다.
![37]

5. ① 권한 그룹 리스트에서 해당 그룹을 클릭하여 사용자를 확인한다. 사용자 리스트에서 삭제할 사용자 ②체크박스 선택한다. ③ 활성화 된 "유저 삭제" 버튼을 클릭한다. 팝업의 ④ "삭제" 를 클릭하여 유저를 삭제한다.
![38]

### <a name="4.6"/>  4.6. 코드 관리

1. PaaS-TA 포탈에서 사용하는 코드를 코드그룹별로 조회한다. 사용여부, 검색유형, 검색어로도 조회가 가능하다.<br>
![39]

2. 코드 그룹의 코드 ID를 클릭하면 상세 코드 정보가 조회된다.<br>
![40] 

3. **Group Table** 에서 ①"등록" 버튼을 클릭한다. 팝업 창에서 코드ID와 코드이름을 입력하고 ②"등록" 버튼을 클릭하여 코드 등록을 한다.
![41]
![42]

4. **Detail Table** 에서 ①"등록" 버튼을 클릭한다. 코드 상세 등록 팝업 창에서 Key, Value, 요약, 사용여부를 입력하고 ②"등록"버튼을 클릭하여 코드를 등록(추가)한다.<br>
![43]
![44]

5. **Detail Table** 에서 해당 코드(Key, Value, 요약, 사용)를 클릭한다. 코드 상세 팝업에서 ①"삭제" 버튼을 클릭하여 코드 삭제를 한다.
![45]

### <a name="4.7"/>  4.7. 설정 정보

1. PaaS-TA 포탈에서 사용하는 설정 정보들을 조회한다.
![60]

2. PaaS-TA 포탈에서 사용하는 설정 정보들을 수정하고 저장한다.
![62]

### <a name="4.8"/>  4.8. 할당량 관리

PaaS-TA 포탈의 조직 및 공간 할당량 관리를 수행하기 위한 메뉴이다.<br>
![46]

1. "조직 할당량 등록" 버튼을 클릭하여 조직 할당량을 등록한다.<br>
![47]

2. 조직 할당량 등록 팝업에 이름, 메모리, 인스턴스 메모리, 라우트, 서비스 인스턴스, APP 인스턴스, 무료여부, 예약된 라우트 포트를 입력한다. "등록" 버튼을 클릭하여 조직 할당량을 등록한다.<br>
![48]

3. 수정 할 조직 할당량을 클릭한다. 조직 할당량 상세 팝업에서 ① "저장" 버튼을 클릭하여 조직 할당량을 저장(수정)한다.
![49]

4. 삭제 할 조직 할당량을 클릭한다. 조직 할당량 상세 팝업에서 ② "삭제" 버튼을 클릭하여 조직 할당량을 삭제한다.
![49]

5. "공간 할당량 등록" 버튼을 클릭하여 공 할당량을 등록한다.<br>
![58]

6. 공간 할당량 등록 팝업에 이름, 생성조직명, 메모리, 인스턴스 메모리, 라우트, 서비스 인스턴스, APP 인스턴스, 무료여부, 예약된 라우트 포트를 입력한다. "등록" 버튼을 클릭하여 공간 할당량을 등록한다..<br>
![59]

7. 수정 할 공간 할당량을 클릭한다. 공간 할당량 상세 팝업에서 ①"저장" 버튼을 클릭하여 공간 할당량을 저장(수정)한다.
![61]

8. 삭제 할 공간 할당량을 클릭한다. 공간 할당량 상세 팝업에서 ② "삭제" 버튼을 클릭하여 공간 할당량을 삭제한다.
![61]

### <a name="4.9"/>  4.9. 분할 관리

1. PaaS-TA 포탈에서 사용하는 분활 관리 내역을 조회한다.<br>
![isolationSegments01]

2. Segment Name 입력 칸에 생성할 이름을 입력한다. "Segment 등록" 버튼을 클릭하여 등록한다.<br>
![isolationSegments02]

3. 분할 관리 리스트 이름을 클릭하면 조직 정보가 조회된다. ① "Add Organization" 리스트에서 추가 할 조직을 클릭하여 추가한다.
![isolationSegments03]

4. Organization Name 리스트 이름을 클릭하면 조직 상세 정보가 조회된다. ① "Isolation Segments Name" 리스트에서 모든 분할관리에 포한 된 조직을 확인 할 수 있다.
![isolationSegments04]

5. ①"삭제" 버튼을 클릭하여 분할 관리를 삭제한다.
![isolationSegments05]


## <a name="5"/>5.  서비스 관리

PaaS-TA 포탈의 서비스 관리를 수행하기 위한 메뉴이다.<br>

### <a name="5.1"/>  5.1. 빌드팩

1. PaaS-TA 포탈에서 사용하는 빌드팩 정보들을 조회한다.<br>
![50]

2. 빌드팩 순서를 변경하기 위해 순서 콤보박스를 변경한다.<br>
![51]

3. 빌드팩 사용유무를 변경하기 위해 사용유무 체크박스를 변경한다.<br>
![52]

4. 빌드팩 Lock유무를 변경하기 위해 Lock 체크박스를 변경한다.<br>
![53]

### <a name="5.2"/>  5.2 서비스 브로커

1. PaaS-TA 포탈에서 사용하는 서비스 브로커 내역을 조회한다.<br>
![54]

2. 서비스 브로커 리스트의 서비스 이름을 클릭하면 해당 서비스 브로커 상세를 볼 수 있다. 수정할 내역이 있을 경우 수정 후 ①"저장" 버튼을 클릭하여 서비스 브로커를 저장한다.<br>
![55]

3. **5.2- 2** ②"삭제" 버튼을 클릭하여 서비스 브로커를 삭제한다.<br>

4.  서비스 브로커 리스트의 ①"서비스 브로커 등록" 버튼을 클릭한다. 서비스 브로커 이름, 사용자 이름, 사용자 패스워드,URL 정보를 입력한다. ②"등록"버튼을 클릭하면 서비스 브로커 등록이 완료된다.<br>
![57]

### <a name="5.3"/>  5.3 서비스 제어

1. PaaS-TA 포탈에서 사용하는 서비스 내역을 조회한다.<br>
![63]

2. 서비스 제어 리스트의 서비스를 클릭하면 해당 서비스 또는 서비스 플랜에 대한 권한 활성화를 볼 수 있다. ①"접근" 리스트를 클릭하여 하위 집합에 대한 권한 활성화 변경이 가능하다. ② 리스트 서비스 또는 서비스 플랜에 지정된 조직 권한을 부여할 수 있다.<br>
**"접근"이 ALL(활성화)의 경우, 하나 또는 모든 조직에 대한 서비스 또는 서비스 플랜 대한 액세스를 'NONE(비 활성화)'으로 변경해야한다.**<br>
![64]

3. ① "Add Organization" 리스트에서 권한 추가 할 조직을 클릭하여 추가한다.
![65]

4. ①"삭제" 버튼을 클릭하여 조직을 삭제한다. 삭제가 완료 되면, 처음의 접근 상태 "ALL"로 변경된다. <br>
![66]
![67]


## <a name="6"/>6.  보안 관리

PaaS-TA 포탈의 보안 관리를 수행하기 위한 메뉴이다.<br>
    
### <a name="6.1"/>  6.1 시큐리티 그룹

1. PaaS-TA 포탈에서 사용하는 시큐리티 그룹 정보들을 조회한다.<br>
![securitygroups01]

2. ①"시큐리티 그룹 등록" 버튼을 클릭하여 시큐리트 그룹을 등록한다. <br>
![securitygroups02]

3. 시큐리티 그룹 등록 팝업에 이름을 입력(필수)하고 staging default설정(필수), running default설정(필수) 체크박스를 선택한다. 팝업의 탭의 내용 입력 부분에 설명, 규칙(필수), 로그, 포트, 프로토콜을 입력한다. ① "+" 를 클릭하여 탭을 증가시킨다. ② "텝 삭제" 버튼을 클릭하여 탭을 삭제한다.<br>
![securitygroups03]

4. 시큐리티 그룹 등록 팝업에서 ①"파일 내보내기" 버튼을 클릭하여 시큐리트 그룹 정보를 ②.txt 파일로 확인 할 수 있다.
![securitygroups04]

5. 시큐리티 그룹 등록 팝업에서 ①"파일 불러오기" 버튼을 클릭하여 시큐리트 그룹 정보를 ②.txt 파일로 확인 할 수 있다. 불러온 파일은 추가 된 새 탭에서 확인 할 수 있다.
![securitygroups05]

6. 시큐리티 그룹 등록 팝업에서 ①"org"를 클릭한다. ②"space"를 클릭한다. ③"등록" 버튼을 클릭하여 org,space 를 등록한다.
![securitygroups06]

7. 시큐리티 그룹 등록 팝업에서 ①"삭제" 버튼을 클릭하여 org,space 를 삭제한다. ②"등록 "버튼을 클릭하여 시큐리티 그룹 정보를 저장한다.
![securitygroups07]

8. 수정 할 시큐리티 그룹을 선택한다. 시큐리티 그룹 상세 팝업에서 ①"수정" 버튼을 클릭하여 시큐리티 그룹을 수정한다.<br>
![securitygroups08]

9. 삭제 할 시큐리티 그룹을 선택한다. 시큐리티 그룹 상세 팝업에서 ①"삭제" 버튼을 클릭하여 시큐리티 그룹을 삭제한다.<br>
![securitygroups09] 



[01]:../images/admin-portal/portal-web-admin-01.png
[02]:../images/admin-portal/portal-web-admin-02.png					
[03]:../images/admin-portal/portal-web-admin-03.png					
[04]:../images/admin-portal/portal-web-admin-04.png					
[05]:../images/admin-portal/portal-web-admin-05.png					
[06]:../images/admin-portal/portal-web-admin-06.png					
[07]:../images/admin-portal/portal-web-admin-07.png					
[08]:../images/admin-portal/portal-web-admin-08.png					
[09]:../images/admin-portal/portal-web-admin-09.png					
[10]:../images/admin-portal/portal-web-admin-10.png					
[11]:../images/admin-portal/portal-web-admin-11.png					
[12]:../images/admin-portal/portal-web-admin-12.png					
[13]:../images/admin-portal/portal-web-admin-13.png					
[14]:../images/admin-portal/portal-web-admin-14.png					
[15]:../images/admin-portal/portal-web-admin-15.png					
[16]:../images/admin-portal/portal-web-admin-16.png					
[17]:../images/admin-portal/portal-web-admin-17.png					
[18]:../images/admin-portal/portal-web-admin-18.png					
[19]:../images/admin-portal/portal-web-admin-19.png					
[20]:../images/admin-portal/portal-web-admin-20.png	
[21]:../images/admin-portal/portal-web-admin-21.png
[22]:../images/admin-portal/portal-web-admin-22.png					
[23]:../images/admin-portal/portal-web-admin-23.png					
[24]:../images/admin-portal/portal-web-admin-24.png					
[25]:../images/admin-portal/portal-web-admin-25.png					
[26]:../images/admin-portal/portal-web-admin-26.png					
[27]:../images/admin-portal/portal-web-admin-27.png
[27-1]:../images/admin-portal/portal-web-admin-27-1.png	
[28]:../images/admin-portal/portal-web-admin-28.png					
[29]:../images/admin-portal/portal-web-admin-29.png					
[30]:../images/admin-portal/portal-web-admin-30.png					
[31]:../images/admin-portal/portal-web-admin-31.png		
[31-1]:../images/admin-portal/portal-web-admin-31-1.png		
[32]:../images/admin-portal/portal-web-admin-32.png					
[33]:../images/admin-portal/portal-web-admin-33.png					
[34]:../images/admin-portal/portal-web-admin-34.png					
[35]:../images/admin-portal/portal-web-admin-35.png					
[36]:../images/admin-portal/portal-web-admin-36.png					
[37]:../images/admin-portal/portal-web-admin-37.png					
[38]:../images/admin-portal/portal-web-admin-38.png					
[39]:../images/admin-portal/portal-web-admin-39.png					
[40]:../images/admin-portal/portal-web-admin-40.png					
[41]:../images/admin-portal/portal-web-admin-41.png
[42]:../images/admin-portal/portal-web-admin-42.png					
[43]:../images/admin-portal/portal-web-admin-43.png					
[44]:../images/admin-portal/portal-web-admin-44.png					
[45]:../images/admin-portal/portal-web-admin-45.png					
[46]:../images/admin-portal/portal-web-admin-46.png					
[47]:../images/admin-portal/portal-web-admin-47.png					
[48]:../images/admin-portal/portal-web-admin-48.png					
[49]:../images/admin-portal/portal-web-admin-49.png					
[50]:../images/admin-portal/portal-web-admin-50.png					
[51]:../images/admin-portal/portal-web-admin-51.png					
[52]:../images/admin-portal/portal-web-admin-52.png					
[53]:../images/admin-portal/portal-web-admin-53.png					
[54]:../images/admin-portal/portal-web-admin-54.png					
[55]:../images/admin-portal/portal-web-admin-55.png					
[56]:../images/admin-portal/portal-web-admin-56.png					
[57]:../images/admin-portal/portal-web-admin-57.png					
[58]:../images/admin-portal/portal-web-admin-58.png					
[59]:../images/admin-portal/portal-web-admin-59.png	
[60]:../images/admin-portal/portal-web-admin-60.png	
[61]:../images/admin-portal/portal-web-admin-61.png	
[62]:../images/admin-portal/portal-web-admin-62.png	
[63]:../images/admin-portal/portal-web-admin-63.png	
[64]:../images/admin-portal/portal-web-admin-64.png	
[65]:../images/admin-portal/portal-web-admin-65.png	
[66]:../images/admin-portal/portal-web-admin-66.png	
[67]:../images/admin-portal/portal-web-admin-67.png	
[pipeline]:../images/admin-portal/portal-web-admin-pipeline.png
[pipeline-app-service-create]:../images/admin-portal/pipeline-app-service-create.png
[pipeline-app-dashboard]:../images/admin-portal/pipeline-app-dashboard.png
[sourcecontroller]:../images/admin-portal/portal-web-admin-sourcecontroller.png
[securitygroups01]:../images/admin-portal/securitygroups01.png			
[securitygroups02]:../images/admin-portal/securitygroups02.png	
[securitygroups03]:../images/admin-portal/securitygroups03.png	
[securitygroups04]:../images/admin-portal/securitygroups04.png	
[securitygroups05]:../images/admin-portal/securitygroups05.png	
[securitygroups06]:../images/admin-portal/securitygroups06.png	
[securitygroups07]:../images/admin-portal/securitygroups07.png	
[securitygroups08]:../images/admin-portal/securitygroups08.png	
[securitygroups09]:../images/admin-portal/securitygroups09.png	
[isolationSegments01]:../images/admin-portal/isolationSegments01.png	
[isolationSegments02]:../images/admin-portal/isolationSegments02.png	
[isolationSegments03]:../images/admin-portal/isolationSegments03.png	
[isolationSegments04]:../images/admin-portal/isolationSegments04.png	
[isolationSegments05]:../images/admin-portal/isolationSegments05.png	


