---
description: >-
  저번 SSH Port Forwarding 시간에서 배웠던 2가지 SSH 연결 방식과는 다른 차원의 개념일 수 있어 어려울 수도 있지만 끝까지
  따라와 주길 바란다.
---

# Dynamic Port Forwarding & ProxyChains

## 로컬 및 리모트 포트 포워딩 vs 다이나믹 포트 포워딩

로컬 및 리모트 포트 포워딩은 로컬 및 리모트 포트를 모두 정의해야 한다. 그래서 로컬이나 리모트 포워딩은 정해준 포트로 밖에 릴레이가 되지 않는다.

하지만 만약에 우리가 사전에 포트를 알 수 없거나 트래픽을 임의의 목표 대상으로 릴레이하려는 경우는 어떻게 해야 할까? 또 일상 생활에서 쓰고 있는 포트는 http, https, ftp, etc 등 다양하기 때문에 일일이 포트를 포워딩하기 어렵다.

이런한 문제를 해결해 주는 개념이 다이나믹 포트 포워딩 방식이다.

## 다이나믹 포트 포워딩

다이나믹 포트 포워딩은 동적 터널링 또는 SSH SOCKS5 프록시라고도 하며, 동적 포트 전달을 사용하여 들어오는 모든 트래픽을 리모트 서버로 동적으로 전달할 연결 포트를 지정할 수 있다. 솔직히 말이 좀 어렵다.&#x20;

일단 다이나믹 포트 포워딩이 쓰이는 이유는 다음과 같다:

1. 인터넷 검열 우회: Dynamic Port Forwarding은 인터넷 검열을 우회하는 데 사용될 수 있다. 예를 들어, 일부 국가에서는 인터넷 검열을 통해 일부 웹 사이트 및 서비스에 액세스가 제한된다. 이런 경우에 SSH를 사용하여 로컬 시스템의 인터넷 트래픽을 SSH 서버로 전달하면, 사용자는 지리적으로 제한된 콘텐츠에 접근할 수 있다.
2. 보안: Dynamic Port Forwarding은 인터넷을 통해 안전한 연결을 설정하기 때문에 중요한 데이터를 안전하게 전송할 수 있다. 예를 들어, 회사에서는 보안상의 이유로 외부 네트워크에서 내부 서버에 직접 액세스하지 못하도록 제한할 수 있다. 이런 경우에 SSH를 사용하여 로컬 시스템의 특정 포트를 SSH 서버를 통해 원격 서버에 노출할 수 있으며, 이를 통해 외부에서 안전하게 내부 서버에 접근할 수 있다.

다시 힌번 정의해 보자. 다이나믹 포트 포워딩은은 SSH 클라이언트를 SOCKS5 프록시 서버로 전환한다. SOCKS5에 대해선 지금은 깊게 알지 않아도 되니 그냥 그런 프록시 프로토콜을 사용한다 정도만 알고 넘어가자.&#x20;

예를 들어보자.

아래와 같은 경우 사용자의 인터넷 검열을 통해 일부 웹 사이트 및 서비스에 접근이 제한된다. 따라서,사용자는 security.grootboan.com을 열람하지 못한다.&#x20;

<div align="center">

<figure><img src="../.gitbook/assets/Pasted image 20230424114206.png" alt=""><figcaption></figcaption></figure>

</div>

이런 경우에 SSH를 사용하여 로컬 시스템의 인터넷 트래픽을 SSH 서버로 전달하면, 사용자는 제한된 콘텐츠에 접근할 수 있다. 다음 다이어그램으로 하나씩 설명을 해보자.&#x20;

<figure><img src="../.gitbook/assets/Pasted image 20230424115734.png" alt=""><figcaption></figcaption></figure>

첫번쨰로 방화벽 우회를 하고자 하는 타겟에서 아래의 커맨드를 통해 localhost의 포트 9090을 바인딩하고 이 포트로 전송되는 모든 트래픽을 REMOTE\_SERVER\_IP로 릴레이한다는 뜻이다.

```sh
ssh -D 127.0.0.1:9090 admin@REMOTE_SERVER_IP
```

![](<../.gitbook/assets/Pasted image 20230424122154.png>)

리모트 서버는 인터넷 접근에 제한이 없는 호스트면 된다. 이 실습에서는 저번과 같이 Digital Ocean에서 기본 Public VPS를 만들어 쓴다.

이제 타겟 머신은 "너가 9090 포트로 보내는 모든 연결은 REMOTE SERVER A.B.C.D한테 보낼꺼야!"를 인지했다.

두번째로, 우리의 목표는 방화벽을 넘어 "인터넷 브라우징하기"이기 때문에 중간 매개체인 "브라우저"를 통과해야 한다.이 실습에서는 Firefox가 쓰였지만 물론 FoxyProxy, 또는 웹 테스팅을 할때는 Burp를 직접 통하도록 설정할 수 도있다.

어쨋든 Firefox 설정으로 들어가 다음과 같이 Proxy를 설정해 준다.

![](<../.gitbook/assets/Pasted image 20230424120937.png>)

마지막으로, 이제 브라우저를 새로고침하여, MY IP를 알아보면 사용자의 실제 IP가 아닌 Remote Server VPS IP로 인식하는 것을 알 수 있다.

<div align="left">

<img src="../.gitbook/assets/Pasted image 20230424122254.png" alt="">

</div>

이로써 다이나믹 포트 포워딩을 이용하여 SOCKS Proxy 설정법에 대해서 다뤄봤다.

## ProxyChains 소개

