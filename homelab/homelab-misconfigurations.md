# 취약한 랩을 위한 설정 커맨드

### 개념

홈 랩을 만들어 내부망 관련된 모의해킹을 하려면 실무에서 마주칠 수 있는 잘못된 설정을 홈랩에 "심어야"한다. 대부분의 모의해커들은 공격할 줄은 알지만, 공격이 가능한 잘못된 설정이 어디서 왜 발생한 것인지는 모르는 경우가 많다. 다음은 AD 공격과 관련된 잘못된 설정을 하는 명령어들이다.

### 주의

다음의 설정들은 자신의 홈랩이 아니라면 절대로 적용해서는 안된다. 이에 따라 발생할 수 있는 피해와 관련해 레드팀 프로젝트는 책임을 지지 않는다.

### 잘못된 설정

LDAP Signing on/off

```
# GPO 
Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options
Domain Controller: LDAP Server Signing Requirements 
```

LDAP Channel Binding

```
# Registry 
Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Services\NTDS\Parameters" -Name "LdapEnforceChannelBinding" -Value 3
```

LDAP Anonymous Bind Enabled

```

# Powershell 
$directoryServicesDN = "CN=Directory Service,CN=Windows NT,CN=Services,CN=Configuration," + (Get-ADDomain).DistinguishedName

$currentValue = (Get-ADObject -Identity $directoryServicesDN -Properties "dSHeuristics").dSHeuristics
if ($currentValue -ne "0000002") {
    Set-ADObject -Identity $directoryServicesDN -Replace @{dSHeuristics="0000002"}
    Write-Output "dSHeuristics updated successfully."
} else {
    Write-Output "dSHeuristics is already set to the desired value."
}

$domainDN = (Get-ADDomain).DistinguishedName
& dsacls "CN=Users,$domainDN" /G 'ANONYMOUS LOGON:GR'

# --------- GUI -----------
# ADSI and dSHeurstics setting 
ADSI Edit > Connection Settings + Select A Well known Naming Context "Configuration > 
CN=Services > CN=Windos NT > CN=Directory Services > Properties > dSHeuristics set to 0000002 (seven zeros)

# Anonymous Logon READ permission on Users Container  
ADUC > Advanced > Users Container > Properties > Permissions > Add > Anonymous Logon > "READ" permission

# ------- DEBUG --------
crackmapexec ldap <Dc> -u '' -p '' --users
```

SMB signing off

```
set-smbserverconfiguration -requiresecuritysignature $False 
```

Defender off

```
set-mppreference -disablerealtimemonitoring $true -DisableScriptScanning $true -DisableIOAVProtection $true

Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender' -name "DisableAntiSpyware" -value 1 -Type DWORD
```

Windows update off

```
reg.exe add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update" /v AUOptions /t REG_DWORD /d 1 /f

reg.exe delete "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" /f

sc.exe config wuauserv start= disabled
net.exe stop wuauserv
```

Firewall off

```
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

Winrm

```
Enable-PSRemoting -Force
winrm set winrm/config/Client '@{AllowUnencrypted = "true"}'
Set-Item WSMan:localhost\client\trustedhosts -value * -Force
winrm set winrm/config/client/auth '@{Basic="true"}'
winrm set winrm/config/service/auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
```

Password Policy

```
Computer Configuration\Policies\Windows Settings\Security Settings\Account Policies\Password Policy

Computer Configuration\Policies\Windows Settings\Security Settings\Account Policies\Account Lockout Policy 
```

NTLMv1 - server

```
reg add HKLM\SYSTEM\CurrentControlSet\Control\Lsa\ /v lmcompatibilitylevel /t REG_DWORD /d 0 /f
```

Two-way Trust

```
1. Allow zone transfer on the DCs (to any server for more misconfiguration!). DNS -> Domain -> Properties -> Zone Transfer -> Allow to any. Do this for both forest's DCs. 

