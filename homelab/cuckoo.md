---
description: >-
  이 문서에서는 Cuckoo Sandbox를 빌드하는 방법에 대해 설명합니다. 이 시리즈를 두 부분으로 나누었습니다. 1부 - Cuckoo
  호스트 소개 및 구성, 2부 - Cuckoo 게스트 구성 및 사용자 지정.
---

# 악성코드 자동화 분석툴 Cuckoo 샌드박스 설치

## 쿠쿠 샌드박스란? <a href="#cuckoo-sandbox" id="cuckoo-sandbox"></a>

Cuckoo SandBox는 악성코드를 자동으로 분석하는 데 사용할 수 있는 오픈 소스 샌드박스입니다. 이것은 오픈 소스이므로 무료로 사용할 수 있지만 설치에는 약간의 시간과 노력이 필요합니다.

### 샌드박싱이란? <a href="#what-is-sandboxing" id="what-is-sandboxing"></a>

먼저 샌드박싱이란 무엇일까요? 샌드박싱은 End User가 실제 운영 환경을 모방해 안전하고 격리된 네트워크 환경에서 파일을 실행하고 관찰 및 분석하는 방식입니다. Cuckoo 샌드박스는 의심스러운 파일을Dumping 할수 있도록 개발되었으며, 이를 "샌드박싱"이라고 부릅니다.

### Cuckoo SandBox 구조의 모습 <a href="#how-the-cuckoo-sandbox-structure-looks-like" id="how-the-cuckoo-sandbox-structure-looks-like"></a>

Cuckoo 샌드박스의 기본적구조에 대해 알아봅시다. 샌드박싱환경을 설치하는 방법은 여러 가지가 있지만 여기서는 간단한 구조를 구축해 보겠습니다.

아래 다이어그램은 기본 Cuckoo 샌드박스환경이 어떻게 구성되는지 설명합니다.

<figure><img src="https://1538861842-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-Luj_UTA-FG9do0VYXMX%2Fuploads%2F963kGS0rYqbi5NxsWekt%2FCuckoo%20Structure.png?alt=media&#x26;token=82f19e66-3ddd-4d98-998c-2936532b481f" alt=""><figcaption></figcaption></figure>

#### **Cuckoo 호스트:**

Cuckoo 호스트는 Cuckoo 게스트와 관리하며 데이터 트래픽 관리를 담당합니다. 분석을 시작하고 트래픽을 덤프하며 보고서를 생성합니다.

**아래와 같은 스팩으로 호스트를 설치합니다:**

* 운영 체제: 우분투 22 LTS
* CPU: 가상 CPU 4 코어
* 메모리: 8GB
* 가상 IP: 192.168.121.133(고정)

**가상 네트워크 (Virtual Network):**

가상 네트워크는 가성 VM호스트와 게스트를 이어주는 네트워크입니다.

* Bridge Adapter로Cuckoo 호스트와 게스트 사이를 연결해 주세요.

**Cuckoo 분석 게스트:**

각   분석 게스트 Analysis Guest는 악성코드가 실행되는 깨끗한 환경입니다. 맬웨어 동작은 Cuckoo 호스트에 다시 보고됩니다.

**아래와 같은 스팩으로 게스트를 설치합니다:**

* 운영 체제: Windows 7 및 Windows 10
* CPU: 가상 CPU 2 코어
* 메모리: 2GB
* 가상 IP: 192.168.121.10(Static) 및 192.168.121.20

_주의! 데이터베이스 쿼리를 저장을 위한 개별 VM을 설치하여 Cuckoo 호스트 VM과 데이터 베이스를 격리할 수도 있습니다. 일반적으로 Cuckoo 호스트 (Ubuntu에 설치할 경우)는 100GB 미만으로 설치되며 호스트 VM에 데이터베이스 저장소를 두는 것은 장기적으 사용하기에는 이상적인 환경이 아닙니다._

### 설치 단계

설치 단계는 다음의 과정을 거칩니다.

Cuckcoo 호스트 Requirement 사항 설정&#x20;

\-> Cuckoo 호스트에 Cuckoo 우분투 서버 설치&#x20;

\-> Cuckoo 돌풍에 윈도우 VM 설치&#x20;

