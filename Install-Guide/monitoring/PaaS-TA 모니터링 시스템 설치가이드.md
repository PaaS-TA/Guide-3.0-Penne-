## Table of Contents
1. [문서 개요](#1)
     * [1.1. 목적](#2)
2. [Metrics Architecture](#3)
     * [2.1.  Architecure](#4)
     * [2.2.  Component 설명](#5)
3. [Log Architecture](#6)
     * [3.1.  Architecture](#7)
     * [3.2.  Component 설명](#8)
4. [설치 전제 조건](#9)
5. [InfluxDB 설치 방법](#10)
6. [Logsearch 설치 방법](#11)
7. [Metrics Collector 서비스 설치 방법](#12)
8. [Bosh Monitoring Agent 설치 방법](#13)

<div id='1'></div>

# 1. 문서 개요
본 문서는 모니터링 시스템의 아키텍처 구조에 대한 설명과 설치 하는 방법을 기술하였다.

<div id='2'></div>

### 1.1. 목적

본 문서는 모니터링 시스템의 Metrics Architecture 구조와 Log Architecture 구조에 대해 설명하고, 모니터링 시스템을 사용하기 위한 Release 파일을 설치하는 방법에 대해 설명하였다.

<div id='3'></div>

# 2.  Metrics Architecture

본 장에서는 모니터링 시스템 정보 중 Metrics 정보를 수집하는 Architecture에 대해 기술하였다.

<div id='4'></div>

### 2.1. Architecture

<kbd>![2-1-1]</kbd>

<div id='5'></div>

### 2.2. Component 설명

<kbd>![2-2-1]</kbd>

<div id='6'></div>

# 3. Log Architecture

본 장에서는 모니터링 시스템 정보 중 Log 정보를 수집하는 Architecture에 대해 기술하였다.

<div id='7'></div>

### 3.1. Architecture

<kbd>![3-1-1]</kbd>

<div id='8'></div>

### 3.2. Component 설명

<kbd>![3-2-1]</kbd>

<div id='9'></div>

# 4. 설치 전제 조건

모니터링 시스템 관련 서비스들을 설치하기전 사전에 Bosh 및 PaaS-TA 3.0 서비스들이 설치되어 있어야 하며, Bosh는 Micro Bosh로 설치되어야 한다.

<div id='10'></div>

# 5. InfluxDB(Metrics Database) 시스템 설치 방법

본 장에서는 모니터링 시스템의 매트릭스(Metrics) 정보를 저장 및 관리하는 데이터베이스 시스템인 InfluxDB과 모니터링 시스템의 화면 UI인 Grafana 서비스를 설치하는 방법에 대해 기술하였다.
자세한 설치 가이드는 IaaS 환경에 맞는 설치가이드를 참조하면 된다.

- [OpenStack 환경](https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Install-Guide/Services/PaaS-TA%20InfluxDB%20%EB%B0%8F%20Grafana%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C(OpenStack)_v1.0.md)
- [Vsphere  환경](https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Install-Guide/Services/PaaS-TA%20InfluxDB%20%EB%B0%8F%20Grafana%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C(VMWare)_v1.0.md)
- [AWS 환경](https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Install-Guide/Services/PaaS-TA%20InfluxDB%20%EB%B0%8F%20Grafana%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C(AWS)_v1.0.md)

<div id='11'></div>

# 6. Logsearch (로그정보 수집) 시스템 설치 방법

본 장에서는 모니터링 시스템의 로그 정보를 저장 및 관리하는 Repository 시스템인 Logsearch를 설치하는 방법에 대해 기술하였다.
자세한 설치 가이드는 IaaS 환경에 맞는 설치가이드를 참조하면 된다.

- [OpenStack 환경](https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Install-Guide/Services/PaaS-TA%20Logsearch%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C(OpenStack)_v1.0.md)
- [Vsphere  환경](https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Install-Guide/Services/PaaS-TA%20Logsearch%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C(VMWare)_v1.0.md)
- [AWS 환경](https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Install-Guide/Services/PaaS-TA%20Logsearch%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C(AWS)_v1.0.md)

<div id='12'></div>

# 7. Metrics Collector 서비스 설치 방법

본 장에서는 PaaSTA 서비스들의 매트릭스(Metrics) 정보를 수집하여 InfluxDB시스템의 데이터베이스에 정보를 저장하는 서비스의 설치하는 방법에 대해 기술하였다.
자세한 설치 가이드는 IaaS 환경에 맞는 설치가이드를 참조하면 된다.

- [OpenStack 환경](https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Install-Guide/Services/PaaS-TA%20Metrics%20Collector%20%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C(OpenStack)_v1.0.md)
- [Vsphere  환경](https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Install-Guide/Services/PaaS-TA%20Metrics%20Collector%20%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C(VMWare)_v1.0.md)
- [AWS 환경](https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Install-Guide/Services/PaaS-TA%20Metrics%20Collector%20%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C(AWS)_v1.0.md)

<div id='13'></div>

# 8. Bosh Monitoring Agent 설치 방법

본 장에서는 Bosh 서비스에서 기본적으로 제공되지 않는 Bosh 서비스 Metrics 수집 Agent 및 로그 수집 Agent 설치 방법에 대해 기술하였다.
자세한 설치 가이드는 아래 링크에 연결된 문서를 참조하면 된다.

- [설치 가이드](https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Install-Guide/BOSH/Bosh%20Monitoring%20Agent%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C_v1.0.md)

[2-1-1]:images/monitoring-architecture/2-1-1.png
[2-2-1]:images/monitoring-architecture/2-2-1.png
[3-1-1]:images/monitoring-architecture/3-1-1.png
[3-2-1]:images/monitoring-architecture/3-2-1.png
