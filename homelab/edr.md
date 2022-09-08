# EDR 솔루션 설치

이 페이지는 다음의 페이지를 기반으로 만들어졌습니다: \
[https://www.elastic.co/security-labs/the-elastic-container-project](https://www.elastic.co/security-labs/the-elastic-container-project)&#x20;

* 참고를 하지는 않았지만 페이지 작성이 끝난 뒤 공식 블로그 글이 있다는 것을 알아서 뒤늦게 Credit 및 레퍼런스로 첨부합니다.&#x20;

오펜시브 시큐리티 관련 리서치를 하다보면 EDR 솔루션과 SIEM을 구축해 공격용 툴이나 스크립트가 실행할 때 어떤 IOC를 남기고 어떤 알람을 울리는지에 대해 잘 알아야한다. 하지만 기업용 EDR 솔루션이나 SIEM을 개인이 구입하기에는 무리가 있다.&#x20;

### Elastic EDR&#x20;

Elasticsearch 로 유명한 Elastic 사에서는 최근 자사의[ EDR 솔루션](https://github.com/elastic/endpoint) 중 일부를 오픈소스화 시키며 무료로 배포하고 있다. 최근 Elastic 사의 엔지니어분 중 하나가 ELK 스택을 도커화 시킨 프로젝트를 오픈소스 했기 때문에 이를 이용해 Elastic SIEM과 EDR 을 모두 설치해본다.&#x20;

### 요구사항&#x20;

* VMware, ESXi, VirtualBox 등의 하이퍼바이저&#x20;
* Ubuntu, Debian, Kali Linux 등의 \*Nix 박스 1대&#x20;
* 윈도우 박스 1대&#x20;

### 설치&#x20;

먼저 [Elastic Container Project](https://github.com/peasead/elastic-container) 를 이용해 ELK 스택을 설치한다. 이때 필요한 의존성 패키지들을 먼저 설치한다. 깃헙 리포의 README 를 읽으셔도 된다. 이 가이드에서는 칼리 리눅스를 기반으로 서맃를 진행한다.&#x20;

```
# Docker installation - Ubuntu 
https://docs.docker.com/engine/install/ubuntu/

# Docker installation - Debian 
https://docs.docker.com/engine/install/debian/

# Docker Install for Kali/Debian
sudo apt update -y 
sudo apt update -y ca-certificates curl gnupg lsb-release 
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
# Install Docker 
sudo apt update -y 
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin 

# Install rest of the dependencies 
sudo apt install jq git curl 
```

이후 Elasitc Container 프로젝트를 설치한다&#x20;

```
sudo git clone https://github.com/peasead/elastic-container.git
cd ./elastic-container
sudo vim ./.env 
# ELASTIC_PASSWORD와 KIBANA_PASSWWORD를 changeme에서 다른 것으로 바꿔준다.
sudo ./elastic-container.sh start 
```



약 10\~15분간 도커 이미지를 다운받고, 도커를 실행할 것이다. 모든 설치가 성공적으로 끝난다면 [https://localhost:5601](https://localhost:5601) 를 방문하라는 메시지와 함께 유저이름 + 비밀번호가 나타난다.&#x20;

### 설정&#x20;

설치가 성공적으로 끝났다면 Elastic Security 과 Elastic EDR 설정을 해준다. 먼저 왼쪽 설정 탭의 맨 아래에 있는 Integrations 로 가 Endpoint and Cloud Security 를 누르고, `Add Endpoint and Cloud Security` 를 눌러 설치한다.&#x20;

<figure><img src="../.gitbook/assets/elastic-container-install-edr.PNG" alt=""><figcaption></figcaption></figure>

그 뒤에는 Integration 설정 화면이 나오는데, 홈랩용으로 설치하는 것이니 랜덤한 이름과 설명을 달아주자. 2번은 그냥 디폴트 상태로 저장한 뒤 설치를 진행한다.&#x20;

<figure><img src="../.gitbook/assets/elastic-container-edr1.PNG" alt=""><figcaption></figcaption></figure>

Endpoint and Cloud Security 설치가 끝나면 `Add Elastic Agent to your hosts` 버튼을 눌러 Agent 생성과 설정을 진행한다.&#x20;

<figure><img src="../.gitbook/assets/elastic-container-edr2.PNG" alt=""><figcaption></figcaption></figure>

마찬가지로 1번은 디폴트 상태로 넘기고, 2번으로 가면 Elastic (EDR) Agent 를 설치해주는 명령어들이 나온다. 이 가이드의 경우 윈도우 호스트에 EDR 에이전트를 설치할 것이기 때문에 Windows 를 누르고, 파워쉘 명령어들을 이용해 설치를 진행한다. 단, 맨 마지막줄에는 `--insecure` 플래그를 넣어준다. 다음과 같은 명령어가 나올 것이다 - 물론 IP 주소와 포트는 다르니 참고한다:&#x20;

```
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.4.1-windows-x86_64.zip -OutFile elastic-agent-8.4.1-windows-x86_64.zip
Expand-Archive .\elastic-agent-8.4.1-windows-x86_64.zip -DestinationPath .
cd elastic-agent-8.4.1-windows-x86_64

# Adding --insecure to ignore self signed certificate 
.\elastic-agent.exe install --url=https://192.168.40.182:8220 --enrollment-token=elROTEdvTUJvYWNVSXA3ZjZCOWk6QXU5R1YtX1NTUEtvT1Y2RzRUU0lkUQ== --insecure 
```

Agent 설치 명령어가 끝나고 다시 Elastic 으로 돌아오면 Agent가 콜백을 성공적으로 했다는 메시지가 나온다.&#x20;

<figure><img src="../.gitbook/assets/elastic-container-ed6.PNG" alt=""><figcaption></figcaption></figure>

오펜시브 시큐리티 연구를 위해서는 악성코드가 꼭 실행되어야만 하는 경우가 많이 있다. 따라서 능동적 방지 (Prevention) 이 아닌 탐지 (Detection) 을 설정한다. `Security -> Manage -> Policies` 로 가 `Protection Level` 을 `Detect` 로 설정한다.&#x20;

<figure><img src="../.gitbook/assets/elastic-container-detect-only.PNG" alt=""><figcaption></figcaption></figure>

추가로 더 많은 악성코드 탐지를 위해 특정한 룰을 설정할 수도 있다. `Security -> Manage -> Rules` 로 가 원하는 룰을 활성화 한다. 예를 들자면 간단한 오픈소스 오펜시브 시큐리티 툴을 탐지하고 싶다면 `malware` 를 검색한 뒤 `Malware - Detected - Elastic Endgame` 룰을 활성화한다.&#x20;

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### 실습&#x20;

Elastic EDR 이 실제로 잘 작동하는지 확인하기 위해 EDR 에이전트가 설치된 테스트 윈도우 박스로 가서 악성코드를 실행해보자.&#x20;

```
iex(new-object net.webclient).downloadfile('https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.5_Any/Rubeus.exe','C:\windows\temp\rubeus.exe')

# DELETE the rubeus malware
rm c:\windows\temp\rubeus.exe
```

악성 파워쉘을 실행하면 바로 `Malware Alert` 라는 경고문구가 뜬다. 이후 약 2\~3분 정도가 지나면 대쉬보드에 알람이 울리기 시작한다. 경우에 따라 조금 더 기다려야 할 수도 있다. &#x20;

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

알람이 들어오기 시작했다면 다양한 `Actions` 를 눌러 알람의 정보를 확인한다.&#x20;

&#x20;

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### 마치며&#x20;

이제 Elastic SIEM과 Elastic EDR 솔루션이 있으니 공격/방어의 창과 방패의 수련을 하면 된다. 오펜시브 시큐리티 툴, 악성코드는 어떤 알람을 만들어내는지, 이는 어떻게 우회할 수 있는지, 어떤 행위가 SIEM에 어떤 알람을 만들어내는지 등에 대해서 알아본다.&#x20;

### 레퍼런스&#x20;

{% embed url="https://www.elastic.co/security-labs/the-elastic-container-project" %}

{% embed url="https://github.com/elastic/endpoint" %}

{% embed url="https://github.com/peasead/elastic-container" %}