\-> 윈도우 VM 구성&#x20;

\-> 네트워크 구성 요소 구성&#x20;

\-> Cuckoo 에이전트 설치&#x20;

\-> 가상 머신 저장&#x20;

\-> Cuckoo 모듈 및 사용자 정의 구성&#x20;

\-> 작동 테스트

#### Cuckcoo 호스트

먼저 Cuckoo 호스트 설치를 진행합니다. Cuckoo 호스트 설치 및 구성을 진행하기 전에 Cuckoo 호스트를 성공적으로 설치하려면 다음과 같은 기본 요구 사항 설정이 필요합니다.

왼쪽 빨간 네모안이 Cuckcoo 호스트입니다. 호스트 VM은 서버의 역할을 수행하며 하나의 호스트를를 설정합니다.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

#### Cuckoo 호스트 요구 사항 설정

VMware Workstation을 다운로드하고 Ubuntu VM을 설치합니다.

본 데모에서는 VMware Workstation Pro를 사용했지만 언제든지 다른 시각화 솔루션을 사용하여도 무관합니다.

본 데모에서는 Ubuntu 리눅스 배포판을 Cuckoo 서버 호스트 OS로 설치했습니다.

* 운영 체제: 우분투 22 LTS
* CPU: 가상 CPU 4 코어
* 가상 IP: 192.168.121.133(고정)
* 메모리: 8GB

![](https://1538861842-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-Luj\_UTA-FG9do0VYXMX%2Fuploads%2F6UWuuzL3BEC1jSYh9jhe%2Fimage.png?alt=media\&token=f7c78d0e-cc06-4a27-bd9e-a6d00bcfca5f)

#### 파이썬 라이브러리 설치하기 (Ubuntu & Debian 기반 배포판용)

호스트를 설치하기 전에 몇 가지 필수 소프트웨어 패키지 및 라이브러리를 설치해야합니다. Cuckoo 샌드박스는 아직 Python 3을 완전히 지원하지 않으므로 다음과 같이 Python 2.7 패키지를 설치해야 합니다.

`sudo apt-get install python2 python-pip python2.7-dev libffi-dev libssl-dev`

`pip2 install virtualenv`

`pip2 install python-setuptools`

`sudo apt-get install libjpeg-dev zlib1g-dev swig`

#### MongoDB 설치

Django 기반 웹 인터페이스를 사용하기 위해서는 MongoDB가 필요합니다.&#x20;

sudo apt-get install mongodb 설치 대신에 수동 설치 (소스 코드에서)방법을 권장합니다:

```
cho "deb http://security.ubuntu.com/ubuntu impish-security main" | sudo tee /etc/apt/sources.list.d/impish-security.list
sudo apt-get update
sudo apt-get install libssl1.1
sudo apt-get install curl
curl -fsSL https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
sudo apt update
sudo apt install mongodb-org -y
```

#### MongoDB 서비스 시작

```
sudo service mongod start && sudo systemctl enable mongod.service
```

#### TCPDUMP 설치

맬웨어 샌드박스  실행 중 발생하는 네트워크 트래픽 덤프하려면 tcpdump 패키지가 필요합니다.

```
sudo apt-get install tcpdump apparmor-utils
sudo aa-disable /usr/sbin/tcpdump
sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/tcpdump
sudo getcap /usr/sbin/tcpdump
```

Tcpdump 실행은 루트 권한이 필요하지만 Cuckoo가 루트로 실행되는 것을 원하지 않으므로 특정 Linux 기능을 바이너리로 설정해야합니다.

```
sudo groupadd pcap
sudo usermod -a -G pcap cuckoo
sudo chgrp pcap /usr/sbin/tcpdump
sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
```

#### Volatility & M2Crypto 설치

자식 복제 https://github.com/volatilityfoundation/volatility3.git

sudo pip2 설치 m2crypto



guacd 설치

guacd는 Cuckoo 웹 인터페이스의 원격 제어 기능을 위해 RDP, VNC 및 SSH에 대한 변환 계층을 제공하는 선택적 서비스입니다.

sudo apt install libguac-client-rdp0 libguac-client-VNC0 libguac-client-ssh0 guacd



