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

본 문서는 IaaS(Infrastructure as a Service) 중 하나인 Vsphere 환경에서 Bosh VM 환경의 시스템 Metrics(CPU, Memory, Disk, Process, Network) 정보를 수집하기 위한 agent를  설치하는 방법을 제공하는데 그 목적이 있다.

<div id='3'></div>

### 1.2. 범위

본 문서는 VSphere 기반에 설치하기 위한 내용으로 한정되어 있다.

<div id='4'></div>

### 1.3. 참고  
      
> <a style="text-decoration:underline" href="https://github.com/PaaS-TA/Guide-3.0-Penne-/blob/master/Install-Guide/monitoring/PaaS-TA%20%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81%20%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EC%B9%98%EA%B0%80%EC%9D%B4%EB%93%9C.md">모니터링 시스템 Architecutre</a>

<div id='5'></div>

# 2.  Bosh 배포

## Prerequisites
Bosh를 설치 하기 위해서는 bosh-init이 PaaS 서비스 설치환경에 설치 되어 있어야 한다.

### 참고  
 PaaS 서비스 설치환경 : PaaS-TA를 설치하기위한 환경으로 Ubuntu OS를 기본으로 한다.

> <a style="text-decoration:underline" href="https://github.com/PaaS-TA/Guide-1.0-Spaghetti-/blob/master/Install-Guide/BOSH/OpenPaaS_PaaSTA_BOSH_Openstack_install_guide.md">Bosh 설치 환경 참조 </a>



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

$ vi bosh-init.yml

```
cloud_provider:
  mbus: https://mbus:mbus-password@10.30.40.105:6868
  properties:
    agent:
      mbus: https://mbus:mbus-password@0.0.0.0:6868
    blobstore:
      path: /var/vcap/micro_bosh/data/cache
      provider: local
    ntp:
    - 0.pool.ntp.org
    - 1.pool.ntp.org
    vcenter:
      address: xxx.xxx.xxx.xxx
      datacenters:
      - clusters:
        - pcf:
            resource_pool: CF_Dev_Pool
        datastore_pattern: Storage
        disk_path: MicroBOSH_Disks
        name: pcf
        persistent_datastore_pattern: Storage
        template_folder: MicroBOSH_Dev_Templates
        vm_folder: MicroBOSH_Dev_VMs
      password: vmware1!
      user: administrator
  template:
    name: vsphere_cpi
    release: bosh-vsphere-cpi
disk_pools:
- disk_size: 100000
  name: disks
jobs:
- instances: 1
  name: bosh
  networks:
  - name: private
    static_ips:
    - 10.30.40.105
  - default:
    - dns
    - gateway
    name: public
    static_ips:
    - xxx.xxx.xxx.xxx
  persistent_disk_pool: disks
  properties:
    logstash:
      ingestor_bosh:
        host: 10.30.152.121                #logsearch-ingestor vm's IP - pre-defined.
        port: 3001
      input:
        file_path: /var/vcap/store/director/tasks/**/debug
    metrics_agent:
      influxdb:
        url: 10.30.200.11:8059            # IP :Port - influx ip : bosh port - pre-defined.
        database: bosh_metric_db
        measurement: bosh_metrics
        processMeasurement: bosh_process_metrics
      origin: bosh
    agent:
      mbus: nats://nats:nats-password@10.30.40.105:4222
    blobstore:
      address: 10.30.40.105
      agent:
        password: agent-password
        user: agent
      director:
        password: director-password
        user: director
      port: 25250
      provider: dav
    director:
      address: 127.0.0.1
      cpi_job: vsphere_cpi
      db:
        adapter: postgres
        database: bosh
        host: 127.0.0.1
        listen_address: 0.0.0.0
        password: postgres-password
        user: postgres
      enable_snapshots: false
      name: bosh
      snapshot_schedule: null
      user_management:
        local:
          users:
          - name: admin
            password: admin
          - name: hm
            password: hm-password
        provider: local
    hm:
      director_account:
        password: hm-password
        user: hm
      resurrector_enabled: true
    nats:
      address: 127.0.0.1
      password: nats-password
      user: nats
    ntp:
    - 0.pool.ntp.org
    - 1.pool.ntp.org
    postgres:
      adapter: postgres
      database: bosh
      host: 127.0.0.1
      listen_address: 0.0.0.0
      password: postgres-password
      user: postgres
    vcenter:
      address: 115.68.46.22
      datacenters:
      - clusters:
        - pcf:
            resource_pool: CF_Dev_Pool
        datastore_pattern: Storage
        disk_path: MicroBOSH_Disks
        name: pcf
        persistent_datastore_pattern: Storage
        template_folder: MicroBOSH_Dev_Templates
        vm_folder: MicroBOSH_Dev_VMs
      password: vmware1!
      user: administrator
  resource_pool: vms
  templates:
  - name: nats
    release: bosh
  - name: postgres
    release: bosh
  - name: blobstore
    release: bosh
  - name: director
    release: bosh
  - name: health_monitor
    release: bosh
  - name: vsphere_cpi
    release: bosh-vsphere-cpi
  - name: metrics_agent
    release: bosh-monitoring-agent
  - name: logstash
    release: bosh-monitoring-agent
name: bootstrap-vsphere
networks:
- name: private
  subnets:
  - cloud_properties:
      name: Internal
    dns:
    - 10.30.20.24
    - 8.8.8.8
    gateway: 10.30.20.23
    range: 10.30.0.0/16
  type: manual
- name: public
  subnets:
  - cloud_properties:
      name: External
    dns:
    - 8.8.8.8
    gateway: xxx.xxx.xxx.xxx
    range: xxx.xxx.xxx.xxx/28
  type: manual
releases:
- name: bosh
  url: file:////home/inception/.bosh_plugin/release/bosh-260.1.tgz
- name: bosh-vsphere-cpi
  url: file:////home/inception/.bosh_plugin/release/bosh-vsphere-cpi-release-35.tgz
- name: bosh-monitoring-agent
  url: file:////home/inception/bosh-space1/paasta3.0/release/bosh-monitoring-agent-1.2.tgz
resource_pools:
- cloud_properties:
    cpu: 2
    disk: 40000
    ram: 4096
  env:
    bosh:
      password: $6$IijxvHjWgc$vPZZgG7KMlgsYU.AcMch/6Jxbw6IrcnSA/x/zWYCxIDlEm5yvjkQFfsjuAUx3cTq/eASOa3KMdHzvV4H0A1Pn0
  name: vms
  network: private
  stemcell:
    url: file:////home/inception/.bosh_plugin/stemcell/bosh-stemcell-3363.10-vsphere-esxi-ubuntu-trusty-go_agent.tgz
```

<div id='8'></div>

### 3.4.  Bosh 배포

bosh-init 명령을 사용하여 bosh deploy한다.

```
$ bosh-init deploy bosh-init.yml
```
