# 간단 디펜더 우회 - C\#

닷넷 어셈블리는 PE파일 포멧과는 다르게 윈도우 디펜더를 우회하는 것이 어렵다. 이는 닷넷 4.8 이후에 적용된 닷넷 AMSI 때문이기도 하고, 닷넷 어셈블리의 Common Intermediate Language (CIL) 형태가 디컴파일이 가능하기 때문이기도 하다.&#x20;

이 페이지에서는 윈도우 디펜더를 우회하기 위해 닷넷 어셈블리를 오픈소스 툴을 이용해 난독화하는 방법에 대해서 간단하게 알아본다. 방법론 자체를 알아보려면 Class, Namespace, Function, Parameter, Variable 맵핑 및 문자열 난독화에 대해서 자세히 적어야겠지만, 그러기에는 레드팀 플레이북이 아니라 레드팀 바이블을 만들어도 페이지가 부족할 것 같아 여기에 따로 적지 않는다.&#x20;

### 문제 및 시나리오&#x20;

Masky 라는 툴은 ADCS를 이용해 LSASS 덤프 없이 호스트 내의 세션을 유지하고 있는 유저들의 프로세스 토큰을 impersonate 하여 유저들의 ADCS 인증서 및 NT 해시를 얻게 해주는 C# 닷넷 기반의 툴이다. C# 툴들은 대부분 파워쉘에 포함시켜 실행하지만, Masky의 경우 ADCS 관련 라이브러리들이 파워쉘에서 실행 되지 않아 파워쉘 실행이 불가능하다. 따라서 어셈블리 난독화를 진행한 뒤 디펜더와 닷넷 AMSI를 모두 우회해본다.&#x20;

### 해결 방법&#x20;

