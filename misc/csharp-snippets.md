# C# snippets

## OffensiveCSharpCheatsheet

C# Cheatsheet with useful code snippets and oneliners

### Converting

#### IntPtr -> String

```
// 1 
string intptrString = intPtrVar.ToInt64().ToString("x2");
Console.WriteLine("0x{0}", intptrString);

// 2 
IntPtr intPtr = <something>;
Console.WriteLine("{0}", String.Format("0x{0:X}", intPtr.ToInt64()));
```

#### Byte\[] -> String - Hex (useful for debugging)

```
public static string ByteArrayToString(byte[] ba)
{
    byte[] returnArray = new byte[ba.Length];
    returnArray = ba.ToArray();
    Array.Reverse(returnArray);
    return "0x" + String.Concat(Array.ConvertAll(returnArray, x => x.ToString("X2")));
}

Console.WriteLine("[+] something: {0}", ByteArrayToString(someBA));
```

#### String -> Byte\[]

```
// ascii 
byte[] ba = Encoding.ASCII.GetBytes("string");

// utf8
byte[] ba = Encoding.UTF8.GetBytes("string");
```

#### Byte\[] -> String

```
byte[] ba = <something>;
string baToStr = System.Text.Encoding.Default.GetString(ba); 
```

#### Base64String -> Byte\[]

```
byte[] ba = Convert.FromBase64String("something");
```

#### Byte\[] -> Base64String

```
string b64Str = Convert.ToBase64String(byte[] ba);
```

#### UInt -> String

```
string uintToStr = String.Format("0x{0:X}", uint something));
```

#### Byte\[] -> IntPtr

```
// 32bit 
IntPtr baPtr = new IntPtr(BitConverter.ToInt32(byteArray, 0));

// 64bit 
IntPtr baPtr = new IntPtr(BitConverter.ToInt64(byteArray, 0));
```

#### IntPtr -> Byte\[]

```
byte[] ba = new byte[<size>];
Marshal.Copy(intPtr, ba, 0, size);

ex)

byte[] ba = new byte[IntPtr.Size];
Marshal.Copy(intPtr, ba, 0, IntPtr.Size);
```

#### String -> Byte\[]

```
byte[] ba = Encoding.Unicode.GetBytes("C:\\windows\\explorer.exe");
```

***

### Memory

#### Allocate unmanaged memory

```
int size = 0x100;
IntPtr pSize = Marshal.AllocHGlobal(size);
```

#### Read memory from IntPtr

```
// 32bit - Read 4 bytes 
IntPtr pBuf = <something>;
int something = Marshal.ReadInt32(pBuf);

// 64bit - Read 8 bytes
IntPtr pBuf = <something>;
long something = Marshal.ReadInt64(pBuf);
```

#### Write memory to IntPtr

```
// https://docs.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.marshal.writeint32?view=net-6.0

IntPtr pSource = <something>;
int offset = <something>;
int value = <something>; 

Marshal.WriteInt32(pSource, offset, value)
```

#### Write IntPtr

```
IntPtr lpValuePointer = Marshal.AllocHGlobal(IntPtr.Size);
Marshal.WriteIntPtr(lpValuePointer, parentHandle);
```

***

### Memory Calculations

#### IntPtr + UInt

```
IntPtr a = <something>;
uint b = <some uint32>;

IntPtr newA = (IntPtr)( (UInt64)a + b );
```

#### ByteArray + Int

```
// 32bit 
byte[] oldBa = <something>; 
int addition = <something>; 
byte[] newBa = BitConverter.GetBytes(BitConverter.ToInt32(oldBa, 0) + addition);
```

***

### Process (debugging)

#### Find Process by name

```
Process explorer = Process.GetProcessByName("explorer")[0];
```

#### Start Process (Notepad)

```
Process notepad = Process.Start("c:\\windows\\system32\\notepad.exe"); 
```

#### Kill Process

```
// by name 
Process[] processes = Process.GetProcessByName("notepad");
foreach(var proc in processes)
{
    proc.Kill(); 
}

// by Id 
int pid = <something>;
Process proc = Process.GetProcessById(pid);
proc.Kill();
```

