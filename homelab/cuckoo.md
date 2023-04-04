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
* 가상 IP: 192.168.121.10(정적) 및 192.168.121.20

_또는 다른 VM을 설치하여 데이터베이스 쿼리를 저장하여 Cuckoo 호스트에서 파일 저장소를 격리할 수 있습니다. 일반적으로 cuckoo 호스트(Ubuntu에 설치)는 100GB 미만으로 실행되며 호스트에 데이터베이스 저장소를 두는 것은 이상적인 장기 사용이 아닐 수 있습니다._