2. Create secondary zone for each DCs. DNS -> Forward Lookup Zones -> ?New zones -> secondary zone -> name blahblah -> type in each others' DC ip -> good! 

3. Create two-way trust! 
- I did selective trust, but more misconfiguration would be forest-wide trust. 
- For some reason, creating domain trust from domainA -> domainB worked, but domainB -> domainA resulted in "domain cannot be contacted". Even after dns restart, reboot, etc. ??? 
```

ShadowCoerce - FSRVP

```
File and Storage service -> File and iSCSI service -> File Server VSS Agent Service 
```

DFSCoerce - DFSNM

```
File and Storage service -> File and iSCSI service -> DFS Replication
```

ADCS Various ESC

```
# Create Cert Template 
- certtmpl.msc -> Duplicate User -> Subject Name -> Supply in request (required for ESC1,2,3) 
- Extensions -> Application Policies -> Edit this for EKU (client auth, any purpose) 
- Security -> Give write/enroll permission to Domain Users (various ESC)
- Change the name of the template 

# Publish Cert Template 
- Use Enterprise admin. Even domain admin doesn't work. 
- certsrv.msc -> Certificate Templates -> New -> the template that we created above 

-------


# ESC6 
- certutil -setreg policy\EditFlags +EDITF_ATTRIBUTESUBJECTALTNAME2
- This requires a reboot. 

--- 

# ESC8 
Install Web Enrollment and Web Service. 
Make sure to install CA first, separately. When CA is installed and configured, then install Web Enrollment and Web Service. 
```

LLMNR powershell scuffed version

```
while($true)
{
  ls \\doesntexist\share\update.ps1
  start-sleep -seconds 5
}

--- 

# Setting the GPO for batch job rights 

Running from a DC -> Assign the user a "Batch job rights" through GPO 

Either open domain policy (if not DC) or Default Domain Controller policy (if DC) 

Computer Configuration > Windows Settings > Security Settings > Local Policies > User Rights Assignment node > Log on as a batch job > specify the user that's running the task 

gpupdate.exe /force 
--- 

# Actually creating Task Scheduler 

New Task -> Run whether user is logged on or not -> Change User or group to whatever user to run 

Triggers -> At Startup 

Actions -> start a program -> powershell.exe (-execution bypass c:\scripts\check.ps1) 
```

DCSync - domain user

```
# Powerview ez clap 
Add-ObjectACL -PrincipalIdentity spotless -Rights DCSync

# Using Active Directory Administrative Center because no internet :sadge: 

domain right click -> properties -> scroll down -> security -> Add specific domain user or group -> Allow the following 
- Replicating Directory Changes 
- Replicating Directory Changes All 
- replicating Directory Chagnes in Filtered Set 

Sometimes ADAC might just crash, but the change goes through (wtf)
```

LSA configuration

```
# Optional (I think) 
Default Domain GPO -> Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > User Rights Assignment > Log on as a service > specify domain user here 

gpupdate.exe /force 

# Go to whatever host and just change print spooler because it's easy
services.msc > UPNP Device Host or Print Spooler > properties > logon > domain/user user + password 
restart > fails. doens't matter, creds are stored in plaintext in lsa secrets hive. 
```

Disable SMBv1 for no MS17-010 (shikata)

```
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force	
```

Shadow Credentials & GenericAll

```
# Giving permissions on user A to have genericAll rights on user B 
ADAC -> Domain -> Users -> user B -> Properties -> scroll down to Extensions -> Security -> Advanced -> user A -> check genericAll rights 

# Remember, add rights on user B that will ALLOW user A to genericAll. So edit user B. 
```

Netlogon share

```
c:\windows\sysvol\sysvol\<domain>\scripts
```

RBCD

```
# ty ired.team - spotless 
# pre-reqs
1. User has local admin rights on WS02 
2. User has WRITE privilege on target WS01 
3. WS02 (or other machines) vulnerable to auth coercion and has WebDAV installed 

