# 파워쉘

### 파워쉘이란&#x20;

"파워쉘은 명령줄 쉘, 스크립팅 언어, 그리고 구성 관리 프레임워크로 구성된 플랫폼 간 작업 자동화 솔루션이다" - MSDN. 가끔 파워쉘 == `powershell.exe` 혹은 파워쉘은 스크립팅 언어다 하는 오해가 있지만, 사실 그 모든 것을 합한 것이 파워쉘이다. 또한, 파워쉘의 근본은 바로 `System.Management.Automation.dll` 이라는 DLL이다. `powershell.exe` 는 그 DLL을 감싸는 wrapper 일 뿐이다.&#x20;

### Why Powershell?&#x20;

파워쉘은 .NET Framework (닷넷 프레임워크) 기반으로 개발되었기 때문에 닷넷 기반의 런타임, 다양한 프로그래밍 언어들과의 소통, 라이브러리, 그리고 개발자 툴을 모두 사용할 수 있는 솔루션이다. 때문에 시스어드민들과 개발자들은 파워쉘을 이용해 데스크탑 어플리케이션, 웹, 클라우드, 모바일, IoT, AI, 레지스트리, COM, WMI, 오피스 솔루션, XML/CSV/JSON, Hyper-V 들과 소통을 할 수 있다.&#x20;

그리고 그것은 곧 공격자들 또한 파워쉘을 이용해 저 모든 것들과 소통할 수 있음을 의미한다.&#x20;

### 공격자의 관점에서의 파워쉘&#x20;

2010년 데프콘에서 David Kennedy 와 Josh Kelly가 발표한 "[Powershell... omfg](https://youtu.be/q5pA49C7QJg)" 토크 이후 파워쉘에 관련된 공격자들의 관심이 크게 증가했다. 공격자들에게 있어서 파워쉘의 매력은 다음과 같다:&#x20;

* 강력함 - 닷넷 프레임워크와 윈도우API 와의 소통, Living off the Land 를 통한 횡적이동&#x20;
* 접근성 - Win7 SPI1, Win2008 R2 SPI1 이후로 모든 워크스테이션과 서버 OS가 자동적으로 설치되어 있고 활성화 되어 있음&#x20;
* 개발 난이도 - 스크립팅 언어이자 매니지드 (managed) 언어이기 때문에 개발에 용이함&#x20;
* 인메모리 실행 - "Fileless Attack" 이라고도 불렸던, 2010년대 초 당시에는 획기적이였던 공격 방식&#x20;
* 블루팀의 약한 인지도&#x20;
  * 시스어드민들 조차 2010년대 초반까지 대중적으로 사용하지 않았음&#x20;
  * 부족한 문서화, OOP 프로그래밍의 입문 난이도 - 파워쉘이 정식 출시된 2006년도 당시 시스어드민들의 프로그래밍 관련된 지식 부족&#x20;
  * 기본적인 파워쉘 로깅 (logging)의 부재 및 당시 AV들의 부족함&#x20;

이 모든 것들이 어우러져 공격자들은 빠른 속도로 당시 PE바이너리 기반이였던 자신들의 툴링을 모두 파워쉘로 변환하기 시작했다. 이는 곧 2010년 \~ 2017년동안 이뤄진 공격자들의 파워쉘 전성기를 열게 된다.&#x20;

### 파워쉘 툴링&#x20;

2012년도 Matt Graeber의 "Why I choose Powershell as an attack platform" 블로그를 기반으로 공격자들의 파워쉘 툴링 시대가 시작됐다.&#x20;

<table><thead><tr><th>종류</th><th>프로젝트</th><th data-hidden></th></tr></thead><tbody><tr><td>C2 프레임워크 </td><td>Powershell Empire, PoshC2, FudgeC2</td><td></td></tr><tr><td>정보 수집</td><td>PowerView, Invoke-PortScan, Powershell-AD-Recon, PSRecon</td><td></td></tr><tr><td>계정 정보 탈취</td><td>Invoke-Mimikatz, Get-Keystrokes, Get-MicrophoneAudio</td><td></td></tr><tr><td>코드 실행</td><td>Invoke-DLLInjection, Invoke-Shellcode, PowerPick </td><td></td></tr><tr><td>Reflective PE Injection</td><td>Invoke-ReflectivePEInjection</td><td></td></tr><tr><td>파워쉘 난독화</td><td>Invoke-Obfuscation, Chameleon </td><td></td></tr><tr><td>Post-Exploitation</td><td>PowerSpoit, PowerTools, Nishang, Powershell Suite </td><td></td></tr></tbody></table>

### 파워쉘 보안&#x20;

공격자들의 파워쉘에 대한 관심도가 증가하면서 마이크로소프트사에서는 파워쉘 버전 5.0과 5.1에서 드디어 보안과 관련된 기능들을 업데이트 하기 시작했다.&#x20;

* Powershell Language Mode - Constrained Lanaguge Mode 로 설정시 .NET 어셈블리 로드, WinAPI, 공격적 cmdlet 금지&#x20;
* Transcript Logging - Over-the-Shoulder Transcription, 메타데이터 모두 로깅&#x20;
* Deep Script Block Logging - 파워쉘 난독화 무력화 -> 메모리상에서 난독화가 풀린 파워쉘 코드를 로깅. 공격용으로 쓰이는 120개의 중요 함수/키워드 금지&#x20;
* AMSI (Anti-Malware Scan Interface) - 윈도우 어플리케이션과 AV/EDR간의 API 제공. 악성 파워쉘 메모리상에서 탐지 후 제거&#x20;

### 파워쉘 근황&#x20;

2022년도 기준으로 몇 몇 APT 그룹들, 랜섬웨어 갱들, 그리고 내부망 모의해커들은 여전히 파워쉘을 많이 사용하고 있는 중이다. 하지만 실력 좋은 (정부 배후/후원) APT 그룹들과 업계 내 레드팀들은 .NET, nimlang, golang, BOF, COFF 등의 트레이드 크래프트 (tradecraft)로 넘어갔다.&#x20;

2010년 \~ 2017년 동안 이뤄진 공격자들의 파워쉘 전성기는 끝난지 5년이나 되었지만, 여전히 파워쉘은 공격에 꽤나 많이 쓰이고 있다.&#x20;
