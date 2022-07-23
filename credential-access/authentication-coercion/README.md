# 강제 인증 (Authentication Coercion)

Remote Procedure Call (RPC) 는 원격 서버에 있는 함수 (RPC call)을 클라이언트가 원격으로 실행시키는 프로세스간 통신 (Inter-process communication) 이다. 서버/클라이언트와 API라는 개념조차 희미 했던 1980년대 시절, 분산환경에서 원격 서버에 클라이언트가 접속해 뭔가를 하기 위해서는 이 RPC라는 것을 사용했다. 윈도우 액티브 디렉토리와 윈도우 운영체제에서는 아직도 이 RPC가 자주 쓰인다. &#x20;

워낙 오래전에 만들어진 기술이다 보니 윈도우 RPC 함수들 중에서는 사용자 인증이나 액세스 제한을 검증하지 않은 RPC들도 있다. 예를 들어 같은 네트워크 안에만 있다면 어떤 사용자 이름/비밀번호도 없이 서버의 RPC 함수를 자기 마음대로 실행할 수 있는 것이다.&#x20;

타겟 서버에서 실행되는 RPC 함수들 중 특정한 RPC 함수는 타겟 서버로 하여금 도메인 머신 계정의 이름과 Net-NTLM 해시화된 비밀번호를 이용해 RPC 함수를 실행시킨 서버 (공격자 서버)에게 인증을 하도록 한다. 이런 형태의 RPC 함수 실행을 "강제 인증 (Authentication Coercion)" 공격이라고 부른다. &#x20;

![](../../.gitbook/assets/print-spooler-rpc.drawio\(2\).png)

위는 RPC 기반의 강제 인증 공격 중 하나인 Printerbug/Print Spooler Attack 을 나타낸 다이어그램이다.&#x20;

1. 공격자는 사용자 인증이 필요없는 취약한 RPC 함수 중 하나인 `RpcRemoteFindFirstPrinterChangeNotificationEx` 를 서버에서 실행하고 있다. 이 함수는 타겟 서버에서 프린터와 관련된 변화가 일어날 시 클라이언트에게 알림을 자동으로 해주는 함수다.&#x20;
2. 서버가 클라이언트가 RPC 함수를 실행했다는 것을 인지한다.&#x20;
3. `RpcRemoteFindFirstPrinterChangeNotificationEx`  함수가 제대로 실행되려면 서버에서 클라이언트로 인증하고 접근 한 뒤, 클라이언트에게 알림 설정을 해줘야한다. 이 설정을 하기 위해 서버는 스스로 (하지만 사실상 클라이언트가 함수를 실행했으니 "강제"로) 공격자 클라이언트에게 서버 머신 계정 + Net-NTLMv1/v2 해시가 포함된 사용자 인증 패킷을 보낸다.&#x20;
4. 공격자는 이 머신 계정의 사용자 인증 패킷을 Responder 와 같은 툴로 받는다.&#x20;
5. 이후 이 머신 계정 이름 + Net-NTLMv1/v2 해시를  가지고 릴레이 공격, 다운그레이드 공격, RBCD 공격 등의 다양한 공격을 진행한다.&#x20;
