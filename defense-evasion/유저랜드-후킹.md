# 유저랜드 후킹

### 후킹 <a href="#ed-9b-84-ed-82-b9" id="ed-9b-84-ed-82-b9"></a>

유저 모드, 커널 모드, 그리고 둘을 이어주는 윈도우 API를 살펴봤으니, 이제 “유저랜드 후킹” 중에서 두번째 요소인 후킹에 대해서 알아본다. 후킹은 특정 시스템의 함수 콜 / 메시지 / 이벤트 등을 가로채 그 시스템의 행동을 바꾸는 기법들을 일컫는다. 개념만 따지고 보면 중간자 공격 (MitM - Man-in-the-Middle) 공격과도 비슷하다. 위에서 알아본대로 악성코드는 자신의 프로세스에 로드된 kernel32.dll 등의 라이브러리를 통해 윈도우 API를 사용한다. 따라서 프로세스에 로드된 kernel32.dll 의 윈도우 API를 후킹하면 악성 행위를 잡아낼 수 있을 것이다.\


![Kernel32.dll을 후킹한다면](https://lh4.googleusercontent.com/MpTo2aAe99Ev6trwUM6xe\_pHEwogCOCo1q7e7RrrUCN5y10m4edSVBm2piNaIT8YSHsF6RVV2lvo39aGo4At4Q6cyOA8IjwoXEfxXnFETAFcemmP5p0fsIS8sPoz1-aw7UKLXXM\_)

예를 들어, 최근 발견된 악성코드가 실행시 `c:\dev\malware.txt` 라는 파일을 생성한다고 가정해보자.

1. 생성되는 모든 프로세스의 kernel32.dll 라이브러리의 `CreateFileW()` 윈도우 API를 후킹한다
2. 악성코드가 kernel32.dll의 `CreateFileW()` 윈도우 API를 이용한다 `CreateFileW(“c:\dev\malware.txt”, …….)`
3. If 악성코드) Kernel32.dll 의 `CreateFileW()` 함수는 후킹되었기 실행시 방어자의 프로세스로 가로채진다. 방어자의 프로세스는 매개변수 중 `C:\dev\malware.txt` 를 발견한 뒤, 악성코드라고 판단, 타겟 프로세스를 종료시킨다
4. Else 정상 프로그램) 후킹 후 `CreateFileW()` 의 매개변수 중 이상한 점을 발견하지 못했다. 악성코드가 아니라고 판단, 다시 실행을 kernel32.dll 로 넘겨준다. 그 뒤 `CreateFileW()` 함수는 정상적으로 실행된다

후킹의 개념은 이해가 되는데, 정확히 어떤 방법을 써서 후킹을 할 수 있을까? 악성코드 프로세스에 로드된 kernel32.dll 의 소스코드를 바꿀 수 있는 것도 아니고, 이미 실행중인 프로세스에 로드된 라이브러리의 특정 함수를 어떻게 바꿀 수 있는 것일까? 다양한 방법이 있겠지만, EDR 솔루션들이 사용하는 후킹 방법중 하나는 DLL 인젝션을 이용한 인라인 후킹 (inline hooking)이다.

인라인 후킹은 타겟 함수의 주소를 찾은 뒤, 첫 n바이트의 어셈블리 명령어들을 JMP등의 명령어로 패치해 코드실행을 가로채간다. 예를 들어 이번에는 ntdll.dll 라이브러리에 있는 `NtProtectVirtualMemory` 윈도우 API 를 패치한다고 가정해보자. 해당 함수는 악성코드들이 쉘코드를 메모리에 작성/옮겨넣기 전, 메모리의 권한을 바꾸는데 자주 사용되는 함수다.

후킹 전 notepad.exe 프로세스에 로드된 ntdll.dll의 정상적인 `NtProtectVirtualMemory` 어셈블리는 다음과 같다.

