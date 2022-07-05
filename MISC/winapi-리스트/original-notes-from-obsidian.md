# original notes from obsidian



## WinAPI Lists

Contains most used WinAPI - links, functionality, parameters, returns, interesting remarks, and personal notes

***

### Process Injection Related WINAPI

[VirtualAlloc](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc)

```
LPVOID VirtualAlloc(
 LPVOID lpAddress,
 SIZE_T dwSize,
 DWORD flAllocationType,
 DWORD flProtect
);
```

LPVOID = Memory Pointer = LongPtr

1. lpAddress = Memory allocation address
   * 0 = API choose the location automatically
2. dwSize = Size of the allocation
3. flAllocationType = Memory allocation type
   * Usually MEM\_COMMIT | MEM\_RESERVE = `0x3000`
4. flProtect = Memory protection flags
   * Usually `0x40` = RWX
   * But you know, better to alloc --> VMProtect --> Write Mem --> VMProtect --> run (but more on that later)

***

**OpenProcess** - Retrieve a handle to a remote process - based on PID

```
HANDLE OpenProcess(
 DWORD dwDesiredAccess,
 BOOL bInheritHandle,
 DWORD dwProcessId
);
```

`dwDesiredAccess` = Access right to obtain in target process. usually PROCESS\_ALL\_ACCESS (`0x001F0FF`)

`bInheritHandle` = True/False on whether the handle can be inherited to child process or not. Usually `False`, because we just don't care.

`dwProcessId` = Target Process's PID

***

**VirtualAllocEx** - Allocate memory in a remote process

```
LPVOID VirtualAllocEx(
 HANDLE hProcess,
 LPVOID lpAddress,
 SIZE_T dwSize,
 DWORD flAllocationType,
 DWORD flProtect
);
```

hProcess = Target process's handle lpAddress = Start address. Usually 0, so VirtualAllocEx can automatically choose the starting address for us

