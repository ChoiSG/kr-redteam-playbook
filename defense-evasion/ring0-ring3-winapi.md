# 유저랜드 커널랜드 윈도우API 개념

### 유저랜드, 커널랜드, 그리고 윈도우 API  <a href="#ec-9c-a0-ec-a0-80-eb-9e-9c-eb-93-9c-ec-bb-a4-eb-84-90-eb-9e-9c-eb-93-9c-ea-b7-b8-eb-a6-ac-ea-b3-a0-e" id="ec-9c-a0-ec-a0-80-eb-9e-9c-eb-93-9c-ec-bb-a4-eb-84-90-eb-9e-9c-eb-93-9c-ea-b7-b8-eb-a6-ac-ea-b3-a0-e"></a>

![윈도우의 Ring0, Ring3](https://lh3.googleusercontent.com/aoZbR6f\_PvI\_6cVt7ZM1iViFQz0k9DBiFZHsypRzr\_11vxiTE6T9iDwIQ41tbJg8xN\_aPfof32y\_NmoO0H-1-YeERAjCXq\_rSEUEZZuaVJkeTP8oCfrUPLlRDG9sAP7bVbdAKiT9)

윈도우 운영체제는 크게 유저 모드 (유저 랜드)와 커널 모드 (커널 랜드)로 나뉘어져있다. 이를 링3, 링0 라고도 부른다. 유저 모드에서는 프로세스가 실행되며, 커널 모드에서는 하드웨어와 소프트웨어를 이어주는 커널이 존재한다. 커널은 운영체제의 핵심이기 때문에 잘못된 접근을 할 경우 머신 전체가 장악당하거나 잘못될 위험(크래시, 블루 스크린)이 있다. 이 때문에 링0과 링3은 분리 되어있다.

하지만 유저 모드가 커널에 접근해야하는 경우도 생기기 마련이다. 예를 들어 유저 모드의 프로세스인 메모장을 이용하다가 파일 저장을 하고 싶으면, 커널을 이용해 하드웨어인 디스크에 파일을 저장해야한다.

예를 들어, 다음과 같은 파이썬 코드를 실행한다고 가정해보자.

```python
input("Setup your procmon/API Monitor or whatever. When you are ready, press enter... ")

fd = open("test.txt", "w")
fd.write("I am python, creating and writing to a file using python's <open/write> function. But under the hood...")
fd.close()
```

이 파이썬 코드는 유저 모드에 존재하지만, 파일 시스템을 통해 하드웨어에 test.txt 라는 파일을 생성해야 한다. 이때 유저 모드와 커널 모드를 이어주는 것이 바로 윈도우 API (winapi, win32api, windows API)다. 윈도우 API는 웹개발을 할 때 자주 보던 API와 비슷한 개념이다. 유저 모드의 프로그램이 커널과 소통을 해야할 경우가 생길 때 윈도우 API를 이용하면 정해진 규칙과 규정대로 커널에 접근할 수 있다.

윈도우 API 함수들은 kernel32.dll, kernelbase.dll, ntdll.dll 과 같은 다양한 라이브러리 파일들의 export table 에 존재한다. 이 중 kernel32.dll, kernelbase.dll, ntdll.dll 과 같은 라이브러리들은 프로세스 실행시 프로세스의 메모리로 기본적으로 로드된다.

실제로 procmon 과 같은 프로그램으로 위 파이썬 코드를 실행하면 다음과 같은 화면이 나온다.

![CreateFile과 WriteFile](https://lh5.googleusercontent.com/LvNFHklcQHzBd7AOMFf-Mg9pgFZMgxDVOXG5\_85dhak21qsMuhm-1T1ibx9e1Bbc8m9iGfAJOfvZNp4Pfly2LO5lbQ\_kA2eOnH9SqSKYakBwaX8Im-k3SAxc4pQ9S4OIdt9vCSEK)

![CreateFile: kernelbase -> ntdll -> ntoskrnl](https://lh4.googleusercontent.com/xsYXtKNTFHhWx\_-GjpWNQzehcghKmeELRpnjpvMkwup3NerKVwtluKp6WE0E\_YSD7VxeSUkc\_\_h2Mm20jnLkm-2iQwQQcF4vStrTMxp9Y2BDHMiShYz5jGsxZjmv07ybXOMZFA6W)

![WriteFile: CreateFile: kernelbase -> ntdll -> ntoskrnl](https://lh6.googleusercontent.com/r-MufJDtjBdMqh8\_-pChjUfEkj4YfHg0nMALQoKHekHTsTgz9DLYHivVkrQx0AxFgX8ARHTQsVh3hhaBwS1wtdudLKMrX3YiJo9VEVRw33lYjvBcVO62DHG898cpbXw7eUmRxdiH)

* CreateFile = 파이썬의 “open(‘test.txt’,’w’)” → CreateFileW (kernelbase.dll) → NtCreateFile (ntdll.dll) → NtCreateFile (ntoskrnl.exe),
* WriteFile = 파이썬의 “fd.write” → WriteFile (kernelbase.dll) → ZwWriteFile (ntdll.dll) → NtWriteFile (ntoskrnl.exe)

위 순서로 함수들이 실행되고 있다. 윈도우 API 함수를 실행하면 kernelbase.dll 의 함수 호출 → ntdll.dll 함수 호출 → ntoskrnl.exe 커널에서 시스템 콜 실행과 같은 방법으로 실행되고 있다. 마지막으로, 똑같은 CreateFileW가 아니라 ntdll.dll과 ntoskrnl.exe 에서는 앞에 “Nt” 접두어가 붙은 네이티브 API (Native API) 가 실행된다.\


![파이썬, WinAPI, NativeAPI, Syscall](https://lh6.googleusercontent.com/\_fELCKQxeGypfaKrYtabIBnBaDO\_qmBPN2R1WpB-335RyV\_9\_cwhLFAAIaxetKfRLGn5IpzP7n7neKFmghyu0xOVrcLC0nc1ENQVn2\_8ViUNTK93FbaGhEQDVUEixr6miLfdhf0c)

사용되는 라이브러리들을 위에서 아래로 (유저모드 → 커널모드)로 정리해보면 다음과 같다.

1. Kernel32.dll = 메모리 I/O 관련 윈도우 API를 가지고 있다. 단, “최근” (윈도우7/서버2008) 윈도우 버전에서는 kernelbase.dll 로 코드 실행을 넘기는 역할을 담당한다.\

2. Kernelbase.dll = “최근” 윈도우 버전부터 kernel32.dll로부터 인수인계를 받고 (?) 실질적으로 관련 윈도우 API 를 가지고 있다. 단, 커널베이스 또한 실질적으로는 ntdll.dll 의 네이티브 API를 실행하는 역할 담당이다.\

3. Ntdll.dll = 윈도우 API보다 한단계 낮은 네이티브 API를 가지고 있다. 사실 윈도우 API는 네이티브 API의 래퍼 (wrapper) 일뿐이다. 이 네이티브 API는 공식적으로 문서화가 되어있지 않고, 윈도우 버전이 바뀔때마다 바뀌는 경우가 많아 이를 추상화 시키기 위해 윈도우 API가 사용된다. Ntdll.dll의 네이티브 API를 통해 최종적으로 시스템 콜 CPU 명령어가 어셈블리 단계에서 실행된다.\

4. Ntoskrnl.exe = 이때부터 코드 실행은 커널로 넘어가 System Service Dispatch Table (SSDT) 를 이용해 시스템 콜 번호를 찾아 실행한다.

평범한 프로그램이나 악성코드나 윈도우와 소통을 하기 위해서는 윈도우 API를 사용할 수 밖에 없다. 따라서 악성코드가 윈도우 API를 사용해 나쁜 일을 한다면, 어떤 윈도우 API 들이 사용되는지 모니터링을 하면 탐지가 가능할 것이다. 하지만 어떤 프로세스가 , 어떤 윈도우 API를, 어떻게 사용하는지 방어자의 입장에서 어떻게 알 수 있을까? 후킹을 통해 알 수 있다.
