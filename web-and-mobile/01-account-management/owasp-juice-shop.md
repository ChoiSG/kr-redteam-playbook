# 세번째, OWASP Juice Shop 로그인 페이지에서의 통신 흐름과 구조 알아보기

그럼 이번에는  OWASP Juice Shop의 로그인 페이지에서 응용해보자.

아래 이미지와 같이 192.168.149.128:3000/#/login의 주소가&#x20;

Juice Shop의 로그인 페이지 주소이고&#x20;

<figure><img src="../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>



Burp Suite로 해당 페이지를 접근하면 아래와 같이 여러 URL에 접근 되는 것이 보인다.

<figure><img src="../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>



지금 목표는 로그인 성공이니 여기에서 다 볼 필요는 없다. 그럼 로그인을 시도할 때 통신 되는 데이터를 찾아보자 로그인을 시도할 시에 /rest/user/login 주소로 통신이 되고 아래 이미지 처럼 email, password 값에 임의로 주소를 넣어야 로그인이 성공 되는 것을 짐작하는데

<figure><img src="../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

Email의 메시지 박스와&#x20;

<div align="left">

<figure><img src="../../.gitbook/assets/image (122).png" alt=""><figcaption></figcaption></figure>

</div>

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>



password의 메시지 박스의 소스코드를 확인 해야 한다.

이 두가지 class를 알아본 이유는 후에 자동 로그인 소스코드를 만들기 위한 정보에 필요해보이기 때문에 우선 확인을 한 것이다.

<div align="left">

<figure><img src="../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

</div>

<figure><img src="../../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>



그럼 계정을 만들고 로그인을 직접 진행 해보자  로그인을 성공할 시에 통신 과정을 보면&#x20;

/rest/user/login&#x20;

/rest/user/whoami&#x20;

/rest/basket/6&#x20;

/api/Quantitys/ 의 과정으로 통신이 진행되면서 로그인이 성공하는 것을 확인 할 수 있는데 여기에서 '/api/Quantitys/' api를 짐작하는 통신을 확인 할 수 있고

<figure><img src="../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>



/rest/user/whoami는 로그인 사용자의 정보로 짐작이 되는데

<figure><img src="../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>



우선적으로 로그인 인증과 세션 유지에 중요해 보이는 것은 API로 보이기 때문에 API를 봐보자

아래 이미지와 같이 로그인을 성공하면 "authentication":{"token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1N ....이하생략  token이 암호화 되어 있는 것과

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>



'/api/Quantitys/' api의 통신 데이터로 보이는 값을 확인 할 수 있는데

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>



암호화 되어있는 token값을 base64로 디코딩해서 보면 나오는 정보가 있다. &#x20;

해당 정보에서의 요점은

<figure><img src="../../.gitbook/assets/Animation5 (22).gif" alt=""><figcaption></figcaption></figure>



첫번째, 'JWT'

<div align="left">

<figure><img src="../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

</div>



두번째, 로그인을 성공할 시의 정보 값을 요점으로 봐야하는데

<figure><img src="../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

JWT을 간략히 소개하자면

JSON 웹 토큰(JWT)은 시스템 간에 암호화 서명된 JSON 데이터를 전송하기 위한 표준 형식이다. 인증,세션 관리 및 액세스 제어 메커니즘 에 일반적으로 사용된다. 이는 공격자가 JWT를 성공적으로 수정할 수 있는 경우 자신의 권한을 에스컬레이션하거나 다른 사용자를 가장할 수 있음을 의미한다.

juice-shop은 DVWA와는 달리 로그인 보안을 신경 쓴 것이 보이기는 하나 결국 JWT도 취약점이 있다는 것을 알게 되었고 다음 글에는 해당 인증 취약점과 세션을 유지하는 방법을 알아보겠다.

참고자료 : [https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/jwts](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/jwts)
