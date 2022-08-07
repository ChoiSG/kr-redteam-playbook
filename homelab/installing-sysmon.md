# 시스몬 (sysmon) 설치

Sysmon (System Monitor)는 윈도우 유저랜드 서비스와 커널랜드 디바이스 드라이버로써, 윈도우 운영체제에서 일어나는 윈도우 이벤트 로그를 수집하고 보여준다. 윈도우 운영체제의 기본적인 로깅은 여러모로 부족하기 때문에, 보안을 위해서라면 시스몬을 설치해 추가로 로그를 수집하는 것이 바람직하다.&#x20;

이렇게 시스몬에서 수집된 로그들은 SIEM으로 전송 되는 경우가 많다. 본인의 홈 랩의 경우는 이를 splunk free version으로 구현하고 있다. 이에 관련해서는 다른 페이지에서 설명한다.&#x20;

### 시스몬 다운과 설치 &#x20;

시스몬 설치를 위해서 sysmon 바이너리와 SwiftOnSecurity의 시스몬 설정 파일을 다운받는다. 설정 파일을 이용해 시스몬 설치를 진행한다.&#x20;

```
iwr https://download.sysinternals.com/files/Sysmon.zip -outfile sysmon.zip
expand-archive sysmon.zip
iwr https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml -outfile sysmonconfig-export.xml

# 시스몬 설치 
sysmon.exe -accepteula -i sysmonconfig-export.xml
```

### 시스몬 설정 파일 업데이트&#x20;

시스몬 설정 파일을 추후 업데이트 한 뒤 적용하고 싶다면 다음과 같이 진행한다.&#x20;

```
sysmon.exe -c sysmonconfig-export.xml
```

### 시스몬 설치/설정 확인&#x20;

시스몬이 제대로 실행되고 있는지 확인하기 위해선 ctrl+R 혹은 파워쉘 실행 후 `eventvwr.exe` 를 실행한다. 이후 `\Applications and Services Logs\Microsoft\Windows\Sysmon\Operational` 로 들어가 로그를 확인한다.&#x20;



### 레퍼런스&#x20;

{% embed url="https://github.com/SwiftOnSecurity/sysmon-config" %}
