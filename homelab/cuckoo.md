---
description: >-
  이 문서에서는 기본 Cuckoo Sandbox를 빌드하는 방법에 대해 설명합니다. 이 시리즈를 두 부분으로 나누었습니다. 1부 -
  Cuckoo 호스트 소개 및 구성, 2부 - Cuckoo 게스트 구성 및 사용자 지정.
---

# 악성코드 자동화 분석툴 Cuckoo 샌드박스 설치

### 쿠쿠 샌드박스 <a href="#cuckoo-sandbox" id="cuckoo-sandbox"></a>

Cuckoo SandBox는 악성코드를 자동으로 분석하는 데 사용할 수 있는 오픈 소스 샌드박스입니다. 이것은 오픈 소스이므로 무료로 사용할 수 있지만 설치에는 약간의 시간과 노력이 필요합니다.

#### 샌드박싱이란? <a href="#what-is-sandboxing" id="what-is-sandboxing"></a>

먼저 샌드박싱이란 무엇입니까?샌드박싱은 최종 사용자 운영 환경을 모방한 네트워크의 안전하고 격리된 환경에서 코드를 실행하고 관찰 및 분석하고 코딩하는 사이버 보안 방식입니다.Cuckoo 샌드박스는 의심스러운 파일을 던질 수 있도록 개발되었으며, 이를 "샌드박싱"이라고 부르는 현실적이면서도 고립된 환경에서 파일을 실행할 것입니다.

#### Cuckoo SandBox 구조의 모습 <a href="#how-the-cuckoo-sandbox-structure-looks-like" id="how-the-cuckoo-sandbox-structure-looks-like"></a>