난독화를 진행하기 위해 [Codeception](https://github.com/Accenture/Codecepticon/) 이라는 툴을 이용한다. Codeception은 닷넷의 Roselyn 라이브러리를 이용해 프로그래밍 방식(?, Programmatically)으로 C# 소스코드를 난독화한다. 소스코드 내 모든 Class, Namespace, Function, Parameter, Enums, Variables 등을 랜덤화된 문자열로 맵핑하고, 일반적인 문자열도 single-byte XOR, Base64, ROT13 등을 이용해 난독화 한다.&#x20;

물론 Codeception 말고도 [invisibilitycloak](https://github.com/h4wkst3r/InvisibilityCloak) 이나 [ConfuserEx](https://github.com/mkaring/ConfuserEx) 등의 다른 난독화 툴을 데이지체인 (Daisy-Chain) 형태로 묶어 이중, 삼중 난독화를 하는 방법도 있다. 하지만 이는 시간적 문제도 있고, 다양하게 디버깅을 해야할 수도 있기 때문에 이 페이지에서는 설명하지 않는다.  &#x20;

### 실습&#x20;

1. 난독화를 적용시킬 툴을 깃 클론 한다.&#x20;

```
git clone https://github.com/Z4kSec/Masky
```

2\. 난독화를 진행할 Codeception 툴을 깃 클론 한 뒤 컴파일 한다.&#x20;

```
git clone https://github.com/Accenture/Codecepticon.git
```

3\. Codeception 은 소스코드 자체를 바꿔버리기 때문에 #1 에서 다운 받은 툴을 새로운 디렉토리에 복사해 복사한 버전을 사용한다.&#x20;

```
cd .. 
mkdir obf-masky 
cp C:\opt\Masky\agent\* . -Recurse
```

4\. 문자열을 난독화 하더라도 디펜더나 AMSI 가 잡아내는 경우가 많기 때문에 본격적인 난독화 전 문제가 될 수 있는 문자열을 제거한다. 이는 [ThreatCheck](https://github.com/rasta-mouse/ThreatCheck) 등의 툴을 쓸 수도 있고, 수동적으로 툴 이름, 툴의 작성자 핸들 등의 문제가 될 법한 문자열을 감으로 찾아 제거할 수도 있다. (**수정**: 2022년도 기준으로 ThreatCheck 등의 툴을 이용하면 오히려 윈도우 디펜더가 머신 러닝등을 이용해 무조건적으로 악성코드로 탐지해내는 경우가 있다. 수동적으로 문자열들을 찾아 바꾼다.)

<figure><img src="../.gitbook/assets/image (18) (1).png" alt=""><figcaption><p>문제가 될만한 문자열의 Masky 를 choirtp 로 대체했다</p></figcaption></figure>

필요한 패키지 들을 설치/제거 해야한다면 지금 진행한다. Masky의 경우 Nuget 매니저로 가 모든 누겟 패키지를 언인스톨 한 뒤, Fody, Costura.Fody, Newtonsoft.Json 등을 다시 재설치 한다.

5\. Codeception 을 이용해 난독화를 진행한다. 클론된 디렉토리 안의 `CommandLineGenerator.html` 유틸리티를 이용해도 되고, 그냥 cli 에서 진행해도 된다.&#x20;

```
.\Codecepticon.exe --action obfuscate --module csharp --verbose --path "C:\opt\testoo\Masky.sln" --rename all --rename-method markov --markov-min-length 7 --markov-max-length 14 --markov-min-words 3 --markov-max-words 5 --string-rewrite --string-rewrite-method single

[ ... ] 
[00:13:28] Applying changes to solution...
[00:13:28] Running profile-specific final actions...
[00:13:28] Applying changes (again) to solution...
[00:13:28] Generating mapping file to: C:\opt\testoo\Masky.sln.html
[00:13:28] Obfuscation complete
```

6\. VisualStudio 등을 이용해 프로젝트 > Properties > `AssemblyInfo.cs` 로 가 `AssemblyTitle, Description, Company, Product` 등을 취향에 맞게 바꾼다. 이때 `AssmeblyVersion, AssemblyFileVersion` 은 최대한 간단한 버전을 지정한다.&#x20;

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

6\. 프로젝트 오른쪽 클릭 > Properties 에서 `Assembly name`, `Default namespace` 등을 바꿔준다. 이때 난독화된 namespace 값을 집어넣는다. Debug/Release, `x86/x64/AnyCPU` 등도 설정한다.&#x20;

<figure><img src="../.gitbook/assets/image (2) (3).png" alt=""><figcaption></figcaption></figure>

7\.  컴파일 한다.&#x20;

### 비교&#x20;

**난독화 되지 않은 Masky의 바이러스 토탈 결과**&#x20;

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

**난독화된 RTP-Moosky의 바이러스 토탈 결과**&#x20;

<figure><img src="../.gitbook/assets/image (1) (1) (2) (2).png" alt=""><figcaption><p><a href="https://www.virustotal.com/gui/file/8220c64fcc5c6477780dacc025cca686e2ec85c7d59f9730be37781e63607292?nocache=1">https://www.virustotal.com/gui/file/8220c64fcc5c6477780dacc025cca686e2ec85c7d59f9730be37781e63607292?nocache=1</a></p></figcaption></figure>

### 실행&#x20;

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

### 마치며&#x20;

닷넷 AMSI, 커널 콜백, 그리고 ETW의 등장으로 AV/EDR 솔루션들이 닷넷 악성코드를 탐지하고 방지하는 비율이 높아지고 있다. 이에 공격자들은 난독화와 CI/CD를 이용한 자동화된 난독화를 이용해 솔루션들을 우회하고 있다. 정적 탐지도 중요하지만, 결국엔 동적 탐지에서 이런 악성코드들을 탐지하고 방지하는 것이 더 중요할 것이다.&#x20;



### 레퍼런스&#x20;

{% embed url="https://github.com/Accenture/Codecepticon/" %}

{% embed url="https://github.com/Z4kSec/Masky" %}
