# DInvoke - 시스템 콜

### DInvoke와 시스템 콜

DInvoke와 시스템콜 함수에 관련해서는 다음 페이지에 서술한다.

theWover와 FuzzySecurity가 만든 DInvoke 라이브러리에는 시스템콜을 도와주는 함수 - `GetSyscallStub` 이 있다.

<details>

<summary>GetSyscallStub</summary>

[https://github1s.com/TheWover/DInvoke/blob/HEAD/DInvoke/DInvoke/DynamicInvoke/Generic.cs#L638-L766](https://github1s.com/TheWover/DInvoke/blob/HEAD/DInvoke/DInvoke/DynamicInvoke/Generic.cs#L638-L766)

```csharp
/// <summary>
/// Read ntdll from disk, find/copy the appropriate syscall stub and free ntdll.
/// </summary>
/// <author>Ruben Boonen (@FuzzySec) and Paul Laîné (@am0nsec)</author>
/// <param name="FunctionName">The name of the function to search for (e.g. "NtAlertResumeThread").</param>
/// <returns>IntPtr, Syscall stub</returns>
public static IntPtr GetSyscallStub(string FunctionName)
{
    // Verify process & architecture
    bool isWOW64 = Native.NtQueryInformationProcessWow64Information((IntPtr)(-1));
    /*if (IntPtr.Size == 4 && isWOW64)
    {
        throw new InvalidOperationException("Generating Syscall stubs is not supported for WOW64.");
    }*/
    ProcessModule NativeModule = null;
    // Find the path for ntdll by looking at the currently loaded module
    string NtdllPath = string.Empty;
    ProcessModuleCollection ProcModules = Process.GetCurrentProcess().Modules;
    foreach (ProcessModule Mod in ProcModules)
    {
        if (Mod.FileName.EndsWith("ntdll.dll", StringComparison.OrdinalIgnoreCase))
        {
            NtdllPath = Mod.FileName;
        }

    }

    foreach (ProcessModule _ in Process.GetCurrentProcess().Modules)
    {
        if (_.FileName.EndsWith("ntdll.dll", StringComparison.OrdinalIgnoreCase))
        {
            NativeModule = _;
            NtdllPath = NativeModule.FileName;
        }
    }

    // Alloc module into memory for parsing
    IntPtr pModule = ManualMap.Map.AllocateFileToMemory(NtdllPath);

    // Fetch PE meta data
    Data.PE.PE_META_DATA PEINFO = GetPeMetaData(pModule);

    // Alloc PE image memory -> RW
    IntPtr BaseAddress = IntPtr.Zero;
    IntPtr RegionSize = PEINFO.Is32Bit ? (IntPtr)PEINFO.OptHeader32.SizeOfImage : (IntPtr)PEINFO.OptHeader64.SizeOfImage;
    UInt32 SizeOfHeaders = PEINFO.Is32Bit ? PEINFO.OptHeader32.SizeOfHeaders : PEINFO.OptHeader64.SizeOfHeaders;

    IntPtr pImage = Native.NtAllocateVirtualMemory(
        (IntPtr)(-1), ref BaseAddress, IntPtr.Zero, ref RegionSize,
        Data.Win32.Kernel32.MEM_COMMIT | Data.Win32.Kernel32.MEM_RESERVE,
        Data.Win32.WinNT.PAGE_READWRITE
    );

    // Write PE header to memory
    UInt32 BytesWritten = Native.NtWriteVirtualMemory((IntPtr)(-1), pImage, pModule, SizeOfHeaders);

    // Write sections to memory
    foreach (Data.PE.IMAGE_SECTION_HEADER ish in PEINFO.Sections)
    {
        // Calculate offsets
        IntPtr pVirtualSectionBase = (IntPtr)((UInt64)pImage + ish.VirtualAddress);
        IntPtr pRawSectionBase = (IntPtr)((UInt64)pModule + ish.PointerToRawData);

        // Write data
        BytesWritten = Native.NtWriteVirtualMemory((IntPtr)(-1), pVirtualSectionBase, pRawSectionBase, ish.SizeOfRawData);
        if (BytesWritten != ish.SizeOfRawData)
        {
            throw new InvalidOperationException("Failed to write to memory.");
        }
    }

    // Get Ptr to function
    IntPtr pFunc = GetExportAddress(pImage, FunctionName);
    if (pFunc == IntPtr.Zero)
    {
        throw new InvalidOperationException("Failed to resolve ntdll export.");
    }

    // Alloc memory for call stub
    BaseAddress = IntPtr.Zero;
    RegionSize = (IntPtr)0x50;
    IntPtr pCallStub = Native.NtAllocateVirtualMemory(
        (IntPtr)(-1), ref BaseAddress, IntPtr.Zero, ref RegionSize,
        Data.Win32.Kernel32.MEM_COMMIT | Data.Win32.Kernel32.MEM_RESERVE,
        Data.Win32.WinNT.PAGE_READWRITE
    );

    // Write call stub
    BytesWritten = Native.NtWriteVirtualMemory((IntPtr)(-1), pCallStub, pFunc, 0x50);
    if (BytesWritten != 0x50)
    {
        throw new InvalidOperationException("Failed to write to memory.");
    }

    // Verify process & architecture
    //bool isWOW64 = Native.NtQueryInformationProcessWow64Information((IntPtr)(-1));

    // Create custom WOW64 stub
    if (IntPtr.Size == 4 && isWOW64)
    {
        IntPtr pNativeWow64Transition = GetExportAddress(NativeModule.BaseAddress, "Wow64Transition");
        byte bRetValue = Marshal.ReadByte(pCallStub, 13);

        // CALL DWORD PTR ntdll!Wow64SystemServiceCall
        Marshal.WriteByte(pCallStub, 5, 0xff);
        Marshal.WriteByte(pCallStub, 6, 0x15);
        Marshal.WriteInt32(pCallStub, 7, pNativeWow64Transition.ToInt32());

        // RET <val>
        Marshal.WriteByte(pCallStub, 11, 0xc2);
        Marshal.WriteByte(pCallStub, 12, bRetValue);
        Marshal.WriteByte(pCallStub, 13, 0x00);

        // NOP for alignment
        Marshal.WriteByte(pCallStub, 14, 0x90);
        Marshal.WriteByte(pCallStub, 15, 0x90);
    }

    // Change call stub permissions
    Native.NtProtectVirtualMemory((IntPtr)(-1), ref pCallStub, ref RegionSize, Data.Win32.WinNT.PAGE_EXECUTE_READ);

    // Free temporary allocations
    Marshal.FreeHGlobal(pModule);
    RegionSize = PEINFO.Is32Bit ? (IntPtr)PEINFO.OptHeader32.SizeOfImage : (IntPtr)PEINFO.OptHeader64.SizeOfImage;

    Native.NtFreeVirtualMemory((IntPtr)(-1), ref pImage, ref RegionSize, Data.Win32.Kernel32.MEM_RELEASE);

    return pCallStub;
}
```

</details>

`GetSyscallStub` 은 다음과 같이 실행된다.&#x20;

1. `ntdll.dll` 라이브러리를 찾아 현재 프로세스에 수동으로 맵핑 한다. 이는 EDR 솔루션이 이미 후킹을 해놓은 `ntdll.dll`을 무시하고 온-디스크에 있는 후킹되어 있지 않은, 새로운 `ntdll.dll`을 구하기 위해서다.
2. 새로운 `ntdll.dll`에서 원하는 NativeAPI 함수를 찾은 뒤, 0x50 만큼 해당 메모리를 "긁어"서 현재 프로세스의 메모리 어딘가에 저장한다. 이 "긁어"온 메모리는 어셈블리어로 되어 있는 시스템 콜 stub다.&#x20;
3. 시스템 콜을 실행할 때 C#의 함수 포인터인 Delegate를 설정한 뒤, Delegate가 #2번에서 긁어온 NativeAPI 함수 stub을 가르키도록 설정한다.&#x20;
4. 이 함수포인터를 실행하면 함수 포인터는 "긁어"온 NativeAPI 함수 Stub를 실행한다. 어셈블리어가 실행되며 `ntdll.dll, kernel32.dll`을 거치지 않고 곧바로 시스템 콜이 실행된다.&#x20;

### 실습&#x20;

TODO - [https://github.com/ChoiSG/sNanoDumpInject](https://github.com/ChoiSG/sNanoDumpInject)

### 레퍼런스&#x20;

{% embed url="https://github.com/ChoiSG/sNanoDumpInject" %}

{% embed url="https://offensivedefence.co.uk/posts/Dynamic-syscalls/" %}
