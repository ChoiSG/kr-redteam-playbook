---
description: 'DNS 개념 편 #1'
---

# 사이버 보안에 필요한 DNS 개념 편 #1

기본 개념 DNS 편은 크게 세편으로 구성된다.

1. 개념 편
2. 공격 & 우회 편 (레드팀) - TBU
3. 방어 & 탐지 편 (블루팀) - TBU

이번 개념편에서는 DNS에 대한 전반적인 개념과 작동 원리, 중요성에 대해 다룬다.

## 개요

보안 관점에서의 DNS의 중요성에 대해 알아보자. 일단 여러 이야기에 앞서 DNS에 대한 지식의 필요성을 강조하기위해 아래 필자의 사례를 가져왔다.

아래 이메일을 필자가 실제 Apple사와 Vulnerability Engineer (Cyber Security / Penetration Testing) 포지션 인터뷰 이메일이다. 총 6번, 각 2시간이 넘는인터뷰를 하고 ~~**떨어졌다**~~.

왜 갑자기 APPLE사와의 인터뷰썰을 말하냐 하면 APPLE사와의 보안 엔지니어 6번 인터뷰중 5번은 DNS에 대해 물어보았다. 그만큼 DNS의 대한 개념이 중요하다는거지 사과 회사와의 최종 인터뷰를 자랑 하는게 아니다. 필자는 처참하게 사과회사에서 떨어졌으니 다시 한번 말하지만 부끄러운 과거?라하겠다.

참고로 당시 필자는 DNS 우회나 공격 기법에 대해선 알고 있었지만 실제 DNS가 어떻게 작동하는지에 대해서는 제대로 설명하지 못하였다. 사실상 떨어진 이유도 기본에 충실하지 못한거였기 때문에 떨어졌어도 그렇게 억울할 일이 없다. ( tmi 필자는 갤럭시 유저다).

어쨌든 빅테크 사이버보안 시장 인터뷰에서도 자주 물어보는 질문이니 DNS에 대한 개념은 중요하다 하겠다.

![](<../obsidian\_resources/apple interview.png>)

가장 효과적인 설명방식인 Golden Circle의 무엇을 어떻게 왜? (What -> How -> Why) 순으로 알아볼테니 제발 잘 따라와 주길 바란다. 또한 최대한 이해하기 쉽게 쉬운 언어와 다이어그램으로 설명하니 이 포스트를 다 읽을 때쯤 당신은 APPLE사의 인터뷰 질문을 아주 훌륭하게 통과할 수 있다.

## DNS란 무엇인가? (What?)

DNS는 Domain Name System의 약자로 쉽게 설명하자면 "온라인사의 전화번호부"라고도 할 수 있다. 라때 시절 전화번호 책에서 전화 번호를 찾거나 종이 지도로 길을 찾아 보았듯 사이버상에도 DNS을 통해 목적지의 주소를 찾아야 한다. 이렇게 컴퓨터 이름(도메인 이름)을 찾아 컴퓨터의 주소인 IP를 얻는 시스템을 DNS라고 부른다. 영어권 국가에 거주한다면 Yello Pages라고도 볼수 있다.&#x20;

{% embed url="https://en.wikipedia.org/wiki/Yellow_pages" %}



![실제 90년대 전화번호부](<../obsidian\_resources/Pasted image 20230525090018.png>)

단, DNS가 전화번호부와 다른점이 있다면 DNS는 인터넷에 연결된 컴퓨터가 "알아서" 목적지를 찾아주는 시스템이라 전화번호북처럼 "직접" 찾아보지 않아도 된다는 점이다.

지금 브라우저에서 https://레드팀.com 을 찾아보자. 브라우저는 "알아서" 레드팀.com을 찾아서 보여준다.

## DNS, 그래서그거 어떻게 하는건데? (How?)

DNS가 어떻게 작동하는지 알아보자.&#x20;

먼저 유저는 브라우저 창에 레드팀.com 을 치는 상황을 가정해 보자. 브라우저는 레드팀.com 이 무엇인지 모르기 때문에 DNS를 통해 아래와 같은 방식으로 레드팀.com 의 IP 주소를 가져와야 한다. 이렇게 DNS가 목적지의 주소 (여기서는 IP 주소)를 찾아주는 행위를 "DNS Lookup"이라 부른다.

사실상 아래 다이어그램은 ISP를 통해 Local DNS Server와 먼저 교신을 하고도 IP를 찾지 못했을 때의 예이다. 예를 들어 KT를 사용한다면 KT에서 제공하는 기본 DNS 서버를 통해 도메인의 IP를 찾을 수 있다면  아래 구조까지 내려가지 않아도 된다.

