# winapi 리스트

<details>

<summary>VirtualAlloc - Allocate memory on current process</summary>

[**MSDN**](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc)&#x20;

#### [**PInvoke.net**](https://pinvoke.net/default.aspx/kernel32/VirtualAlloc.html)

#### **시그니쳐**&#x20;

```
LPVOID VirtualAlloc(
  LPVOID lpAddress,
  SIZE_T dwSize,
  DWORD flAllocationType,
  DWORD flProtect
);
```

#### 파라미터&#x20;

* `lpAddress` - Address of the memory to be allocated&#x20;
  * 0 = API chooses the location automatically&#x20;
* `dwSize` - Size of the allocation&#x20;
* `flAllocationType` - Memory allocation type&#x20;
  * Usually `MEM_COMMIT | MEM_RESERVE` = `0x3000`&#x20;
* flProtect = Memory Protection constants - [link](https://docs.microsoft.com/en-us/windows/win32/memory/memory-protection-constants#constants)
  * 0x20 = RX&#x20;
  * 0x40 = RWX&#x20;
  * 0x04 = RW&#x20;

</details>

<details>

<summary>VirtualAllocEx - Allocate memory on a remote process </summary>

#### [**MSDN**](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualallocex)

#### [**PInvoke.net**](https://pinvoke.net/default.aspx/kernel32/VirtualAllocEx.html)&#x20;

#### **시그니쳐**&#x20;

```
LPVOID VirtualAllocEx(
  HANDLE hProcess,
  LPVOID lpAddress,
  SIZE_T dwSize,
  DWORD flAllocationType,
  DWORD flProtect
);
```

#### 파라미터&#x20;

* `hProcess` - Target process's handle&#x20;
* `lpAddress` - Start address to allocate the memory&#x20;
  * 0 = VirtualAllocEx automatically chooses the starting address for us (checkout DripLoader)
* `dwSize` - Length/Amount of memory to allocate
* `flAllocationType` - Typo of memory allocation. Usually `MEM_COMMIT | MEM_RESERVE = 0x3000`&#x20;
* `flProtect` = Memory Protection constants - [link](https://docs.microsoft.com/en-us/windows/win32/memory/memory-protection-constants#constants)
  * 0x20 = RX&#x20;
  * 0x40 = RWX&#x20;
  * 0x04 = RW&#x20;

</details>

<details>

<summary>OpenProcess - Retrieve a handle to a remote process based on PID </summary>

[**MSDN**](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocess)

#### [**PInvoke.net**](https://pinvoke.net/default.aspx/kernel32/OpenProcess.html)

#### **시그니쳐**&#x20;

```
HANDLE OpenProcess(
  DWORD dwDesiredAccess,
  BOOL bInheritHandle,
  DWORD dwProcessId
);
```

#### 파라미터&#x20;

* `dwDesiredAccess` - Access right to obtain in target process. Usually `PROCESS_ALL_ACCESS (0x001F0FF)`
* `bInheritHandle` - True/False on whether the handle can be inherited to child process or not. Usually `False`, because we just don't care.&#x20;
* `dwProcessId` - Target process's PID&#x20;

</details>

<details>

<summary>VirtualProtect - Change memory protection </summary>

[**MSDN**](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotect)

#### [**PInvoke.net**](https://pinvoke.net/default.aspx/kernel32/VirtualProtect.html)&#x20;

#### **시그니쳐**&#x20;

```
BOOL VirtualProtect(
 LPVOID lpAddress,
 SIZE_T dwSize,
 DWORD flNewProtect,
 PDWORD lpflOldProtect
);
```

#### 파라미터&#x20;

* `lpAddress` - Pointer to the start of the memory address&#x20;
* `dwSize` - Size of the memory to change the protection, in bytes.
  * Usually lpAddress + dwSize, or the shellcode's length&#x20;
* `flNewProtect` - Memory protection constant&#x20;
* `lpflOldProtect` - Pointer to a variable with current memory protection. Usually just `0`.&#x20;

</details>

<details>

<summary><strong>VirtualProtectEx - Change memory protection of a remote process</strong> </summary>



</details>

###

### **VirtualAlloc**&#x20;

#### **MSDN**

#### **PInvoke.net**&#x20;

#### **시그니쳐**&#x20;

```
```

#### 파라미터&#x20;

* a
* b

\---&#x20;





### **VirtualAlloc**&#x20;

#### **MSDN**

#### **PInvoke.net**&#x20;

#### **시그니쳐**&#x20;

```
```

#### 파라미터&#x20;

* a
* b

\---&#x20;





### **VirtualAlloc**&#x20;

#### **MSDN**

#### **PInvoke.net**&#x20;

#### **시그니쳐**&#x20;

```
```

#### 파라미터&#x20;

* a
* b

\---&#x20;