다이나믹 포트 포워딩을 이용해 SOCKS 프록시을 설정하는 법을 배웠으니 실전에선 어떻게 쓰일지 생각해보자.

여기서 한가지만 알고 넘어가야 하는게 SOCKS는 OSI 모델의 레이어 5에서 작동하기 때문에 5 레벨 미만에서 작동하는 프로토콜은 터널링할 수 없다. 즉 ping이나 arp등의 낮은 레벨에서 사용되는 Utility는 SOCKS 프록시의 범주가 아니다.

여하튼 서론이 길었다. 진짜 예로 들어가 보자.

당신은 레드팀에서 내부 침투 ( Intial Access )에 성공, 내부 네트워크 침투에 성공했다. 이제 타겟 네트워크에서 Nmap 스캔을 수행하거나 Public 연결이 불가능한 내부의 여러 호스트를 스캔하려 하거나 여러 다른 기능을 수행하여야한다.

위에서 다뤘던 SOCKS 프록시는 하나의 예일 뿐이며 브라우저인 Firefox를 통해 포트 9090으로 트래픽을 보내도록 SSH를 설정한 거다. 하지만 만약 SMTP, FTP, RDP 등과 같은 다른 모든 TCP 트랙픽을 SOCKS 프록시를 통해 사용하는 방법은 없을까?

NMAP과 같은 대규모? 스캔을 때릴려면 모든 트래픽이 SOCKS 프록시를 거처야하기 때문에 수동으로 하나 하나 프록시를 거치게 설정하기 힘들다.

하지만 이렇게 모든 트래픽이 프록시를 거치게 도와주는 툴이 있다.

### Proxychains를 이용한 SOCKS 프록시

ProxyChains는 현 호스트에서 발생하는 모든 TCP 연결을 SOCKS4, SOCKS5 또는 HTTP 프록시와 같은 프록시를 통하도록 도와주는 오픈소스 툴이다.

따라서, 위의 시나리오와 같이 nmap을 돌린다 가정했을때 ProxyChains를 사용하면 방화벽을 우회해 인터넷에 접근하고, 공격자의 IP ​​주소를 숨기고, 프록시 서버를 통해 SSH/telnet/wget/FTP 및 Nmap과 같은 응용 프로그램을 실행하고, 외부 프록시를 통해 외부에서 로컬 인트라넷에 접근할 수도 있다.

또한 이를 통해 공격자는 탐지를 회피할 수 있는 강력한 무기를 갖게 된다. 여러 프록시가 함께 연결되면 포렌식 전문가가 트래픽을 원래 시스템으로 역추적하는 것이 점점 더 어려워진다.

**ProxyChains 설치**

ProxyChains를 설치하는 법은 간단하다.

{% embed url="https://github.com/haad/proxychains" %}

```sh
sudo apt-get install proxychains
```

윈도우 버전도 제공하니 참고 바란다.&#x20;

{% embed url="https://github.com/shunf4/proxychains-windows" %}

#### ProxyChains 설정 및 실행

ProxyChains4 환경설정 CONFIG 파일을 다음과 같이 설정한다.

```sh
vi /etc/proxychains4.conf
```

![](<../.gitbook/assets/Pasted image 20230424140932.png>)

원하는 포트에 대한 SSH 다이나믹 포워딩을 생성한 다음 이 포트를 proxychains.conf에 추가하면 끝이다.

![](<../.gitbook/assets/Pasted image 20230424122154.png>)

브라우저는 No Proxy Setting으로 돌려준다.

![](<../.gitbook/assets/Pasted image 20230424143815.png>)

이제 proxychains롤 통해 Firefox 브라우저를 실행하여 현재 공용 IP를 찾아보면 Remote Server IP로 뜨는것을 확인할 수 있다. 이로써 진짜 IP를 숨길  수 있다.&#x20;

![](<../.gitbook/assets/Pasted image 20230424141339.png>)

<div align="center">

<img src="../.gitbook/assets/Pasted image 20230424141407.png" alt="">

</div>

## 실제 공격 데모

실제 데모를 위해 Digital Ocean에 공격자와 타겟 박스를 각각 만들자.

* 공격자 실제 IP: A:B:C:D
* SOCKS 프록시 IP: 209.38.230.24
* 타겟 IP: 209.38.242.65

209.38.242.65을 스캔하기 위해 Proxychains를 통해 nmap을 실행한다.

![](<../.gitbook/assets/Pasted image 20230424142005.png>)

타겟 (209.38.242.65) 트랙픽 로그에는 실제 공격자의 IP가 아닌 프록시 IP 209.38.230.24가 보이는 것을 확인할 수 있다.&#x20;

<figure><img src="../.gitbook/assets/Pasted image 20230424142512.png" alt=""><figcaption></figcaption></figure>

#### 최종 우회 시나리오

<figure><img src="../.gitbook/assets/image (116).png" alt=""><figcaption></figcaption></figure>

## 마치며

다이나믹 포트 포워딩을 이용해 Proxy를 구축하는 것은 레드팀뿐 아니라 사이버보안에서 일하는 모든 사람이 숙달해야 하는 중요한 기술이다. 또한 ProxyChains를 통해 Proxy를 효율적으로 적극 활용해 수월한 레드팀을 하기를 바란다.  Happy Hacking!

## 레퍼런스

{% embed url="https://posts.specterops.io/offensive-security-guide-to-ssh-tunnels-and-proxies-b525cbd4d4c6" %}

{% embed url="https://medium.com/swlh/proxying-like-a-pro-cccdc177b081" %}

{% embed url="https://www.ired.team/offensive-security/lateral-movement/ssh-tunnelling-port-forwarding" %}



