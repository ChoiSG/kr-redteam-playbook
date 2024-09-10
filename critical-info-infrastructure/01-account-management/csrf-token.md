# CSRF Token에 관하여

CSRF 토큰이란?

CSRF 토큰은 서버측 애플리케이션에서 생성되고 클라이언트와 공유되는 인증 값이다. 클라이언트는 서버의 통신에 올바른 CSRF 토큰을 포함해야 한다. 그렇지 않으면 서버는 요청된 작업 수행을 거부한다. CSRF 토큰을 클라이언트와 공유하는 일반적인 방법은 HTML 형식의 숨겨진 매개변수로 포함하는 것이다.

<figure><img src="../../.gitbook/assets/image (138).png" alt=""><figcaption></figcaption></figure>



1. 클라이언트는 HTTP GET 메서드를 사용하여 애플리케이션 서버에 액세스한다.&#x20;
2. 서버는 CSRF 토큰을 생성하고 HTTP 세션에 저장한다. 생성된 CSRF 토큰은 HTML 형식의 숨겨진 태그를 사용하여 클라이언트와 연결된다.&#x20;
3. 클라이언트는 HTML 양식의 버튼을 클릭하여 애플리케이션 서버에 요청을 보낸다. CSRF 토큰은 HTML 형식의 숨겨진 필드에 포함되어 있으므로 CSRF 토큰 값이 요청 매개 변수로 전송된다.&#x20;
4. 서버는 요청 파라미터에 지정된 CSRF 토큰 값과 HTTP POST 메소드를 사용하여 액세스할 때 HTTP 세션에 유지된 CSRF 토큰 값이 동일한지 확인한다. 토큰 값이 일치하지 않으면 잘못된 요청으로 오류가 발생한다.&#x20;

CSRF Token은 아래 이미지와 같이 페이지 소스에서 확인이 가능하다. 다만 이 방법은 현재 DVWA의 페이지 소스에서는 hidden 상태가 아니기 때문에 쉽게 찾을 수 있으나 보통 웹 사이트의 Token은 hidden 상태로 보안되어 있다.



CSRF Token은 아래 이미지와 같이 페이지 소스에서 확인이 가능하다. 다만 이 방법은 현재 DVWA의 페이지 소스에서는 hidden 상태가 아니기 때문에 쉽게 찾을 수 있으나 보통 웹 사이트의 Token은 hidden 상태로 보안되어 있다.&#x20;

<figure><img src="../../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>