어떻게 작동합니까? 샌드박싱 환경을 설치하는 방법은 여러 가지가 있지만 여기서는 간단한 구조를 구축해 보겠습니다.아래 다이어그램은 기본 Cuckoo SandBox 환경이 어떻게 보이는지 보여줍니다.![](https://1538861842-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-Luj\_UTA-FG9do0VYXMX%2Fuploads%2F963kGS0rYqbi5NxsWekt%2FCuckoo%20Structure.png?alt=media\&token=82f19e66-3ddd-4d98-998c-2936532b481f)

**뻐꾸기 호스트:**

Cuckoo 호스트는 게스트 및 분석 관리를 담당합니다. 분석을 시작하고 트래픽을 덤프하며 보고서를 생성합니다.

* 운영 체제: 우분투 22 LTS
* CPU: 가상 CPU 4 코어
* 메모리: 8GB
* 가상 IP: 192.168.121.133(고정)

**가상 네트워크:**

가상 네트워크는 분석 가상 머신을 실행하는 격리된 네트워크입니다.

* Cuckoo 호스트와 게스트 사이의 브리지 어댑터.

**분석 게스트:**

Analysis Guest는 악성코드가 실행되는 깨끗한 환경입니다. 맬웨어 동작은 Cuckoo 호스트에 다시 보고됩니다.

* 운영 체제: Windows 7 및 Windows 10
* CPU: 가상 CPU 2 코어
* 메모리: 2GB
* 가상 IP: 192.168.121.10(정적) 및 192.168.121.20

_또는 다른 VM을 설치하여 데이터베이스 쿼리를 저장하여 Cuckoo 호스트에서 파일 저장소를 격리할 수 있습니다. 일반적으로 cuckoo 호스트(Ubuntu에 설치)는 100GB 미만으로 실행되며 호스트에 데이터베이스 저장소를 두는 것은 이상적인 장기 사용이 아닐 수 있습니다._

### 설치 단계 <a href="#installation-steps" id="installation-steps"></a>

설치 단계는 다음과 같이 간단합니다.

| Cuckoo Host 요구사항 설정 -> Cuckoo Host에 Cuckoo Ubuntu 서버 설치-> Cuckoo Gust에 Windows VM 설치 -> Windows VM 구성 -> 네트워크 구성 요소 구성 -> Cuckoo 에이전트 설치 -> 가상 머신 저장 -> Cuckoo 모듈 구성 및 사용자 정의 -> 테스트 |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

### 뻐꾸기 호스트 <a href="#cuckoo-host-1" id="cuckoo-host-1"></a>

먼저 호스트 머신 생성을 진행합니다. Cuckoo 설치 및 구성을 진행하기 전에 cuckoo 호스트를 배포하려면 다음과 같은 기본 요구 사항 설정이 필요합니다.![](https://1538861842-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-Luj\_UTA-FG9do0VYXMX%2Fuploads%2FqnOySmKG9JTsHHCrKbKx%2Fimage.png?alt=media\&token=6f5e8273-bc81-4906-a74b-2c35e1a73aab)뻐꾸기 호스트

#### Cuckoo 호스트 요구 사항 설정 <a href="#cuckoo-host-requirement-setup" id="cuckoo-host-requirement-setup"></a>

**VMware Workstation 다운로드 및 Ubuntu VM 만들기**

VMware Workstation Pro를 사용했지만 언제든지 다른 시각화 솔루션을 다운로드할 수 있습니다. [또한 이 링크](https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html) 에서 무료 버전 VMware Workstation 16 Player를 다운로드할 수 있습니다 .이 데모에서는 Ubuntu OS를 Cuckoo 서버 호스트로 설치했습니다.![](https://1538861842-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-Luj\_UTA-FG9do0VYXMX%2Fuploads%2F6UWuuzL3BEC1jSYh9jhe%2Fimage.png?alt=media\&token=f7c78d0e-cc06-4a27-bd9e-a6d00bcfca5f).

**Python 라이브러리 설치(Ubuntu 및 Debian 기반 배포용)**

호스트를 설치하기 전에 일부 필수 소프트웨어 패키지 및 라이브러리를 설치해야 합니다. Cuckoo 샌드박스는 아직 Python 3를 완전히 지원하지 않으므로 다음과 같이 Python 2.7 패키지를 설치해야 합니다.sudo apt-get install python2 python-pip python2.7-dev libffi-dev libssl-devpip2는 virtualenv를 설치합니다.pip2는 python-setuptools를 설치합니다.sudo apt-get 설치 libjpeg-dev zlib1g-dev swig

**몽고DB 설치**

Django 기반의 Web Interface를 사용하기 위해서는 MongoDB가 필요합니다. 명령 설치 대신,sudo apt-get install mongodb, 수동 설치(소스 코드에서)를 권장합니다.echo "deb http://security.ubuntu.com/ubuntu impish-security main" | sudo 티 /etc/apt/sources.list.d/impish-security.listsudo apt-get 업데이트sudo apt-get 설치 libssl1.1sudo apt-get 설치 컬컬 -fsSL https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key 추가-echo "deb \[ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo 티 /etc/apt/sources.list.d/mongodb-org-5.0.listsudo 적절한 업데이트sudo apt 설치 mongodb-org -y몽고DB 서비스 시작sudo 서비스 mongod 시작 && sudo systemctl enable mongod.service

**TCPDUMP 설치**

맬웨어 실행 중 네트워크 활동을 덤프하려면 tcpdump 패키지가 필요합니다.sudo apt-get install tcpdump apparmor-utilssudo aa-disable /usr/sbin/tcpdumpsudo setcap cap\_net\_raw,cap\_net\_admin=eip /usr/sbin/tcpdumpsudo setcap cap\_net\_raw,cap\_net\_admin=eip /usr/bin/tcpdumpsudo getcap /usr/sbin/tcpdumpTcpdump는 루트 권한이 필요하지만 Cuckoo가 루트로 실행되는 것을 원하지 않기 때문에 특정 Linux 기능을 바이너리로 설정해야 합니다.sudo 그룹 추가 pcapsudo usermod -a -G pcap 뻐꾸기sudo chgrp pcap /usr/sbin/tcpdumpsudo setcap cap\_net\_raw,cap\_net\_admin=eip /usr/sbin/tcpdump

**Volatility 및 M2Crypto 설치**

자식 클론 https://github.com/volatilityfoundation/volatility3.gitsudo pip2 설치 m2crypto

**guacd 설치**

guacd는 Cuckoo 웹 인터페이스에서 원격 제어 기능을 위해 RDP, VNC 및 SSH에 대한 변환 계층을 제공하는 선택적 서비스입니다.sudo apt 설치 libguac-client-rdp0 libguac-client-vnc0 libguac-client-ssh0 guacd

**버추얼박스 설치**

이제 게스트 VM을 생성하려면 가상화 솔루션이 필요합니다. Cuckoo Sandbox는 대부분의 가상화 소프트웨어 솔루션을 지원합니다. 여기에서는 VirtualBox를 사용하고 다음과 같은 수동 명령을 사용하는 것이 좋습니다.에코 뎁 http://download.virtualbox.org/virtualbox/debian xenial contrib | sudo 티 -a /etc/apt/sources.list.d/virtualbox.listwget -q https://www.virtualbox.org/download/oracle\_vbox\_2016.asc -O- | sudo apt-key 추가-sudo apt-get 업데이트sudo apt-get 설치 -y virtualbox-5.1sudo apt-get 설치 가상 상자https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/.

#### Cuckoo 호스트에 Cuckoo 서버 설치 <a href="#installing-cuckoo-server-on-the-cuckoo-host" id="installing-cuckoo-server-on-the-cuckoo-host"></a>

최신 버전의 Cuckoo를 설치하는 방법은 다음과 같이 간단합니다. -U 옵션을 사용하여 "cuckoo"(현재 사용자)로 cuckoo를 설치합니다.sudo pip2 설치 -U 뻐꾸기이제 cuckoo가 제대로 설치되었는지 확인해보자.![](https://1538861842-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-Luj\_UTA-FG9do0VYXMX%2Fuploads%2FS1mZGd65Td71CuoSXAhI%2FCuckoo%20is%20installed!.PNG?alt=media\&token=f2e6d713-2667-4de7-9131-20c2f687a045)이것으로 cuckoo 샌드박스 설치의 첫 번째 부분을 마칩니다. 2부에서는 Cuckoo Analysis Guest VM의 구성을 계속합니다.
