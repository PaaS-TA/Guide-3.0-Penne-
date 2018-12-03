## Table of Contents

[1. 문서 개요](#1)

  -  [1.1. 목적](#11)
  -  [1.2. 범위](#12)
  -  [1.3. 시스템 구성도](#13)
  -  [1.4. 참고 자료](#14)

[2. Container 서비스팩 설치](#2)

  -  [2.1. 설치 전 준비사항](#21)
  -  [2.1.1. Deployment 및 Release 파일 다운로드](#211)
  -  [2.2. Stemcell 업로드](#22)
  -  [2.3. Container 서비스 릴리즈 Deployment 파일 수정 및 배포](#23)
  -  [2.4. Container 서비스 브로커 등록](#24)
  -  [2.5. Container 서비스 UAA Client Id 등록](#25)
  -  [2.6. Container 서비스 Private image Repository 설정 (Optional)](#26)

# <div id='1'/> 1. 문서 개요

### <div id='11'/> 1.1. 목적
본 문서(Container 서비스팩 설치 가이드)는 개방형 PaaS 플랫폼 고도화 및 개발자 지원 환경 기반의 Open PaaS에서 제공되는 서비스팩인 Container 서비스팩을 Bosh를 이용하여 설치 및 서비스 등록하는 방법을 기술하였다.

PaaS-TA 3.5 버전부터는 Bosh 2.0 기반으로 배포(deploy)를 진행한다. 기존 Bosh 1.0 기반으로 설치를 원할 경우에는 PaaS-TA 3.1 이하 버전의 문서를 참고한다.


### <div id='12'/> 1.2. 범위
설치 범위는 Container 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다.


### <div id='13'/> 1.3. 시스템 구성도
본 문서의 설치된 시스템 구성도이다. Container 서비스 Server, Container 서비스 브로커로 최소사항을 구성하였다.

![Architecture]

<table>
  <tr>
    <th>VM명</th>
    <th>인스턴스수</th>
    <th>vCPU 수</th>
    <th>메모리(GB)</th>
    <th>디스크(GB)</th>
  </tr>
  <tr>
    <td>master</td>
    <td>1</td>
    <td>1</td>
    <td>4G</td>
    <td>Root 4G + 영구디스크 20G</td>
  </tr>
  <tr>
    <td>worker</td>
    <td>N</td>
    <td>8</td>
    <td>16G</td>
    <td>Root 4G + 영구디스크 50G</td>
  </tr>
  <tr>
    <td>caas-api<br></td>
    <td>N</td>
    <td>1</td>
    <td>1G</td>
    <td>Root 4G</td>
  </tr>
  <tr>
    <td>caas-common-api</td>
    <td>N</td>
    <td>1</td>
    <td>1G</td>
    <td>Root 4G</td>
  </tr>
  <tr>
    <td>caas-service-broker</td>
    <td>N</td>
    <td>1</td>
    <td>1G</td>
    <td>Root 4G</td>
  </tr>
  <tr>
    <td>caas-dashboard</td>
    <td>1</td>
    <td>1</td>
    <td>1G</td>
    <td>Root 4G</td>
  </tr>
  <tr>
    <td>DBMS<br>(MariaDB)</td>
    <td>1</td>
    <td>1</td>
    <td>2G</td>
    <td>Root 4G + 영구디스크 20G</td>
  </tr>
  <tr>
    <td>HAProxy</td>
    <td>1</td>
    <td>1</td>
    <td>2G</td>
    <td>Root 4G </td>
  </tr>
</table>

### <div id='14'/> 1.4. 참고 자료
>http://bosh.io/docs
http://docs.cloudfoundry.org


# <div id='2'/> 2. Container 서비스팩 설치

### <div id='21'/> 2.1. 설치 전 준비사항
본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다.
서비스팩 설치를 위해서는 먼저 BOSH CLI v2가 설치되어 있어야 하고 BOSH에 로그인이 되어 있어야 한다.<br>
BOSH CLI v2가 설치 되어 있지 않을 경우, 먼저 BOSH 2.0 설치 가이드 문서를 참고하여 BOSH CLI v2를 설치를 하고 사용법을 숙지해야한다.<br>

- BOSH 2.0 사용자 가이드
  >BOSH 2 사용자 가이드 : <https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/v3.5/Use-Guide/Bosh/PaaS-TA_BOSH2_사용자_가이드v1.0.md>
  >
  >BOSH CLI V2 사용자 가이드 : <https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/v3.5/Use-Guide/Bosh/PaaS-TA_BOSH_CLI_V2_사용자_가이드v1.0.md>


#### <div id='211'/> 2.1.1. Deployment 및 Release 파일 다운로드

Container 서비스 설치에 필요한 Deployment 및 릴리즈 파일을 다운로드 받아 서비스 설치 작업 경로로 위치시킨다.

-	설치 파일 다운로드 위치 : https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/v3.5/Download_Page.md
-	Release, deployment 파일은 /home/{user_name}/workspace/paasta-4.0 이하에 다운로드 받아야 한다.
-	설치 작업 경로 생성 및 파일 다운로드

>	Deployment 파일
 >> paasta-container-service-2.0

> Release 파일
 >> bosh-dns-release-1.5.0.tgz<br>
 >> bpm-release-0.6.0.tgz<br>
 >> cfcr-etcd-release-1.3.tgz<br>
 >> docker-32.0.0.tgz<br>
 >> kubo-release.tgz<br>
 >> paasta-container-service-projects-release.tgz<br>


```
- Deployment 다운로드 파일 위치 경로 생성
$ mkdir -p ~/workspace/paasta-4.0/deployment/service-deployment/paasta-container-service-2.0

- 릴리즈 다운로드 파일 위치 경로 생성
$ mkdir -p ~/workspace/paasta-4.0/release/service
```

-	Deployment 파일을 다운로드 받아 ~/workspace/paasta-4.0/deployment/service-deployment/paasta-container-service-2.0 이하 디렉토리에 이동한다.
-	Release 파일을 다운로드 받아 ~/workspace/paasta-4.0/release/service 이하 디렉토리에 이동한다.


### <div id='22'/> 2.2. Stemcell 업로드

- Deploy시 사용할 Stemcell을 확인한다.
  >Stemcell 목록이 존재 하지 않을 경우, BOSH 설치 가이드 문서를 참고하여 Stemcell을 업로드를 해야 한다. (Stemcell 3586.26 버전 사용, PaaSTA-Stemcell.zip)

- Stemcell 다운로드 위치
  >https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/v3.5/Download_Page.md


```
$ bosh -e micro-bosh stemcells
Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

Name                                      Version   OS             CPI  CID  
bosh-vsphere-esxi-ubuntu-trusty-go_agent  3586.26*  ubuntu-trusty  -    sc-109fbdb0-f663-49e8-9c30-8dbdd2e5b9b9  
~                                         3468.51*  ubuntu-trusty  -    sc-bd8fadf2-92f2-495b-b172-d571a388abf5  
~                                         3445.2*   ubuntu-trusty  -    sc-025c70b5-7d6e-4ba3-a12b-7e71c33dad24  
~                                         3309*     ubuntu-trusty  -    sc-22429dba-e5cc-4469-ab3a-882091573277  

(*) Currently deployed

4 stemcells

Succeeded
```


### <div id='23'/> 2.3. Container 서비스 릴리즈 Deployment 파일 수정 및 배포
BOSH Deployment manifest는 Components 요소 및 배포의 속성을 정의한 YAML 파일이다.

Deployment 파일에서 사용하는 network, vm_type 등은 Cloud config를 활용하고 해당 가이드는 BOSH 2.0 가이드를 참고한다.

- Cloud config 내용 조회

  ```
  $ bosh -e micro-bosh cloud-config
  Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

  azs:
  - cloud_properties:
      datacenters:
      - clusters:
        - PAASTA-HA:
            resource_pool: CF_BOSH2_Pool
        name: PAASTA-HA
    name: z1
  - cloud_properties:
      datacenters:
      - clusters:
        - PAASTA-HA:
            resource_pool: CF_BOSH2_Pool
        name: PAASTA-HA
    name: z2
  - cloud_properties:
      datacenters:
      - clusters:
        - PAASTA-HA:
            resource_pool: CF_BOSH2_Pool
        name: PAASTA-HA
    name: z3
  - cloud_properties:
      datacenters:
      - clusters:
        - PAASTA-HA:
            resource_pool: CF_BOSH2_Pool
        name: PAASTA-HA
    name: z4
  - cloud_properties:
      datacenters:
      - clusters:
        - PAASTA-HA:
            resource_pool: CF_BOSH2_Pool
        name: PAASTA-HA
    name: z5
  - cloud_properties:
      datacenters:
      - clusters:
        - PAASTA-HA:
            resource_pool: CF_BOSH2_Pool
        name: PAASTA-HA
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
      gateway: 10.100.20.23
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
      gateway: 10.100.20.23
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
      - 115.68.46.217 - 115.68.46.222
      static:
      - 115.68.46.214 - 115.68.46.216
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
      - host: <CLOUD_PROPERTY_PORT_OF_HOST>
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
  ```
-	Deployment 를 하기 전에 remove-all-addons.sh 을 환경에 맞게 수정한다.
```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment/paasta-container-service-2.0
$ vi remove-all-addons.sh


#!/bin/bash

director_name='micro-bosh'

bosh -e ${director_name} update-runtime-config manifests/ops-files/paasta-container-service/remove-all-addons.yml

```
- Deployment YAML에서 사용하는 변수들을 서버 환경에 맞게 수정한다.

> vSphere용

```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment/paasta-container-service-2.0
$ vi ./manifests/paasta-container-service-vsphere-vars.yml

# INCEPTION OS USER NAME
inception_os_user_name: "inception"

# RELEASE
caas_projects_release_name: "paasta-container-service-projects-release"
caas_projects_release_version: "1.0"

# IAAS
vcenter_master_user: "<VCENTER_MASTER_USER>"
vcenter_master_password: "<VCENTER_MASTER_PASSWORD>"
vcenter_ip: "<VCENTER_IP>"
vcenter_dc: "<VCENTER_DC>"
vcenter_ds: "<VCENTER_DS>"
vcenter_vms: "<VCENTER_VMS>"

# STEMCELL
stemcell_os: "ubuntu-trusty"
stemcell_version: "3586.26"
stemcell_alias: "trusty"

# VM_TYPE
vm_type_small: "small"
vm_type_small_highmem_16GB: "small-highmem-16GB"
vm_type_caas_small: "caas_small"
vm_type_caas_small_api: "caas_small_api"

# NETWORK
service_private_networks_name: "service_private"
service_public_networks_name: "service_public"

# IPS
caas_master_public_url: "115.68.47.178"    # CAAS-MASTER-PUBLIC-URL
haproxy_public_url: "115.68.47.179"        # HAPROXY-PUBLIC-IPS

# CREDHUB
credhub_server_url: "10.30.40.111:8844"    # Bosh credhub server URL
credhub_admin_client_secret: "<CREDHUB_ADMIN_CLIENT_SECRET>"

# CF
cf_uaa_oauth_uri: "https://uaa.<DOMAIN>"
cf_api_url: "https://api.<DOMAIN>"
cf_uaa_oauth_client_id: "<CF_UAA_OAUTH_CLIENT_ID>"
cf_uaa_oauth_client_secret: "<CF_UAA_OAUTH_CLIENT_SECRET>"

# HAPROXY
haproxy_http_port: 8080
haproxy_azs: [z1]

# MARIADB
mariadb_port: "<MARIADB_PORT>"
mariadb_azs: [z2]
mariadb_persistent_disk_type: "10GB"
mariadb_admin_user_id: "<MARIADB_ADMIN_USER_ID>"
mariadb_admin_user_password: "<MARIADB_ADMIN_USER_PASSWORD>"
mariadb_role_set_administrator_code_name: "Administrator"
mariadb_role_set_administrator_code: "RS0001"
mariadb_role_set_regular_user_code_name: "Regular User"
mariadb_role_set_regular_user_code: "RS0002"
mariadb_role_set_init_user_code_name: "Init User"
mariadb_role_set_init_user_code: "RS0003"

# DASHBOARD
caas_dashboard_instances: 1
caas_dashboard_port: 8091
caas_dashboard_azs: [z3]
caas_dashboard_management_security_enabled: false
caas_dashboard_logging_level: "INFO"

# API
caas_api_instances: 1
caas_api_port: 3333
caas_api_azs: [z1]
caas_api_management_security_enabled: false
caas_api_logging_level: "INFO"

# COMMON API
caas_common_api_instances: 1
caas_common_api_port: 3334
caas_common_api_azs: [z2]
caas_common_api_logging_level: "INFO"

# SERVICE BROKER
caas_service_broker_instances: 1
caas_service_broker_port: 8888
caas_service_broker_azs: [z3]

# ADDON
caas_apply_addons_azs: [z1]

# MASTER
caas_master_backend_port: 8443
caas_master_port: 8443
caas_master_azs: [z2]
caas_master_persistent_disk_type: 5120

# WORKER
caas_worker_instances: 3
caas_worker_azs: [z1,z2,z3]

```

> AWS용

```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment/paasta-container-service-2.0
$ vi ./manifests/paasta-container-service-aws-vars.yml

# INCEPTION OS USER NAME
inception_os_user_name: "ubuntu"

# RELEASE
caas_projects_release_name: "paasta-container-service-projects-release"
caas_projects_release_version: "1.0"

# IAAS
aws_access_key_id_master: '<AWS_ACCESS_KEY_ID_MASTER>'
aws_secret_access_key_master: '<AWS_SECRET_ACCESS_KEY_MASTER>'
aws_access_key_id_worker: '<AWS_ACCESS_KEY_ID_WORKER>'
aws_secret_access_key_worker: '<AWS_SECRET_ACCESS_KEY_WORKER>'
kubernetes_cluster_tag: 'kubernetes'      # Do not update!

# STEMCELL
stemcell_os: "ubuntu-trusty"
stemcell_version: "3586.26"
stemcell_alias: "trusty"

# VM_TYPE
vm_type_small: "caas_small"
vm_type_small_highmem_16GB: "caas_small_highmem"
vm_type_caas_small: "small"
vm_type_caas_small_api: "minimal"

# NETWORK
service_private_nat_networks_name: "default"
service_private_networks_name: "service_private"
service_public_networks_name: "service_public"

# IPS
caas_master_public_url: "52.78.21.76" # CAAS-MASTER-PUBLIC-URL
haproxy_public_url: "54.180.13.40" # HAPROXY-PUBLIC-IPS

# CREDHUB
credhub_server_url: "10.0.1.6:8844" # Bosh credhub server URL
credhub_admin_client_secret: "<CREDHUB_ADMIN_CLIENT_SECRET>"

# CF
cf_uaa_oauth_uri: "https://uaa.<DOMAIN>"
cf_api_url: "https://api.<DOMAIN>"
cf_uaa_oauth_client_id: "<CF_UAA_OAUTH_CLIENT_ID>"
cf_uaa_oauth_client_secret: "<CF_UAA_OAUTH_CLIENT_SECRET>"

# HAPROXY
haproxy_http_port: 8080
haproxy_azs: [z1]

# MARIADB
mariadb_port: "<MARIADB_PORT>"
mariadb_azs: [z2]
mariadb_persistent_disk_type: "10GB"
mariadb_admin_user_id: "<MARIADB_ADMIN_USER_ID>"
mariadb_admin_user_password: "<MARIADB_ADMIN_USER_PASSWORD>"
mariadb_role_set_administrator_code_name: "Administrator"
mariadb_role_set_administrator_code: "RS0001"
mariadb_role_set_regular_user_code_name: "Regular User"
mariadb_role_set_regular_user_code: "RS0002"
mariadb_role_set_init_user_code_name: "Init User"
mariadb_role_set_init_user_code: "RS0003"

# DASHBOARD
caas_dashboard_instances: 1
caas_dashboard_port: 8091
caas_dashboard_azs: [z3]
caas_dashboard_management_security_enabled: false
caas_dashboard_logging_level: "INFO"

# API
caas_api_instances: 1
caas_api_port: 3333
caas_api_azs: [z1]
caas_api_management_security_enabled: false
caas_api_logging_level: "INFO"

# COMMON API
caas_common_api_instances: 1
caas_common_api_port: 3334
caas_common_api_azs: [z2]
caas_common_api_logging_level: "INFO"

# SERVICE BROKER
caas_service_broker_instances: 1
caas_service_broker_port: 8888
caas_service_broker_azs: [z3]

# ADDON
caas_apply_addons_azs: [z1]

# MASTER
caas_master_backend_port: 8443
caas_master_port: 8443
caas_master_azs: [z2]
caas_master_persistent_disk_type: 5120

# WORKER
caas_worker_instances: 3
caas_worker_azs: [z1,z2,z3]

```


> OpenStack용

```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment/paasta-container-service-2.0
$ vi ./manifests/paasta-container-service-openstack-vars.yml

# INCEPTION OS USER NAME
inception_os_user_name: "ubuntu"

# RELEASE
caas_projects_release_name: "paasta-container-service-projects-release"
caas_projects_release_version: "1.0"

# IAAS
auth_url: 'http://<IAAS-IP>:5000/v3'
openstack_domain: '<OPENSTACK_DOMAIN>'
openstack_username: '<OPENSTACK_USERNAME>'
openstack_password: '<OPENSTACK_PASSWORD>'
openstack_project_id: '<OPENSTACK_PROJECT_ID>'
region: '<OPENSTACK_REGION>'
ignore-volume-az: true

# STEMCELL
stemcell_os: "ubuntu-trusty"
stemcell_version: "3586.26"
stemcell_alias: "trusty"

# VM_TYPE
vm_type_small: "small"
vm_type_small_highmem_16GB: "small-highmem-16GB"
vm_type_caas_small: "small"
vm_type_caas_small_api: "minimal"

# NETWORK
service_private_networks_name: "default"
service_public_networks_name: "vip"

# IPS
caas_master_public_url: "115.68.151.178"   # CAAS-MASTER-PUBLIC-URL
haproxy_public_url: "115.68.151.177"       # HAPROXY-PUBLIC-IPS

# CREDHUB
credhub_server_url: "10.20.0.7:8844"       # Bosh credhub server URL
credhub_admin_client_secret: "<CREDHUB_ADMIN_CLIENT_SECRET>"

# CF
cf_uaa_oauth_uri: "https://uaa.<DOMAIN>"
cf_api_url: "https://api.<DOMAIN>"
cf_uaa_oauth_client_id: "<CF_UAA_OAUTH_CLIENT_ID>"
cf_uaa_oauth_client_secret: "<CF_UAA_OAUTH_CLIENT_SECRET>"

# HAPROXY
haproxy_http_port: 8080
haproxy_azs: [z1]

# MARIADB
mariadb_port: "<MARIADB_PORT>"
mariadb_azs: [z2]
mariadb_persistent_disk_type: "10GB"
mariadb_admin_user_id: "<MARIADB_ADMIN_USER_ID>"
mariadb_admin_user_password: "<MARIADB_ADMIN_USER_PASSWORD>"
mariadb_role_set_administrator_code_name: "Administrator"
mariadb_role_set_administrator_code: "RS0001"
mariadb_role_set_regular_user_code_name: "Regular User"
mariadb_role_set_regular_user_code: "RS0002"
mariadb_role_set_init_user_code_name: "Init User"
mariadb_role_set_init_user_code: "RS0003"

# DASHBOARD
caas_dashboard_instances: 1
caas_dashboard_port: 8091
caas_dashboard_azs: [z3]
caas_dashboard_management_security_enabled: false
caas_dashboard_logging_level: "INFO"

# API
caas_api_instances: 1
caas_api_port: 3333
caas_api_azs: [z1]
caas_api_management_security_enabled: false
caas_api_logging_level: "INFO"

# COMMON API
caas_common_api_instances: 1
caas_common_api_port: 3334
caas_common_api_azs: [z2]
caas_common_api_logging_level: "INFO"

# SERVICE BROKER
caas_service_broker_instances: 1
caas_service_broker_port: 8888
caas_service_broker_azs: [z3]

# ADDON
caas_apply_addons_azs: [z1]

# MASTER
caas_master_backend_port: 8443
caas_master_port: 8443
caas_master_azs: [z2]
caas_master_persistent_disk_type: 5120

# WORKER
caas_worker_instances: 3
caas_worker_azs: [z1,z2,z3]

```

- Deploy 스크립트 파일을 서버 환경에 맞게 수정한다.
  -	vSphere : **deploy_vsphere.sh**
  - AWS : **deploy_aws.sh**
  - OpenStack : **deploy_openstack.sh**

```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment/paasta-container-service-2.0
$ vi deploy-vsphere.sh

#!/bin/bash

# SET VARIABLES
export CAAS_DEPLOYMENT_NAME='paasta-container-service'
export CAAS_BOSH2_NAME='micro-bosh'
export CAAS_BOSH2_UUID=`bosh int <(bosh -e ${CAAS_BOSH2_NAME} environment --json) --path=/Tables/0/Rows/0/uuid`

# DEPLOY
bosh -e ${CAAS_BOSH2_NAME} -n -d ${CAAS_DEPLOYMENT_NAME} deploy --no-redact manifests/paasta-container-service-deployment.yml \
    -l manifests/paasta-container-service-vsphere-vars.yml \
    -o manifests/ops-files/paasta-container-service/vsphere-network.yml \
    -o manifests/ops-files/paasta-container-service/use-compiled-releases.yml \
    -o manifests/ops-files/iaas/vsphere/cloud-provider.yml \
    -o manifests/ops-files/iaas/vsphere/set-working-dir-no-rp.yml \
    -o manifests/ops-files/rename.yml \
    -o manifests/ops-files/misc/single-master.yml \
    -o manifests/ops-files/misc/first-time-deploy.yml \
    -v director_uuid=${CAAS_BOSH2_UUID} \
    -v director_name=${CAAS_BOSH2_NAME} \
    -v deployment_name=${CAAS_DEPLOYMENT_NAME}


```
**※	Private Image Repository 를 사용할 경우**
```
* 각 IaaS 환경에 따라 아래 명령어를 추가한다.

> vSphere
manifests/ops-files/paasta-container-service/add-private-image-repository-vsphere.yml

> AWS
manifests/ops-files/paasta-container-service/add-private-image-repository-aws.yml

> OpenStack
manifests/ops-files/paasta-container-service/add-private-image-repository-openstack.yml
```

- Container 서비스팩을 배포한다.

```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment/paasta-container-service-2.0
$ ./remove-all-addons.sh
$ ./deploy-vsphere.sh

Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

Using deployment 'paasta-container-service'

######################################################### 100.00% 11.43 KiB/s 0s
Task 62603

############################################################ 100.00% 38.92 KiB/s
Task 62603 | 01:19:56 | Verifying manifest: Verifying manifest (00:00:00)
############################################################ 100.00% 15.33 KiB/sencies (00:00:01)
Task 62603 | 01:19:57 | Processing 3 existing packages: Processing 3 existing packages (00:00:00)
######################################################### 100.00% 12.16 KiB/s 0s00:00:00)

########################################################### 100.00% 139.06 KiB/s
Task 62603 | 01:19:57 | Release has been created: cfcr-etcd/1.3 (00:00:00)

Task 62603 Started  Tue Nov 13 01:19:56 UTC 2018
Task 62603 Finished Tue Nov 13 01:19:57 UTC 2018
Task 62603 Duration 00:00:01
Task 62603 done
############################################################ 100.00% 69.59 KiB/s
Task 62604 | 01:19:57 | Extracting release: Extracting release (00:00:01)
Task 62604 | 01:19:58 | Verifying manifest: Verifying manifest (00:00:00)
######################################################### 100.00% 56.67 KiB/s 0sencies

Task 62605
Task 62604 | 01:19:58 | Resolving package dependencies: Resolving package dependencies (00:00:00)
Task 62604 | 01:19:58 | Processing 4 existing packages: Processing 4 existing packages (00:00:00)
Task 62604 | 01:19:58 | Processing 2 existing jobs: Processing 2 existing jobs (00:00:00)
Task 62604 | 01:19:58 | Release has been created: bpm/0.6.0 (00:00:00)

Task 62604 Started  Tue Nov 13 01:19:57 UTC 2018
Task 62604 Finished Tue Nov 13 01:19:58 UTC 2018
Task 62604 Duration 00:00:01
Task 62604 done

Task 62605 | 01:19:59 | Extracting release: Extracting release (00:00:00)
Task 62605 | 01:19:59 | Verifying manifest: Verifying manifest (00:00:00)
Task 62605 | 01:19:59 | Resolving package dependencies: Resolving package dependencies (00:00:00)
Task 62605 | 01:19:59 | Processing 7 existing packages: Processing 7 existing packages (00:00:00)
Task 62605 | 01:19:59 | Processing 6 existing jobs: Processing 6 existing jobs (00:00:00)
Task 62605 | 01:19:59 | Release has been created: docker/32.0.0 (00:00:00)

Task 62605 Started  Tue Nov 13 01:19:59 UTC 2018
Task 62605 Finished Tue Nov 13 01:19:59 UTC 2018
Task 62605 Duration 00:00:00
Task 62605 done
######################################################## 100.00% 267.40 KiB/s 0s

Task 62606

Task 62606 | 01:20:01 | Extracting release: Extracting release (00:00:00)
Task 62606 | 01:20:01 | Verifying manifest: Verifying manifest (00:00:00)
Task 62606 | 01:20:01 | Resolving package dependencies: Resolving package dependencies (00:00:00)
Task 62606 | 01:20:02 | Processing 6 existing packages: Processing 6 existing packages (00:00:00)
Task 62606 | 01:20:02 | Processing 4 existing jobs: Processing 4 existing jobs (00:00:00)
Task 62606 | 01:20:02 | Release has been created: bosh-dns/1.5.0 (00:00:00)

Task 62606 Started  Tue Nov 13 01:20:01 UTC 2018
Task 62606 Finished Tue Nov 13 01:20:02 UTC 2018
Task 62606 Duration 00:00:01
Task 62606 done
######################################################## 100.00% 243.99 KiB/s 0s

Task 62607

Task 62607 | 01:20:07 | Extracting release: Extracting release (00:00:00)
Task 62607 | 01:20:07 | Verifying manifest: Verifying manifest (00:00:00)
Task 62607 | 01:20:07 | Resolving package dependencies: Resolving package dependencies (00:00:00)
Task 62607 | 01:20:07 | Processing 12 existing packages: Processing 12 existing packages (00:00:01)
Task 62607 | 01:20:08 | Processing 15 existing jobs: Processing 15 existing jobs (00:00:00)
Task 62607 | 01:20:08 | Release has been created: kubo/0.19.0 (00:00:00)

Task 62607 Started  Tue Nov 13 01:20:07 UTC 2018
Task 62607 Finished Tue Nov 13 01:20:08 UTC 2018
Task 62607 Duration 00:00:01
Task 62607 done
######################################################## 100.00% 168.53 KiB/s 0s

Task 62608

Task 62608 | 01:20:14 | Extracting release: Extracting release (00:00:00)
Task 62608 | 01:20:14 | Verifying manifest: Verifying manifest (00:00:00)
Task 62608 | 01:20:14 | Resolving package dependencies: Resolving package dependencies (00:00:00)
Task 62608 | 01:20:14 | Processing 8 existing packages: Processing 8 existing packages (00:00:00)
Task 62608 | 01:20:14 | Processing 6 existing jobs: Processing 6 existing jobs (00:00:00)
Task 62608 | 01:20:14 | Release has been created: paasta-container-service-projects-release/1.0 (00:00:00)

Task 62608 Started  Tue Nov 13 01:20:14 UTC 2018
Task 62608 Finished Tue Nov 13 01:20:14 UTC 2018
Task 62608 Duration 00:00:00
Task 62608 done
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
+     cpu: 1
+     disk: 4096
+     ram: 256
+   name: service_tiny
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 512
+   name: service_small
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 1024
+   name: service_medium
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 2048
+   name: service_medium_1CPU_2G
+ - cloud_properties:
+     cpu: 2
+     disk: 8192
+     ram: 4096
+   name: service_medium_4G
+ - cloud_properties:
+     cpu: 2
+     disk: 10240
+     ram: 2048
+   name: service_medium_2G
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 256
+   name: portal_tiny
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
+     - host: <CLOUD_PROPERTY_PORT_OF_HOST>
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
+     - 115.68.47.175 - 115.68.47.190
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
+     - 115.68.46.217 - 115.68.46.222
+     static:
+     - 115.68.46.214 - 115.68.46.216
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
+ - alias: trusty
+   os: ubuntu-trusty
+   version: '3586.26'

+ releases:
+ - name: kubo
+   url: file://./releases/kubo-release.tgz
+   version: 0.19.0
+ - name: cfcr-etcd
+   sha1: 72a9753b08980229c73b623c4ddeea59a5afc96f
+   url: file://./releases/cfcr-etcd-release-1.3.tgz
+   version: '1.3'
+ - name: docker
+   sha1: f192296243e68150ae66656fc9134a2b2a023184
+   url: file://./releases/docker-32.0.0.tgz
+   version: 32.0.0
+ - name: bosh-dns
+   sha1: 66e74bb601b691ea4836f859c556de8a5c6c1264
+   url: file://./releases/bosh-dns-release-1.5.0.tgz
+   version: 1.5.0
+ - name: bpm
+   sha1: 4f0f239abdc801d71de9063625aa56e3c42634b5
+   url: file://./releases/bpm-release-0.6.0.tgz
+   version: 0.6.0
+ - name: paasta-container-service-projects-release
+   url: file://./releases/paasta-container-service-projects-release.tgz
+   version: '1.0'

+ update:
+   canaries: 1
+   canary_watch_time: 10000-300000
+   max_in_flight: 100%
+   update_watch_time: 10000-300000

+ addons:
+ - include:
+     stemcells:
+     - os: ubuntu-trusty
+   jobs:
+   - name: bosh-dns
+     properties:
+       api:
+         client:
+           tls: "((/dns_api_client_tls))"
+         server:
+           tls: "((/dns_api_server_tls))"
+       cache:
+         enabled: true
+       health:
+         client:
+           tls: "((/dns_healthcheck_client_tls))"
+         enabled: true
+         server:
+           tls: "((/dns_healthcheck_server_tls))"
+     release: bosh-dns
+   name: bosh-dns
+ - jobs:
+   - name: kubo-dns-aliases
+     release: kubo
+   name: bosh-dns-aliases

+ variables:
+ - name: kubo-admin-password
+   type: password
+ - name: kubelet-password
+   type: password
+ - name: kubelet-drain-password
+   type: password
+ - name: kube-proxy-password
+   type: password
+ - name: kube-controller-manager-password
+   type: password
+ - name: kube-scheduler-password
+   type: password
+ - name: route-sync-password
+   type: password
+ - name: kubo_ca
+   options:
+     common_name: ca
+     is_ca: true
+   type: certificate
+ - name: tls-kubelet
+   options:
+     alternative_names: []
+     ca: kubo_ca
+     common_name: kubelet.cfcr.internal
+     organization: system:nodes
+   type: certificate
+ - name: tls-kubelet-client
+   options:
+     ca: kubo_ca
+     common_name: kube-apiserver.cfcr.internal
+     extended_key_usage:
+     - client_auth
+     organization: system:masters
+   type: certificate
+ - name: tls-kubernetes
+   options:
+     alternative_names:
+     - 115.68.47.178
+     - 10.100.200.1
+     - kubernetes
+     - kubernetes.default
+     - kubernetes.default.svc
+     - kubernetes.default.svc.cluster.local
+     - master.cfcr.internal
+     ca: kubo_ca
+     common_name: master.cfcr.internal
+     organization: system:masters
+   type: certificate
+ - name: service-account-key
+   type: rsa
+ - name: tls-docker
+   options:
+     ca: kubo_ca
+     common_name: docker.cfcr.internal
+   type: certificate
+ - name: tls-etcd-v0-17-0
+   options:
+     ca: kubo_ca
+     common_name: "*.etcd.cfcr.internal"
+     extended_key_usage:
+     - client_auth
+     - server_auth
+   type: certificate
+ - name: tls-etcdctl
+   options:
+     ca: kubo_ca
+     common_name: etcdClient
+     extended_key_usage:
+     - client_auth
+   type: certificate
+ - name: tls-heapster
+   options:
+     alternative_names:
+     - heapster.kube-system.svc.cluster.local
+     ca: kubo_ca
+     common_name: heapster
+   type: certificate
+ - name: tls-influxdb
+   options:
+     alternative_names: []
+     ca: kubo_ca
+     common_name: monitoring-influxdb
+   type: certificate
+ - name: kubernetes-dashboard-ca
+   options:
+     common_name: ca
+     is_ca: true
+   type: certificate
+ - name: tls-kubernetes-dashboard
+   options:
+     alternative_names: []
+     ca: kubernetes-dashboard-ca
+     common_name: kubernetesdashboard.cfcr.internal
+   type: certificate
+ - name: "/dns_healthcheck_tls_ca"
+   opsname: global_dns_healthcheck_tls_ca
+   options:
+     common_name: dns-healthcheck-tls-ca
+     is_ca: true
+   type: certificate
+ - name: "/dns_healthcheck_server_tls"
+   opsname: global_dns_healthcheck_server_tls
+   options:
+     ca: "/dns_healthcheck_tls_ca"
+     common_name: health.bosh-dns
+     extended_key_usage:
+     - server_auth
+   type: certificate
+ - name: "/dns_healthcheck_client_tls"
+   opsname: global_dns_healthcheck_client_tls
+   options:
+     ca: "/dns_healthcheck_tls_ca"
+     common_name: health.bosh-dns
+     extended_key_usage:
+     - client_auth
+   type: certificate
+ - name: "/dns_api_tls_ca"
+   opsname: global_dns_api_tls_ca
+   options:
+     common_name: dns-api-tls-ca
+     is_ca: true
+   type: certificate
+ - name: "/dns_api_server_tls"
+   opsname: global_dns_api_server_tls
+   options:
+     ca: "/dns_api_tls_ca"
+     common_name: api.bosh-dns
+     extended_key_usage:
+     - server_auth
+   type: certificate
+ - name: "/dns_api_client_tls"
+   opsname: global_dns_api_client_tls
+   options:
+     ca: "/dns_api_tls_ca"
+     common_name: api.bosh-dns
+     extended_key_usage:
+     - client_auth
+   type: certificate

+ features:
+   use_dns_addresses: true

+ instance_groups:
+ - azs:
+   - z1
+   instances: 1
+   jobs:
+   - name: haproxy
+     properties:
+       caas_api:
+         server:
+           port: 3333
+         urls:
+         - 10.30.107.40
+       caas_common_api:
+         server:
+           port: 3334
+         urls:
+         - 10.30.107.41
+       caas_dashboard:
+         server:
+           port: 8091
+         urls:
+         - 10.30.107.42
+       caas_service_broker:
+         server:
+           port: 8888
+         urls:
+         - 10.30.107.43
+       host_ip: 10.30.107.44
+       http_port: 8080
+       mariadb:
+         port: '<MARIADB_PORT>'
+         urls:
+         - 10.30.107.45
+     release: paasta-container-service-projects-release
+   name: haproxy
+   networks:
+   - name: service_private
+     static_ips: 10.30.107.44
+   - default:
+     - dns
+     - gateway
+     name: service_public
+     static_ips: 115.68.47.179
+   stemcell: trusty
+   update:
+     max_in_flight: 1
+     serial: true
+   vm_type: caas_small
+ - azs:
+   - z2
+   instances: 1
+   jobs:
+   - name: mariadb
+     properties:
+       admin_user:
+         id: <MARIADB_ADMIN_USER_ID>
+         password: <MARIADB_ADMIN_USER_PASSWORD>
+       port: '<MARIADB_PORT>'
+     release: paasta-container-service-projects-release
+   name: mariadb
+   networks:
+   - name: service_private
+     static_ips:
+     - 10.30.107.45
+   persistent_disk_type: 10GB
+   stemcell: trusty
+   update:
+     max_in_flight: 1
+     serial: true
+   vm_type: caas_small
+ - azs:
+   - z3
+   instances: 1
+   jobs:
+   - name: container-service-dashboard
+     properties:
+       caas:
+         url: https://115.68.47.178:8443
+       caasApi:
+         authorization:
+           id: <CAAS_API_AUTH_ID>
+           password: <CAAS_API_AUTH_PASSWORD>
+         port: 3333
+         url: 115.68.47.179
+       cf:
+         api:
+           url: https://api.<DOMAIN>
+         uaa:
+           oauth:
+             authorization:
+               uri: https://uaa.<DOMAIN>/oauth/authorize
+             client:
+               id: <CF_UAA_OAUTH_CLIENT_ID>
+               secret: <CF_UAA_OAUTH_CLIENT_SECRET>
+             info:
+               uri: https://uaa.<DOMAIN>/userinfo
+             logout:
+               url: https://uaa.<DOMAIN>/logout
+             token:
+               access:
+                 uri: https://uaa.<DOMAIN>/oauth/token
+               check:
+                 uri: https://uaa.<DOMAIN>/check_token
+       commonApi:
+         authorization:
+           id: <CAAS_COMMON_API_AUTH_ID>
+           password: <CAAS_COMMON_API_AUTH_PASSWORD>
+         port: 3334
+         url: 115.68.47.179
+       java_opts: "-XX:MaxMetaspaceSize=104857K -Xss349K -Xms681574K -XX:MetaspaceSize=104857K
+         -Xmx681574K"
+       logging:
+         file: logs/application.log
+         level:
+           ROOT: INFO
+         path: classpath:logback-spring.xml
+       management:
+         security:
+           enabled: false
+       server:
+         port: 8091
+       spring:
+         freemarker:
+           template-loader-path: classpath:/templates/
+     release: paasta-container-service-projects-release
+   name: container-service-dashboard
+   networks:
+   - name: service_private
+     static_ips:
+     - 10.30.107.42
+   stemcell: trusty
+   update:
+     max_in_flight: 1
+     serial: true
+   vm_type: caas_small
+ - azs:
+   - z1
+   instances: 1
+   jobs:
+   - name: container-service-api
+     properties:
+       caasMaster:
+         api:
+           url: https://115.68.47.178:8443
+       commonApi:
+         authorization:
+           id: <CAAS_COMMON_API_AUTH_ID>
+           password: <CAAS_COMMON_API_AUTH_PASSWORD>
+         port: 3334
+         url: 115.68.47.179
+       java_opts: "-XX:MaxMetaspaceSize=104857K -Xss349K -Xms681574K -XX:MetaspaceSize=104857K
+         -Xmx681574K"
+       logging:
+         file: logs/application.log
+         level:
+           ROOT: INFO
+         path: classpath:logback-spring.xml
+       management:
+         security:
+           enabled: false
+       server:
+         port: 3333
+     release: paasta-container-service-projects-release
+   name: container-service-api
+   networks:
+   - name: service_private
+     static_ips:
+     - 10.30.107.40
+   stemcell: trusty
+   update:
+     max_in_flight: 1
+     serial: true
+   vm_type: caas_small_api
+ - azs:
+   - z2
+   instances: 1
+   jobs:
+   - name: container-service-common-api
+     properties:
+       caasApi:
+         authorization:
+           id: <CAAS_API_AUTH_ID>
+           password: <CAAS_API_AUTH_PASSWORD>
+         port: 3333
+         url: 115.68.47.179
+       java_opts: "-XX:MaxMetaspaceSize=104857K -Xss349K -Xms681574K -XX:MetaspaceSize=104857K
+         -Xmx681574K"
+       logging:
+         file: logs/application.log
+         level:
+           ROOT: INFO
+         path: classpath:logback-spring.xml
+       server:
+         port: 3334
+       spring:
+         datasource:
+           driver_class_name: com.mysql.cj.jdbc.Driver
+           password: <MARIADB_ADMIN_USER_PASSWORD>
+           url: jdbc:mysql://115.68.47.179:3306/caas?autoReconnect=true&useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Seoul&useLegacyDatetimeCode=false
+           username: <MARIADB_ADMIN_USER_ID>
+           validationQuery: SELECT 1
+         jpa:
+           database: mysql
+           generate-ddl: false
+           hibernate:
+             ddl-auto: none
+             naming:
+               strategy: org.hibernate.cfg.EJB3NamingStrategy
+           properties:
+             hibernate:
+               dialect: org.hibernate.dialect.MySQLInnoDBDialect
+               format_sql: true
+               show_sql: true
+               use_sql_comments: true
+     release: paasta-container-service-projects-release
+   name: container-service-common-api
+   networks:
+   - name: service_private
+     static_ips:
+     - 10.30.107.41
+   stemcell: trusty
+   update:
+     max_in_flight: 1
+     serial: true
+   vm_type: caas_small_api
+ - azs:
+   - z3
+   instances: 1
+   jobs:
+   - name: container-service-broker
+     properties:
+       auth:
+         id: <CAAS_SERVICE_BROKER_AUTH_ID>
+         password: <CAAS_SERVICE_BROKER_AUTH_PASSWORD>
+       caas:
+         api_server_url: https://115.68.47.178:8443
+         cluster: micro-bosh/paasta-container-service
+         common_api_url: http://115.68.47.179:3334/adminToken
+         exit_code: caas_exit
+         service_broker:
+           auth: <CAAS_SERVICE_BROKER_AUTHORIZED_HEADER>
+           url: http://115.68.47.179:8888/tokens
+       caasMaster:
+         cluster:
+           command: "/var/vcap/jobs/container-service-broker/script/set_caas_service_info.sh"
+         common:
+           port: 3334
+           url: 115.68.47.179
+         url: https://115.68.47.178:8443
+       credhub:
+         admin_client_secret: <CREDHUB_ADMIN_CLIENT_SECRET>
+         server_url: 10.30.40.111:8844
+       dashboard:
+         url: http://115.68.47.179:8091/caas/intro/overview/
+       datasource:
+         driver_class_name: com.mysql.jdbc.Driver
+         password: <MARIADB_ADMIN_USER_PASSWORD>
+         url: jdbc:mysql://<MARIADB_URI>/broker?autoReconnect=true&useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Seoul&useLegacyDatetimeCode=false
+         username: <MARIADB_ADMIN_USER_ID>
+       freemarker:
+         template-loader-path: classpath:/templates/
+       java_opts: "-XX:MaxMetaspaceSize=104857K -Xss349K -Xms681574K -XX:MetaspaceSize=104857K
+         -Xmx681574K"
+       jpa:
+         database_platform: org.hibernate.dialect.MySQL5InnoDBDialect
+         hibernate_ddl_auto: none
+         show_sql: true
+       logging:
+         config: classpath:logback.xml
+         level:
+           org:
+             hibernate: info
+             openpaas:
+               servicebroker: DEBUG
+       server:
+         port: 8888
+       serviceDefinition:
+         bindable: false
+         desc: for CaaS Plans, You can choose plan about CPU, Memory, disk.
+         id: 8a3f2d14-5283-487f-b6c8-6663639ad8b1_test
+         image_url: data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGQAAABkCAMAAABHPGVmAAAC/VBMVEVHcEwxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQxbOQ1/RrDAAAA/nRSTlMAA/wI/SAC/gHfBvoEBwsJ6BD3GPUK+QXt7vMS4fIT+A3wFOQl+93sGhUb8Tnr6sX2Dir0g+Dm2+fXDBa23JQXuxEPzx8dVrpH4lfl7zJEOhnp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
+         name: container-service
+         plan1:
+           cpu: 2
+           desc: 2 CPUs, 2GB Memory (free)
+           disk: 10GB
+           id: f02690d6-6965-4756-820e-3858111ed674test
+           memory: 2GB
+           name: Micro
+           type: A
+           weight: 1
+         plan2:
+           cpu: 4
+           desc: 4 CPUs, 6GB Memory (free)
+           disk: 20GB
+           id: a5213929-885f-414a-801f-c66ddb5e48f1test
+           memory: 6GB
+           name: Small
+           type: B
+           weight: 2
+         plan3:
+           cpu: 8
+           desc: 8 CPUs, 12GB Memory (free)
+           disk: 40GB
+           id: 056d05b6-4039-40ec-8619-e68490b79255test
+           memory: 12GB
+           name: Advanced
+           type: C
+           weight: 3
+         planupdatable: 'true'
+         tags: CaaS,Containers as a Service
+     release: paasta-container-service-projects-release
+   name: container-service-broker
+   networks:
+   - name: service_private
+     static_ips:
+     - 10.30.107.43
+   stemcell: trusty
+   update:
+     max_in_flight: 1
+     serial: true
+   vm_type: caas_small_api
+ - azs:
+   - z1
+   instances: 1
+   jobs:
+   - consumes:
+       cloud-provider:
+         from: master-cloud-provider
+     name: apply-specs
+     properties:
+       admin-password: "((kubo-admin-password))"
+       admin-username: <CAAS-KUBO-ADMIN-USERNAME>
+       api-token: "((kubelet-password))"
+       tls:
+         heapster: "((tls-heapster))"
+         influxdb: "((tls-influxdb))"
+         kubernetes: "((tls-kubernetes))"
+         kubernetes-dashboard: "((tls-kubernetes-dashboard))"
+     release: kubo
+   lifecycle: errand
+   name: apply-addons
+   networks:
+   - name: default
+   stemcell: trusty
+   vm_type: caas_small
+ - azs:
+   - z2
+   instances: 1
+   jobs:
+   - name: bpm
+     release: bpm
+   - name: flanneld
+     release: kubo
+   - consumes:
+       cloud-provider:
+         from: master-cloud-provider
+     name: kube-apiserver
+     properties:
+       admin-password: "((kubo-admin-password))"
+       admin-username: <CAAS-KUBO-ADMIN-USERNAME>
+       backend_port: 8443
+       kube-controller-manager-password: "((kube-controller-manager-password))"
+       kube-proxy-password: "((kube-proxy-password))"
+       kube-scheduler-password: "((kube-scheduler-password))"
+       kubelet-drain-password: "((kubelet-drain-password))"
+       kubelet-password: "((kubelet-password))"
+       port: 8443
+       route-sync-password: "((route-sync-password))"
+       service-account-public-key: "((service-account-key.public_key))"
+       tls:
+         kubelet-client: "((tls-kubelet-client))"
+         kubernetes:
+           ca: "((tls-kubernetes.ca))"
+           certificate: "((tls-kubernetes.certificate))"
+           private_key: "((tls-kubernetes.private_key))"
+     release: kubo
+   - consumes:
+       cloud-provider:
+         from: master-cloud-provider
+     name: kube-controller-manager
+     properties:
+       api-token: "((kube-controller-manager-password))"
+       cluster-signing: "((kubo_ca))"
+       service-account-private-key: "((service-account-key.private_key))"
+       tls:
+         kubernetes: "((tls-kubernetes))"
+     release: kubo
+   - name: kube-scheduler
+     properties:
+       api-token: "((kube-scheduler-password))"
+       tls:
+         kubernetes: "((tls-kubernetes))"
+     release: kubo
+   - consumes:
+       cloud-provider:
+         from: master-cloud-provider
+     name: kubernetes-roles
+     properties:
+       admin-password: "((kubo-admin-password))"
+       admin-username: <CAAS-KUBO-ADMIN-USERNAME>
+       tls:
+         kubernetes: "((tls-kubernetes))"
+     release: kubo
+   - name: etcd
+     properties:
+       etcd:
+         dns_suffix: etcd.cfcr.internal
+       tls:
+         etcd:
+           ca: "((tls-etcd-v0-17-0.ca))"
+           certificate: "((tls-etcd-v0-17-0.certificate))"
+           private_key: "((tls-etcd-v0-17-0.private_key))"
+         etcdctl:
+           ca: "((tls-etcdctl.ca))"
+           certificate: "((tls-etcdctl.certificate))"
+           private_key: "((tls-etcdctl.private_key))"
+         peer:
+           ca: "((tls-etcd-v0-17-0.ca))"
+           certificate: "((tls-etcd-v0-17-0.certificate))"
+           private_key: "((tls-etcd-v0-17-0.private_key))"
+     release: cfcr-etcd
+   - name: smoke-tests
+     release: kubo
+   - name: cloud-provider
+     properties:
+       cloud-provider:
+         type: vsphere
+         vsphere:
+           datacenter: <VCENTER-NAME>
+           datastore: <VCENTER-DATASTORE-NAME>
+           insecure-flag: 1
+           password: <VCENTER-MASTER-PASSWORD>
+           server: <VCENTER-SERVER-IP>
+           user: <VCENTER-MASTER-ID>
+           vms: CF_BOSH2_VMs
+           working-dir: "/BD-DC/vm/CF_BOSH2_VMs"
+     provides:
+       cloud-provider:
+         as: master-cloud-provider
+     release: kubo
+   name: master
+   networks:
+   - name: service_private
+   - default:
+     - dns
+     - gateway
+     name: service_public
+     static_ips: 115.68.47.178
+   persistent_disk: 5120
+   stemcell: trusty
+   vm_type: small
+ - azs:
+   - z1
+   - z2
+   - z3
+   instances: 3
+   jobs:
+   - name: flanneld
+     release: kubo
+   - name: docker
+     properties:
+       bridge: cni0
+       default_ulimits:
+       - nofile=65536
+       env: {}
+       flannel: true
+       ip_masq: false
+       iptables: false
+       log_level: error
+       log_options:
+       - max-size=128m
+       - max-file=2
+       storage_driver: overlay2
+       store_dir: "/var/vcap/data"
+       tls_cacert: "((tls-docker.ca))"
+       tls_cert: "((tls-docker.certificate))"
+       tls_key: "((tls-docker.private_key))"
+     release: docker
+   - name: kubernetes-dependencies
+     release: kubo
+   - name: kubelet
+     properties:
+       api-token: "((kubelet-password))"
+       cloud-provider: vsphere
+       drain-api-token: "((kubelet-drain-password))"
+       tls:
+         kubelet: "((tls-kubelet))"
+         kubelet-client-ca:
+           certificate: "((tls-kubelet-client.ca))"
+         kubernetes: "((tls-kubernetes))"
+     release: kubo
+   - name: kube-proxy
+     properties:
+       api-token: "((kube-proxy-password))"
+       cloud-provider: vsphere
+       tls:
+         kubernetes: "((tls-kubernetes))"
+     release: kubo
+   name: worker
+   networks:
+   - name: service_private
+   stemcell: trusty
+   vm_type: small-highmem-16GB

+ name: paasta-container-service

Task 62609

Task 62609 | 01:20:17 | Preparing deployment: Preparing deployment (00:00:16)
Task 62609 | 01:21:25 | Preparing package compilation: Finding packages to compile (00:00:01)
Task 62609 | 01:21:26 | Creating missing vms: haproxy/ff28d149-3f58-4be6-9e5d-c3e599d48bd9 (0)
Task 62609 | 01:21:26 | Creating missing vms: mariadb/689775ed-d80a-4301-934a-4a053563c672 (0)
Task 62609 | 01:21:26 | Creating missing vms: container-service-api/f8f7ea3f-7451-4bfc-93c8-8cd74fec7506 (0)
Task 62609 | 01:21:26 | Creating missing vms: container-service-dashboard/acbd03d9-f2c2-44b7-be9a-c60eb9706a93 (0)
Task 62609 | 01:21:26 | Creating missing vms: container-service-broker/21285375-f0a0-420e-a65c-20c656a4df0e (0)
Task 62609 | 01:21:26 | Creating missing vms: container-service-common-api/7b1874b7-83bc-4aea-9d88-ed4b8523ab41 (0)
Task 62609 | 01:21:26 | Creating missing vms: worker/562d3b1a-2b1d-4741-8d0e-51aad9efc38e (0)
Task 62609 | 01:21:26 | Creating missing vms: master/e663affd-016f-4132-86ab-7fcba028dad5 (0)
Task 62609 | 01:21:26 | Creating missing vms: worker/5f73b4ec-3c18-43cd-997b-6f21af34e99c (1)
Task 62609 | 01:21:26 | Creating missing vms: worker/dac5e512-55a1-466a-b111-a41590a280c1 (2)
Task 62609 | 01:25:12 | Creating missing vms: container-service-api/f8f7ea3f-7451-4bfc-93c8-8cd74fec7506 (0) (00:03:46)
Task 62609 | 01:25:15 | Creating missing vms: container-service-broker/21285375-f0a0-420e-a65c-20c656a4df0e (0) (00:03:49)
Task 62609 | 01:25:24 | Creating missing vms: container-service-common-api/7b1874b7-83bc-4aea-9d88-ed4b8523ab41 (0) (00:03:58)
Task 62609 | 01:25:34 | Creating missing vms: worker/5f73b4ec-3c18-43cd-997b-6f21af34e99c (1) (00:04:08)
Task 62609 | 01:25:37 | Creating missing vms: mariadb/689775ed-d80a-4301-934a-4a053563c672 (0) (00:04:11)
Task 62609 | 01:25:42 | Creating missing vms: haproxy/ff28d149-3f58-4be6-9e5d-c3e599d48bd9 (0) (00:04:16)
Task 62609 | 01:25:43 | Creating missing vms: container-service-dashboard/acbd03d9-f2c2-44b7-be9a-c60eb9706a93 (0) (00:04:17)
Task 62609 | 01:25:56 | Creating missing vms: worker/562d3b1a-2b1d-4741-8d0e-51aad9efc38e (0) (00:04:30)
Task 62609 | 01:25:56 | Creating missing vms: worker/dac5e512-55a1-466a-b111-a41590a280c1 (2) (00:04:30)
Task 62609 | 01:26:06 | Creating missing vms: master/e663affd-016f-4132-86ab-7fcba028dad5 (0) (00:04:40)
Task 62609 | 01:26:10 | Updating instance haproxy: haproxy/ff28d149-3f58-4be6-9e5d-c3e599d48bd9 (0) (canary) (00:00:45)
Task 62609 | 01:26:55 | Updating instance mariadb: mariadb/689775ed-d80a-4301-934a-4a053563c672 (0) (canary) (00:02:43)
Task 62609 | 01:29:38 | Updating instance container-service-dashboard: container-service-dashboard/acbd03d9-f2c2-44b7-be9a-c60eb9706a93 (0) (canary) (00:00:44)
Task 62609 | 01:30:22 | Updating instance container-service-api: container-service-api/f8f7ea3f-7451-4bfc-93c8-8cd74fec7506 (0) (canary) (00:00:41)
Task 62609 | 01:31:03 | Updating instance container-service-common-api: container-service-common-api/7b1874b7-83bc-4aea-9d88-ed4b8523ab41 (0) (canary) (00:00:42)
Task 62609 | 01:31:45 | Updating instance container-service-broker: container-service-broker/21285375-f0a0-420e-a65c-20c656a4df0e (0) (canary) (00:00:52)
Task 62609 | 01:32:37 | Updating instance master: master/e663affd-016f-4132-86ab-7fcba028dad5 (0) (canary) (00:03:04)
Task 62609 | 01:35:41 | Updating instance worker: worker/562d3b1a-2b1d-4741-8d0e-51aad9efc38e (0) (canary) (00:03:40)
Task 62609 | 01:39:21 | Updating instance worker: worker/5f73b4ec-3c18-43cd-997b-6f21af34e99c (1) (00:01:23)
Task 62609 | 01:40:44 | Updating instance worker: worker/dac5e512-55a1-466a-b111-a41590a280c1 (2) (00:03:08)

Task 62609 Started  Tue Nov 13 01:20:17 UTC 2018
Task 62609 Finished Tue Nov 13 01:43:52 UTC 2018
Task 62609 Duration 00:23:35
Task 62609 done

Succeeded

```


- 업로드된 Container 서비스 릴리즈를 확인한다.

```
$ bosh -e micro-bosh releases
Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

Name                               Version   Commit Hash  
bosh-dns                           1.5.0*    f5a8d25  
bpm                                0.9.0*    c9b7136  
cfcr-etcd                          1.3*      6a62d8f  
docker                             32.0.0*   542c382  
kubo                               0.19.0*   c9294bb  
paasta-container-service-projects-release       1.0*      0126121+  


(*) Currently deployed
(+) Uncommitted changes

6 releases

Succeeded
```


- 배포된 Container 서비스팩을 확인한다.


```
$ bosh -e micro-bosh -d paasta-container-service vms
Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

Task 62642 done

Deployment 'paasta-container-service'

Instance                                                               Process State  AZ  IPs            VM CID                                   VM Type             Active  
container-service-api/f8f7ea3f-7451-4bfc-93c8-8cd74fec7506             running        z1  10.30.107.40   vm-5e7f69e5-dc49-46db-9390-1b3a3053f0ca  caas_small_api      true  
container-service-common-api/7b1874b7-83bc-4aea-9d88-ed4b8523ab41      running        z2  10.30.107.41   vm-b12711c7-8811-4579-82de-b2e264f4e80f  caas_small_api      true  
container-service-dashboard/acbd03d9-f2c2-44b7-be9a-c60eb9706a93       running        z3  10.30.107.42   vm-35dccdc1-f939-41c7-a726-2388f6124920  caas_small          true  
container-service-broker/21285375-f0a0-420e-a65c-20c656a4df0e          running        z3  10.30.107.43   vm-2f9d63ac-e3fe-4e6e-96eb-ef725acc01e1  caas_small_api      true  
haproxy/ff28d149-3f58-4be6-9e5d-c3e599d48bd9                           running        z1  10.30.107.44   vm-c3488820-2eeb-40a2-84a6-e4bbec5e27aa  caas_small          true  
                                                                                          115.68.47.179                                                                 
mariadb/689775ed-d80a-4301-934a-4a053563c672                           running        z2  10.30.107.45   vm-b9a18b3e-281d-471f-ae42-859b65f58a29  caas_small          true  
master/e663affd-016f-4132-86ab-7fcba028dad5                            running        z2  10.30.107.0    vm-58219f94-023c-4fb0-ba5a-ba870071215d  small               true  
                                                                                          115.68.47.178                                                                 
worker/562d3b1a-2b1d-4741-8d0e-51aad9efc38e                            running        z1  10.30.108.0    vm-04f5b88b-1b48-490c-84c4-578cbc4364b2  small-highmem-16GB  true  
worker/5f73b4ec-3c18-43cd-997b-6f21af34e99c                            running        z2  10.30.108.1    vm-468dbbdb-3034-4be8-8d60-3e6d6ba5e0c1  small-highmem-16GB  true  
worker/dac5e512-55a1-466a-b111-a41590a280c1                            running        z3  10.30.108.2    vm-04ca0cad-4282-472b-8ac0-18a3171e6381  small-highmem-16GB  true  

10 vms

Succeeded
```


### <div id='24'/> 2.4. Container 서비스 브로커 등록
Container 서비스팩 배포가 완료되었으면 PaaS-TA 포탈에서 서비스 팩을 사용하기 위해서 먼저 Container 서비스 브로커를 등록해 주어야 한다. 서비스 브로커 등록 시 개방형 클라우드 플랫폼에서 서비스 브로커를 등록할 수 있는 사용자로 로그인이 되어있어야 한다.

- 서비스 브로커 목록을 확인한다.

```
$ cf service-brokers
Getting service brokers as admin...

name                               url
delivery-pipeline-service-broker   http://10.30.107.64:8080
```

- Container 서비스 브로커를 등록한다.


>$ cf create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{서비스팩 URL}
> - 서비스팩 이름 : 서비스 팩 관리를 위해 개방형 클라우드 플랫폼에서 보여지는 명칭
> - 서비스팩 사용자 ID/비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID/비밀번호
> - 서비스팩 URL : 서비스팩이 제공하는 API를 사용할 수 있는 URL

```
$ cf create-service-broker container-service-broker admin cloudfoundry http://115.68.47.179:8888
```

- 등록된 Container 서비스 브로커를 확인한다.

```
$ cf service-brokers
Getting service brokers as admin...

name                               url
container-service-broker                http://10.30.107.43:8888
delivery-pipeline-service-broker   http://10.30.107.64:8080
```

- 접근 가능한 서비스 목록을 확인한다.

```
$ cf service-access
Getting service access as admin...
broker: container-service-broker
   service                plan       access   orgs
   container-service      Micro      none
   container-service      Small      none
   container-service      Advanced   none

broker: delivery-pipeline-service-broker
   service                plan                          access   orgs
   delivery-pipeline-v2   delivery-pipeline-shared      all
   delivery-pipeline-v2   delivery-pipeline-dedicated   all

```

- 특정 조직에 해당 서비스 접근 허용을 할당한다.

```
$ cf enable-service-access caas
Enabling access to all plans of service caas for all orgs as admin...
OK
```

- 접근 가능한 서비스 목록을 확인한다.

```
$ cf service-access
Getting service access as admin...
broker: container-service-broker
   service                plan       access   orgs
   container-service      Micro      all
   container-service      Small      all
   container-service      Advanced   all

broker: delivery-pipeline-service-broker
   service                plan                          access   orgs
   delivery-pipeline-v2   delivery-pipeline-shared      all
   delivery-pipeline-v2   delivery-pipeline-dedicated   all

```


### <div id='25'/> 2.5. Container 서비스 UAA Client Id 등록
UAA 포털 계정 등록 절차에 대한 순서를 확인한다.

- Container 서비스 대시보드에 접근이 가능한 IP를 알기 위해 **haproxy IP** 를 확인한다.

```
$ bosh -e micro-bosh -d paasta-container-service vms
Deployment 'paasta-container-service'

Instance                                                               Process State  AZ  IPs            VM CID                                   VM Type             Active  
container-service-api/f8f7ea3f-7451-4bfc-93c8-8cd74fec7506             running        z1  10.30.107.40   vm-5e7f69e5-dc49-46db-9390-1b3a3053f0ca  caas_small_api      true  
container-service-common-api/7b1874b7-83bc-4aea-9d88-ed4b8523ab41      running        z2  10.30.107.41   vm-b12711c7-8811-4579-82de-b2e264f4e80f  caas_small_api      true  
container-service-dashboard/acbd03d9-f2c2-44b7-be9a-c60eb9706a93       running        z3  10.30.107.42   vm-35dccdc1-f939-41c7-a726-2388f6124920  caas_small          true  
container-service-broker/21285375-f0a0-420e-a65c-20c656a4df0e          running        z3  10.30.107.43   vm-2f9d63ac-e3fe-4e6e-96eb-ef725acc01e1  caas_small_api      true  
haproxy/ff28d149-3f58-4be6-9e5d-c3e599d48bd9                           running        z1  10.30.107.44   vm-c3488820-2eeb-40a2-84a6-e4bbec5e27aa  caas_small          true  
                                                                                          115.68.47.179                                                                 
mariadb/689775ed-d80a-4301-934a-4a053563c672                           running        z2  10.30.107.45   vm-b9a18b3e-281d-471f-ae42-859b65f58a29  caas_small          true  
master/e663affd-016f-4132-86ab-7fcba028dad5                            running        z2  10.30.107.0    vm-58219f94-023c-4fb0-ba5a-ba870071215d  small               true  
                                                                                          115.68.47.178                                                                 
worker/562d3b1a-2b1d-4741-8d0e-51aad9efc38e                            running        z1  10.30.108.0    vm-04f5b88b-1b48-490c-84c4-578cbc4364b2  small-highmem-16GB  true  
worker/5f73b4ec-3c18-43cd-997b-6f21af34e99c                            running        z2  10.30.108.1    vm-468dbbdb-3034-4be8-8d60-3e6d6ba5e0c1  small-highmem-16GB  true  
worker/dac5e512-55a1-466a-b111-a41590a280c1                            running        z3  10.30.108.2    vm-04ca0cad-4282-472b-8ac0-18a3171e6381  small-highmem-16GB  true  

10 vms

Succeeded

```

- uaac server의 endpoint를 설정한다.

```
$ uaac target
Target: https://10.30.40.111:8443
Context: uaa_admin, from client uaa_admin
```

- URL을 변경하고 싶을 경우 아래와 같이 입력하여 변경 가능하다. <br>
```
uaac target https://uaa.<DOMAIN>
```

- UAAC 로그인을 한다.

```
$ uaac token client get
Client ID: ****************
Client secret: ****************

Successfully fetched token via client credentials grant.
Target: https://uaa.<DOMAIN>
Context: admin, from client admin

```

- Container 서비스 계정 생성을 한다.

> $ uaac client add caasclient -s {클라이언트 비밀번호} --redirect_uri {컨테이너 서비스 대시보드 URI} --scope {퍼미션 범위} --authorized_grant_types {권한 타입} --authorities={권한 퍼미션} --autoapprove={자동승인권한}
> - 클라이언트 비밀번호 : uaac 클라이언트 비밀번호를 입력한다.
> - 컨테이너 서비스 대시보드 URI : 성공적으로 리다이렉션 할 컨테이너 서비스 대시보드 URI를 입력한다.
> - 퍼미션 범위: 클라이언트가 사용자를 대신하여 얻을 수있는 허용 범위 목록을 입력한다.
> - 권한 타입 : 서비스팩이 제공하는 API를 사용할 수 있는 권한 목록을 입력한다.
> - 권한 퍼미션 : 클라이언트에 부여 된 권한 목록을 입력한다.
> - 자동승인권한: 사용자 승인이 필요하지 않은 권한 목록을 입력한다.

```
$ uaac client add caasclient -s clientsecret --redirect_uri "http://localhost:8091, http://115.68.47.179:8091" \
--scope "cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" \
--authorized_grant_types "authorization_code , client_credentials , refresh_token" \
--authorities="uaa.resource" \
--autoapprove="openid , cloud_controller_service_permissions.read"
```


- Container 서비스 계정 수정을 한다. (이미 uaac client가 등록되어 있는 경우)

> $ uaac client update caasclient --redirect_uri={컨테이너 서비스 대시보드 URI}
> - 컨테이너 서비스 대시보드 URI : 성공적으로 리다이렉션 할 컨테이너 서비스 대시보드 URI를 입력한다.

```
$ uaac client update caasclient --redirect_uri="http://54.180.13.40:8091 http://115.68.151.177:8091 http://localhost:8091 http://115.68.47.179:8091 http://115.68.47.176:8091
```

### <div id='26'/> 2.6. Container 서비스 Private Image Repository 설정 (Optional)

해당 설정은 Container 서비스에서 설치된 Private Image Repository에 저장된 Image를 참조하기 위해
각 Work Node Host의 Private Image Repository와 Docker 간의 액세스를 위한 설정이다.

-	insecure registry 항목 등록(모든 Work Node마다 수정 필요)


```
$ vi /etc/docker/daemon.json  
{
    "insecure-registries": [
        "10.30.111.41:5000"
    ]
}

$ sudo /var/vcap/jobs/docker/bin/ctl stop
$ sudo /var/vcap/jobs/docker/bin/ctl start
```


[Architecture]:/Service-Guide/images/caas/CaaS_Architecture.png