* (Driploader changes this to trick EDR solutions, but that's another story)

dwSize = Length to allocate. Usually shellcode's length flAllocationType = Type of allocation. Usually MEM\_COMMIT | MEM\_RESERVE = `0x3000` flProtect = Memory protection type. Usually 0x20 (Read) or 0x40 (RWX)

***

VirtualProtect - Change memory protection

```
BOOL VirtualProtect(
 LPVOID lpAddress,
 SIZE_T dwSize,
 DWORD flNewProtect,
 PDWORD lpflOldProtect
);
```

lpAddress = `IntPtr` - Page address - Start pointer of the memory address dwSize = `UInt32` - Size of area wish to modify. `1 ~ 0xFFF` is same. Usually `3`. flNewProtect = `UInt32` - Memory protection constant. RWX = `0x40`, RX = `0x20` lpflOldProtect = `[ref] UInt32` = Current memory protection

***

**WriteProcessMemory** - Write to a region of a memory in a remote process

```
BOOL WriteProcessMemory(
 HANDLE hProcess,
 LPVOID lpBaseAddress,
 LPCVOID lpBuffer,
 SIZE_T nSize,
 SIZE_T *lpNumberOfBytesWritten
);
```

hProcess = Handle to target process lpBaseAddress = Starting address to write memory. Usually `alloc` returned from `VirtualAllocEx` lpBuffer = Address of source byte array to be copied nSize = Length/Size of the lpBuffer. Usually shellcode length lpNumberOfBytesWritten = Pointer to a location in memory to output how much data was copied

* Usually used with `out` keyword from C#
* Pass by reference, not value - because it's a pointer that will hold data copied.

***

**CreateRemoteThread**

```
HANDLE CreateRemoteThread(
 HANDLE hProcess,
 LPSECURITY_ATTRIBUTES lpThreadAttributes,
 SIZE_T dwStackSize,
 LPTHREAD_START_ROUTINE lpStartAddress,
 LPVOID lpParameter,
 DWORD dwCreationFlags,
 LPDWORD lpThreadId
);
```

hProcess = Handle to the target process lpThreadAttributes = Thread attribute. Usually 0, unless PPID spoofing... but that's another story. dwStackSize = Usually 0 lpStartAddress = Start address of the thread. Usually `alloc` from `VirtualAllocEx`. This is where the shellcode starts. lpParameter = Pointer to variables if parameter. Since shellcodes usually don't have parameters, usually 0. dwCreationFlags = Usually IntPtr.Zero lpThreadId = Usually 0.

***

**ReadProcessMemory** = Read memory from remote process

```
BOOL ReadProcessMemory(
 HANDLE hProcess,
 LPCVOID lpBaseAddress,
 LPVOID lpBuffer,
 SIZE_T nSize,
 SIZE_T *lpNumberOfBytesRead
);
```

hProcess = `IntPtr` / Handle to remote process lpBaseAddress = `IntPtr` / Base address to start reading from lpBuffer = `byte[]` / Destination buffer to copy the content from nSize = `int` / Number of bytes to be read lpNumberOfBytesRead = `out` / Number of bytes that were ACTUALLY read

Example

```
byte[] addrBuf = new byte[IntPtr.Size];
IntPtr nRead = IntPtr.Zero;
ReadProcessMemory(hProcess, ptrToImageBaseAddress, addrBuf, addrBuf.Length, out nRead);
```

***

**CreateProcessW** - Create a Process

```
BOOL CreateProcessW(
 LPCWSTR lpApplicationName,
 LPWSTR lpCommandLine,
 LPSECURITY_ATTRIBUTES lpProcessAttributes,
 LPSECURITY_ATTRIBUTES lpThreadAttributes,
 BOOL bInheritHandles,
 DWORD dwCreationFlags,
 LPVOID lpEnvironment,
 LPCWSTR lpCurrentDirectory,
 LPSTARTUPINFOW lpStartupInfo,
 LPPROCESS_INFORMATION lpProcessInformation
);
```

lpApplicationName = Name of the application - Usually `null` lpCommandLine = Full commandline to be executed - Usually full path of executable lpProcessAttribute & lpThreadAttribute = Specify security descriptor - Usually `null` and `false` dwCreationFlag = Creation Flag. CREATE\_SUSPENDED = `0x4` lpEnvironMent, lpCurrentDirectory = Straight forward, but usually `null` for default. lpStartupInfo = `STARTUPINFO` Structure that includes values related with how new process should be configured. Included in PInvoke.net's cheatsheet lpProcessInformation = `PROCESS_INFORMATION` structure with handles, threads, pid, tid. Provided in PInvoke.net.

Example

```
STARTUPINFO si = new STARTUPINFO();
PROCESS_INFORMATION pi = new PROCESS_INFORMATION();

bool result = CreateProcess(null, @"C:\windows\system32\svchost.exe", IntPtr.Zero, IntPtr.Zero, false, 0x4, IntPtr.Zero, null, ref si, out pi); 
```

***

**ZwQueryInformationProcess** - Return PEB information inside a PI (Process Information) variable

```
NTSTATUS WINAPI ZwQueryInformationProcess(
 _In_ HANDLE ProcessHandle,
 _In_ PROCESSINFOCLASS ProcessInformationClass,
 _Out_ PVOID ProcessInformation,
 _In_ ULONG ProcessInformationLength,
 _Out_opt_ PULONG ReturnLength
);
```

NTSTATUS (return type) = hex value from the kernel showing the status of the call.

ProcessHandle = `IntPtr` / Handle to process from `PROCESS_INFORMATION` structure ProcessInformationClass = Usually `0` ProcessInformation = `ref PROCESS_BASIC_INFORMATION()` / structure PorcesInformationLength = `uint` / Size of input structure (6 IntPtr) ReturnLength = `ref uint` variable that hold the size of fetched data

ex)

```
PROCESS_BASIC_INFORMATION bi = new PROCESS_BASIC_INFORMATION();
uint tmp = 0; 
IntPtr hProcess = pi.hProcess;
// PROCESS_BASIC_INFORMATION has 6 IntPtr, thus (IntPtr.Size*6)
ZwQueryInformationProcess(hProcess, 0, ref bi, (uint)(IntPtr.Size*6), ref tmp);

// Now "bi" has the PEB address. 
IntPtr ptrToImageBase = (IntPtr)((Int64)bi.PebAddress + 0x10); 
```

***

**GetCurrentProcess** - Return current process's handle

```
[DllImport("kernel32.dll")]
static extern IntPtr GetCurrentProcess();
```

***

### Token Related WINAPI

`CreateNamedPipe` - Creates a named pipe

```
HANDLE CreateNamedPipeA(
 LPCSTR lpName,
 DWORD dwOpenMode,
 DWORD dwPipeMode,
 DWORD nMaxInstances,
 DWORD nOutBufferSize,
 DWORD nInBufferSize,
 DWORD nDefaultTimeOut,
 LPSECURITY_ATTRIBUTES lpSecurityAttributes
);
```

lpName = `string` = Name of the pipe (ex. `\\.\pipe\pipe_name`) dwOpenMode = `uint` = Mode of the pipe = Mostly `3` = `PIPE_ACCESES_DUPLEX` dwPipeMode = `uint` = Mode of the pipe operation = `PIPE_TYPE_BYTE|PIPE_WAIT`, Usually `0`. nMaxInstances = `uint` = Maximum number of instances of the pipe. Anything 1\~255. nOutBufferSize = `uint` = Number of bytes to use for input/output buffer. Usually `0x1000` bytes. nInBufferSize = `uint` = Number of bytes to use for input/output buffer. Usually `0x1000` bytes. nDefaultTimeout = `uint` = Default timeout. Usually 0. lpSecurityAttributes = `IntPtr` = SID of clients that can interact with the pipe. Usually `NULL/IntPtr.Zero`, because we want any clients to connect to our dummy pipe server.

