## Table of Contents

1. [개요](#1)
  * [목적](#2)
  * [범위](#3)
  * [참고자료](#4)
2. [BOSH](#5)
	* [BOSH1](#6)
	* [BOSH2](#7)
	* [Bosh 컴포넌트 구성](#8)
3. [BOSH 설치 환경 구성 및 설치](#9)
	* [BOSH 설치 절차](#10)
    * [Inception 서버 구성](#11)
    * [Inception 설치](#12)
    	*  [Pre-requsite](#13)
    	*  [BOSH cli 및 dependency 설치](#14)
    	*  [Deployment 및 release 파일 다운로드](#15)
    	*  [BOSH 환경 설정 및 디렉토리 설명](#16)
			*  [Bosh 환경 설정](#17)
            	*  [OPENSTACK BOSH 환경 설정](#18)
            	*  [AWS BOSH 환경 설정](#19)
            	*  [VSPHERE BOSH 환경 설정](#20)
            	*  [AZURE BOSH 환경 설정](#21)
            	*  [GOOGLE(GCP) BOSH 환경 설정](#22)
            	*  [BOSH-LITE 환경 설정](#23)
        *  [BOSH Deploy](#24)
        *  [BOSH Login](#25)
        *  [jumpbox](#26)
        *  [credhub](#27)
			*  [credhub cli install](#28)

## Executive Summary

본 문서는 BOSH2의 설명 및 설치하는 가이드 문서로 BOSH를 실행할 수 있는 환경을 구성하여 실행하고 사용하는 방법에 대해서 설명하였다.

# <div id='1'/>1.  문서 개요 

## <div id='2'/>1.1.  목적
클라우드 환경에 서비스 시스템을 배포할 수 있는 BOSH는 릴리즈 엔지니어링, 개발, 소프트웨어 라이프사이클 관리를 통합한 오픈소스 프로젝트로 본 문서에서는 Inception환경(설치환경)에서 BOSH를 설치하여 BOSH를 기반으로 paasta-3.5를 설치하는데 그 목적이 있다. 

## <div id='3'/>1.2.  범위
본 가이드에서는 Linux 환경(Ubuntu 16.04)을 기준으로 BOSH 설치를 위한 패키지 및 라이브러리를 설치, 구성을 하고 이를 이용하여 BOSH를 설치하는 것을 기준으로 작성 하였다.
Paasta-3.5에서 사용하는 bosh는 기존 3.1 이하 버전과 설치 방식이 이전과 다르게 변경되었다.
3.1이하 버전은 cloud-foundry bosh1을 기반으로 bosh를 설치 했지만 bosh2에서는 bosh-deployment를 제공하며 이를 기반으로 bosh를 설치한다. 

본문서는 bosh-deployment v1.1.0을 기준으로 작성 되었다.

## <div id='4'/>1.3.  참고자료

본 문서는 Cloud Foundry의 BOSH Document와 Cloud Foundry Document를 참고로 작성하였다.

BOSH Document: [http://bosh.io](http://bosh.io)

BOSH DEPLOYMENT: [https://github.com/cloudfoundry/bosh-deployment](https://github.com/cloudfoundry/bosh-deployment)


Cloud Foundry Document: [https://docs.cloudfoundry.org/](https://docs.cloudfoundry.org/)



# <div id='5'/>2. BOSH
BOSH는 초기에 Cloud Foundry PaaS를 위해 개발 되었지만 현재는 jenkins, Hadoop 등 Yaml 파일 형식으로 소프트웨어를 쉽게 배포할 수 있으며 수 백 가지의 VM을 설치 할 수 있고 각 각의 VM에 대해 모니터링, 장애복구 등 라이프 사이클을 관리 할 수 있는 통합 프로젝트이다.

BOSH가 지원이 되는 IaaS는 vSphere, Google Cloud Platform, AWS, OpenStack, Azure, vCloud, RackHD, SoftLayer가 있다. 현재 PaaS-TA에서 지원 되는 IaaS 항목은 vSphere, AWS, Openstack, Google Cloud , Azure Platform이 지원이 되고 있다.

PaaS-TA 3.1까지는 cloud-foundry bosh1을 기준으로 bosh 설치를 했다. PaaS-TA 3.5부터는 bosh2를 기준으로 Bosh를 설치하게 된다. Bosh2는 cloud-foundry에서 제공하는 bosh-deployment를 bosh를 설치한다.

## <div id='6'/>2.1. BOSH1

BBosh1은 bosh-init을 통하여 Bosh를 생성하고, bosh1 cli를 통하여 PaaS-TA Controller, Container를 생성하였다.

![PaaSTa_BOSH_Use_Guide_Image1]

## <div id='7'/>2.2. BOSH2

Bosh2는 Bosh2-cli를 통하여 Bosh와 PaaS-TA 를 모두 생성 시켜준다. Bosh생성시 Bosh-deployment를 이용하여 Bosh를 생성한다. Bosh생성 후 paasta-deployment를 활용하여 paasta를 생성한다.
Paasta-3.1 버전까지는  PaaS-TA Container, Controller를 별도로 deployment로 설치 해야 했지만 3.5부터는 paasta deployment 하나로 통합 되었으며, 한번에 PaaS-TA를 설치 할 수 있다.

![PaaSTa_BOSH_Use_Guide_Image2]

## <div id='8'/>2.3. Bosh 컴포넌트 구성

BOSH의 컴포넌트 구성은 다음과 같다.

![PaaSTa_BOSH_Use_Guide_Image3]

1.	Command Line Interface(CLI): Director와 상호작용을 위한 Command Line Interface
2.	Director: Director가 VM을 생성 또는 수정할 때 설정 정보를 레지스트리에 저장한다. 저장된 레지스트리 정보는 VM의 bootstrapping stage에서 이용된다.
3.	Database: Director가 사용하는 Postgres 데이터베이스로 Deployment시에 필요로하는 Stemcell / Release / Deployment의 메타 정보들을 저장한다.
4.	Message Bus(Nats): Message Bus는 Director와 Agent간 통신하기 위한 publish-subscribe 방식의 Message System으로 VM 모니터링과 특정 명령을 수행하기 위해 사용된다.
5.	Registry: VM 생성을 위한 설정 정보 저장
6.	Health Monitor: Health Monitor는 BOSH Agent로부터 클라우드의 상태정보들을 수집한다. 클라우드로부터 특정 Alert이 발생하면 Resurrector를 하거나 Notification Plug-in을 통해 Alert Message를 전송할 수도 있다.
7.	Agent: Agent는 클라우드에 배포되는 모든 VM에 설치되고 Director로부터 특정 명령을 받고 수행하는 역할을 하게된다. Agent는 Director로부터 수신 받은 Job Specification(설치할 패키지 및 구성 방법)정보로부터 해당 VM에 Director의 지시대로 지정된 패키지를 설치하고, 필요한 구성정보를 설정하게 된다.


## <div id='9'/>3. BOSH 설치 환경 구성 및 설치

## <div id='10'/>3.1. BOSH 설치 절차
Inception은 bosh 및 paasta를 설치하기 위한 설치 환경으로 vm또는 server 장비 이다. Os version은 Ubuntu 16.04를 기준으로 한다.

![PaaSTa_BOSH_Use_Guide_Image4]
	
### <div id='11'/>3.2.	Inception 서버 구성

Inception 서버는 BOSH 설치와 BOSH의 Director를 설정하여 PaaS-TA를 설치 하기 위해 필요한 패키지 및 라이브러리, Manifest 파일 등의 환경을 가지고 있는 배포 작업 및 실행 서버이다. 환경 구성에 있어서 전제조건으로 Inception 서버는 외부와 통신이 가능해야 한다.


BOSH 및 PaaS-TA 설치를 위해 Inception 서버에 구성해야 할 컴포넌트는 다음과 같다.

-	BOSH Cli 2.x 이상 
-	BOSH Dependency : ruby, ruby-dev, openssl 등
-	BOSH Deployment: Bosh 설치 하기위한 manifest Deployment (bosh-deployment 1.1 기준) 
-	paasta Deployment : paasta를 설치하기 위한 manifest deployment (cf-deployment 2.9 기준)


### <div id='12'/>3.3.	Inception 설치

### <div id='13'/>3.3.1.	Pre-requsite

-	본 설치 가이드는 Ubuntu 16.04 버전을 기준으로 한다.
-	Release, deployment 파일은 /home/{user_name}/workspace/paasta-3.5 이하에 다운로드 받아 야 한다.

### <div id='14'/>3.3.2.	Bosh cli 및 dependency 설치

※  Bosh cli 설치
```
$ sudo apt-get update
$ mkdir workspace
$ cd workspace
$ curl -Lo ./bosh https://s3.amazonaws.com/bosh-cli-artifacts/bosh-cli-5.1.2-linux-amd64
$ chmod +x ./bosh
$ sudo mv ./bosh /usr/local/bin/bosh
$ bosh -v
```

※ Bosh dependency 설치
```
$ sudo apt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt-dev libxml2-dev libssl-dev libreadline6 libreadline6-dev libyaml-dev libsqlite3-dev sqlite3
```

※  bosh2 cli 
bosh2 cli는 bosh deploy시 bosh certificate 정보를 생성해 주는 기능이 있다. 
bosh site에서 기본으로 받는 bosh cli는 인증서가 1년으로 제한되어 있다. 

만약 인증서 기간을 늘리고 싶다면 bosh-cli 소스를 받아 compile해서 사용 해야 한다.

bosh cli 소스를 컴파일하여 인증서 기간을 늘리고 싶다면 다음 가이드를 따라야 한다.

※ 소스 build 전제 조건
 - go 1.9.2 버전 이상이 설치 되어 있어야 한다.
 - 빌드 환경은 ubuntu 이다.

```
$ mkdir -p ~/workspace/bosh-cli/src/
$ cd ~/workspace/bosh-cli

$ export GOPATH=$PWD
$ export PATH=$GOPATH/bin:$PATH

$ go get -d github.com/cloudfoundry/bosh-cli
$ cd $GOPATH/src/github.com/cloudfoundry/bosh-cli
$ git checkout v5.1.2
$ vi ./vendor/github.com/cloudfoundry/config-server/types/certificate_generator.go

func generateCertTemplate 함수에 아래 내용중 365(일)을 원하는 기간 만큼 수정한다.

  - notAfter := now.Add(365 * 24 * time.Hour) 

$ ./bin/build
cd out
sudo cp ./bosh /usr/local/bin/bosh

bosh -version
```

### <div id='15'/>3.3.3.	Deployment 및 release 파일 다운로드

1.	다운로드 파일이 위치할 경로 디렉토리를 만든다.

- [설치 파일 다운로드 받기](../../Download_Page.md)

```
$ mkdir -p ~/workspace/paasta-3.5/deployment
$ mkdir -p ~/workspace/paasta-3.5/release
$ mkdir -p ~/workspace/paasta-3.5/stemcell

```

2.	파스타 다운로드 URL에서 [PaaS-TA Deployment] 파일을 다운로드 받아 ~/workspace/paasta-3.5/deployment 이하 디렉토리에 압축을 푼다.

3.	파스타 다운로드 URL에서 [PaaS-TA BOSH 릴리즈 다운로드] 파일을 다운로드 받아 ~/workspace/paasta-3.5/release 이하 디렉토리에 압축을 푼다.

4.	파스타 다운로드 URL에서 [PaaS-TA 스템셀 이미지] 파일을 다운로드 받아 ~/workspace/paasta-3.5/stemcell 이하 디렉토리에 압축을 푼다.



### <div id='16'/>3.3.4.	Bosh 환경 설정 및 디렉토리 설명

- 다운로드 받은 파일이 아래 경로에 존재하는지 확인한다



- paasta-3.5 이하 디렉토리

![PaaSTa_BOSH_Use_Guide_Image5]

<table>
<tr>
<td>deployment</td>
<td>paasta-deployment, bosh-deployment, cloud-config 가 존재 한다.</td>
</tr>
<tr>
<td>release</td>
<td>paasta Release File이 존재한다.</td>
</tr>   
<tr>
<td>stemcell</td>
<td>IaaS별 Stemcell 이 존재한다.</td>
</tr>
</table>

- paasta-3.5/deployment 이하 디렉토리


![PaaSTa_BOSH_Use_Guide_Image6]

<table>
<tr>
<td>bosh-deployment</td>
<td>Bosh 설치할 manifest및 설치 파일이 존재한다.</td>
</tr>
<tr>
<td>cloud-config</td>
<td>Paasta 설치하기 위한 iaas 관련 network/storage/vm 관련 설정들을 정의 한다. IaaS/network/disk등 상황에 따라 설정이 다르다.(중요)</td>
</tr>   
<tr>
<td>paasta-deployment-3.5</td>
<td>paasta 설치할 manifest및 설치 파일이 존재한다.</td>
</tr>
</table>


- paasta-3.5/release 이하 디렉토리

![PaaSTa_BOSH_Use_Guide_Image7]


<table>
<tr>
<td>bosh</td>
<td>Bosh 설치 시 필요한 release 파일이 존재하는 디렉토리</td>
</tr>
<tr>
<td>paasta</td>
<td>Paasta 설치 시 필요한 release 파일이 존재하는 디렉토리</td>
</tr>
</table>


- paasta-3.5/stemcell  이하 디렉토리

![PaaSTa_BOSH_Use_Guide_Image8]


<table>
<tr>
<td>stemcell</td>
<td>Paasta가 설치될 때 사용할 stemcell이다. (paasta vm image)</td>
</tr>
</table>

### <div id='17'/>3.3.5.	Bosh 환경 설정



※ Bosh-deployment 디렉토리로 이동한다.
```
$ cd ~/workspace/paasta-3.5/deployment/bosh-deployment
$ chmod 755 *.sh  
```

~/workspace/paasta-3.5/deployment/bosh-deployment 이하 디렉토리에는 iaas별 bosh를 설치 하는 shell이 존재한다. Shell 파일을 이용하여 bosh를 설치 한다.
파일명은 deploy-{iaaS-name}.sh 로 만들어 졌다. 


<table>
<tr>
<td>deploy-aws.sh</td>
<td>amazon 설치 bosh 설치 shell</td>
</tr>
<tr>
<td>deploy-openstack.sh</td>
<td>openstack 설치 bosh 설치 shell</td>
</tr>   
<tr>
<td>deploy-azure.sh</td>
<td>vsphere 설치 bosh 설치 shell</td>
</tr>
<tr>
<td>deploy-google.sh</td>
<td>GCP(google) 설치 bosh 설치 shell</td>
</tr>
<tr>
<td>deploy-azure.sh</td>
<td>Azure 설치 bosh 설치 shell</td>
</tr>
<tr>
<td>deploy-bosh-lite.sh</td>
<td>Bosh-lite(local Test용) 설치 bosh 설치 shell</td>
</tr>
</table>

설치 shell 파일은 각 iaas별로 존재하며 bosh 설치 시 명령어는 create-env 로 시작한다. Shell 이 아닌 bosh command로 실행이 가능하며 설치하는 IaaS 환경에 따라 Option들이 달라진다.

<table>
<tr>
<td>--state</td>
<td>Bosh 설치 명령어를 실행할 때 생성되는 file로 설치된 bosh의 iaas 설정 정보를 보관히고 있다. (중요, backup 필요)</td>
</tr>
<tr>
<td>--vars-store</td>
<td>Bosh 설치 명령어를 실행할 때 생성되는 file로 bosh가 설치 될 때 Bosh Cli는 Bosh 내부 컴포넌트가 사용하는 인증서 및 인증정보를 생성 저장한다. (중요, backup 필요)</td>
</tr>   
<tr>
<td>-o</td>
<td>Bosh 설치시 Option File을 설정할 수 있는데 IaaS별 Cpi선택 또는 jumpbox, credhub 등 설정적용을 할 수 있다.</td>
</tr>
<tr>
<td>-v</td>
<td>Bosh 설치시 사용되는 yml 파일 또는 option 파일에 변수에 값을 설정할 경우 사용 할 수 있다. Yml, optionfile 속성에 따라 필수인 경우와 option인 경우가 있다.</td>
</tr>
<tr>
<td>--var-file</td>
<td>주로 인증서를 사용하는 경우 사용하는 option 이다.</td>
</tr>
</table>


#### <div id='18'/>3.3.5.1. OPENSTACK BOSH 환경 설정

```
bosh create-env bosh.yml \
    --state=openstack/state.json \      # bosh 설치 시 생성되는 파일로 절대 삭제 되면 않됨. (backup 필요)
    --vars-store=openstack/creds.yml \  # bosh 내부 인증서 파일 중요 (Backup필요)
    -o openstack/cpi.yml \             # openstack  cpi 적용
    -o uaa.yml  \
    -o credhub.yml  \
    -o jumpbox-user.yml  \
    -v internal_cidr=10.20.0.0/24 \       # internal ip range
    -v internal_gw=10.20.0.1 \            # internal ip gateway
    -v internal_ip=10.20.0.6 \            # internal ip 
    -v director_name=micro-bosh \         # bosh director 명
    -v auth_url=http://xxx.xxx.xxx.xxx:5000/v3/  \   # keystone url
    -v az=zone1 \                                # bosh 설치될 az zone
    -v default_key_name=openpaas \               # openstack key name
    -v default_security_groups=[openpaas] \      # openstack security group
    -v net_id=51b96a68-aded-4e73-aa44-f44a812b9b30 \  # network id
    -v multizone=true \                          # Compute Node Multizone 설정 여부
    -v openstack_password=xxxx \                 # openstack user password
    -v openstack_username=xxxx\                  # openstack user name
    -v openstack_domain=default \                # bosh 설치될 openstack domain name
    -v openstack_project=monitoring \            # bosh 설치될 openstack project
    -v private_key=~/.ssh/OpenPaas.pem \         # openstack 접속 pem file
    -v region=RegionOne                          # bosh 설치될 openstack 설치 될 region
```

#### <div id='19'/>3.3.5.2. AWS BOSH 환경 설정

```
bosh create-env bosh.yml \
     --state=aws/state.json \                 #bosh 설치 시 생성되는 파일로 절대 삭제 되면 않됨. (backup 필요)
     --vars-store aws/creds.yml \   # bosh 내부 인증서 파일 중요 (Backup필요)
     -o aws/cpi.yml \                     # aws cpi 적용
     -o uaa.yml  \
     -o credhub.yml  \
     -o jumpbox-user.yml  \
     -v internal_cidr=10.0.0.0/24 \        # internal ip range
     -v internal_gw=10.0.0.1 \             # internal ip gateway
     -v internal_ip=10.0.0.6 \             # internal ip 
     -v director_name=micro-bosh \         # bosh director 명
     -v access_key_id=xxxxx \              # aws access_key
     -v secret_access_key=xxxxx \          # aws secret_key
     -v region=ap-northeast-1 \            # bosh가 설치될 aws region
     -v az=ap-northeast-1a \               # bosh가 설치될 aws availability zone
     -v default_key_name=aws-paasta-rnd \  # aws key name
    -v default_security_groups=[paasta-rnd] \  # aws security-group
    --var-file private_key=~/.ssh/aws-paasta-rnd.pem \  # aws 접속 pem file
    -v subnet_id=subnet-ba1e15f3                # bosh가 설치될 aws subnet

```

#### <div id='20'/>3.3.5.3. VSPHERE BOSH 환경 설정

```
bosh create-env bosh.yml \
    --state=vsphere/state.json \ #bosh 설치 시 생성되는 파일로 절대 삭제 되면 않됨. (backup 필요)
    --vars-store=vsphere/creds.yml \   #bosh 내부 인증서 파일 중요 (Backup필요)
    -o vsphere/cpi.yml \                 # vsphere 적용
    -o uaa.yml  \
    -o credhub.yml  \
    -o vsphere/resource-pool.yml  \                # vsphere 적용
    -v director_name=micro-bosh-vsphere \          # micro-bosh director 이름
    -v internal_cidr=10.30.0.0/16 \                # internal ip range
    -v internal_gw=10.30.20.23 \                   # internal ip gateway
    -v internal_ip=10.30.40.111 \                  # internal ip 
    -v network_name="Internal" \                   # internal 네트워크 이름 (vcenter)
    -v vcenter_dc=BD-DC \                          # vcenter datacenter 이름
    -v vcenter_ds=iSCSI-28-Storage \               # vcenter data storage 이름
    -v vcenter_ip=10.30.20.22 \                    # vcenter internal ip
    -v vcenter_user=administrator \                # vcenter user 이름
    -v vcenter_password=sdfsfsdeee \               # vcenter user 패스워드
    -v vcenter_templates=CF_BOSH2_Templates \      # vcenter templates 폴더명
    -v vcenter_vms=CF_BOSH2_VMs \                  # vcenter vms 이름
    -v vcenter_disks=CF_BOSH2_Disks \              # vcenter disk 이름
    -v vcenter_cluster=BD-HA \                     # vcenter cluster 이름
    -v vcenter_rp=CF_BOSH2_Pool                    # vcenter resource pool 이름
```



#### <div id='21'/>3.3.5.4. AZURE BOSH 환경 설정
```
bosh create-env bosh.yml \
     --state=azure/state.json \  #bosh 설치 시 생성되는 파일로 절대 삭제 되면 않됨. (backup 필요)
     --vars-store azure/creds.yml \ #bosh 내부 인증서 파일 중요 (Backup필요)
     -o azure/cpi.yml \               # azure CPI 적용
     -o uaa.yml  \
     -o credhub.yml  \
     -o jumpbox-user.yml  \
     -v internal_cidr=10.0.0.0/24 \   # internal ip range
     -v internal_gw=10.0.0.1 \        # internal ip gateway
     -v internal_ip=10.0.0.6 \        # internal ip
     -v director_name=micro-bosh \    # bosh director 명
     -v vnet_name=paasta-net \        # Azure VNet 명
     -v subnet_name=bosh-net \        #Azure VNet Subnet 명
     -v subscription_id=816-91e9-4ba6-806c2ccb8630 \   # Azure Subscription Id
     -v tenant_id=aeacdca2-4f9e-8c8b-b0403fbdcfd1 \    # Azure Tenant Id
     -v client_id=779dae-49b05-464d3b877b7b \          # Azure Client Id
     -v client_secret=lNYSOm4j/HbeN5jiA8vNIC4rXs= \    # Azure Client Secret
     -v resource_group_name=paas-ta-resoureceGorup \   # Azure Resource Group
     -v storage_account_name=paasta \                  # Azure Storage Account
     -v default_security_group=bosh-security           # Azure Security Group
```


#### <div id='22'/>3.3.5.5. GOOGLE(GCP) BOSH 환경 설정
```
bosh create-env bosh.yml \
     --state=gcp/state.json \   # bosh 설치 시 생성되는 파일로 절대 삭제 되면 않됨. (backup 필요)
     --vars-store gcp/creds.yml \ # bosh 내부 인증서 파일 중요 (Backup필요)
     -o gcp/cpi.yml \                   # google CPI 적용
     -o uaa.yml  \
     -o credhub.yml  \
     -o jumpbox-user.yml  \
     -v internal_cidr=192.168.10.0/24 \ # internal ip range
     -v internal_gw=192.168.10.1 \      # internal ip gateway
     -v internal_ip=192.168.10.6 \      # internal ip
     -v director_name=micro-bosh \      # bosh director 명
     -v network=paas-ta-network \       # google Network Name
     -v subnetwork=bosh-net \           # google Subnet Name
     -v tags=[bosh-security] \          # google tag
     -v project_id=paas-ta-198701 \     # google Project Id
     -v private_key=~/.ssh/vcap.pem \   # ssh private key path
     -v zone=asia-northeast1-a \        # google zone
     --var-file gcp_credentials_json=~/.ssh/PaaS-TA-1e47e2554132.json  google service account key
```


#### <div id='23'/>3.3.5.6. BOSH-LITE 환경 설정
```
bosh create-env bosh.yml \
     --state=warden/state.json \   # bosh 설치 시 생성되는 파일로 절대 삭제 되면 않됨. (backup 필요)
     --vars-store warden/creds.yml \ # bosh 내부 인증서 파일 중요 (Backup필요)
     -o virtualbox/cpi.yml \           # virtualbox CPI 적용
     -o virtualbox/outbound-network.yml \  
     -o bosh-lite.yml \            
     -o bosh-lite-runc.yml \
     -o uaa.yml \
     -o credhub.yml \
     -o jumpbox-user.yml \
     -v director_name=vbox \
     -v internal_ip=192.168.150.4 \       # internal ip range
     -v internal_gw=192.168.150.1 \       # internal gateway
     -v internal_cidr=192.168.150.0/24 \  # internal ip rang
     -v network_name=vboxnet0 \           # network name
     -v outbound_network_name=NatNetwork  # outbound network
```


### <div id='24'/>3.3.6. BOSH Deploy

```
$ cd ~/workspace/paasta-3.5/deployment/bosh-deployment
$ ./deploy-{iaas}.sh
```

다음은 bosh 설치 실행하는 화면이다. 
![PaaSTa_BOSH_Use_Guide_Image9]

다음은 bosh 설치 완료 화면이다. 
![PaaSTa_BOSH_Use_Guide_Image10]

### <div id='25'/>3.3.7. BOSH Login
bosh가 설치 되면 bosh설치 디렉토리 이하 {iaas}/creds.yml 이 생성된다. creds.yml은 bosh 인증정보를 가지고 있으며 creds.yml을 활용하여 bosh에 login 한다. Bosh 로그인 후 bosh-cli 명령어를 이용하어 paasta를 설치 할 수 있다

```
$ cd ~/workspace/paasta-3.5/deployment/bosh-deployment
$ export BOSH_CA_CERT=$(bosh int ./{iaas}/creds.yml --path /director_ssl/ca)
$ export BOSH_CLIENT=admin
$ export BOSH_CLIENT_SECRET=$(bosh int ./{iaas}/creds.yml --path /admin_password)
$ bosh alias-env {director_name} -e {bosh-internal-ip} --ca-cert <(bosh int {iaas}/creds.yml --path /director_ssl/ca)
$ bosh –e {director_name} env
```

### <div id='26'/>3.3.8. jumpbox
BOSH설치시 option file에 jumpbox-user.yml을 추가 하였다. Jumpbox는 BOSH VM에 접근하기 위한 인증을 적용하게 된다. 인증 key는 Bosh 자체적으로 생성하며 인증키를 통해 BOSH VM에 접근 할 수 있다.

```
$ cd ~/workspace/paasta-3.5/deployment/bosh-deployment
$ bosh int {iaas}/creds.yml --path /jumpbox_ssh/private_key > jumpbox.key 
$ chmod 600 jumpbox.key
$ ssh jumpbox@{bosh_ip} -i jumpbox.key
```

![PaaSTa_BOSH_Use_Guide_Image11]

### <div id='27'/>3.3.9. credhub
BOSH설치시 option file에 credhub.yml을 추가 하였다. Credhub은 인증정보를 저장소이다. Bosh 설치 시 credhub.yml을 적용하면 paasta 설치시 인증정보를 credhub에 저장하게 된다. Credhub에 로그인 하기 위해서는 credhub cli를 통해 인증정보를 조회 수정 삭제 할 수 있다

#### <div id='28'/>3.3.9.1 credhub cli install

```
$ wget https://github.com/cloudfoundry-incubator/credhub-cli/releases/download/2.0.0/credhub-linux-2.0.0.tgz
$ tar -xvf credhub-linux-2.0.0.tgz 
$ chmod +x credhub
$ sudo mv credhub /usr/local/bin/credhub 
$ credhub –version
```

#### <div id='27'/>3.3.8.1 credhub login

```
$ export CREDHUB_CLIENT=credhub-admin
$ export CREDHUB_SECRET=$(bosh int --path /credhub_admin_client_secret {iaas}/creds.yml)
$ export CREDHUB_CA_CERT=$(bosh int --path /credhub_tls/ca {iaas}/creds.yml)
$ credhub login -s https://10.20.0.6:8844 --skip-tls-validation  (bosh internal ip)
$ credhub find
```

Credhub login 후 find를 해보면 비어 있는 것을 알 수 있다. Paasta를 설치 하면 인증 정보가 저장되어 조회 할 수 있다

```
$ credhub find
ex) uaa 인증정보 조회
$ credhub get -n /{director}/{deployment}/uaa_ca
```


[PaaSTa_BOSH_Use_Guide_Image1]:../images/Bosh2/bosh1.png
[PaaSTa_BOSH_Use_Guide_Image2]:../images/Bosh2/bosh2.png
[PaaSTa_BOSH_Use_Guide_Image3]:../images/Bosh2/bosh3.png
[PaaSTa_BOSH_Use_Guide_Image4]:../images/Bosh2/bosh-install-flow.png
[PaaSTa_BOSH_Use_Guide_Image5]:../images/Bosh2/directory_1.png
[PaaSTa_BOSH_Use_Guide_Image6]:../images/Bosh2/directory_2.png
[PaaSTa_BOSH_Use_Guide_Image7]:../images/Bosh2/directory_3.png
[PaaSTa_BOSH_Use_Guide_Image8]:../images/Bosh2/directory_4.png
[PaaSTa_BOSH_Use_Guide_Image9]:../images/Bosh2/deploy_1.png
[PaaSTa_BOSH_Use_Guide_Image10]:../images/Bosh2/deploy_2.png
[PaaSTa_BOSH_Use_Guide_Image11]:../images/Bosh2/jumpbox.png