### 1. Root Name Server&#x20;

Root Name 서버가 등장하는 시점은 다음과 같다. 아래 다이어그램의 과정과 순서 번호를 적어 놓았으니 그대로 따라오면 된다.

* (1) 유저: www.레드팀.com 아이피 뭐야?
* DNS Resolving Name Server: Local DNS Server한테 물어볼께. Local DNS Server! www.레드팀.com 아이피 뭐야?
* Local DNS Server: 잠시만...찾아볼께... ! 나도 모르겠어. 대신 Root Name Server에 물어봐.

이제 DNS Resolving Nmae Server는 Root Name Server에 레드팀.com의 IP 주소를 물어본다.

* (2) DNS Resolving Name Server: Root Name Server! 레드팀com IP 주소 좀 알려줘 !

이때 Root Name Server가 레드팀.com에 대한 정보가 없다면 다음과 같이 최상위 Root에서 시작해 순서대로 Child Name 서버 노드에게 물어보는 구조이다.

*   (3) Root Name Server: 나도 레드팀.com의 IP주소 정보가 없어 대신 .com TLD를 가지고 있으니 .com TLD 서버한테 물어봐!&#x20;

    <figure><img src="../obsidian_resources/Pasted image 20230525115224.png" alt=""><figcaption></figcaption></figure>

Root Name Server는 레드팀.com의 IP 주소를 알진 못하지만 .com TLD 네임 서버의 이름을 알고 있어 DNS Query를 다음 노드인 TLD 서버에 전달하게 된다. 이렇게 하면 DNS Resolving Name Server가 이 com TLD 서버에 레드팀.com의 IP 주소를 요청할 수 있다.

따라서, Root Name Server는 각 기존 TLD에 대한 최상위 도메인(TLD) 이름 서버의 이름을 미리 알아야 한다. Root Name Server는 IANA(Internet Assigned Numbers Authority)에서 감독하에 전 세계 13개가 존재한다.

전 세계 Root Name Server:&#x20;

{% embed url="https://root-servers.org/" %}

### 2. Top-Level Domain Name Server

TLD Server는 TLD에 따라 모든 도메인 정보를 제공한다. 예를 들어 도메인 레드팀.com을 등록하려면 com에 대한 TLD registry를 사용하여 등록해야 하며 ccTLD ( Counry-code TLD)라 부르는 국가 TLD느 국가 차원에서 관리를 하는 TLD로 분류된다. 대한민국의 경우 .kr가 ccTLD이며 KISA에서 관리한다.

다행스럽게도? 도메인 등록회사 ( Domai Registrant)에서 TLD를 선택할수 있기때문에 TLD registry에 직접 등록할 필요는 없다.

예를 들면 Namecheap, GoDaddy에서 공식 TLD registry의 대리 승인을 거쳐 다른 TLD의 도메인을 등록할 수 있다. (단 .kr과 같은 국가 코드는 예외로 여러 단계의 신분 확인을 거처야 한다).

com TLD Server는 레드팀.com의 IP 주소를 모르지만, 레드팀.com 도메인을 담당하는 Authoritative Name Server의 이름은 알고있다. 따라서 이 Authoritative Name Server의 이 이름을 DNS Resolving Name Server로 반환한다. 이렇게 하면 DNS Resolving Name Server가 해당 Authoritative Name Server를 찾아 레드팀.com의 IP 주소를 요구하게 된다.

아래 다이어그램의 (4) (5) 단계에 해당한다.

* (4) DNS Resolving Name Server: .com TLD Name Server! 레드팀.com IP주소 좀 알려줘 !
* (5) TLD Name Server: 잠시만.. 찾아볼께! ... 레드팀.com 도메인은 있는데 IP 주소 정보는 없어. 대신 레드팀.com이 사용하는 Authoritative Name Server는 알고 있어. 이 Authoritative Name Server가 레드팀.com IP를 알려 줄꺼야!

### &#x20;3. Authoritative Name Server&#x20;

이제 DNS Resolving Name Server는 TLD Name Server를 통해 레드팀.com의 IP 주소를 알고 있는 특정 Authoritative Name Server의 이름을 알아냈다. 위에서 설명했듯이 Authroitative Name Server는 도메인 이름을 IP 주소로 변환하는 권한을 가지고 있는 마지막 단계의 Name Server 이므로 레드팀.com의 IP 주소를 DNS Resolving Name Server에게 알려준다.

<figure><img src="../obsidian_resources/Pasted image 20230525115224.png" alt=""><figcaption></figcaption></figure>

