## Table of Contents

1. [문서 개요](#문서-개요)
     * [1.1. 목적](#목적)
     * [1.2. 범위](#범위)
     * [1.3. 시스템 구성도](#시스템-구성도)
     * [1.4. 참고자료](#참고자료)

2. [Pinpoint 서비스팩 설치](#pinpoint-서비스팩-설치)
     * [2.1. 설치전 준비사항](#설치전-준비사항)
     * [2.2. Pinpoint 서비스 릴리즈 업로드](#pinpoint-서비스-릴리즈-업로드)
     * [2.3. Pinpoint 서비스 Deployment 파일 수정 및 배포](#pinpoint-서비스-deployment-파일-수정-및-배포)
     * [2.4. HBase 기본 데이터 실행](#hbase-기본-데이터-실행)
     * [2.4. Pinpoint 서비스 브로커 등록](#pinpoint-서비스-브로커-등록)

3. [Sample Web App 연동 Pinpoint 연동](#sample-web-app-연동-pinpoint-연동)
     * [3.1. Sample App 구조](#sample-web-app-구조)
     * [3.2. PaaS-TA에서 서비스 신청](#PaaS-TA에서-서비스-신청)
     * [3.3. Sample App에 서비스 바인드 신청 및 App 확인](#sample-web-app에-서비스-바인드-신청-및-app-확인)

<br>
<div id = 문서-개요></div>
#1.  문서 개요

<br>
<div id='목적'></div>
### 1.1. 목적

본 문서(Pinpoint 서비스팩 설치 가이드)는 전자정부표준프레임워크 기반의 PaaS-TA에서 제공되는 서비스팩인 Pinpoint 서비스팩을 Bosh를 이용하여 설치하는 방법과 PaaS-TA의 SaaS 형태로 제공하는 Application 에서 Pinpoint 서비스를 사용하는 방법을 기술하였다.

<br>
<div id='범위'></div>
### 1.2. 범위 

설치 범위는 Pinpoint 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다.

<br>
<div id='시스템-구성도'></div>
### 1.3. 시스템 구성도

본 문서의 설치된 시스템 구성도입니다. Pinpoint Server, HBase의 HBase Master2, HBase Slave2, Collector 2, Pinpoint 서비스 브로커, WebUI3로 최소사항을 구성하였다. 

![시스템구성도][pinpoint_image_01]

<table>
  <tr>
    <th>구분</th>
    <th>Resource Pool</th>
    <th>스펙</th>
  </tr>
  <tr>
  <td>Collector/0      </td><td>pinpoint\_medium</td><td>2vCPU / 2GB RAM / 8GB Disk</td>
  </tr>
  <tr>
  <td>Collector/1      </td><td>pinpoint\_medium</td><td>2vCPU / 2GB RAM / 8GB Disk</td>
  </tr>
  <tr>
  <td>h\_master/0      </td><td>pinpoint\_medium</td><td>2vCPU / 2GB RAM / 8GB Disk</td>
  </tr>
  <tr>
  <td>h\_secondary     </td><td>pinpoint\_ small</td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
  <tr>
  <td>h\_slave/0       </td><td>services-small </td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
  <tr>
  <td>h\_slave/1       </td><td>services-small </td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
  <tr>
  <td>haproxy\_webui/0 </td><td>services-small </td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
  <tr>
  <td>pinpoint\_broker/0</<td>services-small </td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
  <tr>
  <td>webui/0          </0><td>services-small </td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
  <tr>
  <td>webui/1          </td><td>services-small </td><td>1vCPU / 1GB RAM / 4GB Disk</td>
  </tr>
</table>

<br>
<div id='참고자료'></div>
### 1.4. 참고자료

[**http://bosh.io/docs**](http://bosh.io/docs)

[**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/)

<br>
<div id='#pinpoint-서비스팩-설치'></div>
#   2.Pinpoint 서비스팩 설치

<br>
<div id='설치전-준비사항'></div>
### 2.1.  설치전-준비사항

본 서비스팩 설치를 위해서는 먼저 BOSH CLI 가 설치 되어 있어야 하고 BOSH에 로그인 및 target 설정이 되어 있어야 한다.

BOSH CLI 가 설치 되어 있지 않을 경우 먼저 BOSH 설치 가이드 문서를 참고하여 BOSH CLI를 설치 해야 한다.

PaaS-TA에서 제공하는 압축된 릴리즈 파일들을 다운받는다. (PaaSTA-Deployment.zip, PaaSTA-Sample-Apps.zip, PaaSTA-Services.zip)

<br>
- 다운로드 위치

>PaaSTA-Services : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Services.zip>**  
>PaaSTA-Deployment : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Deployment.zip>**  
>PaaSTA-Sample-Apps : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Sample-Apps.zip>**  

<br>
<div id='pinpoint-서비스-릴리즈-업로드'></div>
### 2.2. Pinpoint 서비스 릴리즈 업로드

<br>
-  Github에서 다운로드 받은 paasta-pinpoint-release 폴더로 이동하여 Pinpoint 릴리즈 파일을 확인한다.

```
$ls paasta-pinpoint-2.0.tgz

```

<br>
-  릴리즈를 업로드한다.
```
$ bosh upload release paasta-pinpoint-2.0.tgz
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on 'bosh'
Verifying manifest...
Extract manifest OK
Manifest exists OK
Release name/version OK

File exists and readable OK
Read package 'webui' (1 of 10) OK
Package 'webui' checksum                                     OK
Read package 'ssh-keys' (2 of 10)                            OK
Package 'ssh-keys' checksum                                  OK
Read package 'bosh-helpers' (3 of 10)                        OK
Package 'bosh-helpers' checksum                              OK
Read package 'hadoop' (4 of 10)                              OK
Package 'hadoop' checksum                                    OK
Read package 'broker' (5 of 10)                              OK
Package 'broker' checksum                                    OK
Read package 'haproxy' (6 of 10)                             OK
Package 'haproxy' checksum                                   OK
Read package 'hbase' (7 of 10)                               OK
Package 'hbase' checksum                                     OK
Read package 'tomcat' (8 of 10)                              OK
Package 'tomcat' checksum                                    OK
Read package 'java' (9 of 10)                                OK
Package 'java' checksum                                      OK
Read package 'collector' (10 of 10)                          OK
Package 'collector' checksum                                 OK
Package dependencies                                         OK
Checking jobs format                                         OK
Read job 'h_secondary' (1 of 8), version 5c00b321b374d2b166908d5bc12a0c6d16cbe49c OK
Job 'h_secondary' checksum                                   OK
Extract job 'h_secondary'                                    OK
Read job 'h_secondary' manifest                              OK
Check template 'bin/h_secondary_ctl.erb' for 'h_secondary'   OK
Check template 'config/hadoop/core-site.xml.erb' for 'h_secondary' OK
Check template 'config/hadoop/hadoop-env.sh' for 'h_secondary' OK
Check template 'config/hadoop/hdfs-site.xml.erb' for 'h_secondary' OK
Check template 'config/hadoop/mapred-site.xml.erb' for 'h_secondary' OK
Check template 'config/hadoop/slaves.erb' for 'h_secondary'  OK
Check template 'config/hadoop/yarn-env.sh' for 'h_secondary' OK
Check template 'config/hadoop/yarn-site.xml.erb' for 'h_secondary' OK
Check template 'data/properties.sh.erb' for 'h_secondary'    OK
Check template 'etc/hosts.erb' for 'h_secondary'             OK
Check template 'config/hadoop/capacity-scheduler.xml' for 'h_secondary' OK
Check template 'config/hadoop/configuration.xsl' for 'h_secondary' OK
Check template 'config/hadoop/container-executor.cfg' for 'h_secondary' OK
Check template 'config/hadoop/hadoop-env.cmd' for 'h_secondary' OK
Check template 'config/hadoop/hadoop-metrics.properties' for 'h_secondary' OK
Check template 'config/hadoop/hadoop-metrics2.properties' for 'h_secondary' OK
Check template 'config/hadoop/hadoop-policy.xml' for 'h_secondary' OK
Check template 'config/hadoop/httpfs-env.sh' for 'h_secondary' OK
Check template 'config/hadoop/httpfs-log4j.properties' for 'h_secondary' OK
Check template 'config/hadoop/httpfs-signature.secret' for 'h_secondary' OK
Check template 'config/hadoop/httpfs-site.xml' for 'h_secondary' OK
Check template 'config/hadoop/log4j.properties' for 'h_secondary' OK
Check template 'config/hadoop/mapred-env.cmd' for 'h_secondary' OK
Check template 'config/hadoop/mapred-env.sh' for 'h_secondary' OK
Check template 'config/hadoop/mapred-queues.xml.template' for 'h_secondary' OK
Check template 'config/hadoop/mapred-site.xml.template' for 'h_secondary' OK
Check template 'config/hadoop/ssl-client.xml.example' for 'h_secondary' OK
Check template 'config/hadoop/ssl-server.xml.example' for 'h_secondary' OK
Check template 'config/hadoop/yarn-env.cmd' for 'h_secondary' OK
Job 'h_secondary' needs 'bosh-helpers' package               OK
Job 'h_secondary' needs 'ssh-keys' package                   OK
Job 'h_secondary' needs 'java' package                       OK
Job 'h_secondary' needs 'hadoop' package                     OK
Monit file for 'h_secondary'                                 OK
Read job 'h_master' (2 of 8), version 4b1213e73d0c35d133c7582d68d1fd1bf4de6969 OK
Job 'h_master' checksum                                      OK
Extract job 'h_master'                                       OK
Read job 'h_master' manifest                                 OK
Check template 'bin/h_master_ctl.erb' for 'h_master'         OK
Check template 'config/hadoop/core-site.xml.erb' for 'h_master' OK
Check template 'config/hadoop/hadoop-env.sh' for 'h_master'  OK
Check template 'config/hadoop/hdfs-site.xml.erb' for 'h_master' OK
Check template 'config/hadoop/mapred-site.xml.erb' for 'h_master' OK
Check template 'config/hadoop/slaves.erb' for 'h_master'     OK
Check template 'config/hadoop/yarn-env.sh' for 'h_master'    OK
Check template 'config/hadoop/yarn-site.xml.erb' for 'h_master' OK
Check template 'config/hbase/hbase-env.sh' for 'h_master'    OK
Check template 'config/hbase/hbase-site.xml.erb' for 'h_master' OK
Check template 'config/hbase/regionservers.erb' for 'h_master' OK
Check template 'data/properties.sh.erb' for 'h_master'       OK
Check template 'etc/hosts.erb' for 'h_master'                OK
Check template 'config/hadoop/capacity-scheduler.xml' for 'h_master' OK
Check template 'config/hadoop/configuration.xsl' for 'h_master' OK
Check template 'config/hadoop/container-executor.cfg' for 'h_master' OK
Check template 'config/hadoop/hadoop-env.cmd' for 'h_master' OK
Check template 'config/hadoop/hadoop-metrics.properties' for 'h_master' OK
Check template 'config/hadoop/hadoop-metrics2.properties' for 'h_master' OK
Check template 'config/hadoop/hadoop-policy.xml' for 'h_master' OK
Check template 'config/hadoop/httpfs-env.sh' for 'h_master'  OK
Check template 'config/hadoop/httpfs-log4j.properties' for 'h_master' OK
Check template 'config/hadoop/httpfs-signature.secret' for 'h_master' OK
Check template 'config/hadoop/httpfs-site.xml' for 'h_master' OK
Check template 'config/hadoop/log4j.properties' for 'h_master' OK
Check template 'config/hadoop/mapred-env.cmd' for 'h_master' OK
Check template 'config/hadoop/mapred-env.sh' for 'h_master'  OK
Check template 'config/hadoop/mapred-queues.xml.template' for 'h_master' OK
Check template 'config/hadoop/mapred-site.xml.template' for 'h_master' OK
Check template 'config/hadoop/ssl-client.xml.example' for 'h_master' OK
Check template 'config/hadoop/ssl-server.xml.example' for 'h_master' OK
Check template 'config/hadoop/yarn-env.cmd' for 'h_master'   OK
Check template 'config/hbase/hadoop-metrics2-hbase.properties' for 'h_master' OK
Check template 'config/hbase/hbase-env.cmd' for 'h_master'   OK
Check template 'config/hbase/hbase-policy.xml' for 'h_master' OK
Check template 'config/hbase/log4j.properties' for 'h_master' OK
Job 'h_master' needs 'bosh-helpers' package                  OK
Job 'h_master' needs 'ssh-keys' package                      OK
Job 'h_master' needs 'java' package                          OK
Job 'h_master' needs 'hadoop' package                        OK
Job 'h_master' needs 'hbase' package                         OK
Monit file for 'h_master'                                    OK
Read job 'webui' (3 of 8), version 9cec7497c1b103f22f3ce39bee82e92e9a50e934 OK
Job 'webui' checksum                                         OK
Extract job 'webui'                                          OK
Read job 'webui' manifest                                    OK
Check template 'bin/webui_ctl.erb' for 'webui'               OK
Check template 'common/ctl_common.sh' for 'webui'            OK
Check template 'data/properties.sh.erb' for 'webui'          OK
Check template 'etc/hosts.erb' for 'webui'                   OK
Job 'webui' needs 'bosh-helpers' package                     OK
Job 'webui' needs 'java' package                             OK
Job 'webui' needs 'tomcat' package                           OK
Job 'webui' needs 'webui' package                            OK
Monit file for 'webui'                                       OK
Read job 'haproxy_webui' (4 of 8), version 481dac37a2d65b6ce4edfa5d91a4cddd5cce9516 OK
Job 'haproxy_webui' checksum                                 OK
Extract job 'haproxy_webui'                                  OK
Read job 'haproxy_webui' manifest                            OK
Check template 'bin/haproxy_webui_ctl.erb' for 'haproxy_webui' OK
Check template 'config/haproxy/haproxy.cfg.erb' for 'haproxy_webui' OK
Check template 'data/properties.sh.erb' for 'haproxy_webui'  OK
Check template 'etc/hosts.erb' for 'haproxy_webui'           OK
Job 'haproxy_webui' needs 'bosh-helpers' package             OK
Job 'haproxy_webui' needs 'java' package                     OK
Job 'haproxy_webui' needs 'haproxy' package                  OK
Monit file for 'haproxy_webui'                               OK
Read job 'broker' (5 of 8), version f17d59dd28cc876ea3de8959b0bceed8df8c6eea OK
Job 'broker' checksum                                        OK
Extract job 'broker'                                         OK
Read job 'broker' manifest                                   OK
Check template 'bin/broker_ctl' for 'broker'                 OK
Check template 'bin/monit_debugger' for 'broker'             OK
Check template 'data/properties.sh.erb' for 'broker'         OK
Check template 'helpers/ctl_setup.sh' for 'broker'           OK
Check template 'helpers/ctl_utils.sh' for 'broker'           OK
Check template 'config/broker.yml.erb' for 'broker'          OK
Check template 'config/application-mvc.properties.erb' for 'broker' OK
Check template 'config/collector.json.erb' for 'broker'      OK
Check template 'config/logback.xml.erb' for 'broker'         OK
Job 'broker' needs 'broker' package                          OK
Job 'broker' needs 'java' package                            OK
Monit file for 'broker'                                      OK
Read job 'h_master_register' (6 of 8), version 40d0e9f56b63ac5025f523597754a2ee6a799107 OK
Job 'h_master_register' checksum                             OK
Extract job 'h_master_register'                              OK
Read job 'h_master_register' manifest                        OK
Check template 'errand.sh.erb' for 'h_master_register'       OK
Job 'h_master_register' needs 'ssh-keys' package             OK
Monit file for 'h_master_register'                           OK
Read job 'h_slave' (7 of 8), version ca15d7c5e57fbd7ecfb7a4614dd8bee8d43efe84 OK
Job 'h_slave' checksum                                       OK
Extract job 'h_slave'                                        OK
Read job 'h_slave' manifest                                  OK
Check template 'bin/h_slave_ctl.erb' for 'h_slave'           OK
Check template 'config/hadoop/core-site.xml.erb' for 'h_slave' OK
Check template 'config/hadoop/hadoop-env.sh' for 'h_slave'   OK
Check template 'config/hadoop/hdfs-site.xml.erb' for 'h_slave' OK
Check template 'config/hadoop/mapred-site.xml.erb' for 'h_slave' OK
Check template 'config/hadoop/slaves.erb' for 'h_slave'      OK
Check template 'config/hadoop/yarn-env.sh' for 'h_slave'     OK
Check template 'config/hadoop/yarn-site.xml.erb' for 'h_slave' OK
Check template 'config/hbase/hbase-env.sh' for 'h_slave'     OK
Check template 'config/hbase/hbase-site.xml.erb' for 'h_slave' OK
Check template 'config/hbase/regionservers.erb' for 'h_slave' OK
Check template 'data/properties.sh.erb' for 'h_slave'        OK
Check template 'etc/hosts.erb' for 'h_slave'                 OK
Check template 'config/hadoop/capacity-scheduler.xml' for 'h_slave' OK
Check template 'config/hadoop/configuration.xsl' for 'h_slave' OK
Check template 'config/hadoop/container-executor.cfg' for 'h_slave' OK
Check template 'config/hadoop/hadoop-env.cmd' for 'h_slave'  OK
Check template 'config/hadoop/hadoop-metrics.properties' for 'h_slave' OK
Check template 'config/hadoop/hadoop-metrics2.properties' for 'h_slave' OK
Check template 'config/hadoop/hadoop-policy.xml' for 'h_slave' OK
Check template 'config/hadoop/httpfs-env.sh' for 'h_slave'   OK
Check template 'config/hadoop/httpfs-log4j.properties' for 'h_slave' OK
Check template 'config/hadoop/httpfs-signature.secret' for 'h_slave' OK
Check template 'config/hadoop/httpfs-site.xml' for 'h_slave' OK
Check template 'config/hadoop/log4j.properties' for 'h_slave' OK
Check template 'config/hadoop/mapred-env.cmd' for 'h_slave'  OK
Check template 'config/hadoop/mapred-env.sh' for 'h_slave'   OK
Check template 'config/hadoop/mapred-queues.xml.template' for 'h_slave' OK
Check template 'config/hadoop/mapred-site.xml.template' for 'h_slave' OK
Check template 'config/hadoop/ssl-client.xml.example' for 'h_slave' OK
Check template 'config/hadoop/ssl-server.xml.example' for 'h_slave' OK
Check template 'config/hadoop/yarn-env.cmd' for 'h_slave'    OK
Check template 'config/hbase/hadoop-metrics2-hbase.properties' for 'h_slave' OK
Check template 'config/hbase/hbase-env.cmd' for 'h_slave'    OK
Check template 'config/hbase/hbase-policy.xml' for 'h_slave' OK
Check template 'config/hbase/log4j.properties' for 'h_slave' OK
Job 'h_slave' needs 'bosh-helpers' package                   OK
Job 'h_slave' needs 'ssh-keys' package                       OK
Job 'h_slave' needs 'java' package                           OK
Job 'h_slave' needs 'hadoop' package                         OK
Job 'h_slave' needs 'hbase' package                          OK
Monit file for 'h_slave'                                     OK
Read job 'collector' (8 of 8), version 062974e137db2e4262036a38198c2c167fbc6f0c OK
Job 'collector' checksum                                     OK
Extract job 'collector'                                      OK
Read job 'collector' manifest                                OK
Check template 'bin/collector_ctl.erb' for 'collector'       OK
Check template 'common/ctl_common.sh' for 'collector'        OK
Check template 'data/properties.sh.erb' for 'collector'      OK
Check template 'etc/hosts.erb' for 'collector'               OK
Job 'collector' needs 'bosh-helpers' package                 OK
Job 'collector' needs 'java' package                         OK
Job 'collector' needs 'tomcat' package                       OK
Job 'collector' needs 'collector' package                    OK
Monit file for 'collector'                                   OK

Release info
------------
Name:    paasta-pinpoint-service
Version: 2.0

Packages
- webui (655536239b03532d40b99159865520d8ee9e870d)
- ssh-keys (a92619994e7c69a7c52d30f4bc87c949ca2e5b77)
- bosh-helpers (10d1d5cf2e9dfd6c8279aa99386c560ed32eb6fe)
- hadoop (1d3ea90f6e3173e51ce42eacccd380c8c821f3a4)
- broker (9c8209bff0c69156bcb15e1ed7f1c7f3b73df2cd)
- haproxy (88e2cea38299d775fda4bac58eba167eefae6dc5)
- hbase (5d5427c7c567de557a7f899f2c83f347723007c3)
- tomcat (462a5a0b802cb559e23fe8e59b804c594c641428)
- java (788dd8b3824f7ed89a51f0d870d8a5b6376820a5)
- collector (e0b59e5385885af84c3cb4363821d5d6d1be0eae)

Jobs
- h_secondary (5c00b321b374d2b166908d5bc12a0c6d16cbe49c)
- h_master (4b1213e73d0c35d133c7582d68d1fd1bf4de6969)
- webui (9cec7497c1b103f22f3ce39bee82e92e9a50e934)
- haproxy_webui (481dac37a2d65b6ce4edfa5d91a4cddd5cce9516)
- broker (f17d59dd28cc876ea3de8959b0bceed8df8c6eea)
- h_master_register (40d0e9f56b63ac5025f523597754a2ee6a799107)
- h_slave (ca15d7c5e57fbd7ecfb7a4614dd8bee8d43efe84)
- collector (062974e137db2e4262036a38198c2c167fbc6f0c)

License
- none

Checking if can repack release for faster upload...
webui (655536239b03532d40b99159865520d8ee9e870d) UPLOAD
ssh-keys (a92619994e7c69a7c52d30f4bc87c949ca2e5b77) UPLOAD
bosh-helpers (10d1d5cf2e9dfd6c8279aa99386c560ed32eb6fe) UPLOAD
hadoop (1d3ea90f6e3173e51ce42eacccd380c8c821f3a4) UPLOAD
broker (9c8209bff0c69156bcb15e1ed7f1c7f3b73df2cd) UPLOAD
haproxy (88e2cea38299d775fda4bac58eba167eefae6dc5) UPLOAD
hbase (5d5427c7c567de557a7f899f2c83f347723007c3) UPLOAD
tomcat (462a5a0b802cb559e23fe8e59b804c594c641428) UPLOAD
java (788dd8b3824f7ed89a51f0d870d8a5b6376820a5) UPLOAD
collector (e0b59e5385885af84c3cb4363821d5d6d1be0eae) UPLOAD
h_secondary (5c00b321b374d2b166908d5bc12a0c6d16cbe49c) UPLOAD
h_master (4b1213e73d0c35d133c7582d68d1fd1bf4de6969) UPLOAD
webui (9cec7497c1b103f22f3ce39bee82e92e9a50e934) UPLOAD
haproxy_webui (481dac37a2d65b6ce4edfa5d91a4cddd5cce9516) UPLOAD
broker (f17d59dd28cc876ea3de8959b0bceed8df8c6eea) UPLOAD
h_master_register (40d0e9f56b63ac5025f523597754a2ee6a799107) UPLOAD
h_slave (ca15d7c5e57fbd7ecfb7a4614dd8bee8d43efe84) UPLOAD
collector (062974e137db2e4262036a38198c2c167fbc6f0c) UPLOAD
Uploading the whole release

Uploading release
paasta-pinpoi:  96% |ooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo     | 512.4MB  26.5MB/s ETA:  00:00:00
Director task 441
Started extracting release > Extracting release. Done (00:00:06)

Started verifying manifest > Verifying manifest. Done (00:00:00)

Started resolving package dependencies > Resolving package dependencies. Done (00:00:00)

Started creating new packages
Started creating new packages > webui/655536239b03532d40b99159865520d8ee9e870d. Done (00:00:01)
Started creating new packages > ssh-keys/a92619994e7c69a7c52d30f4bc87c949ca2e5b77. Done (00:00:00)
Started creating new packages > bosh-helpers/10d1d5cf2e9dfd6c8279aa99386c560ed32eb6fe. Done (00:00:00)
Started creating new packages > hadoop/1d3ea90f6e3173e51ce42eacccd380c8c821f3a4. Done (00:00:02)
Started creating new packages > broker/9c8209bff0c69156bcb15e1ed7f1c7f3b73df2cd. Done (00:00:00)
Started creating new packages > haproxy/88e2cea38299d775fda4bac58eba167eefae6dc5. Done (00:00:00)
Started creating new packages > hbase/5d5427c7c567de557a7f899f2c83f347723007c3. Done (00:00:02)
Started creating new packages > tomcat/462a5a0b802cb559e23fe8e59b804c594c641428. Done (00:00:00)
Started creating new packages > java/788dd8b3824f7ed89a51f0d870d8a5b6376820a5. Done (00:00:04)
Started creating new packages > collector/e0b59e5385885af84c3cb4363821d5d6d1be0eae. Done (00:00:00)
   Done creating new packages (00:00:09)

Started creating new jobs
Started creating new jobs > h_secondary/5c00b321b374d2b166908d5bc12a0c6d16cbe49c. Done (00:00:00)
Started creating new jobs > h_master/4b1213e73d0c35d133c7582d68d1fd1bf4de6969. Done (00:00:00)
Started creating new jobs > webui/9cec7497c1b103f22f3ce39bee82e92e9a50e934. Done (00:00:00)
Started creating new jobs > haproxy_webui/481dac37a2d65b6ce4edfa5d91a4cddd5cce9516. Done (00:00:00)
Started creating new jobs > broker/f17d59dd28cc876ea3de8959b0bceed8df8c6eea. Done (00:00:01)
Started creating new jobs > h_master_register/40d0e9f56b63ac5025f523597754a2ee6a799107. Done (00:00:00)
Started creating new jobs > h_slave/ca15d7c5e57fbd7ecfb7a4614dd8bee8d43efe84. Done (00:00:00)
Started creating new jobs > collector/062974e137db2e4262036a38198c2c167fbc6f0c. Done (00:00:00)
   Done creating new jobs (00:00:01)

Started release has been created > paasta-pinpoint-service/2.0. Done (00:00:00)

Task 441 done

Started   2016-12-29 07:29:24 UTC
Finished  2016-12-29 07:29:40 UTC
Duration  00:00:16
paasta-pinpoi:  96% |ooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo     | 513.7MB  14.3MB/s Time: 00:00:35

Release uploaded

```

<br>
-   업로드 되어 있는 릴리즈 목록을 확인한다.

```
$ bosh releases
```
```
RSA 1024 bit CA certificates are loaded ue to old openssl compatibility
Acting as user 'admin' on 'bosh'

+-------------------------+----------+-------------+
| Name                    | Versions | Commit Hash |
+-------------------------+----------+-------------+
| cf                      | 247*     | af4efe9f+   |
| cflinuxfs2-rootfs       | 1.40.0*  | 19fe09f4+   |
| diego                   | 1.1.0*   | 2298c8d4    |
| empty-release           | 1+dev.1* | 00000000    |
| etcd                    | 86*      | 2dfbef00+   |
| garden-runc             | 1.0.3*   | c6c4c73c    |
| monitoring-api-server   | 0+dev.3* | 00000000    |
|                         | 0+dev.4* | 00000000    |
|                         | 0+dev.5  | 00000000    |
|                         | 0+dev.6  | 00000000    |
|                         | 0+dev.7  | 00000000    |
| paasta-glusterfs        | 2.0*     | 85e3f01e+   |
| paasta-pinpoint-service | 2.0      | 85e3f01e+   |
| swift-keystone-release  | 0+dev.1* | 00000000    |
+-------------------------+----------+-------------+
(*) Currently deployed
(+) Uncommitted changes
Releases total: 11
```
<br>
<div id="Pinpoint-서비스-Deployment-파일-수정-및-배포"></div>
###2.3. Pinpoint 서비스 Deployment 파일 수정 및 배포

BOSH Deployment manifest 는 components 요소 및 배포의 속성을 정의한 YAML  파일이다.
Deployment manifest 에는 sotfware를 설치 하기 위해서 어떤 Stemcell (OS, BOSH agent) 을 사용할 것인지와 Release (Software packages, Config templates, Scripts)의 이름과 버전, VMs 용량, Jobs params 등이 정의 되어 있다.
등을 정의가 되어 있다.

- PaaSTA-Deployment.zip 파일 압축을 풀고 폴더안에 있는 IaaS별 Pinpoint 서비스팩 Deployment 파일을 복사한다. 

- Director UUID를 확인한다.
BOSH CLI가 배포에 대한 모든 작업을 허용하기 위한 현재 대상 BOSH Director의 UUID와 일치해야 한다. ‘bosh status’ CLI 을 통해서 현재 BOSH Director 에 target 되어 있는 UUID를 확인할 수 있다.
```
$ bosh status
```
```
Config
         /home/inception/.bosh_config

Director
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Name       bosh
URL        https://10.30.40.105:25555
Version    260.1.0 (00000000)
User       admin
UUID       d363905f-eaa0-4539-a461-8c1318498a32
CPI        vsphere_cpi
dns        disabled
compiled_package_cache disabled
snapshots  disabled

Deployment
Manifest   /home/inception/bosh-space/kimdojun/swift-keystone-release/swift-keystone.yml
```

<br>
- Deploy시 사용할 Stemcell을 확인한다.
```
$ bosh stemcells
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on 'bosh'

+------------------------------------------+---------------+---------+-----------------------------------------+
| Name                                     | OS            | Version | CID                                     |
+------------------------------------------+---------------+---------+-----------------------------------------+
| bosh-vsphere-esxi-ubuntu-trusty-go_agent | ubuntu-trusty | 3147    | sc-ac0d095d-89c9-46f9-b337-7eecf0f2edf0 |
| bosh-vsphere-esxi-ubuntu-trusty-go_agent | ubuntu-trusty | 3263.8* | sc-af443b65-9335-43b1-9b64-6d1791a10428 |
+------------------------------------------+---------------+---------+-----------------------------------------+

(*) Currently in-use

Stemcells total: 2 
```
Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드 해야 한다.

<br>
- Deployment 파일을 서버 환경에 맞게 수정한다. (vSphere 용으로 설명, 다른 IaaS는 해당 Deployment 파일의 주석내용을 참고)

```
$ vi paasta-pinpoint-vsphere-2.0.yml
```
```yaml
# paasta-pinpoint-vsphere 설정 파일 내용
name: paasta-pinpoint-service                           # 서비스 배포이름(필수)
director_uuid: d363905f-eaa0-4539-a461-8c1318498a32     # bosh status 에서 확인한 Director UUID을 입력(필수)
compilation:                             # 컴파일시 필요한 가상머신의 속성(필수)
  workers: 6                             # 컴파일 하는 가상머신의 최대수(필수)
  network: default                       # Networks block에서 선언한 network 이름(필수)
  cloud_properties:                      # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성 (instance_type, availability_zone), 직접 cpu,disk,ram 사이즈를 넣어도 됨
    ram: 4096
    disk: 8096
    cpu: 4
update:
  canaries: 1                            # canary 인스턴스 수(필수)
  canary_watch_time: 120000              # canary 인스턴스가 수행하기 위한 대기 시간(필수)
  update_watch_time: 120000              # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)
  max_in_flight: 8                       # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)
networks:
- name: default                           # Networks block에서 선언한 network 이름(필수)
  subnets:
  - cloud_properties:
      name: Internal          # vsphere 에서 사용하는 network 이름(필수)
    dns:                      # DNS 정보
    - 10.30.20.24
    - 8.8.8.8
    gateway: 10.30.20.23
    name: default_unused
    range: 10.30.0.0/16
    reserved:                         # 설치시 제외할 IP 설정
    - 10.30.20.0 - 10.30.20.22
    - 10.30.20.24 - 10.30.20.255
    - 10.30.40.0 - 10.30.40.255
    - 10.30.60.0 - 10.30.60.112
    static:
    - 10.30.70.0 - 10.30.70.255       #사용 가능한 IP 설정
- name: public_network                #퍼블릭 네트워크
  type: manual
  subnets:
  - cloud_properties:
      name: External
    dns:
    - 8.8.8.8
    range: 115.68.46.176/28
    gateway: 115.68.46.177
    static:
    - 115.68.46.182

resource_pools:               # 배포시 사용하는 resource pools를 명시하며 여러 개의 resource pools 을 사용할 경우 name 은 unique 해야함(필수)
- name: pinpoint_smallc        # 고유한 resource pool 이름
  network: default
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent      # stemcell 이름(필수)
    version: '3263.8'                                   # stemcell 버전(필수)
  cloud_properties:         # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성을 설명 (instance_type, availability_zone), 직접 cpu, disk, 메모리 설정가능
    cpu: 1
    disk: 4096
    ram: 1024
- name: pinpoint_medium
  network: default
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent      # stemcell 이름(필수)
    version: '3263.8'                                   # stemcell 버전(필수)
  cloud_properties:         # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성을 설명 (instance_type, availability_zone), 직접 cpu, disk, 메모리 설정가능
    cpu: 2
    disk: 8192
    ram: 2048
jobs:
- name: h_slave                           #작업 이름(필수)
  template: h_slave                       # job template 이름(필수)
  instances: 2                            # job 인스턴스 수(필수)
  resource_pool: pinpoint_small           # resource_pools block에 정의한 resource pool 이름(필수)
  networks:                               # 네트워크 구성정보
  - name: default                         # Networks block에서 선언한 network 이름(필수)
    static_ips:                           # 사용할 IP addresses 정의(필수): Pinpoint 서버 IP
    - 10.30.70.90
    - 10.30.70.91
- name: h_secondary                       #작업 이름(필수)
  template: h_secondary                   # job template 이름(필수)
  instances: 1                            # job 인스턴스 수(필수)
  resource_pool: pinpoint_small           # resource_pools block에 정의한 resource pool 이름(필수)
  networks:                               # 네트워크 구성정보
  - name: default                         # Networks block에서 선언한 network 이름(필수)
    static_ips:                           # 사용할 IP addresses 정의(필수): Pinpoint 서버 IP
    - 10.30.70.76                       
- name: h_master                          #작업 이름(필수)
  template: h_master                      # job template 이름(필수)
  instances: 1                            # job 인스턴스 수(필수)
  resource_pool: pinpoint_medium          # resource_pools block에 정의한 resource pool 이름(필수)
  networks:                               # 네트워크 구성정보          
  - name: default                         # Networks block에서 선언한 network 이름(필수)
    static_ips:                           # 사용할 IP addresses 정의(필수): Pinpoint 서버 IP  
    - 10.30.70.75                         
- name: h_master_register                 #작업 이름(필수)
  template: h_master_register             # job template 이름(필수)
  instances: 1                            # job 인스턴스 수(필수)
  lifecycle: errand                       # resource_pools block에 정의한 resource pool 이름(필수)
  resource_pool: pinpoint_small           # 네트워크 구성정보
  networks:                               # Networks block에서 선언한 network 이름(필수)
  - name: default
- name: collector                         #작업 이름(필수)
  template: collector                     # job template 이름(필수)
  instances: 2                            # job 인스턴스 수(필수)
  resource_pool: pinpoint_medium          # resource_pools block에 정의한 resource pool 이름(필수)
  networks:                               # 네트워크 구성정보
  - name: default                         # Networks block에서 선언한 network 이름(필수)
    static_ips:                           # 사용할 IP addresses 정의(필수): Pinpoint 서버 IP                          
    - 10.30.70.45
    - 10.30.70.46
- name: haproxy_webui                     #작업 이름(필수)
  template: haproxy_webui                 # job template 이름(필수)
  instances: 1                            # job 인스턴스 수(필수)
  resource_pool: pinpoint_small           # resource_pools block에 정의한 resource pool 이름(필수)
  networks:                               # 네트워크 구성정보
  - name: default                         # Networks block에서 선언한 network 이름(필수)
    static_ips:                           # 사용할 IP addresses 정의(필수): Pinpoint 서버 IP                          
    - 10.30.70.78
  - name: public_network
    default:
    - dns
    - gateway
    static_ips: 115.68.46.182
- name: webui                              #작업 이름(필수)
  template: webui                          # job template 이름(필수)
  instances: 2                             # job 인스턴스 수(필수)
  resource_pool: pinpoint_small            # resource_pools block에 정의한 resource pool 이름(필수)
  networks:                                # 네트워크 구성정보
  - name: default                          # Networks block에서 선언한 network 이름(필수)
    static_ips:                            # 사용할 IP addresses 정의(필수): Pinpoint 서버 IP                          
    - 10.30.70.79
    - 10.30.70.80
- name: pinpoint_broker                    #작업 이름(필수)
  template: broker                         # job template 이름(필수)
  instances: 1                             # job 인스턴스 수(필수)
  resource_pool: pinpoint_small            # resource_pools block에 정의한 resource pool 이름(필수)
  networks:                                # 네트워크 구성정보
  - name: default                          # Networks block에서 선언한 network 이름(필수)
    static_ips:                            # 사용할 IP addresses 정의(필수): Pinpoint 서버 IP                          
    - 10.30.70.82
properties:                                # Pinpoint 설정정보
  master:                                  # Pinpoint master 설정정보
    host_name: h-master                    # Pinpoint master name
    host_ip: 10.30.70.75                   # Pinpoint master IP
    port: 9000                             # Pinpoint master port
    http_port: 50070                       # Pinpoint master http port
    replication: 2                         # Pinpoint master 복제개수
    tcp_listen_port: 29994                 # Pinpoint master tcp port
    udp_stat_listen_port: 29995            # Pinpoint master udp stat port
    udp_span_listen_port: 29996            # Pinpoint master udp span port
  broker:                                  # Pinpoint broker 설정정보
    collector_ips:                         # Pinpoint collector ip 설정정보
    - 10.30.70.45 
    - 10.30.70.46
    collector_tcp_port: 29994              # Pinpoint collector tcp port 설정정보
    collector_stat_port: 29995             # Pinpoint collector stat port 설정정보
    collector_span_port: 29996             # Pinpoint collector span port 설정정보
    dashboard_uri: http://115.68.46.182:80/#/main   # Pinpoint dashboard url 설정정보
  secondary:                               # Pinpoint secondary 설정정보
    host_name: h-secondary                 # Pinpoint secondary name
    host_ip: 10.30.70.76                   # Pinpoint secondary IP
    http_port: 50090                       # Pinpoint secondary port
  yarn:                                    # Pinpoint yarn 설정정보      
    host_name: h-master                    # Pinpoint yarn name
    host_ip: 10.30.70.75                   # Pinpoint yarn ip
    resource_tracker_port: 8025            # Pinpoint yarn resource_tracker_port
    scheduler_port: 8030                   # Pinpoint yarn scheduler_port
    resourcemanager_port: 8040             # Pinpoint yarn resourcemanager_port
  slave:                                   # Pinpoint slave 설정정보      
    host_names:                            # Pinpoint slave name
    - h-slave0
    - h-slave1
    host_ips:                              # Pinpoint slave ip
    - 10.30.70.90
    - 10.30.70.91
  collector:                               # Pinpoint collector 설정정보  
    host_names:                            # Pinpoint collector host name
    - h-collector0
    - h-collector1
    host_ips:                              # Pinpoint collector host IP
    - 10.30.70.45
    - 10.30.70.46
  haproxy:                                 # Pinpoint haproxy 설정정보  
    host_name: haproxy-webui               # Pinpoint haproxy host name
    host_ip: 10.30.70.78                   # Pinpoint haproxy host IP
    http_port: 80                          # Pinpoint haproxy host port
  webui:                                   # Pinpoint webui 설정정보  
    host_names:                            # Pinpoint webui host name
    - h-webui0
    - h-webui1
    host_ips:                              # Pinpoint webui host IP
    - 10.30.70.79
    - 10.30.70.80
releases:
- name: paasta-pinpoint-service                             # 서비스 릴리즈 이름(필수)
  version: 2.0                                              # 서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전

```

<br>
-   Deploy 할 deployment manifest 파일을 BOSH 에 지정한다.
```
$ bosh deployment paasta-pinpoint-vsphere-2.0.yml
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
```

<br>
-   Pinpoint 서비스팩을 배포한다.
```
$ bosh deploy
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on deployment 'paasta-pinpoint' on 'bosh'
Getting deployment properties from director...
Unable to get properties list from director, trying without it...

Detecting deployment changes
----------------------------
resource_pools:
- name: pinpoint_small
network: default
stemcell:
  name: bosh-vsphere-esxi-ubuntu-trusty-go_agent
  version: '3263.8'
cloud_properties:
  cpu: 1
  disk: 4096
  ram: 1024
- name: pinpoint_medium
network: default
stemcell:
  name: bosh-vsphere-esxi-ubuntu-trusty-go_agent
  version: '3263.8'
cloud_properties:
  cpu: 2
  disk: 8192
  ram: 2048

compilation:
workers: 6
network: default
cloud_properties:
  ram: 4096
  disk: 8096
  cpu: 4

networks:
- name: default
subnets:
- cloud_properties:
    name: Internal
  dns:
  - 10.30.20.24
  - 8.8.8.8
  gateway: 10.30.20.23
  name: default_unused
  range: 10.30.0.0/16
  reserved:
  - 10.30.20.0 - 10.30.20.22
  - 10.30.20.24 - 10.30.20.255
  - 10.30.40.0 - 10.30.40.255
  - 10.30.60.0 - 10.30.60.112
  static:
  - 10.30.70.0 - 10.30.70.255
- name: public_network
type: manual
subnets:
- cloud_properties:
    name: External
  dns:
  - 8.8.8.8
  range: 115.68.46.176/28
  gateway: 115.68.46.177
  static:
  - 115.68.46.182

update:
canaries: 1
canary_watch_time: 120000
update_watch_time: 120000
max_in_flight: 8

jobs:
- name: h_slave
template: h_slave
instances: 2
resource_pool: pinpoint_small
networks:
- name: default
  static_ips:
  - 10.30.70.73
  - 10.30.70.74
- name: h_secondary
template: h_secondary
instances: 1
resource_pool: pinpoint_small
networks:
- name: default
  static_ips:
  - 10.30.70.76
- name: h_master
template: h_master
instances: 1
resource_pool: pinpoint_medium
networks:
- name: default
  static_ips:
  - 10.30.70.75
- name: h_master_register
template: h_master_register
instances: 1
lifecycle: errand
resource_pool: pinpoint_small
networks:
- name: default
- name: collector
template: collector
instances: 2
resource_pool: pinpoint_medium
networks:
- name: default
  static_ips:
  - 10.30.70.40
  - 10.30.70.41
- name: haproxy_webui
template: haproxy_webui
instances: 1
resource_pool: pinpoint_small
networks:
- name: default
  static_ips:
  - 10.30.70.78
- name: public_network
  default:
  - dns
  - gateway
  static_ips: 115.68.46.182
- name: webui
template: webui
instances: 2
resource_pool: pinpoint_small
networks:
- name: default
  static_ips:
  - 10.30.70.79
  - 10.30.70.80
- name: pinpoint_broker
template: broker
instances: 1
resource_pool: pinpoint_small
networks:
- name: default
  static_ips:
  - 10.30.70.82

name: paasta-pinpoint

director_uuid: d363905f-eaa0-4539-a461-8c1318498a32

release:
name: paasta-pinpoint-service
version: latest

properties:
master:
  host_name: "<redacted>"
  host_ip: "<redacted>"
  port: "<redacted>"
  http_port: "<redacted>"
  replication: "<redacted>"
  tcp_listen_port: "<redacted>"
  udp_stat_listen_port: "<redacted>"
  udp_span_listen_port: "<redacted>"
broker:
  collector_ips:
  - "<redacted>"
  - "<redacted>"
  collector_tcp_port: "<redacted>"
  collector_stat_port: "<redacted>"
  collector_span_port: "<redacted>"
  dashboard_uri: "<redacted>"
secondary:
  host_name: "<redacted>"
  host_ip: "<redacted>"
  http_port: "<redacted>"
yarn:
  host_name: "<redacted>"
  host_ip: "<redacted>"
  resource_tracker_port: "<redacted>"
  scheduler_port: "<redacted>"
  resourcemanager_port: "<redacted>"
slave:
  host_names:
  - "<redacted>"
  - "<redacted>"
  host_ips:
  - "<redacted>"
  - "<redacted>"
collector:
  host_names:
  - "<redacted>"
  - "<redacted>"
  host_ips:
  - "<redacted>"
  - "<redacted>"
haproxy:
  host_name: "<redacted>"
  host_ip: "<redacted>"
  http_port: "<redacted>"
webui:
  host_names:
  - "<redacted>"
  - "<redacted>"
  host_ips:
  - "<redacted>"
  - "<redacted>"
Please review all changes carefully

Deploying
---------
Are you sure you want to deploy? (type 'yes' to continue): yes

Director task 446
Deprecation: Ignoring cloud config. Manifest contains 'networks' section.

Started preparing deployment > Preparing deployment. Done (00:00:00)

Started preparing package compilation > Finding packages to compile. Done (00:00:00)

Started compiling packages
Started compiling packages > haproxy/88e2cea38299d775fda4bac58eba167eefae6dc5
Started compiling packages > java/788dd8b3824f7ed89a51f0d870d8a5b6376820a5
Started compiling packages > ssh-keys/a92619994e7c69a7c52d30f4bc87c949ca2e5b77
Started compiling packages > bosh-helpers/10d1d5cf2e9dfd6c8279aa99386c560ed32eb6fe. Done (00:01:24)
   Done compiling packages > ssh-keys/a92619994e7c69a7c52d30f4bc87c949ca2e5b77 (00:01:29)
   Done compiling packages > haproxy/88e2cea38299d775fda4bac58eba167eefae6dc5 (00:02:16)
   Done compiling packages > java/788dd8b3824f7ed89a51f0d870d8a5b6376820a5 (00:02:17)
Started compiling packages > broker/9c8209bff0c69156bcb15e1ed7f1c7f3b73df2cd
Started compiling packages > tomcat/462a5a0b802cb559e23fe8e59b804c594c641428
Started compiling packages > hadoop/1d3ea90f6e3173e51ce42eacccd380c8c821f3a4
Started compiling packages > hbase/5d5427c7c567de557a7f899f2c83f347723007c3
   Done compiling packages > tomcat/462a5a0b802cb559e23fe8e59b804c594c641428 (00:02:00)
Started compiling packages > webui/655536239b03532d40b99159865520d8ee9e870d
Started compiling packages > collector/e0b59e5385885af84c3cb4363821d5d6d1be0eae
   Done compiling packages > broker/9c8209bff0c69156bcb15e1ed7f1c7f3b73df2cd (00:02:07)
   Done compiling packages > hadoop/1d3ea90f6e3173e51ce42eacccd380c8c821f3a4 (00:02:16)
   Done compiling packages > hbase/5d5427c7c567de557a7f899f2c83f347723007c3 (00:02:36)
   Done compiling packages > webui/655536239b03532d40b99159865520d8ee9e870d (00:01:33)
   Done compiling packages > collector/e0b59e5385885af84c3cb4363821d5d6d1be0eae (00:01:41)
   Done compiling packages (00:05:58)

Started creating missing vms
Started creating missing vms > h_slave/c6ed5066-be08-4dfa-b9cf-40c8c9c89778 (1)
Started creating missing vms > h_slave/ae8f48e2-5273-4b23-b63f-b2a3ab77eb4c (0)
Started creating missing vms > h_secondary/6b29e5c9-0aac-48fb-96d1-c029415451f6 (0)
Started creating missing vms > h_master/50b79e72-c97f-4402-98ed-c08d97723291 (0)
Started creating missing vms > collector/2af508e8-1751-41be-905a-43f5884c4402 (0)
Started creating missing vms > collector/e741680d-b612-4ccf-8495-7855006dfc80 (1)
Started creating missing vms > haproxy_webui/2a977c16-331b-430c-9690-314e31eea9c8 (0)
Started creating missing vms > webui/b23ab30f-c0f9-473a-9609-30fc5f2408b2 (0)
Started creating missing vms > webui/0b3a252a-9657-4efa-98ff-1eeaac368bc9 (1)
Started creating missing vms > pinpoint_broker/82756b8b-f1a1-4bbe-9408-6875bb2a9976 (0)
   Done creating missing vms > haproxy_webui/2a977c16-331b-430c-9690-314e31eea9c8 (0) (00:01:11)
   Done creating missing vms > h_master/50b79e72-c97f-4402-98ed-c08d97723291 (0) (00:01:22)
   Done creating missing vms > webui/b23ab30f-c0f9-473a-9609-30fc5f2408b2 (0) (00:01:25)
   Done creating missing vms > collector/2af508e8-1751-41be-905a-43f5884c4402 (0) (00:01:25)
   Done creating missing vms > webui/0b3a252a-9657-4efa-98ff-1eeaac368bc9 (1) (00:01:26)
   Done creating missing vms > pinpoint_broker/82756b8b-f1a1-4bbe-9408-6875bb2a9976 (0) (00:01:27)
   Done creating missing vms > h_slave/ae8f48e2-5273-4b23-b63f-b2a3ab77eb4c (0) (00:01:28)
   Done creating missing vms > h_secondary/6b29e5c9-0aac-48fb-96d1-c029415451f6 (0) (00:01:30)
   Done creating missing vms > h_slave/c6ed5066-be08-4dfa-b9cf-40c8c9c89778 (1) (00:01:30)
   Done creating missing vms > collector/e741680d-b612-4ccf-8495-7855006dfc80 (1) (00:01:40)
   Done creating missing vms (00:01:40)

Started updating instance h_slave
Started updating instance h_slave > h_slave/ae8f48e2-5273-4b23-b63f-b2a3ab77eb4c (0) (canary). Done (00:02:51)
Started updating instance h_slave > h_slave/c6ed5066-be08-4dfa-b9cf-40c8c9c89778 (1). Done (00:02:52)
   Done updating instance h_slave (00:05:43)
Started updating instance h_secondary > h_secondary/6b29e5c9-0aac-48fb-96d1-c029415451f6 (0) (canary). Done (00:02:43)
Started updating instance h_master > h_master/50b79e72-c97f-4402-98ed-c08d97723291 (0) (canary). Done (00:02:48)
Started updating instance collector
Started updating instance collector > collector/2af508e8-1751-41be-905a-43f5884c4402 (0) (canary). Done (00:02:34)
Started updating instance collector > collector/e741680d-b612-4ccf-8495-7855006dfc80 (1)

. Done (00:02:32)
   Done updating instance collector (00:05:06)
Started updating instance haproxy_webui > haproxy_webui/2a977c16-331b-430c-9690-314e31eea9c8 (0) (canary). Done (00:02:29)
Started updating instance webui
Started updating instance webui > webui/b23ab30f-c0f9-473a-9609-30fc5f2408b2 (0) (canary). Done (00:02:37)
Started updating instance webui > webui/0b3a252a-9657-4efa-98ff-1eeaac368bc9 (1). Done (00:02:37)
   Done updating instance webui (00:05:14)
Started updating instance pinpoint_broker > pinpoint_broker/82756b8b-f1a1-4bbe-9408-6875bb2a9976 (0) (canary). Done (00:02:32)

Task 446 done

Started   2016-12-29 07:54:37 UTC
Finished  2016-12-29 08:28:51 UTC
Duration  00:34:14

Deployed 'paasta-pinpoint' to 'bosh'
```

<br>
-   배포된 Pinpoint 서비스팩을 확인한다.

```
Deployment 'paasta-pinpoint'

Director task 503

Task 503 done

+----------------------------------------------------------+---------+-----+-----------------+---------------+
| VM                                                       | State   | AZ  | VM Type         | IPs           |
+----------------------------------------------------------+---------+-----+-----------------+---------------+
| collector/0 (2af508e8-1751-41be-905a-43f5884c4402)       | running | n/a | pinpoint_medium | 10.30.70.40   |
| collector/1 (e741680d-b612-4ccf-8495-7855006dfc80)       | running | n/a | pinpoint_medium | 10.30.70.41   |
| h_master/0 (50b79e72-c97f-4402-98ed-c08d97723291)        | running | n/a | pinpoint_medium | 10.30.70.75   |
| h_secondary/0 (6b29e5c9-0aac-48fb-96d1-c029415451f6)     | running | n/a | pinpoint_small  | 10.30.70.76   |
| h_slave/0 (ae8f48e2-5273-4b23-b63f-b2a3ab77eb4c)         | running | n/a | pinpoint_small  | 10.30.70.73   |
| h_slave/1 (c6ed5066-be08-4dfa-b9cf-40c8c9c89778)         | running | n/a | pinpoint_small  | 10.30.70.74   |
| haproxy_webui/0 (2a977c16-331b-430c-9690-314e31eea9c8)   | running | n/a | pinpoint_small  | 10.30.70.78   |
|                                                          |         |     |                 | 115.68.46.182 |
| pinpoint_broker/0 (82756b8b-f1a1-4bbe-9408-6875bb2a9976) | running | n/a | pinpoint_small  | 10.30.70.82   |
| webui/0 (b23ab30f-c0f9-473a-9609-30fc5f2408b2)           | running | n/a | pinpoint_small  | 10.30.70.79   |
| webui/1 (0b3a252a-9657-4efa-98ff-1eeaac368bc9)           | running | n/a | pinpoint_small  | 10.30.70.80   |
+----------------------------------------------------------+---------+-----+-----------------+---------------+
```
<br>
<div id ='HBase-기본-데이터-실행'></div>
### 2.4. HBase 기본 데이터 실행

> Pinpoint 서비스팩 배포가 완료 되었으면 HBase 14개의 기본 Table이
> 생성되어야 Application에서 서비스 팩을 정상 사용할 수 있다.

```
$ bosh run errand h_master_register
Acting as user 'admin' on deployment 'paasta-pinpoint' on 'my-bosh'

Director task 1875
Started preparing deployment > Preparing deployment. Done (00:00:00)

Started preparing package compilation > Finding packages to compile. Done (00:00:00)

Started creating missing vms > h_master_register/0 (3cccc024-c528-44ea-88fc-d6fea301ded5). Done (00:00:59)

Started updating job h_master_register > h_master_register/0 (3cccc024-c528-44ea-88fc-d6fea301ded5) (canary)^C
Do you want to cancel task 1875? [yN] (^C again to detach): n

. Done (00:02:13)

Started running errand > h_master_register/0. Done (00:00:34)

Started fetching logs for h_master_register/3cccc024-c528-44ea-88fc-d6fea301ded5 (0) > Finding and packing log files. Done (00:00:01)

Started deleting errand instances h_master_register > h_master_register/0 (3cccc024-c528-44ea-88fc-d6fea301ded5). Done (00:00:09)

Task 1875 done

Started   2016-12-14 06:41:06 UTC
Finished  2016-12-14 06:45:02 UTC
Duration  00:03:56

[stdout]
0 row(s) in 1.6690 seconds

0 row(s) in 2.2490 seconds

0 row(s) in 1.2360 seconds

0 row(s) in 1.2220 seconds

0 row(s) in 1.2240 seconds

0 row(s) in 2.2290 seconds

0 row(s) in 1.2290 seconds

0 row(s) in 2.2430 seconds

0 row(s) in 4.2790 seconds

0 row(s) in 2.2410 seconds

0 row(s) in 2.2270 seconds

0 row(s) in 2.2390 seconds

0 row(s) in 1.2260 seconds

0 row(s) in 1.2300 seconds

TABLE
AgentEvent
AgentInfo
AgentLifeCycle
AgentStat
ApiMetaData
ApplicationIndex
ApplicationMapStatisticsCallee_Ver2
ApplicationMapStatisticsCaller_Ver2
ApplicationMapStatisticsSelf_Ver2
ApplicationTraceIndex
HostApplicationMap_Ver2
SqlMetaData_Ver2
StringMetaData
Traces
14 row(s) in 0.0360 seconds

[stderr]
Warning: Permanently added 'h-master,10.244.2.21' (ECDSA) to the list of known hosts.
Unauthorized use is strictly prohibited. All access and activity
is subject to logging and monitoring.
2016-12-14 06:44:22,899 WARN [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable

Errand 'h_master_register' completed successfully (exit code 0)
```
<br>
<div id='pinpoint-서비스-브로커-등록'></div>
### 2.5. Pinpoint 서비스 브로커 등록

Pinpoint 서비스팩 배포가 완료 되었으면 Application에서 서비스 팩을
사용하기 위해서 먼저 Pinpoint 서비스 브로커를 등록해 주어야 한다.

서비스 브로커 등록시 PaaS-TA에서 서비스브로커를 등록 할
수 있는 사용자로 로그인이 되어 있어야 한다.

<br>
-   서비스 브로커 목록을 확인한다.

```
$ cf service-brokers
```
```
Getting service brokers as admin...

name   url
No service brokers found
```

<br>
-   Pinpoint 서비스 브로커를 등록한다.

```
$ cf create-service-broker {서비스브로커 이름} {서비스브로커 사용자ID} {서비스브로커 사용자비밀번호} http://{서비스브로커 URL(IP)}
```
```
서비스브로커 이름 : 서비스브로커 관리를 위해 PaaS-TA에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스브로커 명칭이다.

서비스브로커 사용자ID / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID입니다. 서비스브로커도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.

서비스브로커 URL : 서비스브로커가 제공하는 API를 사용할 수 있는 URL을 입력한다.
```

```
$ cf create-service-broker pinpoint-service-broker admin cloudfoundry http:// 10.30.70.82:8080
```
```
Creating service broker pinpoint-service-broker as admin...
OK
```

<br>
-   등록된 Pinpoint 서비스 브로커를 확인한다.

```
$ cf service-brokers
```
```
Getting service brokers as admin...
name url
pinpoint-service-broker http:// 10.30.70.82:8080
```
<br>
-   접근 가능한 서비스 목록을 확인한다.

```
$ cf service-access
```
```
Getting service access as admin...
broker: Pinpoint-service-broker
service plan access orgs
Pinpoint Pinpoint\_standard none
```
서비스 브로커 생성시 디폴트로 접근을 허용하지 않는다.

<br>
-   특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)

```
$ cf enable-service-access Pinpoint
```

```
Enabling access to all plans of service Pinpoint for all orgs as admin...
OK
```
<br>
서비스 접근 허용을 확인한다.
```
$ cf service-access
```
```
Getting service access as admin...
broker: Pinpoint-service-broker
service plan access orgs
Pinpoint Pinpoint\_standard all
```
<br>
<div id='sample-web-app-연동-pinpoint-연동'></div>
#   3. Sample Web App 연동 Pinpoint 연동

본 Sample Web App은 개방형 클라우드 플랫폼에 배포되며 Pinpoint의 서비스를 Provision과 Bind를 한 상태에서 사용이 가능하다.
<br>
<div id ='sample-web-app-구조'></div>
### 3.1. Sample App 구조

Sample Web App은 PaaS-TA에 App으로 배포가 된다. 배포된
App에 Pinpoint 서비스 Bind 를 통하여 초기 데이터를 생성하게 된다. 바인드
완료 후 연결 url을 통하여 브라우저로 해당 App에 대한 Pinpoint 서비스
모니터링을 할 수 있다.

-   Spring-music App을 이용하여 Pinpoint 모니터링을 테스트 하였다.
-   앱을 다운로드 후 –b 옵션을 주어 buildpack을 지정하여 push 해 놓는다.

```
$ cf push -b java_buildpack_pinpoint --no-start
```

```
Using manifest file /home/ubuntu/workspace/bd_test/spring-music/manifest.yml

Creating app spring-music-pinpoint in org org / space space as admin...
OK

Creating route spring-music-pinpoint.monitoring.open-paas.com...
OK

Binding spring-music-pinpoint.monitoring.open-paas.com to spring-music-pinpoint...
OK

Uploading spring-music-pinpoint...
Uploading app files from: /tmp/unzipped-app175965484
Uploading 21.2M, 126 files
Done uploading               
OK
```

```
$ cf apps
```
```
Getting apps in org org / space space as admin...
OK

name                    requested state   instances   memory   disk   urls
php-demo                started           1/1         256M     1G     php-demo.monitoring.open-paas.com
spring-music            stopped           0/1         512M     1G     spring-music.monitoring.open-paas.com
spring-music-pinpoint   stopped           0/1         512M     1G     spring-music-pinpoint.monitoring.open-paas.com
```

<br>
<div id='PaaS-TA에서-서비스-신청'></div>
### 3.2. PaaS-TA에서 서비스 신청

Sample Web App에서 Pinpoint 서비스를 사용하기 위해서는 서비스
신청(Provision)을 해야 한다.

\*참고: 서비스 신청시 PaaS-TA에서 서비스를 신청 할 수 있는
사용자로 로그인이 되어 있어야 한다.

-   먼저 PaaS-TA Marketplace에서 서비스가 있는지 확인을
    한다.

```
$ cf marketplace
```

```
Getting services from marketplace in org org / space space as admin...
OK

service    plans               description
Pinpoint   Pinpoint_standard   A simple pinpoint implementation 
```

<br>
-   Marketplace에서 원하는 서비스가 있으면 서비스 신청(Provision)을 하여
    서비스 인스턴스를 생성한다.

```  
$ cf create-service {서비스명} {서비스플랜} {내서비스명}
```
```
서비스명 : p-Pinpoint로 Marketplace에서 보여지는 서비스 명칭이다.
서비스플랜 : 서비스에 대한 정책으로 plans에 있는 정보 중 하나를 선택한다. Pinpoint 서비스는 10 connection, 100 connection 를 지원한다.
내서비스명 : 내 서비스에서 보여지는 명칭이다. 이 명칭을 기준으로 환경설정정보를 가져온다.

```

```
$ cf create-service Pinpoint Pinpoint_standard PS1
```
```
Creating service instance PS1 in org org / space space as admin...
OK
```

<br>
-   생성된 Pinpoint 서비스 인스턴스를 확인한다.

```
$ cf services
```
```
Getting services in org org / space space as admin...
OK

name             service         plan                bound apps               last operation
app_log_drain    user-provided
PS1              Pinpoint        Pinpoint_standard                            create succeeded
syslog_service   user-provided                       php-demo, spring-music 
```

<br>
<div id='#sample-web-app에-서비스-바인드-신청-및-app-확인'></div>
### 3.3. Sample App에 서비스 바인드 신청 및 App 확인
-------------------------------------------------

서비스 신청이 완료되었으면 Sample Web App 에서는 생성된 서비스
인스턴스를 Bind 하여 App에서 Pinpoint 서비스를 이용한다.

\*참고: 서비스 Bind 신청시 PaaS-TA 플랫폼에서 서비스 Bind신청 할
수 있는 사용자로 로그인이 되어 있어야 한다.

-   Sample Web App에서 생성한 서비스 인스턴스 바인드 신청을 한다.

<br>
- 서비스 인스턴스 확인
```
$ cf s
```
```
Getting services in org org / space space as admin...
OK

name             service         plan                bound apps               last operation
app_log_drain    user-provided
PS1              Pinpoint        Pinpoint_standard                            create succeeded
syslog_service   user-provided                       spring-music, php-demo

<br>
- 서비스 바인드
```
$ cf bind-service spring-music-pinpoint PS1 -c '{"application_name":"spring-music"}'
```
Binding service PS1 to app spring-music-pinpoint in org org / space space as admin...
OK
TIP: Use 'cf restage spring-music-pinpoint' to ensure your env variable changes take effect
```
  
cf cli 리눅스 버전 : 
```
cf bind-service <application이름> PS1 -c ‘{"application_name\":"<application이름>"}
```  

cf cli window 버전 : 
```
cf bind-service <application이름> PS1 -c "{\"application_name\":\"<application이름>\"}"
```
<br>
-   바인드가 적용되기 위해서 App을 restage한다.

```
$ cf restage spring-music-pinpoint
```
```

Restaging app spring-music-pinpoint in org org / space space as admin...
Downloading binary_buildpack...
Downloading go_buildpack...
Downloading staticfile_buildpack...
Downloading java_buildpack...
Downloading ruby_buildpack...
Downloading nodejs_buildpack...
Downloading python_buildpack...
Downloading php_buildpack...
Downloaded python_buildpack
Downloaded binary_buildpack
Downloaded go_buildpack
Downloaded java_buildpack
Downloaded ruby_buildpack
Downloaded nodejs_buildpack
Downloaded staticfile_buildpack
Downloaded php_buildpack
Creating container
Successfully created container
Downloading app package...
Downloaded app package (24.5M)
Downloading build artifacts cache...
Downloaded build artifacts cache (54.1M)
Staging...
-----> Java Buildpack Version: v3.7.1 | https://github.com/cloudfoundry/java-buildpack.git#78c3d0a
-----> Downloading Open Jdk JRE 1.8.0_91-unlimited-crypto from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_91-unlimited-crypto.tar.gz (found in cache)
     Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.6s)
-----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
     Memory Settings: -XX:MaxMetaspaceSize=64M -Xss995K -Xmx382293K -Xms382293K -XX:MetaspaceSize=64M
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
-----> Downloading Tomcat Instance 8.0.39 from https://java-buildpack.cloudfoundry.org/tomcat/tomcat-8.0.39.tar.gz (found in cache)
     Expanding Tomcat Instance to .java-buildpack/tomcat (0.1s)
-----> Downloading Tomcat Lifecycle Support 2.5.0_RELEASE from https://java-buildpack.cloudfoundry.org/tomcat-lifecycle-support/tomcat-lifecycle-support-2.5.0_RELEASE.jar (found in cache)
-----> Downloading Tomcat Logging Support 2.5.0_RELEASE from https://java-buildpack.cloudfoundry.org/tomcat-logging-support/tomcat-logging-support-2.5.0_RELEASE.jar (found in cache)
-----> Downloading Tomcat Access Logging Support 2.5.0_RELEASE from https://java-buildpack.cloudfoundry.org/tomcat-access-logging-support/tomcat-access-logging-support-2.5.0_RELEASE.jar (found in cache)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading droplet...
Uploading build artifacts cache...
Uploaded build artifacts cache (54.1M)
Uploaded droplet (77.3M)
Uploading complete

0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK
```
<br>
-   App이 정상적으로 Pinpoint 서비스를 사용하는지 확인한다.

![pinpoint_image_03]

<br>
-  환경변수 확인

```
$ cf env spring-music-pinpoint
```
```
Getting env variables for app spring-music-pinpoint in org org / space space as admin...
OK

System-Provided:
{
"VCAP_SERVICES": {
"Pinpoint": [
 {
  "credentials": {
   "application_name": "spring-music",
   "collector_host": "10.244.2.30",
   "collector_span_port": 29996,
   "collector_stat_port": 29995,
   "collector_tcp_port": 29994
  },
  "label": "Pinpoint",
  "name": "PS1",
  "plan": "Pinpoint_standard",
  "provider": null,
  "syslog_drain_url": null,
  "tags": [
   "pinpoint"
  ],
  "volume_mounts": []
 }
]
}
}

{
"VCAP_APPLICATION": {
"application_id": "b010e6e9-5431-4198-81f8-7d6ba9c14f40",
"application_name": "spring-music-pinpoint",
"application_uris": [
 "spring-music-pinpoint.monitoring.open-paas.com"
],
"application_version": "9a600116-97bd-45da-a33e-3b0d5592b1d0",
"limits": {
 "disk": 1024,
 "fds": 16384,
 "mem": 512
},
"name": "spring-music-pinpoint",
"space_id": "bc70b951-d870-49ca-b57d-5c7137060e5e",
"space_name": "space",
"uris": [
 "spring-music-pinpoint.monitoring.open-paas.com"
],
"users": null,
"version": "9a600116-97bd-45da-a33e-3b0d5592b1d0"
}
}

No user-defined env variables have been set

No running env variables have been set

No staging env variables have been set
```

<br>
- App 정상 구동 확인
```
$ curl http://115.68.151.187/#/main/spring-music-pinpoint@TOMCAT
```

[pinpoint_image_01]:/Service-Guide/images/pinpoint/pinpoint-image1.png
[pinpoint_image_02]:/Service-Guide/images/pinpoint/pinpoint-image2.png
[pinpoint_image_03]:/Service-Guide/images/pinpoint/pinpoint-image3.png
