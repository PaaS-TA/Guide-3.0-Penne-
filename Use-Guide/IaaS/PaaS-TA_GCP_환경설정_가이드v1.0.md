## Table of Contents

1. [개요](#1)
  * [목적](#2)
  * [범위](#3)
  * [참고자료](#4)
2. [Google Cloud Platform 기본 환경 설정](#5)
	* [Google Cloud Platform 기본 환경 설정 절차](#6)
		* [Google Cloud Platform 프로젝트 생성](#7)
		* [Google Cloud Platform API Enable 설정](#8)
		* [Google Cloud Platform Service Account 생성](#9)
		* [Google Cloud Platform Service Account Key 생성](#10)
		* [Google Cloud Platform VPC networks 생성](#11)
		* [Google Cloud Platform External IP Address 생성](#12)
		* [Google Cloud Platform Firewall rules 설정](#13)
		* [Google Cloud Platform Key pair 생성](#14)
		* [Google Cloud Platform Meta Data 생성](#15)


## Executive Summary

본 문서는 PaaS-TA가 지원하는 IaaS 환경 중 Google에 대한 BOSH 설치 환경 설정하는 방법에 대해서 설명하였다.

# <div id='1'/>1.  문서 개요 

## <div id='2'/>1.1.  목적
본 문서에서는 Inception 서버 환경에 BOSH를 설치하여 기능을 테스트할 수 있는 환경을 구축 시 Google Cloud Platform (이하 GCP) 기본 환경 설정을 하는 데 목적이 있다. 

## <div id='3'/>1.2.  범위
본 가이드에서는 Linux 환경(Ubuntu 14.04 TRUSTY) 을 설치환경 구성 기준으로 작성 하였으며, 전제 조건으로는 Google Cloud Platform 계정이 있어야 한다.

## <div id='4'/>1.3.  범위
본 문서는 Cloud Foundry의 BOSH Document와 Google Cloud Platform Document를 참고로 작성하였다.

BOSH Document: http://bosh.io

GCP Document: https://cloud.google.com/docs/

# <div id='5'/>2.  Google Cloud Platform 환경 설정

BOSH는 클라우드 환경에 서비스를 배포 관리하는 소프트웨어로 BOSH 자체도 클라우드에 배포 되어야 
하는 서비스이다. 본 문서에서는 BOSH를 Google Cloud Platform 환경에서 배포 시 필요한 기본 환경 설정에 목적이 있다.

## <div id='6'/>2.1. Google Cloud Platform 기본 환경 설정 절차
Google Cloud Platform 기본 환경 설정 절차는 다음과 같다.

-	Google Cloud Platform 프로젝트 생성 
-	Google Cloud Platform API Enable 설정 
-	Google Cloud Platform Service Account 생성 및 권한 관리
-	Google Cloud Platform Service Account Key 생성
-	Google Cloud Platform VPC networks생성
-	Google Cloud Platform External IP Address생성
-	Google Cloud Platform Firewall Rules 설정 
-	Google Cloud Platform KeyPair 생성
-	Google Cloud Platform Meta Data생성

### <div id='7'/>2.1.1. Google Cloud Platform 프로젝트 생성

위에서 범위에서 언급 하였듯이 사전에  Google Cloud Platform 계정이 존해 해야 한다.

1.https://console.cloud.google.com GCP 콘솔 접속

![PaaSTa_Google_Use_Guide_Image1]

2.추가 버튼을 눌러 프로젝트를 생성한다.

![PaaSTa_Google_Use_Guide_Image2]

### <div id='8'/>2.1.2. Google Cloud Platform API Enable 설정
- API를 통하여 Token 및 데이터를 가져오기 위해 API를 Enable 설정 한다.

		Enable the GCE API for your project
		Enable the IAM API for your project
		Enable the Cloud Resource Manager API

1.Google Cloud Platform 콘솔 화면에서 API Manager메뉴를 선택한다.
![PaaSTa_Google_Use_Guide_Image3]

2.Library 메뉴를 선택한다.
![PaaSTa_Google_Use_Guide_Image4]


3.GCP API를 Enable 하기 위해 compute를 검색창에 쓰고, 검색 결과 목록에서 Google Compute Engine API (GCE API)를 클릭한다.

4.GCE API 상태를 Enable로 변경 한다.
![PaaSTa_Google_Use_Guide_Image5]

5.위와 같은 방법으로 IAM, Cloud Resource Manager API의 상태도 활성화 한다. 


### <div id='9'/>2.1.3. Google Cloud Platform Service Account 생성

1.Google Cloud Platform IAM & ADMIN 메뉴의 하위 Service accounts 메뉴에서 Service Account를 생성 한다.
![PaaSTa_Google_Use_Guide_Image6]

![PaaSTa_Google_Use_Guide_Image7]

![PaaSTa_Google_Use_Guide_Image8]

2.Service Account 권한
- 보안 상의 이유로 Owner 권한을 주면 안될 경우 
	- Compute Instance Admin, Compute Network Admin, Compute Network User, 
	- Compute Storage Admin, Service Account Actor, Storage Admin 5가지 권한을 부여 하면 된다.

![PaaSTa_Google_Use_Guide_Image9]

### <div id='10'/>2.1.4. Google Cloud Platform Service Account Key 생성

1.Service Account에 대한 접근 가능  key를 JSON으로 생성 한다.
-	Service Account 목록에서 해당 Service Account의 우측에 Create Key메뉴를 클릭하면 된다.

![PaaSTa_Google_Use_Guide_Image10]

![PaaSTa_Google_Use_Guide_Image11]


### <div id='11'/>2.1.5. Google Cloud Platform VPC networks 생성

1.Networking의 하위메뉴 VPC networks 메뉴를 선택 하고 CREATE VPC NETWORK 버튼을 클릭 한다.

![PaaSTa_Google_Use_Guide_Image12]

2.NETWORK 및 서브넷 정보를 입력 하고 Create 버튼을 클릭 한다.

![PaaSTa_Google_Use_Guide_Image13]

### <div id='12'/>2.1.6. Google Cloud Platform External IP Address 생성

1.Networking의 하위메뉴 External IP address 메뉴를 선택 하고 Reserve static address 버튼을 클릭 한다.

![PaaSTa_Google_Use_Guide_Image14]

2.External IP address 정보를 입력하고 Reserve 버튼을 클릭 한다.

![PaaSTa_Google_Use_Guide_Image15]

### <div id='13'/>2.1.7. Google Cloud Platform External IP Address 생성

1.Networking의 하위메뉴 Firewall rules 메뉴를 선택 하고 CREATE FIREWALL RULE 버튼을 클릭 한다.

![PaaSTa_Google_Use_Guide_Image16]

2.firewall 아래 테이블의 rule 정보 입력 한다.

![PaaSTa_Google_Use_Guide_Image17]

3.firewall rule 정보를 입력하고 Create 버튼을 클릭 한다.

![PaaSTa_Google_Use_Guide_Image18]

4.	다음과 같이 firewall rules가 생성되는 것을 확인 할 수 있다

![PaaSTa_Google_Use_Guide_Image19]

### <div id='14'/>2.1.7. Google Cloud Platform Key pair 생성 
1.Ubuntu Linux 명령어를 사용하여 Key Pair를 생성 한다.

	$ ssh-keygen -t rsa -f ~/.ssh/vcap -C vcap

### <div id='15'/>2.1.7. Google Cloud Platform ssh Meta Data 생성
1.Compute Engine 하위 메뉴 Metadata 메뉴를 클릭 하고 ssh Keys 탭 메뉴의 Add SSH keys를 선택 한다.

![PaaSTa_Google_Use_Guide_Image20]

![PaaSTa_Google_Use_Guide_Image21]

[PaaSTa_Google_Use_Guide_Image1]:../images/IaaS/google/1.png
[PaaSTa_Google_Use_Guide_Image2]:../images/IaaS/google/2.png
[PaaSTa_Google_Use_Guide_Image3]:../images/IaaS/google/3.png
[PaaSTa_Google_Use_Guide_Image4]:../images/IaaS/google/4.png
[PaaSTa_Google_Use_Guide_Image5]:../images/IaaS/google/5.png
[PaaSTa_Google_Use_Guide_Image6]:../images/IaaS/google/6.png
[PaaSTa_Google_Use_Guide_Image7]:../images/IaaS/google/7.png
[PaaSTa_Google_Use_Guide_Image8]:../images/IaaS/google/8.png
[PaaSTa_Google_Use_Guide_Image9]:../images/IaaS/google/9.png
[PaaSTa_Google_Use_Guide_Image10]:../images/IaaS/google/10.png
[PaaSTa_Google_Use_Guide_Image11]:../images/IaaS/google/11.png
[PaaSTa_Google_Use_Guide_Image12]:../images/IaaS/google/12.png
[PaaSTa_Google_Use_Guide_Image13]:../images/IaaS/google/13.png
[PaaSTa_Google_Use_Guide_Image14]:../images/IaaS/google/14.png
[PaaSTa_Google_Use_Guide_Image15]:../images/IaaS/google/15.png
[PaaSTa_Google_Use_Guide_Image16]:../images/IaaS/google/16.png
[PaaSTa_Google_Use_Guide_Image17]:../images/IaaS/google/17.png
[PaaSTa_Google_Use_Guide_Image18]:../images/IaaS/google/18.png
[PaaSTa_Google_Use_Guide_Image19]:../images/IaaS/google/19.png
[PaaSTa_Google_Use_Guide_Image20]:../images/IaaS/google/20.png
[PaaSTa_Google_Use_Guide_Image21]:../images/IaaS/google/21.png
[PaaSTa_Google_Use_Guide_Image22]:../images/IaaS/google/22.png


