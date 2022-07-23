# 도메인 분류와 신뢰도

공격자가 도메인을 구입해 당일날 도메인 세팅을 하고 공격하던 시대는 지난지 오래되었다. 많은 보안 업체들과 공공기관들의 협력으로 인해 요새는 도메인 신뢰도와 도메인 카테고리화라는 기술을 이용해 도메인 블랙리스트와 화이트리스트를 공유한다. 이 리스트들은 각 기관의 내부망에 있는 포워드 프록시에 자동으로 적용되어 매일매일 새로운 도메인들이 블랙/화이트리스트에 업데이트 된다.

까다로운 도메인 분류와 신뢰도를 모두 우회하기 위해서 도메인 프론팅(Domain Fronting)이라는 기술을 사용할 수도 있다. 이에 관련해서는 다른 페이지에서 따로 서술한다.

### 도메인 분류 (Domain Categorization)

도메인 분류는 해당 도메인이 어느 카테고리에 들어가는지 분류하는 작업을 일컫는다. 예를 들어 `cnn.com` 은 "언론" 카테고리로, `seoul.go.kr` 은 "공공기관" 카테고리, `youtube.com` 은 "미디어/엔터테인먼트" 등으로 분류된다. 다양한 업체들의 스캐너들이 인터넷을 돌아다니며 자동적으로 도메인 카테고리화를 적용하거나, 유저들이 자발적으로 도메인 카테고리화 요청을 넣어 카테고리화를 진행하기도 한다.

예를 들어, Symantec의 bluecoat 도메인 분류 서비스/솔루션을 이용해보자. 유투브를 검색하면 다음과 같이 Audio/Vdieo Clip 및 Mixed Content/Potentially Adult 로 분류된다.

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png>)

레드팀에서 사용할 도메인 또한 분류가 되면 좋다. 카테고리화가 되려면 도메인에 정상적인 컨텐츠를 올려놓거나, 다른 페이지로 리다이렉트 되도록 설정하면 된다. 물론 분류가 성공적일수도 있고, 실패할수도 있다. 도메인 카테고리화는 도메인 신뢰도에 따라 결과가 달라질 수 있으니, 이에 대해 알아보자.

### 도메인 신뢰도 (Domain Reputation)

도메인 신뢰도는 해당 도메인이 얼마나 신뢰될 수 있는지를 보여주는 지표다. 도메인 신뢰도를 평가하는 솔루션/업체들도 다양하고, 다 각자만의 기준이 있어 모두 설명하기가 어렵다. 하지만 대체로 다음과 같은 지표들을 사용해 도메인 신뢰도를 평가한다:

* 이메일 보안 - 주소, 반송 주소, DKIM Signing, SPF, DMARC 등.
* 도메인 나이 - 등록된지 얼마나 지났는가? 1주일 전에 등록된 도메인이라면 신뢰하기 어려울 것이다.
* 도메인 분류 - 위에서 살펴본 도메인 분류/카테고리에 따라 신뢰 점수를 다양하게 부여한다.
* SSL/TLS 인증서 - 검증받은 CA에서 제대로 받은 인증서인지 확인한다.
* 아이피 신뢰도 - Threat Intelligence 데이터베이스나 바이러스 토탈 등에서 발견된 공격에 쓰이던 IP인지 등을 확인한다.
* 컨텐츠 - 웹서버를 운영중인가? 그렇다면, 퀄리티가 좋은 컨텐츠가 있는가.
* 링크 - 해당 도메인이 링크된 적이 있는지, 있다면 어떤 웹사이트들이 얼마나 많이 링크 했는지.

### 구입

레드팀은 고객사와 정식으로 계약을 맺고 합법적으로 일을 진행하기 때문에 도메인 구입에 있어서 딱히 작전보안을 구축할 필요는 없다\*. 따라서 가짜 신분이나 가짜 카드, 페이퍼 페이팔 계정, 믹싱한 암호화폐등을 준비하지 않아도 된다. 그냥 본인의 이름과 회사 법인 카드 등으로 도메인을 구입하면 된다.

도메인 네임 등록 대행 업체 또한 스케치한 제3국의 대행 업체에서 진행할 필요 없이 잘 알려진 업체에서 구입하면 된다. 단, 고객사의 요청에 의해 특정 APT가 자주 사용하는 대행 업체가 있다면, APT 시뮬레이션을 위해 해당 대행 업체에서 도메인을 구입하면 된다.

### 만료된 도메인 구입

새로운 도메인을 구입하지 않고 만료된 도메인을 구입해 사용하는 것 또한 효율적이다. 만료된 도메인들은 새로운 도메인들보다 상대적으로 도메인 신뢰도나 분류가 쉬울 수 있기 때문이다. 심지어 만료된 도메인을 다시 구입해 "살려내는" 경우, 이전에 갖고 있던 신뢰도나 분류가 유지되는 경우도 있다. [ExpiredDomains.net](https://www.expireddomains.net) 같은 사이트들은 만료된 도메인들을 보여주는데, 여기서 원하는 도메인 이름을 구입해 사용하면 된다.



### 레퍼런스

[도메인 카테고리화 및 차단리스트](https://github.com/bluscreenofjeff/Red-Team-Infrastructure-Wiki#categorization-and-blacklist-checking-resources)

[OpenDNS 도메인 카테고리화](https://community.opendns.com/domaintagging/)

[도메인 신뢰도 - 이메일](https://postmarkapp.com/blog/how-to-check-your-domain-reputation)

[만료된 도메인 검색](https://www.expireddomains.net)

[도메인 에이징](https://posts.specterops.io/being-a-good-domain-shepherd-57754edd955f)