***

### Encryption & Decryption

#### XOR ByteArray

```
// --encrypt-key - bash -c 'echo $RANDOM | md5sum | head -c 32; echo;'
// --encrypt-iv - bash -c 'echo $RANDOM | md5sum | head -c 16; echo;'
public static byte[] xorByteArray(byte[] sc, int scLength, byte[] key, int keyLength)
{
    byte[] decryptedSC = new byte[scLength];
    int j = 0;

    int c = 0;
    for (int i = 0; i < scLength; i++)
    {
        if (j == keyLength - 1)
        {
            j = 0;
        }

        decryptedSC[i] = (byte)(sc[i] ^ key[j]);
        j++;
    }
    return decryptedSC;
}
```

#### AES CBC mode Decrypt

```
// https://stackoverflow.com/questions/53653510/c-sharp-aes-encryption-byte-array
// byte[] buf = Aes256Decrypt(bufEncrypted, Encoding.ASCII.GetBytes(key), Encoding.ASCII.GetBytes(iv));
public static byte[] Aes256Decrypt(byte[] data, byte[] key, byte[] iv)
{
    using (var aes = Aes.Create())
    {
        aes.KeySize = 128;
        aes.BlockSize = 128;
        aes.Padding = PaddingMode.Zeros;

        aes.Key = key;
        aes.IV = iv;

        using (var decryptor = aes.CreateDecryptor(aes.Key, aes.IV))
        {
            return PerformCryptography(data, decryptor);
        }
    }
}

public static byte[] PerformCryptography(byte[] data, ICryptoTransform cryptoTransform)
{
    using (var ms = new MemoryStream())
    using (var cryptoStream = new CryptoStream(ms, cryptoTransform, CryptoStreamMode.Write))
    {
        cryptoStream.Write(data, 0, data.Length);
        cryptoStream.FlushFinalBlock();

        return ms.ToArray();
    }
}
```

#### AES CBC mode Encrypt

```
// https://stackoverflow.com/questions/53653510/c-sharp-aes-encryption-byte-array
// byte[] buf = Aes256Decrypt(bufEncrypted, Encoding.ASCII.GetBytes(key), Encoding.ASCII.GetBytes(iv));
public byte[] Encrypt(byte[] data, byte[] key, byte[] iv)
{
    using (var aes = Aes.Create())
    {
        aes.KeySize = 128;
        aes.BlockSize = 128;
        aes.Padding = PaddingMode.Zeros;

        aes.Key = key;
        aes.IV = iv;

        using (var encryptor = aes.CreateEncryptor(aes.Key, aes.IV))
        {
            return PerformCryptography(data, encryptor);
        }
    }
}

public static byte[] PerformCryptography(byte[] data, ICryptoTransform cryptoTransform)
{
    using (var ms = new MemoryStream())
    using (var cryptoStream = new CryptoStream(ms, cryptoTransform, CryptoStreamMode.Write))
    {
        cryptoStream.Write(data, 0, data.Length);
        cryptoStream.FlushFinalBlock();

        return ms.ToArray();
    }
}
```

***

### Web

#### DownloadData - HTTP

```
string urlString = "http://127.0.0.1:9999/something";
WebClient webclient = new WebClient();
byte[] downloadData = webclient.DownloadData(urlString);
```

#### DownloadData - HTTPS

```
// TODO 
ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;
string urlString = "https://127.0.0.1:9999/something";
WebClient webclient = new WebClient();
byte[] downloadData = webclient.DownloadData(urlString);
```

***

### DInvoke

***

### MISC

#### is64Bit

```
public static bool is64Bit()
{
    if(IntPtr.Size == 8)
    {
        return true;
    }
    else
    {
        return false;
    }
}
```

#### Sleep

```
// ms 
System.Threading.Thread.Sleep(1000);
```

## References

[AES Encrypt, Decrypt](https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions/Invoke-AESEncryption.ps1)

[SharpSploit](https://github.com/cobbr/SharpSploit)

[SharpBlock](https://github.com/CCob/SharpBlock)

[Sharp-Suite](https://github.com/FuzzySecurity/Sharp-Suite)
