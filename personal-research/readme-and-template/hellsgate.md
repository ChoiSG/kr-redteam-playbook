# HellsGate

## Disclaimer&#x20;

이곳은 제 개인 공부 + 연구 섹션입니다. 오펜시브 시큐리티 관련 블로그 글을 읽고, 요약한 뒤, 거기에 있는 코드들 따라치기 정도의 낮은 퀄리티 글 밖에 없을거기 때문에 안 읽으셔도 무방합니다. 이 섹션의 모든 페이지들의 내용 및 코드는 제것이 아닙니다.&#x20;

페이퍼 링크: [https://vxug.fakedoma.in/papers/VXUG/Exclusive/HellsGate.pdf](https://vxug.fakedoma.in/papers/VXUG/Exclusive/HellsGate.pdf)

툴/코드 링크: [https://github.com/am0nsec/HellsGate](https://github.com/am0nsec/HellsGate)

### 문제

* AV/EDR의 유저랜드 후킹을 우회하기 위해 시스템 콜을 이용하는데, 이를 위해서는 각 NTAPI 함수들의 시스템 콜 번호를 알아야한다. Hell's Gate 발표  전까지 이 번호들을 런타임 중 프로그래매틱 하게 찾아낼 수 있는 방법이 없었다.
* 따라서 레드팀들은 모든 윈도우 버전의 모든 시스템 콜 번호들을 스태틱하게 구조체로 만들어 하드코딩 + 수많은 조건문을 이용해 시스템 콜을 사용하고 있었다. 물론, 이는 매우 불안정하고 비효율적인 방법이였다.

### 해결

* HellsGate (헬즈 게이트)는 NTAPI 함수들의 시스템 콜 번호를 런타임 도중 프로그래매틱하게 찾아주는 기법을 문서화 + POC화 시킨 연구다. 헬즈 게이트 덕분에 추후 수많은 XYZ 게이트 기법들이 생겨나며 시스템 콜 시대가 시작된다.

### 중요 개념 - 프로그래매틱하게 시스템 콜 번호 찾기

* HellsGate의 중요 개념은 ntdll의 export된 NTAPI 함수의 시스템 콜 관련 바이트 패턴(mov r10, rcx // mov eax, \<syscall-number>)의 시작지점으로부터 index로 5번째 (0부터 시작 기준, 실제로는 6번째 바이트) 바이트를 bitwise shift left 8 연산을 한 뒤, index로 4번째 (0부터 시작 기준, 실제로는 5번째 바이트) 바이트를 OR 연산을 하면 시스템 콜 번호를 프로그래매틱하게 구할 수 있다는 것이다.

페이퍼의 예시를 보자면, NtPlugPlayControl 함수의 시작 지점으로 부터의 명령어들은 `4c 8b d1 b8 32 01 00 00` 이다. 각각의 명령어들은 다음과  같이 풀이된다.

```
0:000> uf ntdll!NtPlugPlayControl  
ntdll!NtPlugPlayControl:  
00007fff`b040d3b0 4c8bd1 mov r10,rcx       // 인덱스 기준 0,1,2     - 4c 8b d1
00007fff`b040d3b3 b832010000 mov eax,132h  // 인덱스 기준 3,4,5,6,7 - b8 32 01 00 00
```

이때, 0부터 시작하는 인덱스 기준으로 5번째 `0x01` 바이트를 bitwise shift left 8 연산 후, 인덱스로 4번째 바이트 `0x32` 로 OR 연산을 하면 다음과 같이 된다.

<pre><code>// 01 == 5번째 바이트, 32 == 4번째 바이트 
(0x01 &#x3C;&#x3C; 8) | 0x32 = 132h

<strong>// 풀이
</strong>0x01 &#x3C;&#x3C; 8 == 0x0100 
0x0100 | 0x32 == 0x132h 
</code></pre>

페이퍼에서도 Windbg를 통한 풀이를 다음과 같이 보여준다.

```
0:000> db (ntdll!NtPlugPlayControl + 0x4) L2
00007fff`b040d3b4 32 01 2.    // 32 == 4번째 바이트, 01 == 5번째 바이트 
0:000> ? (0x01 << 8) | 0x32
Evaluate expression: 306 = 00000000`00000132
```

### 코드 - 시스템 콜 번호 찾기 - 개념&#x20;

위 개념/공식은 Hell's Gate 의 공식 깃헙 리포 `main.c` 의 `GetVxTableEntry` 함수에 적용되어 있다.

```
// main.c, line: 143~156 https://github.com/am0nsec/HellsGate/blob/master/HellsGate/main.c#L143-L156

// First opcodes should be :
//    MOV R10, RCX
//    MOV RCX, <syscall>
if (*((PBYTE)pFunctionAddress + cw) == 0x4c
	&& *((PBYTE)pFunctionAddress + 1 + cw) == 0x8b
	&& *((PBYTE)pFunctionAddress + 2 + cw) == 0xd1
	&& *((PBYTE)pFunctionAddress + 3 + cw) == 0xb8
	&& *((PBYTE)pFunctionAddress + 6 + cw) == 0x00
	&& *((PBYTE)pFunctionAddress + 7 + cw) == 0x00) {
	BYTE high = *((PBYTE)pFunctionAddress + 5 + cw);
	BYTE low = *((PBYTE)pFunctionAddress + 4 + cw);
	pVxTableEntry->wSystemCall = (high << 8) | low;
	break;
}
```

먼저, 제대로 시스템 콜 번호 관련 명령어가 있는 부분까지 왔는지 조건문을 통해 확인한다. `pFunctionAddress`의 0, 1, 2, 3 번째 오프셋이 `4c 8b d1 b8` 인지 알아본다. 이는 `move r10, rcx, mov` 까지의 어셈 코드를 나타낸다. 그 뒤, 6번째와 7번째 오프셋이 `00 00` 인지도 확인하고 있다. 6번째와 7번째 바이트를 확인하는 이유는 `mov eax, <syscall>h` 가 실행될 경우 인덱스로 6번째와 7번째 오프셋의 바이트들은 무조건 `00 00` 이 되기 때문이다.

0,1,2,3,6,7 번의 바이트를 모두 확인했고, 조건문을 통과했다면, 우리가 찾고 있던 시스템 콜 관련 명령어  "패턴" 을 찾은 것이다.

```
// 시스템 콜 관련 명령어 "패턴" 
mov r10, rcx
mov rcx, <syscall>
```

이제 남은 4번째, 5번째 바이트와 Hell's Gate의 시스템 콜 번호 찾기 공식을 이용해 시스템 콜 번호를 찾는다. 앞서 얘기한대로, 인덱스(NT함수의 시작지점)로부터 5번째 바이트는 bitwise shift left 8 연산을, 그 뒤에 4번째 바이트는 OR 연산을 실행한다. 그 뒤, 찾아낸 시스템 콜 번호를 `pVxTableEntry->wSystemCall` 에다가 저장한다.

```
BYTE high = *((PBYTE)pFunctionAddress + 5 + cw);
BYTE low = *((PBYTE)pFunctionAddress + 4 + cw);
pVxTableEntry->wSystemCall = (high << 8) | low;
```

### 적용

* 시스템 콜 번호는 찾았다. 근데 이건 어떻게 사용되는걸까?
* Hell's Gate 가 NT 함수들의 메모리 주소, 함수 해시, 시스템 콜 번호를 편리하게 저장하기 위한 `_VX_TABLE_ENTRY` 와 `_VX_TABLE` 에 관련된 설명은 생략한다.
* 일단 VX\_TABLE 및 엔트리들을 최대한 설정해준 뒤, `GetVxTableEntry` 함수를 이용해 위에서 설명한 개념을 이용해 시스템 콜을 찾는다.
* 그 뒤, HellsGate 과 HellDescent 어셈블리 코드들을 이용해 실제로 시스템 콜을 실행한다.

```
.data
	wSystemCall DWORD 000h

.code 
	HellsGate PROC
		mov wSystemCall, 000h
		mov wSystemCall, ecx
		ret
	HellsGate ENDP

	HellDescent PROC
		mov r10, rcx
		mov eax, wSystemCall

		syscall
		ret
	HellDescent ENDP
end
```

HellsGate 는 `GetVxTableEntry` 함수에서 찾아낸 시스템 콜을 `wSystemCall` 이라는 변수에 저장한다. HellDescent 는 실제 NT 함수들처럼 `mov r10, rcx // mov eax, <systemcall>h // syscall` 를 통해 eax 레지스터에 시스템 콜 번호를 올려놓은 뒤, syscall 명령어를 실행해 CPU에게 커널 모드로 진입해 해당 syscall 을 실행시킨다.

### 예시

페이퍼에서 나온 간단한 예시다. 예를 들어, `NtAllocateVirtualMemory` 의 시스템 콜 번호를 알아낸 뒤, 이를 직접 어셈블리 코드를 이용해 사용해보자.

```
// VX_TABLE 의 Entry 중 하나인 NtAllocateVirtualMemory의 시스템 콜을 GetVxTableEntry 함수를 이용해 찾아냄. 
VX_TABLE Table = { 0 };
Table.NtAllocateVirtualMemory.dwHash = 0xf5bd373480a6b89b;
if (!GetVxTableEntry(pLdrDataEntry->DllBase, pImageExportDirectory, &Table.NtAllocateVirtualMemory))
	return 0x1;

// NtAllocateVirtualMemory의 시스템 콜 번호를 HellsGate + HellDescent 콤보를 이용해 실행 
HellsGate(pVxTable->NtAllocateVirtualMemory.wSystemCall);
status = HellDescent((HANDLE)-1, &lpAddress, 0, &sDataSize, MEM_COMMIT, PAGE_READWRITE);
```

### 마치며

헬즈 게이트는 2018\~2020년 유저랜드 후킹 및 시스템 콜 관련된 포스트들이 많이 나올때 프로그래매틱하게 시스템 콜 번호를 찾아내는 방법을 깔끔한 문서화 + POC 코드까지 제공한 전설적인(?) 연구다.

헬즈 게이트 이후 수많은 XYZ게이트 기법들이 쏟아져 나오기도 했다. 추후 나온 기법들에 비하면 부족하기는 하지만, 게이트 기법들의 선조격이자 처음으로 나온 기법이니 이해된다. 다시 한 번 페이퍼를 읽으면서 저자들의 지식에 감탄했다.

### MISC - 혼잣말

* `GetVxTableEntry` 에서 `cw` 변수가 왜 quick and dirty fix in case the function has been hooked 인지 몰랐는데, 말이 된다. NT함수의 시작지점 오프셋이라고도 볼 수 있는 cw를 하나씩 증가하며 NTAPI 함수의 메모리를 시작부터 1바이트씩 쭉 훑으면서 조건문을 통해 `mov r10, rcx // mov` 패턴의 명령어들을 찾는다. 이렇게 되면 NT 함수 초반 부분에 훅을 걸어놓건 뭘 하던 간에 어쨌든 "난 `mov r10, rcx // mov` 패턴을 가진 바이트들만 찾는다" 느낌으로 쭉 메모리를 훑게 된다. 후킹을 우회할 수 있게 된다는게 무슨 말인지 알겠다.