```nasm
1:045> u ntdll!NtProtectVirtualMemory
ntdll!NtProtectVirtualMemory:
00007ffb`78bcd710 4c8bd1          mov     r10,rcx
00007ffb`78bcd713 b850000000      mov     eax,50h
00007ffb`78bcd718 f604250803fe7f01 test    byte ptr [SharedUserData+0x308 (00000000`7ffe0308)],1
00007ffb`78bcd720 7503            jne     ntdll!NtProtectVirtualMemory+0x15 (00007ffb`78bcd725)
00007ffb`78bcd722 0f05            syscall
00007ffb`78bcd724 c3              ret
```



몇가지 옵코드들을 실행한 후, 마지막에 syscall을 실행해 커널로 시스템 콜을 호출한다. 이 함수를 후킹한다고 가정하면 다음과 같이 변한다.

```nasm
1:045> u ntdll!NtProtectVirtualMemory
ntdll!NtProtectVirtualMemory:
00007ffb`78bcd710 <...>ff	  jmp	  <방어자_DLL의_후킹_함수_주소>
00007ffb`78bcd713 b850000000      mov     eax,50h
00007ffb`78bcd718 f604250803fe7f01 test    byte ptr [SharedUserData+0x308 (00000000`7ffe0308)],1
00007ffb`78bcd720 7503            jne     ntdll!NtProtectVirtualMemory+0x15 (00007ffb`78bcd725)
00007ffb`78bcd722 0f05            syscall
00007ffb`78bcd724 c3              ret
```

`NtProtectVirtualMemory` 함수의 가장 첫번째 명령어가 후킹되어 DLL 인젝션으로 인젝트된 방어자 DLL의 함수로 점프(JMP)한다. 이제 코드 실행이 방어자 DLL로 넘어갔으니, 방어자는 해당 함수의 매개변수 등을 살펴본 후, 이게 악성코드인지 아닌지에 따라 처리를 한 뒤, 다시 ntdll 함수로 코드 실행을 되돌려준다.

### 후킹 - 실습 <a href="#ed-9b-84-ed-82-b9-ec-8b-a4-ec-8a-b5" id="ed-9b-84-ed-82-b9-ec-8b-a4-ec-8a-b5"></a>

아까 위에서 “...후킹 방법중 하나는 DLL 인젝션을 이용한 인라인 후킹 (inline hooking)이다” 라고 했다. 게임 해킹, CTF, 악성코드 분석 등을 해보신 분들이라면 DLL 인젝션이 익숙하실 것이다. 타겟 프로그램의 코드 실행에 변화를 주기 위한 목적으로 DLL 인젝션이 사용되는데, 보안 프로그램이 악성 프로세스의 코드 실행을 탐지/후킹 하기 위한 목적으로도 사용된다. 이번 실습에서는 [@EthicalChaos](https://twitter.com/\_EthicalChaos\_) 가 만든 연습용 유저랜드 후킹 프로그램을 가지고 실습을 진행한다. 실습에는 Visual Studio, Windbg (Preview) 등이 사용된다.

1. 가상의 유저랜드 후킹 보안 프로그램 (SylantStrike - made by @EthicalChaos) - [https://github.com/CCob/SylantStrike](https://github.com/CCob/SylantStrike)\

2. 가상의 악성코드 (메시지 박스 출력) - [https://gist.github.com/ChoiSG/2e86452b01d95b1d4725938eeab2717a](https://gist.github.com/ChoiSG/2e86452b01d95b1d4725938eeab2717a)\

3. 비주얼 스튜디오와 Windbg - 구글링

실습 시나리오는 다음과 같다.

1. 가상의 보안 프로그램 SylantStrike 는 최신 EDR 솔루션으로, 유저랜드 후킹을 이용해 프로세스의 ntdll.dll의 `NtProtectVirtualMemory` 를 감시한다. 만약 프로세스가 `NtProtectVirtualMemory` 를 사용해 메모리의 특정 부분을 RWX (읽기/쓰기/실행하기)로 바꾼다면, 악성코드로 간주하고 프로세스를 종료시킨다.\

2. 가상의 악성코드는 메모장 프로세스를 생성한 뒤, 보안 탐지 우회를 위해 메모장에 프로세스 인젝션을 실행한다. 쉘코드는 메시지 박스를 출력한다 (대부분 쉘코드는 C2 에이전트 실행이지만, PoC니까).\

3. 각각의 파일을 빌드하고, 실행한 뒤, Windbg 를 통해 어떻게 유저랜드 후킹이 이뤄지는지 디버깅을 해본다.

먼저 SylantStrike 를 깃-클론 한 뒤, 전체 솔루션을 빌드한다.

별 탈 없이 빌드됐다면 SylantStrike.dll 과 SylantStrikeInject.exe 이 생길것이다. SylantStrike.dll 은 실제로 유저랜드 후킹이 일어나는 (가상의 보안) DLL이고, SylantStrikeInject.exe 는 위 DLL을 악성코드로 인젝트하는 프로그램이다.

다음으론 .NET Framework 4.0 Console 프로젝트를 만든뒤, 가상의 악성코드 hooktester 의 소스코드를 붙여넣고  빌드한다.

이제 가상의 악성코드를 실행하면 다음과 같은 결과가 나올 것이다.

가상의 악성코드가 제대로 실행된 모습

![메모장이 생성된 뒤, 쉘코드 (메시지 박스)가 출력된다.](https://lh5.googleusercontent.com/Qnt7s5K\_mZQS-ViZb\_s14aaId9naqkP-UvqwCyf3UUCmtt\_BZl0v5es1KjgLhO70L17Nfj74grCRZm4pN5ArSw-2LjVL3zbueYAmjptS2Z9AMaSbDic4\_tAEHm8mlklSRsLcLoZT)

이제 SylantStrikeInject.exe 를 이용해 보안 프로그램을 실행한다. 악성코드의 이름을 프로세스 이름으로 지정한 뒤, DLL 인젝션을 실행할 SylantStrike.dll 을 지정한다.

`.\SylantStrikeInject.exe --process=hooktester.exe --dll=C:\opt\SylantStrike\x64\Debug\SylantStrike.dll`

![](https://lh4.googleusercontent.com/dYTkmUl9bdZE1Vve22ZlBYresqyZaFgiwLX\_DVcsWrQ4XQRnNHsz9ahxoIsfDozq7XbqsolOjUSRpuKrHkZkj3-NXIOCBmFFiw6JwBwsw33c9BdGgiOmNmcKGmjSaU2lYvj6VxGv)

다시 악성코드를 실행하면 이번에는 SylantStrike 의 경고 메시지가 나오며 잡아냈다 이녀석이라고 얘기해준다. 악성코드 또한 실행을 끝마치지 못하고 중간에 종료된다.

실제로 Process Hacker 나 Process Explorer 등으로 확인해보면 SylantStrike.dll 이 인젝트 된 것을 알 수 있다.

![](https://lh4.googleusercontent.com/FJVdzLI2G31OJsaB6FoWMCMXU3Ovc3ZZIaqb1KbdYrpG-eEAGTZgFKh3TKeh47oXfMnIwADBHvznwpV\_fOexffU3ZbK-t3p9xgRWjEteTLFaXNrkNlm8gOcYR0QwpXa3IiQqut8z)

악성코드 프로세스에 인젝트된 SylantStrike.dll 모습

그렇다면 후킹은 어디서, 어떻게 일어나고 있을까? 이를 위해서는 Windbg 를 사용한다. 먼저, SylantStrikeInject.exe 를 종료시켜 탐지를 멈춘 뒤, 악성코드를 실행하되 프로세스 인젝션을 진행하기 전 멈춘다.

![](https://lh3.googleusercontent.com/JBQdxgVLkjUZbtmJ3hK0Rhvsybim5WXE9AbC4XRvY3Vh8ZRFusH5eVgBsk5RtX9uKWR-qc78eGyTlRgEle-DJdvsF-1VlIO9az3GHqx8u1cB76mM8VzyFf5Utnw9VLpVs7IMBEZE)

이 화면에서 멈추면 된다. 엔터를 안누르면 진행하지 않는다.

이제 Windbg 를 실행해 hooktester.exe 에 붙인 뒤 디버깅을 시작한다. 이 글에서는 Windbg Preview 를 사용하고 있지만, Windbg 을 사용해도 큰 차이는 없다. 디버깅을 할 프로세스의 이름을 검색한 뒤, 디버거를 붙여주자.

![](https://lh3.googleusercontent.com/HRE0Oht1WQnCnoYIoHJiMXeaoaZ01fWZQLQvOp\_5e\_f56j-ffb-2wonvA2bevBEwPWlMzfmf\_mvdVrLY07CYZyx8sg5CF5MEckAk55flavHA95A2BF58trtXz1WdkMZIHWkaOAEo)

지금은 SylantStrike 를 끈 상태이기 때문에 악성코드의 ntdll.dll 은 후킹이 되어있지 않다. 확인을 위해 ntdll.dll 의 `NtProtectVirtualMemory` 를 확인해보자.

![정상적인 NtProtectVirtualMemory 의 모습이다.](https://lh6.googleusercontent.com/keQ2Yyb1v7GT63VEQPJbLJOcslJIa0gZVQ3QFEyzt2wL1MxWtZY1lJ2ShFEz00oXs-sotrfmt0JVCZsrqKC26KRebixJz5JNrH67k4sotMpM4o7iUoN-8RpfedRYZrALst\_HHvHw)

이제 유저랜드 후킹이 실행된 이후를 알아본다. Windbg에서 디버깅 그만하기를 눌러 디버깅을 종료하고 악성코드를 종료시킨다. 그 뒤, 이번에는 SylantStrikeInject.exe 를 먼저 실행하고 악성코드를 실행한다.

다시 Windbg 로 돌아와 hooktester.exe 프로세스를 디버깅 해보자. 이번에 새롭게 시작된 악성코드 프로세스는 SylantStrike의 유저랜드 후킹이 적용된 상태다. 다시 한 번  악성코드의 ntdll.dll의 `NtProtectVirtualMemory` 를 살펴보자.

![유저 랜드 후킹이 적용된 악성코드의 ntdll!NtProtectVirtualMemory](https://lh5.googleusercontent.com/rGw1RMdzvn5TyofKqaNAE2tsWU09yHYQbfPNpwxV5ZmmMi-\_VZ07uS0gsBfZXaiTakVS2uuFcVrTeWoRa3F\_rVdXLt81w3Jgrt8HW21uP9mut7IyTG15-u\_HS6VRXJrjjZoywBCP)

악성코드에 로드된 ntdll.dll 의 `NtProtectVirtualMemory` 의 옵코드들이 변했다. 후킹이 된 것이다. 가장 눈에 띄는 것은 맨 처음 명령어 - `jmp 0x00007ff93de40fd6` - 해당 주소로 점프를 하는 명령어다. 해당 주소를 따라가보면 다음과 같이 또 다른 점프가 나온다.

![](https://lh4.googleusercontent.com/jxCWx-\_Ws7eeDC7VTU-xEDpDP8uakaTTjs3W\_uIQwW8KrOR3ZE2Ss-kiVJoKJreoYbA\_63pHN3aGcVdNYeeNLJYKIF-Der0TXvAVyEBrkCNc7yuiY4yoEprze7Z235BGnFI3K6nD)

0x00007ff93de40fd6 로 찾아가보면 `qword ptr [0x00007ff93de40fdc]` 를 통해 해당 주소에 있는 32 비트의 값으로 또 점프한다. 0x00007ff93de40fdc 주소에 어떤 값이 저장되어 있고, 그 값으로 점프를 하면 어디로 가는지 한 번 더 따라간다.

![](https://lh4.googleusercontent.com/KP68rGC8tO9oItBbtuVvbiwjG870GfcmikBuAmDnO544rbp3cg9jxzur2QOgsXyit43en6usKOudJO11OGGU8v9CzqVGlT7lUraiAwNQpXszjWdgTl8zByzeAfnoSlsUAib9saV8)

0x00007ff93de40fdc  주소로 가보니 0x00007ff92ef0115e 값이 나왔다. 바로 저 값으로 점프하면 된다. 저 주소에는 뭐가 있을까?

![](https://lh4.googleusercontent.com/h763k3bXlofUZQmIsKo71QwiMpVtumATL3hNPw5KG6ab8CgxiprnbRNrQgUMdH3KJYgvbFHZue5nqV77rQYRhAQV0SLpC1QeYT5YFcFZuxJ8WACWlXLp2poAzK8RyLF23diDEcAS)

SylantStrike DLL 파일로 점프를 하고 있다. 이후 SylantStrike DLL의 NtProtectVirtualMemory 로 코드가 진행된다. 즉, 유저랜드 후킹이 성공적으로 이뤄진 것이다.

정리해보자면,

1. ntdll!NtProtectVirtualMemory:  `jmp 0x00007ff93de40fd6`  // 후킹 성공!
2. 0x00007ff93de40fd6: `jmp qword ptr [0x00007ff93de40fd6]`(0x00007ff93de40fd6 주소에는 0x00007ff92ef0115e 값이 저장되어 있음)(사실상 jmp 0x00007ff92ef0115e 과 동일)
3. 0x00007ff92ef0115e: `jmp SylantStrike!NtProtectVirtualMemory` (SylantStrike 로 코드 진행)
4. `SylantStrike!NtProtectVirtualMemory` 코드 진행이 모두 끝난 뒤 다시 `ntdll!NtProtectVirtualMemory`의 코드를 마저 진행



이렇게 유저랜드 후킹의 개념에 대해서 알아봤다.&#x20;



### 레퍼런스&#x20;

{% embed url="https://blog.sunggwanchoi.com/kr-yujeo-raendeu-huking/" %}