아래 다이어그램에서의 (6) (7) 과정을 마지막으로 DNS Resolving Name Server는 유저에게 (8)레드팀.com의 IP를 최종적으로 반환한다.

* (6) DNS Resolving Name Server: Authoritative Name Server ! 레드팀.com IP주소 좀 알려줘 !
* (7) Authroitative Name Server: Roger! 레드팀.com IP는 A.B.C.D야

![](<../obsidian\_resources/Pasted image 20230525115224.png>)

그 외 (9) (10) 과정: 이제 유저가 도메인의 IP를 알아냈으니 이 IP의 호스팅된 여러 서비스 ( Web, Mail 등 )와 구동이 가능하다.

이로써 (1)에서 유저가 브라우저에 레드팀.com을 서버에 요청된 값이 DNS를 통해 (10)의 레드팀.com웹 페이지로 반환되는 모든 과정을 설명하였다.

## 그래서 DNS는 왜 중요한가? (Why?)

그래서 DNS는 왜 중요한가?에 대해 알아보자.

DNS는 사이버 보안 관점에서 굉장히 중요한 역할을 한다.

### 레드팀/공격자의 관점에서의 DNS:

1. DNS Cache Poisoning: 공격자는 DNS 캐시를 조작하여 악의적인 IP 주소를 도메인 이름에 반환하여 사용자가 악성 사이트로 리디렉션되거나 악성 행위를 수행할 수 있다.
2. DNS Spoofing: 공격자는 DNS 응답 (Response)을 위조하여 사용자를 속여 정상적인 사이트로 접속하는 것처럼 보이지만, 사실은 악성 사이트에 접속하게 유도할수 있다.
3. DNS 터널링 (DNS Tunneling): 악성 터널링은 DNS 프로토콜을 이용하여 네트워크 보안 제한을 우회하고 데이터를 전송하는 공격 기법이다.
4. DoS: DNS 서버를 과부하로 몰거나 구성 설정을 변경하여 DNS 서버의 기능을 마비시킬 수 있다.

그렇다면 방어자의 관점에서 DNS는 어떠한 역할을 할까? 어렵게 생각말고 위에 내용을 180도 바꿔 생각해보면 된다.

### 블루팀/방어자의 관점에서의 DNS:

1. DNS 악성 활동 탐지: DNS 로그는 악성 활동 탐지와 분석에 중요한 정보를 제공한다. 공격자의 악성 도메인 또는 C\&C(Command and Control) 서버와의 연결은 DNS를 통과하기 떄문에 DNS 로그를 분석하는 것은 탐지 활동에 있어서 중요한 역할은 한다.
2. DNS 웹 필터링 (검열) : DNS는 웹 필터링에 사용될 수 있다. 말 그래도 악성 웹 사이트나 공격 도메인등에 대한 접근을 DNS 차원에서 차단하거나 제어할 수 있기 때문에 보안과 안전을 강화할 수 있다. 또한 DNS 필터링을 거쳐 위조된 웹 사이트에 접속하는 것을 방지할 수 있다.
3. DNS 터널링 탐지: DNS는 악성 터널링 활동을 탐지하는 데 사용된다. 악성 터널링은 보안 제한을 우회하고 내부 네트워크에서 외부로 데이터를 전송하는 기술이기 때문에 DNS 로그를 분석하고 터널링을 탐지,차단 역할이 중요하다.
4. DDoS 대응: DNS 하면 빠지지 않는 DDoS(Distributed Denial of Service) 공격이 있다. 적절한 보호 대응을 구현하여 DNS 서버를 DDoS 공격으로부터 보호해야 한다.

## 마치며

이번 시간에는 DNS에 대한 개념을 알아봤다. 사이버 보인 입장에서 DNS를 모르는 것은 수학을 모르고 로켓을 만든 다거나,  글자를 모르고 책을 쓰는 것과 같다 할 수.있다. 또한훌륭한 해커?가 되고자 하여도 DNS와 기본 개념은 확실히 알아야 공격 활동에 유용하게 써먹을 수 있다.

다음 2편과 3편에서는 공격 기법과 DNS 필터를 우회하는 방법과 약간의 Internet Censorship ( 예, 중국의 DNS 사용법 a.k.a Great Firewall)에 대해서도 알아보겠다.

## 레퍼런스

{% embed url="https://aws.amazon.com/ko/route53/what-is-dns/" %}

{% embed url="https://attack.mitre.org/techniques/T1071/004/" %}

{% embed url="https://www.imperva.com/learn/application-security/dns-hijacking-redirection" %}

{% embed url="https://www.smartinsights.com/digital-marketing-strategy/online-value-proposition/start-with-why-creating-a-value-proposition-with-the-golden-circle-model" %}
