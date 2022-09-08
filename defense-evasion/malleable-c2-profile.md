# 가변적 C2 프로필

네트워크 안 어딘가에서 실행중인 비컨/에이전트 등의 RAT 악성코드들을 구분해내는 전통적인 방법 중 하나는 바로 콜백 트래픽의 특성을 살펴보는 것이였다. 아직도 많은 악성코드 분석 및 포렌식 보고서를 살펴보면 다음과 같은 네트워크 기반 IoC들이 언급되곤 한다:&#x20;

* IP 주소 및 도메인 주소&#x20;
* HTTP(S) -엔드포인트 주소 &#x20;
* HTTP(S) - GET/POST 등에서 사용되는 파라미터 이름과 값 등&#x20;
* HTTP(S) - HTTP 헤더 이름과 값&#x20;

이들을 모두 합쳐 "APT 999 그룹의 darkchoi 악성코드는 <공격자도메인> 의 /login.php에 GET과 POST 리퀘스트를 보내며, 파라미터는 "name", 쿠키는 "sessionID-choi" 등을 쓴다..." 라는 문구를 많이 볼 수 있다. 블루팀들은 이와 같이 네트워크 IoC 기반으로 탐지룰을 짜기도 한다.&#x20;

예를 들어 슬리버 C2의 기본적인 HTTP 네트워크 트래픽은 다음과 같다.&#x20;

![](<../.gitbook/assets/image (3) (2).png>)

본인이 블루팀이라면 어떤 논리들로 네트워크 트래픽 탐지룰을 작성할 것인가?&#x20;

* 슬리버 서버는 세션 관련 HTTP 요청에 HTTP 202 Accepted를 반환한다 - 정기적으로 202 응답을 보내는 서버를 찾아본다.
* 슬리버 서버는 웹서버 관련 이름, 버전이 들어간 HTTP 헤더가 아예 존재하지 않는다.&#x20;
* 슬리버 비컨은 하트비트 콜백을 할 때 `.js` 파일을 요청한다 - 정기적으로 특정 서버에게 `.js` 파일을 요청하는 트래픽을 찾아본다.&#x20;
* 슬리버 비컨은 세션 관련 콜백을 할 때 `.php` 파일을 요청한다 - 정기적으로 특정 서버에게 `.php` 파일을 요청하는 트래픽을 찾아본다.&#x20;
* 슬리버 비컨은 기본적으로 `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.3380.71 Safari/537.36` 유저 에이전트를 사용한다.&#x20;
* 슬리버 비컨은 세션 구축 후 `APISID` 라는 세션 쿠키를 사용하고 있다.&#x20;

해당 탐지룰을 잘 조합해 탐지룰을 구축한다면, 앞으로 슬리버 C2를 사용하는 모든 공격자들을 탐지할 수 있을 것이다 - 라고 모든 블루팀들이 생각했고, 실제로 옛날 공격자들은 이런 탐지룰에 걸리면 전체 작전이 끝나곤 했다. 많은 네트워크 트래픽 관련된 특성들이 하드코딩 되었기 때문이다.&#x20;

### 가변적 C2 프로필&#x20;

이런 문제점들을 해결하고자 2014년 코발트 스트라이크의 제작자인 Raphael Mudge는 [Malleable C2 Profile](https://www.cobaltstrike.com/blog/malleable-command-and-control/) (가변적 C2 프로필) 기능을 발표한다. 가변적 C2 프로필은 C2 서버와 비컨이 통신을 할 때 사용하는 네트워크 특성들을 바꾼 프로필을 생성한 뒤, 적용시키는 기능이다. 요새는 HTTP/S, DNS, ICMP 등의 다양한 프로토콜에도 가변적 프로토콜이 적용되곤 한다. &#x20;

예를 들어 위 슬리버 비컨/서버가 가지고 있던 네트워크 IoC 들 - `.js, .php, APISID, 202 Accepted, Mozilla/5.0 (Windows NT 10.0; Win64; x64` 을 바꾸는 가변적 C2 프로필을 적용시켜보자.&#x20;

![](<../.gitbook/assets/image (12) (1).png>)

가변적 C2 프로필을 적용시키고 나니 몇가지 바뀐점이 보인다&#x20;

* 엔드 포인트의 이름들이 모두 바뀌고, 엔드포인트 확장자는 `.asp` 로 바뀌었다.&#x20;
* User-Agent 도 바뀌었다&#x20;
* 쿠키도 `session-id` 로 바뀌었다&#x20;
* 서버 헤더 `Server: Microsoft-IIS/10.0, X-Powered-By: ASP.NET` 등이 추가됐다
* 이 외 GET/POST 파라미터 또한 바꿀 수 있다&#x20;

적용한 가변적 C2 프로필은 다음과 같다.&#x20;

<details>

<summary>http-c2.json</summary>

```json
{
    "implant_config": {
        "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:101.0) Gecko/20100101 Firefox/101.0",
        "url_parameters": null,
        "headers": null,
        "max_files": 3,
        "min_files": 1,
        "max_paths": 3,
        "min_paths": 1,
        "stager_file_ext": ".woff",
        "poll_file_ext": ".asp",
        "poll_files": [
            "aboutus",
            "lsp",
            "session",
            "account"
        ],
        "poll_paths": [
            "en-us",
            "about",
            "today",
            "view",
            "shopping",
            "store"
        ],
        "start_session_file_ext": ".html",
        "session_file_ext": ".aspx",
        "session_files": [
            "login",
            "signin",
            "view",
            "api",
            "index",
            "admin",
            "register",
            "sign-up"
        ],
        "session_paths": [
            "upload",
            "actions",
            "rest",
            "v1",
            "auth",
            "oauth2",
            "oauth2callback",
            "api"
        ],
        "close_file_ext": ".png",
        "close_files": [
            "favicon",
            "sample",
            "example"
        ],
        "close_paths": [
            "static",
            "www",
            "assets",
            "images",
            "icons",
            "image",
            "icon",
            "png"
        ]
    },
    "server_config": {
        "random_version_headers": false,
        "headers": [
            {
                "name": "Cache-Control",
                "value": "no-store, no-cache, must-revalidate",
                "probability": 100
            },
            {
                "name": "Server",
                "value": "Microsoft-IIS/10.0",
                "probability": 100
            },
            {
                "name": "X-Powered-By",
                "value": "ASP.NET",
                "probability": 100
            }
        ],
        "cookies": [
            "ASPSESSIONID",
            "ASP.NET_SessionId",
            "session-id"
        ]
    }
}
```

</details>

따라서 위에서 적용한 탐지룰은 이제 적용하기가 힘들어졌다. 새로운 탐지룰이 나오면 공격자는 또 가변적 C2 프로필을 바꿀 것이다.&#x20;

### 미래&#x20;

2014년도 이후 가변적 C2 프로필은 많이 진화해 다양한 "가변적 XYZ"가 많이 나왔다.&#x20;

* Hot-Swappable 가변적 C2 프로필 - 실시간으로 C2 프로필을 바꿀 수 있는 기능&#x20;
* Malleable PE - 생성하는 모든 PE 프로필에 자동적 난독화, IoC 랜덤화&#x20;
* Malleable Process Injection - 프로세스 인젝션을 실행할때마다 다른 인젝션 기법을 사용&#x20;
* Malleable Post Exploitation - 메모리상에서 실행되는 Post-Ex .NET 어셈블리, BOF, COFF 등의 난독화 및 IoC 랜덤화&#x20;

시간이 지날수록 점점 더 유연하고 가변적인 공격용 툴과 방어용 툴의 대결이 계속 될 것 같다.&#x20;