seems like the target need to be a privileged box like DC for RBCD to be useful? 
```

LDAP Signing off

```
Default Domain Controller Policy > Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options > Domain Controller: LDAP server signing requirements -> None

Default Domain Controller Policy > Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options > Network security: LDAP client signing requirements -> None 

gpupdate.exe /force 
```

WebDAV

```
Features -> IIS -> WebDAV, basic auth, windows auth 

IIS -> Default Web site -> WebDAV authoring rules -> *, All Users, read+write+source, then "Enable WebDAV"

IIS -> Default Web Site -> Authentication -> Enable Anonymous, Basic, and Windows auth 

IIS -> Default Web site -> Advanced (WebDAV Settings) -> Allow Anonymous Property Queries "True" 

IIS -> Default WEb site -> WebDAV authoring rules -> Disable and Enable webdav (restart) 

Fingers crossed it works now 

# Having WebDAV on CA server with Web Enrollment may not work - need more testing. 
```

MachineAccountQuota - for shared lab environment

```
Get-ADObject -Identity ((Get-ADDomain).distinguishedname) -Properties ms-DS-MachineAccountQuota
Set-ADDomain -Identity <DomainName> -Replace @{"ms-DS-MachineAccountQuota"="500"}
```

Adding Users

```
# Loop this with random passwords from rockyou.txt 

New-ADUser -Name "username" -Accountpassword (Read-Host -AsSecureString "password") -Enabled $true -PasswordNeverExpires $true 
```

See my license

```
slmgr /dlv 
```

Rearm

```
slmgr.vbs -rearm 
```

Full clone machine Sysprep change SID

```
# Sometimes, fully cloned machines have duplicate SID, which prevents the machines from joining the domain. Use Sysprep to change that. 

c:\windows\system32\sysprep\sysprep.exe /oobe /generalize /reboot 

# change the hostname and the static ip address after the sysprep reboot 

# get machine sid for sanity check 
get-adcomputer computername -prop sid
```

Allow Anonymous access on shares

```
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa' -name "everyoneincludesanonymous" -value 1 -Type DWORD
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa' -name "restrictanonymous" -value 0 -Type DWORD
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa' -name "restrictanonymoussam" -value 0 -Type DWORD
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa' -name "LimitBlankPasswordUse" -value 0 -Type DWORD
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters' -name "requiresecuritysignature" -value 0 -Type DWORD
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters' -name "restrictnullsessaccess" -value 0 -Type DWORD
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters' -name "NullSessionPipes" -value "*" -Type String
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters' -name "NullSessionShares" -value "*" -Type String

# Right click on share -> Properties 
- "Security" > Add Everyone and add all permissions
- "Sharing" > Advanced Sharing > permissions > Add Everyone and all permissions

# Test it, or use cme 
net use \\servername\sharename "" /user:""
```

Guest/Anonymous access on shares ver.2&#x20;

```
# Allow anonymous on Windows 10
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters" -Name "AllowInsecureGuestAuth" -Value 1 -Type DWord

# Activate guest account
net user guest /active:yes

# Ensure directory structure for SMB share exists
New-Item -Path "C:\" -ItemType Directory -Force

# Add Everyone full control to the directory
$acl = Get-Acl -Path "C:\"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule("Everyone", "FullControl", "ContainerInherit, ObjectInherit", "None", "Allow")
$acl.SetAccessRule($accessRule)
Set-Acl -Path "C:\" -AclObject $acl

# Create SMB share
New-SmbShare -Name "C" -Path "C:\" -FullAccess "Everyone" -Description "Entier C Filesystem"
```

Remove History

```
echo "hi" >> (Get-PSReadlineOption).HistorySavePath
clear-history 
```

SCCM Server Installation&#x20;

```
References: 
- https://www.prajwaldesai.com/sccm-1902-install-guide-using-baseline-media/#Step-by-Step-SCCM-1902-Install-Guide
- https://www.prajwal.org/sccm-network-access-account/

