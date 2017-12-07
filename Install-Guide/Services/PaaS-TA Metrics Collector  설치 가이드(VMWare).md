## Table of Contents
1. [문서 개요](#1)
     * [1.1. 목적](#2)
     * [1.2. 범위](#3)
     * [1.3. 전제조건](#4)
     * [1.4. 참고](#5)
2. [Metrics Collector Release 배포](#6)
     * [2.1.  upload release](#7)
     * [2.2.  manifest 파일 설정](#8)
     * [2.3.  deploy](#9)
     * [2.4.  확인](#10)

<div id='1'></div>

# 1. 문서 개요

<div id='2'></div>

### 1.1. 목적

본 문서는 IaaS(Infrastructure as a Service) 중 하나인 VMWare 환경에서 모니터링 시스템의 주요 정보인 PaaS-TA 각 서비스에서 생성한 시스템 Metrics(CPU, Memory, Disk) 정보를 수집하여, 데이터베이스 시스템(InfluxDB)에 저장하기 위한 Metrics Collector Release를 설치하기 위한 가이드를 제공하는데 그 목적이 있다.

<div id='3'></div>

### 1.2. 범위

본 문서는 VMWare 기반에 설치하기 위한 내용으로 한정되어 있다.

<div id='4'></div>

### 1.3. 전제조건

1. Metrics Collector 서비스를 설치하기 위해서는 사전에 PaaS-TA 서비스가 배포되어 서비스되고 있어야 한다.
2. PaaS-Ta 서비스에서 발생한 시스템 Metrics를 저장하기 위한 Database 시스템(InfluxDB)이 배포되어 서비스되고 있어야 한다

<div id='5'></div>

### 1.4. 참고

> <a style="text-decoration:underline" href="https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/master/Install-Guide/monitoring/PaaS-TA%20%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81%20%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EC%B9%98%EA%B0%80%EC%9D%B4%EB%93%9C.md">모니터링 시스템 Architecutre</a>

<div id='6'></div>

# 2.  Metrics Collector Release 배포

본 장에서는 Metrics Collector Release를 배포하는 방법에 대해서 기술하였다.

<div id='7'></div>

### 2.1.  upload "Metrics Collector" release

하단 링크로 접속하여 PaaS-TA 모니터링 패키지 파일을 다운로드하여, 압축해제한다.

>PaaS-TA 모니터링 : **<https://paas-ta.kr/data/packages/3.0/PaaSTA-Monitoring.zip>** <br>
>다운로드 받은 PaaSTA-Monitoring.zip 파일을 압축해제한다. <br>
>paasta-metrics-collector-3.0.tgz 파일을 확인한다. <br>

다음의 명령어를 이용하여 릴리즈 파일을 bosh에 업로드한다.

$ bosh upload release paasta-metrics-collector-2.0.tgz

<kbd>![2-1-1]</kbd>

<div id='8'></div>

### 2.2.  manifest 파일 설정

> <a style="text-decoration:underline" href="https://github.com/OpenPaaSRnD/Documents-PaaSTA-2.0/blob/master/Use-Guide/PaaS-TA%20%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81%20DB%20%EB%B0%8F%20Metrics%20%EA%B0%80%EC%9D%B4%EB%93%9C.md">InfluxDB 참조</a>

1. "Metrics Collector" 서비스가 배포되는 환경에 맞게 manifest 파일의 설정 정보를 수정한다.

$ vi metrics-collector-release.yml

```
---
---
compilation:
  cloud_properties:
    cpu: 2
    disk: 8192
    ram: 2048
  network: paasta-metrics-collector-network			#network name
  reuse_compilation_vms: true
  workers: 1


director_uuid: <%= `bosh status --uuid` %>				#bosh uuid

jobs:
- instances: 3
  name: metrics_collector_z1							#service name
  networks:
  - name: paasta-metrics-collector-net					#network name
    static_ips:
    - 10.30.152.161										#local static ip
    - 10.30.152.162										
    - 10.30.152.163										#instance 개수와 ip 개수 일치
  properties: {}
  resource_pool: metrics-collector						#resource name
  templates:
  - name: metrics_collector
    release: paasta-metrics-collector
  update:
    max_in_flight: 1
    serial: true

name: paasta-metrics-collector							#deployment name

networks:

- name: paasta-metrics-collector-net					#network name
  type: manual
  subnets:
  - cloud_properties:
      name: Internal
    dns:
    - 10.30.20.27										#dns
    - 8.8.8.8
    gateway: 10.30.20.23								#gateway
    range: 10.30.0.0/16									#static ip range
    reserved:
    - 10.30.0.1 - 10.30.152.159						    #reserved ip range
    static:
    - 10.30.152.160 - 10.30.152.165				        #available ip range
  type: manual

properties:
  syslog_daemon_config:
    address: null
    port: null
  metrics_collector:
    uaaUrl: https://uaa.115.68.46.186.xip.io			#uaa target url
    clientId: cf										#uaa client id
    clientPass: null									#uaa client password
    influx:
      url: 10.30.152.51:8089							#target influxdb url
      database: cf_metric_db							#target influxdb database
      cf_measurement: cf_metrics						#target influxdb measurement
      cf_process_measurement : cf_process_metrics		#target influxdb process measurement
    dopplerUrl:
    - wss://doppler.xxx.xxx.xxx.xxx.xip.io:443			#doppler websocket url
    log_level: info

releases:
- name: paasta-metrics-collector						#release name
  version: latest   										    #release version				

resource_pools:
- cloud_properties:
    cpu: 1
    disk: 4096
    ram: 1024
  env:
    bosh:
      password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0
  name: metrics-collector								#resource name
  network: paasta-metrics-collector-network 			#network name
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent		#stemcell name
    version: latest										#stemcell version

update:
  canaries: 1
  canary_watch_time: 30000-100000
  max_errors: 1
  max_in_flight: 1
  update_watch_time: 30000-100000

```

2. manifest 파일 지정

$ bosh deployment metrics-collector-release.yml

<kbd>![2-2-1]</kbd>

<div id='9'></div>

### 2.3.  배포

$ bosh -n deploy

<kbd>![2-3-1]</kbd>
<kbd>![2-3-2]</kbd>

<div id='10'></div>

### 2.4.  확인

$ bosh deployments

<kbd>![2-4-1]</kbd>


[2-1-1]:images/metrics-collector/2-1-1.png
[2-2-1]:images/metrics-collector/2-2-1.png
[2-3-1]:images/metrics-collector/2-3-1.png
[2-3-2]:images/metrics-collector/2-3-2.png
[2-4-1]:images/metrics-collector/2-4-1.png