***

`ConnectNamedPipe` - Connect to a named pipe

```
BOOL ConnectNamedPipe(
 HANDLE hNamedPipe,
 LPOVERLAPPED lpOverlapped
);
```

hNamedPipe = `IntPTr` =Handle to the named pipe to connect to lpOverlapped = `IntPtr` = Pointer to a structure for advnaced cases. Usually `IntPtr.Zero/Null` for us.

***

`ImpersonateNamedPipeClient` - Impersonate the access token of the pipe client that connected to our pipe server. Stolen token will be assigned to the current thread of the process.

```
BOOL ImpersonateNamedPipeClient(
 HANDLE hNamedPipe
);
```

hNamedPipe = `IntPtr` = Handle to the named pipe (server).

***

`OpenThreadToken` - Open thread's token. Used for confirmation that impersonation was successful.

```
BOOL OpenThreadToken(
 HANDLE ThreadHandle,
 DWORD DesiredAccess,
 BOOL OpenAsSelf,
 PHANDLE TokenHandle
);
```

ThreadHandle = `IntPtr` = Handle to the thread to check its token. In this case, we want to check our thread from our process. So just use `GetCurrentThread`. DesiredAccess = `uint` = Desired Access to the thread. Usually `TOKEN_ALL_ACCESS` = `0xF01FF` OpenAsSelf = `bool` = Should API use security context of current process? Usually `false/no`, because we are using the impersonated token of the target process. TokenHandle = `IntPtr hToken; out hToken` = (Out) pointer that will be populated with a handle to the token that is opened.

***

`GetTokenInformation` - Return token information from the token handle. One of those "Call twice" weird API.

```
BOOL GetTokenInformation(
 HANDLE TokenHandle,
 TOKEN_INFORMATION_CLASS TokenInformationClass,
 LPVOID TokenInformation,
 DWORD TokenInformationLength,
 PDWORD ReturnLength
);
```

TokenHandle = `IntPtr` = Token Handle. Usually retrieved through `OpenThreadToken` TokenInformationClass = `uint` = Enum of token information. Usually `1`, which is TokenUser (SID) value. TokenInformation = `IntPtr` = Pointer to output buffer that will be populated by the API. TokenInformationLength = `int` = Size of output buffer ReturnLength = `out int` = Output length.

So this winAPI need to be called twice. It's because we don't know required size of the buffer.

ex)

```
[DllImport("advapi32.dll", SetLastError = true)]
static extern bool GetTokenInformation(IntPtr TokenHandle, uint TokenInformationClass, IntPtr TokenInformation, int TokenInformationLength, out int ReturnLength);

int TokenInfLength = 0;

// Calling first time, because we don't know TokenInformationLength
GetTokenInformation(hToken, 1, IntPtr.Zero, TokenInfLength, out TokenInfLength);

// Calling second time, because we now KNOW TokenInformationLength. Allocate memory of TokenInformation with the length of TokenInformationLength, and call again.
IntPtr TokenInformation = Marshal.AllocHGlobal((IntPtr)TokenInfLength);
GetTokenInformation(hToken, 1, TokenInformation, TokenInfLength, out TokenInfLength);
```

***

`ConvertSidToStringSid` - Convert binary SID to string SID

```
BOOL ConvertSidToStringSidW(
 PSID Sid,
 LPWSTR *StringSid
);
```

Sid = `IntPtr` = Pointer to the SID. It's inside output buffer from `GetTokenInformation`. This needs to be extracted from a structure. StringSid = `out IntPtr` = Output string that will contain the SID string.

Example

```
// Setting structure of SID. Used for SID extraction later on. 
[StructLayout(LayoutKind.Sequential)]
public struct SID_AND_ATTRIBUTES
{
 public IntPtr Sid;
 public int Attributes;
}
public struct TOKEN_USER
{
 public SID_AND_ATTRIBUTES User;
}

[DllImport("advapi32", CharSet = CharSet.Auto, SetLastError = true)]
static extern bool ConvertSidToStringSid(IntPtr pSID, out IntPtr ptrSid);

// Extracting SID(TokenUser) from the TokenInformation structure 
TOKEN_USER TokenUser = (TOKEN_USER)Marshal.PtrToStructure(TokenInformation, typeof(TOKEN_USER));

// Pointer to contain output SID string 
IntPtr pstr = IntPtr.zero;
Boolean ok = ConvertSidToStringSid(TokenUser.User.Sid, out pstr);

// Converting pointer to actual string 
string sidstr = Marshal.PtrToStringAuto(pstr);
Console.WriteLine(@"[+] Sid: ", sidstr); 
```
