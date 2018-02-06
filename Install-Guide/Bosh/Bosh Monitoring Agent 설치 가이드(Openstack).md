## Table of Contents
1. [문서 개요](#1)
     * [1.1. 목적](#2)
     * [1.2. 범위](#3)
     * [1.3. 참고](#4)
2. [Bosh 배포](#5)
     * [2.1.  download "Bosh Agent"](#6)
     * [2.2.  manifest 파일 설정](#7)
     * [2.3.  Bosh 배포](#8)

<div id='1'></div>

# 1. 문서 개요

<div id='2'></div>

### 1.1. 목적

본 문서는 IaaS(Infrastructure as a Service) 중 하나인 Openstack 환경에서 Bosh VM 환경의 시스템 Metrics(CPU, Memory, Disk, Process, Network) 정보를 수집하기 위한 agent를  설치하는 방법을 제공하는데 그 목적이 있다.

<kbd>![1-1-1]</kbd>

<div id='3'></div>

### 1.2. 범위

본 문서는 Openstack 기반에 설치하기 위한 내용으로 한정되어 있다.

<div id='4'></div>

### 1.3. 참고  
      
> <a style="text-decoration:underline" href="https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/master/Install-Guide/monitoring/PaaS-TA%20%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81%20%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EC%B9%98%EA%B0%80%EC%9D%B4%EB%93%9C.md">모니터링 시스템 Architecutre</a>

<div id='5'></div>

# 2.  Bosh 배포

##### Bosh를 설치하는 방법에는 [수동 설치]와 [PaaS-TA Bosh 플랫롬 설치 자동화 Tool]를 이용한 자동설치 2가지 방법이 있다. 본 문서는 수동으로 Bosh 수동 설치시 모니터링 Agent를 설치하는 방법을 제공한다. 만약 수동설치시 Bosh 모니터링 Agent 설치를 원하지 않는다면 아래 [Bosh 수동설치 사용자 가이드 ]를 참고하기 바란다.

## Prerequisites
Bosh를 설치 하기 위해서는 bosh-init이 PaaS 서비스 설치환경에 설치 되어 있어야 한다.

### 참고  
 PaaS 서비스 설치환경 : PaaS-TA를 설치하기위한 환경으로 Ubuntu OS를 기본으로 한다.


 Bosh개념 이해 및 수동 설치를 하기 위해서는 아래 Link를 참조
 > <a style="text-decoration:underline" href="https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/master/Use-Guide/Bosh/PaaS-TA_BOSH_%EC%82%AC%EC%9A%A9%EC%9E%90_%EA%B0%80%EC%9D%B4%EB%93%9Cv1.0.md">Bosh 수동설치 사용자 가이드 </a>

 PaaS-TA Bosh 플랫롬 설치 자동화 Tool을 활용하여 Bosh 설치하려면 아래 Link를 참조
 자동화 Tool에서 Bosh 모니터링을 사용하기 위해서는 Link 3.6.6의 [BOOTSTRAP 설치 – 기본 정보] 부분을 참조
 > <a style="text-decoration:underline" href="https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/master/Use-Guide/platform/PaaS_TA_%ED%94%8C%EB%9E%AB%ED%8F%BC_%EC%84%A4%EC%B9%98_%EC%9E%90%EB%8F%99%ED%99%94_%EC%82%AC%EC%9A%A9_%EA%B0%80%EC%9D%B4%EB%93%9Cv3.0.md#40">Bosh 자동화 설치 사용자 가이드 </a>


본 장에서는 Bosh를 배포하는 방법에 대해서 기술하였다. PaaS-TA 2.0 에서는 Bosh Monitoring Agent를 수동으로 배포 하였으나 PaaS-TA3.0에서는 Bosh Deploy명령으로 Bosh Monitoring Agent를 자동 배포 할 수 있다.

<div id='6'></div>

### 2.1.  download "Bosh Agent"

하단 링크로 접속하여 PaaS-TA 모니터링 패키지 파일을 다운로드하여, 압축해제한다.

>PaaS-TA 모니터링 : **<https://paas-ta.kr/data/packages/3.0/PaaSTA-Monitoring.zip>**

>다운로드 받은 PaaSTA-Monitoring.zip 파일을 압축해제한다. <br>
>bosh-monitoring-agent-1.2.tgz 파일을 확인한다. <br>
~/workspace/bosh-agent 디렉토리로 이동한다. <br>

<div id='7'></div>

### 2.2.  manifest 파일 설정

1. "bosh" 서비스가 배포되는 환경에 맞게 manifest 파일의 설정 정보를 수정한다.


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
10. Template: Bosh VM에서 실행될 Process Job

$ vi bosh-init.yml

```
---
name: paasta-monitoring-bosh

releases:
- name: bosh
  url: https://bosh.io/d/github.com/cloudfoundry/bosh?v=261.4
  sha1: 4da9cedbcc8fbf11378ef439fb89de08300ad091
- name: bosh-openstack-cpi
  url: https://bosh.io/d/github.com/cloudfoundry-incubator/bosh-openstack-cpi-release?v=32
  sha1: ed97641c755f455f2494fe1d7016b9fd1c3ec1e8
# Bosh 모니터링 Agent Release (Option)
- name: bosh-monitoring-agent
  url: file:///home/ubuntu/workspace/bosh-agent/bosh-monitoring-agent-1.2.tgz   #Monitoring Agent
  sha1: e4442afcc64123e11f2b33cc2be799a0b59207d0  

resource_pools:
- name: vms
  network: private
  env:
    bosh:
      password: $6$vfBOZYF/t$fapFotTmMh93k1rloiZP4/Z2o8KY3Xf.VXcHfexE/j5U7TPaa4sQvbp4c.2v9xS7yrucYUphQrBVsxdqYsfez0 #crossent1234
  stemcell:
    url: https://bosh.io/d/stemcells/bosh-openstack-kvm-ubuntu-trusty-go_agent?v=3445.2
    sha1: 72f4c939eacd7396d5f1a5f3382673251f4468c0
  cloud_properties:
    instance_type: m1.large

disk_pools:
- name: disks
  disk_size: 50_000

networks:
- name: private
  type: manual
  subnets:
  - range: 10.10.10.0/24
    gateway: 10.10.10.1
    dns: [8.8.8.8]
    cloud_properties: {net_id: 46f55d82-a239-4538-b95e-a20609bd2312}
- name: public
  type: vip

jobs:
- name: bosh
  instances: 1

  templates:
  - {name: nats, release: bosh}
  - {name: postgres-9.4, release: bosh}
  - {name: blobstore, release: bosh}
  - {name: director, release: bosh}
  - {name: health_monitor, release: bosh}
  - {name: registry, release: bosh}
  - {name: openstack_cpi, release: bosh-openstack-cpi}
  # Bosh 모니터링 Metric Agent Release Job 추가 (Option)
  - {name: metrics_agent, release: bosh-monitoring-agent}
  # Bosh 모니터링 Log Agent Release Job 추가 (Option)
  - {name: logstash, release: bosh-monitoring-agent}
  resource_pool: vms
  persistent_disk: 50_000

  networks:
  - name: private
    static_ips: [10.10.10.26]
    default: [dns, gateway]
  - name: public
    static_ips: [xxx.xxx.xxx.xxx]      #Bosh VM의 public IP

  properties:
  ####### Monitoing Agent 설정 Start #######
    logstash:
      ingestor_bosh:
        host: 10.10.6.37		            # Bosh Log Data(CPU/Memory/Disk) 정보를 전송할 Ingestor VM IP(PaaS-TA Logsearch 설치시 Log Data 전송됨)
        port: 3001
      input:
        file_path: /var/vcap/store/director/tasks/**/debug
    metrics_agent:
      influxdb:
        url: 10.10.6.11:8059	        	# Metric Data(CPU/Memory/Disk) 정보를 전송할 InfluxDB VM IP(Paas-TA-Influx-Grafana 설치시 Data 전송됨)
        database: bosh_metric_db
        measurement: bosh_metrics
        processMeasurement: bosh_process_metrics
      origin: my-bosh
   ####### Monitoing Agent 설정 Stop #######
    nats:
      address: 127.0.0.1
      user: nats
      password: nats-password

    postgres: &db
      listen_address: 127.0.0.1
      host: 127.0.0.1
      user: postgres
      password: postgres-password
      database: bosh
      adapter: postgres

    registry:
      address: 10.10.10.26
      host: 10.10.10.26
      db: *db
      http: {user: admin, password: admin, port: 25777}
      username: admin
      password: admin
      port: 25777

    blobstore:
      address: 10.10.10.26
      port: 25250
      provider: dav
      director: {user: director, password: director-password}
      agent: {user: agent, password: agent-password}

    director:
      address: 127.0.0.1
      name: my-bosh
      db: *db
      cpi_job: openstack_cpi
      max_threads: 3
      user_management:
        provider: local
        local:
          users:
          - {name: admin, password: admin}
          - {name: hm, password: hm-password}

    hm:
      director_account: {user: hm, password: hm-password}
      resurrector_enabled: true

    openstack: &openstack
      auth_url: http://xxx.xxx.xxx.xxx:5000/v3/     #openstack auth ip and port
      project: paasta                               #openstack project name
      domain: default
      username: paasta
      api_key: xxxxxx
      default_key_name: xxxxxx
      default_security_groups: [xxxxxx]
      ignore_server_availability_zone: true
      connection_options:
        connect_timeout: 360

    agent: {mbus: "nats://nats:nats-password@10.10.10.26:4222"}

    ntp: &ntp [0.pool.ntp.org, 1.pool.ntp.org]

cloud_provider:
  template: {name: openstack_cpi, release: bosh-openstack-cpi}

  ssh_tunnel:
    host: xxx.xxx.xxx.xxx
    port: 22
    user: vcap
    private_key: /home/ubuntu/.ssh/monitoring-hostway.pem

  mbus: "https://mbus:mbus-password@xxx.xxx.xxx.xxx:6868"

  properties:
    openstack: *openstack
    agent: {mbus: "https://mbus:mbus-password@0.0.0.0:6868"}
    blobstore: {provider: local, path: /var/vcap/micro_bosh/data/cache}
    ntp: *ntp
```

<div id='8'></div>

### 3.4.  Bosh 배포

bosh-init 명령을 사용하여 bosh deploy한다.

```
$ bosh-init deploy bosh-init.yml
```

[1-1-1]:images/1-1-1.png