The references are pretty straight forward. 
- Domain container, user, group configuration 
- Install SQL server on the server that SCCM will be also installed. (SQL2017, SCCM 2203 worked) 
- If SCCM complains about not finding the correct MSSQL instance, 
do NOT do "install primary standalone" option. Just go full manual and specify the SQL server 
FQDN and the instance name.
- ENSURE to install the "primary site". DO NOT install the configuration site.  
- Ensure current user has a SQL administrator priv - during SQL installation  
- Ensure SQL service is running through SQL Server Configuration manager. 
- Ensure SQL service has dynamic port <none> and tcp port 1433 via SQL server configuration manager
- If the default MSSQLSERVER instance got messed up during SQL installation, just install another SQL instance like SCCMSQL and use that during SCCM splash installation.
- gl! 

```

Remote Desktop Service for Windows Server machines&#x20;

```
Get-WindowsFeature -Name RDS-RD-Server | Select-Object -ExpandProperty Installed
Install-WindowsFeature -Name RDS-RD-Server -IncludeManagementTools
restart-computer -force 
(Get-WmiObject -class Win32_TSGeneralSetting -Namespace root\cimv2\terminalservices -Filter "TerminalName='RDP-tcp'").SetUserAuthenticationRequired(0)

# Scheduled Task to reset RDS 120 days trial 
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$folderPath = 'C:\scripts'
if (-not (Test-Path $folderPath)) { New-Item -Path $folderPath -ItemType Directory }
Invoke-WebRequest 'https://raw.githubusercontent.com/pbv7/reset-rds-grace-period/main/reset-rds-grace-period.ps1' -OutFile "$folderPath\reset-rds-grace-period.ps1"
$Action = New-ScheduledTaskAction -Execute 'PowerShell.exe' -Argument "-NoProfile -File '$folderPath\reset-rds-grace-period.ps1' -Force -RestartTS"
$Trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Monday -At 9am
Register-ScheduledTask -TaskName 'ResetRDSGracePeriod' -Action $Action -Trigger $Trigger -RunLevel Highest
```

RDPWrap for Win10/Win11 client machines&#x20;

```
https://woshub.com/how-to-allow-multiple-rdp-sessions-in-windows-10/

invoke-webrequest -uri "https://github.com/stascorp/rdpwrap/releases/download/v1.6.2/RDPWrap-v1.6.2.zip" -outfile "C:\rdpwrap.zip"
Expand-archive -path "C:\rdpwrap.zip" -destinationpath "C:\rdpwrap"
start-process cmd.exe "/c c:\rdpwrap\install.bat" -Wait -NoNewWindow
Stop-Service termservice -Force
Invoke-WebRequest https://raw.githubusercontent.com/sebaxakerhtc/rdpwrap.ini/master/rdpwrap.ini -outfile "C:\Program Files\RDP Wrapper\rdpwrap.ini"
restart-computer -force 
```

XP\_CMDSHELL vulnerable MSSQL server (kobitwave, hello - osint folks! -  check git history) &#x20;

<pre><code>1. Create cmd_shell proxy domain user account 

- domain.com\proxy_user:Password123!

2. Enable xp_cmdshell by default, to all users 

USE master;
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
GRANT EXEC ON xp_cmdshell TO PUBLIC;

3. Configure proxy user 

USE master;
-- Create the credential for the proxy account
CREATE CREDENTIAL [##xp_cmdshell_proxy_account##]
WITH IDENTITY = 'domain.com\user', 
SECRET = 'pass';
-- Set the xp_cmdshell proxy account
EXEC sp_xp_cmdshell_proxy_account 'domain.com\proxy_user', 'pass';

4. Test
<strong>└─# mssqlclient.py doamin.com/user:pass@1.1.1.1 -windows-auth
</strong>xp_cmdshell whoami
</code></pre>



