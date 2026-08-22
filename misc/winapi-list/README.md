# winapi 리스트

<details>

<summary>VirtualAlloc - Allocate memory on current process</summary>

[**MSDN**](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc)

[**PInvoke.net**](https://pinvoke.net/default.aspx/kernel32/VirtualAlloc.html)

**시그니쳐**

```
LPVOID VirtualAlloc(
  LPVOID lpAddress,
  SIZE_T dwSize,
  DWORD flAllocationType,
  DWORD flProtect
);
```

**파라미터**

* `lpAddress` - Address of the memory to be allocated
  * 0 = API chooses the location automatically
* `dwSize` - Size of the allocation
* `flAllocationType` - Memory allocation type
  * Usually `MEM_COMMIT | MEM_RESERVE` = `0x3000`
* flProtect = Memory Protection constants - [link](https://docs.microsoft.com/en-us/windows/win32/memory/memory-protection-constants#constants)
  * 0x20 = RX
  * 0x40 = RWX
  * 0x04 = RW

</details>

<details>

<summary>VirtualAllocEx - Allocate memory on a remote process</summary>

[**MSDN**](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualallocex)

[**PInvoke.net**](https://pinvoke.net/default.aspx/kernel32/VirtualAllocEx.html)

**시그니쳐**

```
LPVOID VirtualAllocEx(
  HANDLE hProcess,
  LPVOID lpAddress,
  SIZE_T dwSize,
  DWORD flAllocationType,
  DWORD flProtect
);
```

**파라미터**

* `hProcess` - Target process's handle
* `lpAddress` - Start address to allocate the memory
  * 0 = VirtualAllocEx automatically chooses the starting address for us (checkout DripLoader)
* `dwSize` - Length/Amount of memory to allocate
* `flAllocationType` - Typo of memory allocation. Usually `MEM_COMMIT | MEM_RESERVE = 0x3000`
* `flProtect` = Memory Protection constants - [link](https://docs.microsoft.com/en-us/windows/win32/memory/memory-protection-constants#constants)
  * 0x20 = RX
  * 0x40 = RWX
  * 0x04 = RW

</details>

<details>

<summary>OpenProcess - Retrieve a handle to a remote process based on PID</summary>

[**MSDN**](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocess)

[**PInvoke.net**](https://pinvoke.net/default.aspx/kernel32/OpenProcess.html)

**시그니쳐**

```
HANDLE OpenProcess(
  DWORD dwDesiredAccess,
  BOOL bInheritHandle,
  DWORD dwProcessId
);
```

**파라미터**

* `dwDesiredAccess` - Access right to obtain in target process. Usually `PROCESS_ALL_ACCESS (0x001F0FF)`
* `bInheritHandle` - True/False on whether the handle can be inherited to child process or not. Usually `False`, because we just don't care.
* `dwProcessId` - Target process's PID

</details>

<details>

<summary>VirtualProtect - Change memory protection</summary>

[**MSDN**](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotect)

[**PInvoke.net**](https://pinvoke.net/default.aspx/kernel32/VirtualProtect.html)

**시그니쳐**

```
BOOL VirtualProtect(
 LPVOID lpAddress,
 SIZE_T dwSize,
 DWORD flNewProtect,
 PDWORD lpflOldProtect
);
```

**파라미터**

* `lpAddress` - Pointer to the start of the memory address
* `dwSize` - Size of the memory to change the protection, in bytes.
  * Usually lpAddress + dwSize, or the shellcode's length
* `flNewProtect` - Memory protection constant
* `lpflOldProtect` - Pointer to a variable with current memory protection. Usually just `0`.

</details>

<details>

<summary><strong>VirtualProtectEx - Change memory protection of a remote process</strong></summary>



</details>

###

### **VirtualAlloc**

#### **MSDN**

#### **PInvoke.net**

#### **시그니쳐**

```
```

#### 파라미터

* a
* b

\---

### **VirtualAlloc**

#### **MSDN**

#### **PInvoke.net**

#### **시그니쳐**

```
```

#### 파라미터

* a
* b

\---

### **VirtualAlloc**

#### **MSDN**

#### **PInvoke.net**

#### **시그니쳐**

```
```

#### 파라미터

* a
* b

\---
