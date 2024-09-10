# 레드팀 글로벌 동향 (2024)

## 들어가며

<figure><img src="https://blog.sunggwanchoi.com/content/images/2024/03/sword.png" alt=""><figcaption></figcaption></figure>

레드팀(공격자 시뮬레이션)에 관련된 얘기를 나누다보면 가장 많이 받는 질문들은 다음과 같다.

1. 레드팀이 뭐예요?
2. 다른 나라들은 어떻게 하고 있나요?

1번에 대한 답은 이미 [레드팀이란?](https://www.xn--hy1b43d247a.com/what-even-is-redteam) 이라는 글에 정리해놨다.

2번과 관련된 이야기를 나누다보면 결국 다른 나라들의 레드팀에 관련된 평가 체계, 프레임워크, 법, 컴플라이언스에 대해 설명하게 된다. 모의해킹이야 서비스를 제공한지 20년이 지나며 이제는 `취약점 점검 및 조치` 등의 항목으로 다양한 법과 컴플라이언스에 포함되어 가고 있지만, 레드팀은 과연 어떨까? 이 글에서는 레드팀과 관련된 평가 체계, 프레임워크, 그리고 법률들을 통해 전세계적으로 공격자 시뮬레이션이 어떻게 받아드려지고 있는지에 대해 알아본다. 글로벌 동향을 살펴보며 다른 나라들은 어떤 방안들을 내놓고 있는지, 왜 이런 방안들이 나오게 되었는지, 구체적으로 어떤 내용을 포함하고 있는지, 공공과 민간에서는 이를 어떻게 받아드리고 있는지, 그리고 마지막으로 국내 상황은 어떤지에 대해 살펴본다.

### 목차

1. 글로벌 동향
2. 왜? - 레드팀 평가 체계, 프레임워크, 법률이 만들어진 이유
3. 무엇을? - 레드팀 평가 체계, 프레임워크, 법률의 내용
4. 어떻게? - 실무에서 방안들이 어떻게 활용되고 있는가
5. 국내 상황
6. 마치며

`레드팀 프레임워크` 라는 표현이 자주 나올텐데, 아래의 정의를 참고한다.

* **레드팀 프레임워크:** 레드팀 프레임워크는 레드팀 서비스 실행에 있어 필요한 주요 용어, 단계, 방법론, 활동, 결과물, 상호작용 방법을 체계화한 가이드라인이다. 레드팀의 정의를 제공하며, 해당 서비스를 이용하려는 기관이나 기업에게 구체적인 수행 방법론을 제시한다. 특히 영국과 유럽에서는 단순한 방법론을 넘어 위협 인텔리전스 공급자와 레드팀 제공 업체 및 개인의 자격 요건, 그리고 해당 업체들을 선정하는 방법 및 계약을 맺고 프로젝트를 실행하는 방법에 이르기까지 포괄적인 가이드라인들을 일컫는다.

## 1. 글로벌 동향

<figure><img src="https://blog.sunggwanchoi.com/content/images/2024/03/earth.png" alt=""><figcaption></figcaption></figure>

### 유럽 연합, 홍콩, 싱가포르, 사우디아라비아

영국과 유럽 연합을 필두로 한 서구권에서는 2010년대 중후반부터 레드팀(Red team(ing)), 위협 주도 침투 테스트(Threat Led Penetration Testing, TLPT), 혹은 인텔리전스 기반 고급 침투 테스트(Intelligence-Led/Based Advanced Penetration Testing) 등의 다양한 명칭으로 레드팀/공격자 시뮬레이션에 관련된 프레임워크와 규정들을 개발했다. 많이 알려진 TIBER-EU 부터, 최근에 만들어진 FEER, 그리고 아예 법안으로 제정된 DORA까지, 유럽 연합 국가들과 영국, 싱가포르, 홍콩, 사우디아라비아 등 약 20개국에서는 아래와 같은 레드팀 프레임워크 및 규정들을 운영하고 있다. 아래 표의 이름을 클릭하면 각 문서들로 링크를 해놨으니 원문이 궁금하다면 참고한다.

| 이름                                                                                                                                                                         | 종류    | 연도         | 발의 기관                                                     | 적용 나라                                                                               |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----- | ---------- | --------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| [CBEST](https://www.bankofengland.co.uk/financial-stability/operational-resilience-of-the-financial-sector/cbest-threat-intelligence-led-assessments-implementation-guide) | 평가 체계 | 2014       | 영국 중앙은행 (영란은행)                                            | 영국                                                                                  |
| iCAST                                                                                                                                                                      | 프레임워크 | 2016       | 홍콩 금융 관리국                                                 | 홍콩                                                                                  |
| [GBEST](https://www.crest-approved.org/membership/gbest/)                                                                                                                  | 평가 체계 | 2017\~2018 | 영국 정부                                                     | 영국                                                                                  |
| [AASE](https://abs.org.sg/docs/library/abs-red-team-adversarial-attack-simulation-exercises-guidelines-v1-06766a69f299c69658b7dff00006ed795.pdf)                           | 프레임워크 | 2018       | 싱가포르 은행 협회(ABS)                                           | 싱가포르                                                                                |
| [TIBER-EU](https://www.ecb.europa.eu/pub/pdf/other/ecb.tiber\_eu\_framework.en.pdf)                                                                                        | 프레임워크 | 2018       | 유럽 중앙 은행(ECB)                                             | 프랑스, 독일, 이탈리아, 스페인, 스웨덴, 네덜란드, 오스트리아, 벨기에, 덴마크, 핀란드, 아이슬란드, 아일랜드, 룩셈부르크, 노르웨이, 포르투갈 |
| [FEER](https://uploads-ssl.webflow.com/59d28ad983887e000196f803/5fecc27919c71c17d5a10851\_Financial%20Entities%20Ethical%20Red%20Teaming%20Framework.pdf)                  | 프레임워크 | 2019       | 사우디아라비아 통화청                                               | 사우디아라비아                                                                             |
| [STAR-FS](https://www.bankofengland.co.uk/financial-stability/operational-resilience-of-the-financial-sector)                                                              | 프레임워크 | 2024       | 영국 중앙은행                                                   | 영국                                                                                  |
| [DORA - Article 26,27](https://www.digital-operational-resilience-act.com/Article\_26.html)                                                                                | 법안/규정 | 2023/2025  | 유럽 은행 당국(EBA), 유럽 보험 및 직업 연금 당국(EIOPA), 유럽 시장 관리 기구(ESMA) | 모든 유럽 연합 국가                                                                         |

가장 눈에 띄는 것은 영국이다. 영국은 무려 2014년도부터 CBEST 위협 인텔리전스 주도 평가(Threat Intelligence-Led Assessments)를 만들어 영국 중앙은행(BoE), 건전성감독청(PRA), 금융행위감독청(FCA) 등의 기관이 금융 관련 기관/기업 및 인프라 운영 업체들의 보안을 평가하는데 사용했다. CBEST가 금융쪽에 자리잡기 시작하자 이를 기반으로 GBEST라는 새로운 평가 체계를 만들어 영국의 정부 부서에 적용하기도 했다. CBEST와 GBEST는 추후 유럽 중앙 은행의 TIBER-EU 프레임워크를 제작하는데 많이 참고되었다고 한다.

TIBER-EU는 위협 인텔리전스 기반 윤리적 레드팀 운용 프레임워크이며, 금융 인프라 및기관들의 사이버 공격에 대한 회복력을 시험하고 향상시키는데 사용되고 있다. TIBER-EU는 금융쪽에서 레드팀 서비스를 받기 위해 필요한 주요 용어, 단계, 활동, 방법론, 결과물 및 상호작용 방법을 체계화한 가이드라인이다. 제작된 이후 많은 나라들에 적용되었으며, 각 나라마다 기본 TIBER 프레임워크를 조금씩 변형해 운영하고 있다. 예를 들어 프랑스는 TIBER-FR, 스페인은 TIBER-ES, 독일은 TIBER-DE 를 자체적으로 만들어 운영중이다.

### CREST

<figure><img src="https://blog.sunggwanchoi.com/content/images/2024/03/image.png" alt=""><figcaption></figcaption></figure>

영국과 유럽 연합의 가장 특징적인 점은 바로 CREST라는 비영리 인증 기관을 통해 사이버 보안 기술 및 서비스의 품질을 체계화 시키고 있다는 점이다. 레드팀을 예로 들자면 프레임워크의 인증, 수행 기업과 인원의 자격, 심지어는 수행 인원이 받은 교육 프로그램의 인증까지, 모든 것들을 국내의 소위 `8대 전문직` 처럼 체계화 시키고 있다. 이처럼 CREST는 레드팀 뿐만 아니라 사이버 보안과 관련된 모든 것에 대한 자격을 부여하고 인증을 심사하는 기관이다. 국내에서는 [SK인포섹이 2018년](https://m.blog.naver.com/skinfosec2000/221330540930) CREST 인증을 획득하며 알려지기 시작했다.

예를 들어 유럽 내의 은행이 TIBER-EU 평가 체계를 통과하기 위해 레드팀 서비스를 받아야한다면,

* TIBER-EU 평가 체계가 CREST 인증을 받았음을 확인하고 - [링크](https://www.crest-approved.org/membership/tiber-eu/)
* TIBER-EU 레드팀 서비스를 진행할 수 있는 인증된 기업 목록을 CREST에서 확인한 뒤 - [링크](https://www.crest-approved.org/members/?filter\_government\_scheme\_10717=TIBER%20EU%20\(Europe\))
* 수행 인원들의 레드팀 전문 자격증 CCSAS 취득 여부를 확인하고 - [링크](https://www.crest-approved.org/skills-certifications-careers/crest-certified-simulated-attack-specialist/)
* 마지막으로 수행 인원들이 인증된 교육 기관/기업에서 교육을 받았는지 확인할 수 있다 - [링크](https://www.crest-approved.org/skills-certifications-careers/approved-training-providers/)

이처럼 서구권에서는 CREST를 필두로 해 레드팀 뿐만 아니라 정보보안과 관련된 자격, 인증 심사, 교육을 모두 표준화, 체계화 하려는 움직임을 보이고 있다.

### 미국

정보보안의 선도 국가인 미국은 오히려 레드팀 관련 법, 프레임워크, 평가 체계를 갖추지 않고 있다. 그러나 사이버 공격 이후의 징벌적 과징금, 집단 소송, 주가 변동 등으로 인해 정보보안에 대해 진지하게 생각하는 기업들이 레드팀 서비스를 받는 사례들이 증가하고 있다. 이는 보안 컨설팅 업체들이 최근 10년 내에 레드팀/공격자 시뮬레이션 부서를 신설한 것만 보더라도 알 수 있다. 또한, 레드팀 수요 증가에는 아래의 다음 사건들도 어느정도 영향을 미친 것으로 보인다.

1. 2021년 4월 - 바이든 행정부의 `국가 사이버 보안 개선에 관한 행정 명령` - [링크](https://www.whitehouse.gov/briefing-room/presidential-actions/2021/05/12/executive-order-on-improving-the-nations-cybersecurity/)

솔라윈즈 해킹, 익스체인지 사태, 콜로니얼 파이프라인 해킹 등 대형 사건 사고들을 겪으며 백악관에서 행정 명령이 나올 정도로 미국에서는 사이버 보안과 관련된 관심이 높아졌다. 대규모 사이버 공격 및 OT 인프라가 마비될 정도의 고도화된 공격을 받았기 때문에 레드팀 서비스를 통해 이를 시뮬레이션 하고 탐지 및 대응 하려는 노력들도 많아지고 있다. 이와 관련한 국내 기사는 다음의 링크를 참고한다 - [링크](https://m.boannews.com/html/detail.html?idx=97506\&page=2\&mkind=1\&kind=1)

2. 2023년 7월 - 미국 증권거래위원회의 사이버 공격 4일 공시 의무화 - [링크](https://www.sec.gov/rules/2022/03/cybersecurity-risk-management-strategy-governance-and-incident-disclosure#33-11216)

과거 대형 해킹 사건이 났을 때 이를 은폐하거나 피해를 최소화해 공표하려는 시도들 때문인지, 미국 증권거래위원회는 사이버 공격 발생 시 4일 이내에 정보 공시를 의무화 하는 규정을 제정했다. 이 때문에 단순히 로비 활동을 통해 언론이 해킹 공격을 "덮는" 경우가 많이 줄어들었고, 이는 곧 기업들의 사이버 보안 투자 증가로도 이어졌다. 이와 관련된 국내 기사는 다음의 링크를 참고한다 - [링크](https://www.techtube.co.kr/news/articleView.html?idxno=3613)

## 2. 왜? - 레드팀 평가 체계, 프레임워크, 법률이 만들어진 이유

<figure><img src="https://blog.sunggwanchoi.com/content/images/2024/03/why.png" alt=""><figcaption></figcaption></figure>

그렇다면 왜 서구권, 홍콩, 싱가포르, 사우디아라비아쪽에서는 레드팀과 관련된 평가 체계, 프레임워크, 법률들을 만들었을까? 이 질문에 답하기 위해 관련 문서들에서 언급된 부분들을 살펴보고 분석해봤다.

참고로 문서들에서는 레드팀 서비스를 받는 기관들을 엔티티(Entity) 라는 단어로 표현하는데, 국내 정서에 맞게 일단은 "기관"이라는 용어를 사용했다. 또한 전문 번역가는 아니기에 번역에 있어서 어느정도 오류나 의역에 있다는 점을 양해 바란다.

### TIBER-EU 섹션 2.5 - 왜 위협 주도 레드티밍 테스트인가?

TIBER-EU에서는 금융 기관들의 IT 환경이 얼마나 많이 변화되고, 고도화 되고, 복잡하게 바뀌었는지에 대해 섹션 2.1에서 서술한다. 그 뒤, 현재 제공 되고 있는 모의해킹 서비스들의 한계에 대해서 섹션 2.5에서 다음과 같이 서술한다:

> 모의해킹은 단일 고립된 시스템이나 환경에서 기술적, 환경적 취약점에 대한 상세하고 유용한 평가를 제공한다. 하지만, 모의해킹은 기관 전체(사람, 프로세스, 기술)에 대한 표적 공격의 전체 시나리오를 평가하지 않는다.   ("Penetration tests have provided a detailed and useful assessment of technical and configuration vulnerabilities, often within isolation of a single system or environment. However, they do not assess the full scenario of a targeted attack against an entire entity (including the complete scope of its people, processes and technologies.")

[레드팀이란?](what-even-is-redteam.md) 글에서도 잠깐 언급했지만 모의해킹은 고립되고 지엽적인 시스템이나 환경의 취약점을 알아내는데 특화된 서비스다. 하지만 실제 공격자들의 공격은 특정 도메인, 특정 시스템들"만" 공격하지 않는다. 실제 공격자들은 타겟의 모든 자산을 유기적으로 넘나드며 공격을 진행한다. 따라서 모의해킹이 평가하지 못하는 부분 - 타겟 전체에 대한 표적 공격 - 을 커버하기 위해 레드팀 서비스를 받아야한다는 것을 강조하고 있다.

### DORA Article 26 - 2번

> 각 위협 주도 침투 테스트의 타겟은 기관에서 운영중인 중요 기능들의 일부, 혹은 전체를 포함해야 하며, 실제 해당 기능을 운영중인 라이브 프로덕션 시스템들에서 수행되어야 한다. (Each threat-led penetration test shall cover several or all critical or important functions of a financial entity, and shall be performed on live production systems supporting such functions.)

DORA 아티클 26번의 중요한 부분이다. 단순한 사이버 레인지(Cyber Range)등에서 이뤄지는 공격/방어 형식의 레드팀은 충분하지 않고, 실제 프로덕션 서버에서 실제 기능들을 수행하는 시스템들을 상대로 TLPT(위협 주도 침투 테스트)를 진행해야 한다고 명시하고 있다. 모의해킹은 가용성의 문제 때문에 UAT, DEV, 스테이징 등의 시스템이나 네트워크에서 이뤄지는 경우가 많은데, 이를 허용하지 않겠다는 의미이기도 하다. 실제 공격자들은 라이브 프로덕션 시스템들을 공격하지, 친절하게 UAT 환경으로 옮겨가서 공격하지 않는다. 사이버 공격 시뮬레이션을 실제 시스템에서 실행함으로서 가장 실전에 가까운 현실적인 훈련을 받으라는 것으로 이해하면 될 듯 하다.

> 기관들은 ICT(정보 통신 기술) 서비스를 운영하는데 있어 관련된 ICT 시스템, 프로세스, 기술을 식별해야한다. 여기엔 ICT 제3자 서비스 제공업체에게 아웃소싱 되었거나 외주된 것들도 포함된다. (Financial entities shall identify all relevant underlying ICT systems, processes and technologies supporting critical or important functions and ICT services, including those supporting the critical or important functions which have been outsourced or contracted to ICT third-party service providers.)

DORA에서 위협 모델링(Threat Modeling)을 간접적으로 언급한 부분이다. 레드팀 서비스를 받을 각 기관들에게 1) 주요 기능은 무엇인가? 2) 해당 기능과 관련된 시스템, 프로세스, 기술은 무엇인가? 3) 어떤 것들이 외주로 만들어졌는가? 에 대해서 생각하도록 하고 있다. 위협 모델링의 첫번째 단계 중 하나인 자산 식별 및 목표 설정을 안내하고 있는 것이다. 풀어 쓰자면 `뭘 지켜야 하는가?` 에 대해 고심해보라는 것이다.

### AASE - Executive Summary

> 금융 기관에 대한 사이버 보안 공격은 범위, 복잡성, 정교함 측면에서 빠르게 진화하고 있다. < . . . > 기관들이 직면한 특별한 위협을 상대로 효율적인 자원 배분을 하기 위해 기관들은 위협 모델링을 통해 가능한 공격자들과 공격 경로들을 식별하고 공격 시뮬레이션 및 시나리오를 제작하는 것이 권장된다. (Cyber security attacks against organisations such as financial institutions (FIs) are evolving rapidly in scope, complexity and sophistication. In order to efficiently allocate their resources to the unique threats they are facing, FIs are encouraged to create scenarios for their attack simulation by identifying the most likely adversaries and the attack vectors through threat modelling.)

싱가포르 은행 협회의 AASE 개요 섹션의 한 부분이다. 별 다른 설명이 필요없을만큼 레드팀/공격자 시뮬레이션이 왜필요한지에 대해 설명하고 있다.

이외에도 다양한 방안들을 살펴보고 종합해보면, 다음과 같은 이유 때문에 이런 방안들과 레드팀 서비스가 만들어졌다고 볼 수 있다:

**사이버 공격이 점차 고도화 됨에 따라, 기존의 지엽적인 모의해킹 및 취약점/소스코드 분석은 방어자의 전체적인 인력, 프로세스, 기술에 대한 전반적인 평가를 내리는데에 부족함이 있다. 따라서, 기관의 관리적, 기술적 보안을 담당하는 인력, 대응 절차, 보안 통제, 기술 등의 현황과 부족한 간극을 최대한 현실적으로 평가하기 위해 실제 공격자들의 공격 라이프사이클과 전략, 전술, 절차(TTP)를 에뮬레이트 (emulate), 혹은 시뮬레이트 (simulate) 하는 가상의 사이버 공격을 진행하는 레드팀 서비스를 도입할 필요가 있다.**

## 3. 무엇을? - 레드팀 평가 체계, 프레임워크, 법률의 내용

<figure><img src="https://blog.sunggwanchoi.com/content/images/2024/03/what.png" alt=""><figcaption></figcaption></figure>

높은 수준의 APT 그룹들의 행동을 시뮬레이트하는 레드팀 서비스는 명확한 방법론, 규칙, 자격, 인증 없이 수행될 경우 심각한 문제를 초래할 수 있다. 따라서 레드팀 평가 체계, 프레임워크, 법률은 프로젝트를 진행하는데 있어 모든 이해관계자가 동일한 용어, 방법론, 활동, 결과물, 상호작용 단계들을 사용할 수 있도록 이를 표준화하고 체계화하는데 중요한 역할을 한다.

TIBER-EU를 예로 들자면, 레드팀과 관련된 다음의 항목들에 대한 표준화를 하고 있다. [TIBER-EU 링크](https://www.ecb.europa.eu/pub/pdf/other/ecb.tiber\_eu\_framework.en.pdf). 아래는 TIBER-EU의 1 페이지 요약이다.

### 1. 레드팀 프로세스

모든 기관들이 체계화된 레드팀 서비스를 받을 수 있도록 다음과 같이 표준화했다.

* **준비 단계:** 위협 배경, 프로젝트 스코핑, 위협 인텔리전스 및 서비스 제공자와의 계약
* **테스트 단계:** 위협 인텔레전스 및 레드팀 서비스 실행
* **마무리 단계:** 대응 방안 마련 및 결과 공유

이와 관련된 세부 사항은 섹션 4.2 Process Overview를 참고한다.

### 2. 준비 단계

**주요 이해관계자 역할 및 설정:** 레드팀 서비스를 진행하는데 있어 필요한 인력 및 이해관계자를 설정한다. 테스트 매니저, 화이트팀, 블루팀, 위협 인텔리전스 제공 업체, 레드팀 서비스 제공 업체, 정보기관 및 국가 사이버 보안 센터 등을 명시해놨다. 섹션 5.3 - Test Management와 5.4 Test Implementation 참고.

**위협 관리:** 레드팀 서비스를 진행하며 위협 관리와 위협 평가는 어떤 주체가 실행하는지, 그리고 위협 인텔리전스 및 레드팀 제공 업체를 선정하는데 있어 필요한 최소한의 요구 사항 및 계약 사항, 비밀 유지 계약서 등에 대해 설명한다. 섹션 6 - Risk Management for TIBER-EU tests 참고.

**준비 단계:** 프로젝트 브리핑, 계약 과정에 필요한 업체 자격/인증, 프로젝트 스코프 및 목표 설정, 플래그 및 가짜 데이터 생성에 대해 설명한다. 섹션 7 - Preparation Phase 참고.

### 3. 테스팅 단계

**위협 인텔리전스 테스팅:** 레드팀을 진행하기 전, 위협 인텔리전스 테스팅을 먼저 진행한다. GTL 보고서를 통해 금융 업계를 위협하는 공격자들에 대한 전반적인 정리를 한 뒤, TTI 보고서를 통해 기관의 현재 방어 체계와 공격 표면을 바탕으로 한 현실적인 공격 시나리오를 구상한다. 더 현실적인 공격 시나리오를 만들기 위해 가능하다면 국가 정보기관의 도움을 받는다. 가장 중요한 목표중 하나는 바로 공격할 타겟과 현실적인 위협을 정의하는 것이며, 이는 모두 실제 예시 및 정보기관의 정보를 바탕으로 만들어져야한다. 섹션 8 - Testing Phase: Threat Intelligence and Scenarios 참고

**레드팀 테스팅:** 사이버 킬체인이 살짝 변형된 - Reconnaissance, Weaponization, Delivery, Exploitation, Control and Movement, Actions on Target - 형태로 이뤄진다. 레드팀 서비스를 수행하기 전, 위협 인텔리전스 단계의 TTI 보고서를 기반으로 공격 시나리오 및 계획을 만들어 이해관계자에게 전달한다. 공격 시나리오에는 공격할 타겟 시스템들 및 플래그가 명확해야한다. 사용할 TTP는 인젤리전스를 바탕으로 한 높은 수준의 실제 공격자들의 TTP와 비슷하면서도, 상황에 맞게 TTP를 수정 할 수 있는 유연성과 창의성을 갖춰야한다. 실제 공격자들과는 달리 레드팀들은 정해진 시간, 인력, 예산이 있기 때문에 내부 시스템에 관련된 정보가 너무 없거나 찾기 불가능한 경우에는 화이트팀이나 타겟 기관과 소통해 어느정도의 정보를 얻는다. 특정 구간/단계에서 막힌 경우 침해 가정 시나리오(Assumed Breach)나 특정 권한 및 플래그를 준 뒤 재개해 모든 프로세스를 다 시험 할 수 있도록 한다. 섹션 9 - Testing Phase: Red Team Testing 참고.

### 4. 마무리 단계 세부 설명

마무리 단계에서는 각 이해관계자들이 역할에 맞는 보고서를 작성한 뒤, 디브리핑을 진행한다.

* **레드팀 보고서** - 기술적/비-기술적 취약점, 공격 방식, 발견점들에 대한 서술
* **블루팀 보고서** - 레드팀 보고서를 기반으로 한 공격자 TTP와 그에 따른 탐지/대응 방안 맵핑
* **레드팀 & 블루팀 리플레이 워크샵** - 레드팀과 블루팀 보고서를 기반으로 한 퍼플팀을 진행하며 특정 공격/탐지/대응에 관련되서 레드팀과 블루팀이 협력한다.
* **360도 피드백** - 기관, TCT, 위협 인텔리전스 업체, 레드팀 업체들이 모두 모여, 진행된 TIBER-EU 테스트에 관련된 피드백 워크샵을 진행한다.
* **대응 방안 계획 및 테스트 요약 보고서** - 기관은 모든 보고서, 디브리핑, 워크샵을 기반으로 한 대응 방안 보고서를 작성한 뒤, TIBER-EU 테스트의 요약 보고서를 작성한다.

위와 관련된 모든 결과물들이 제출되고, 모든 이해관계자가 동의 한다면 TIBER-EU 테스트 종료 확인/증명 프로세스를 진행해 모든 테스트가 성공적으로 끝남을 알린다.

섹션 10 - Closure Phase 참고.

## 4. 어떻게? - 실무에서 방안들이 어떻게 활용되고 있는가

<figure><img src="https://blog.sunggwanchoi.com/content/images/2024/03/how.png" alt=""><figcaption></figcaption></figure>

평가 체계, 프레임워크, 법안들은 실무에서 사용되고 있을까? 만들어놓기만 한 건 아닐까?

유럽 연합에서 TIBER-EU 및 TIBER 계열의 평가 체계는 실제로 활발히 사용되고 있다. 유럽 중앙 은행은 2023년 4월, 약 5년 동안 총 100회의 TIBER 테스트가 성공적으로 수행되었다는 것을 발표했다([링크](https://www.ecb.europa.eu/paym/intro/news/html/ecb.mipnews230427.en.html)). 물론 중복 테스트도 있었겠지만, 러프하게 보자면 2018년에서 2023년 동안 유럽 내 금융 기관 약 100곳에서 TIBER-EU를 실행했다는 것을 의미한다.

영국 및 유럽 보안 컨설팅 업체들은 평가 체계와 프레임워크를 기반으로 한 레드팀 서비스 제공을 마케팅 포인트로 삼고 있다. 예를 들어 NightHawk C2 프레임워크로 유명한 MDSEC은 CREST 인증을 통해 CBEST, TIBER, STAR 관련 프로젝트를 진행할 수 있다는 홍보를 한다.

<figure><img src="https://blog.sunggwanchoi.com/content/images/2024/03/image-1.png" alt=""><figcaption><p>MDSEC의 레드팀 홍보 페이지 중 (출처: https://www.mdsec.co.uk/our-services/adversary-simulation/red-team-operations/)</p></figcaption></figure>

NightHawk의 탄생 비화도 재미있다. MDSEC에서 레드팀 서비스를 제공하며 코발트 스트라이크(Cobalt Strike)를 많이 사용했었는데, 작전을 뛸 때마다 CS가 EDR 솔루션들에게 너무 많이 걸려 아예 작정하고 NightHawk를 만들었다는 것이다. 이처럼 유럽의 보안 컨설팅 업체들에서는 TIBER/CBEST를 기반으로 하는 레드팀 서비스를 많이 제공하고 있는 듯 하다. 유럽에서 제대로된 레드팀 서비스를 제공하는 업체들이 궁금하다면 MDSEC, Outflank(Fortra에 인수), NCC Group, Pen Test Partners, Security Alliance, WithSecure 등을 참고한다.

DORA는 기관들이 적어도 3년마다 한번씩 레드팀(TLPT)를 실시하도록 법적으로 규정하고 있다 ("Financial entities . . . shall carry out at least every 3 years advanced testing by means of TLPT"). 물론 이 주기는 기관과 환경에 따라 더 짧아질수도, 길어질수도 있지만, 아예 법률에 이렇게 제정해놨다는 것 자체에 의미가 있다. DORA는 2023년도 1월에 발효되었으며, 2025년 1월부터 적용된다.

미국의 경우에는 앞서 언급했듯 레드팀 관련 평가 체계나 프레임워크가 아직 없다. 하지만 자본주의의 나라 답게 집단 소송과 금융 치료를 피하고 싶은 업체들이 레드팀 서비스를 많이 받고 있다. 미국 내 최상위 정보보안 컨설팅 업체들이 10년 사이 레드팀 부서들을 신설한 이유다. 어떤 업체들이 제대로된 레드팀 서비스를 제공하는지 궁금하다면 IBM X-Force, TrustedSec, SpecterOps, Mandiant/Google, NetSPI, BishopFox, Rapid7, Fortra 등의 회사들을 참고한다.

<figure><img src="https://blog.sunggwanchoi.com/content/images/2024/03/image-2.png" alt=""><figcaption><p>IBM X-Force의 오펜시브 시큐리티 서비스 중 레드팀 (출처: https://www.ibm.com/downloads/cas/JZ38L39E)</p></figcaption></figure>

인하우스 레드팀들도 빼놓을 수 없다. 개인적으로 유럽에서는 일을 안해봐서 모르지만, 미국에서는 Fortune 100을 필두로 해 많은 대기업들에서 사내 레드팀을 운영하고 있다. 단순히 IT나 빅테크 뿐만 아니라, 다른 업계의 대기업들 또한 레드팀을 운영한다. 예를 들어 Walmart, Target, Wegmans 와도 같이 IT와는 좀 거리가 먼 업체들 또한 사내 레드팀을 최근 10년내 신설한 뒤 운영중에 있다.

## 국내 상황

<figure><img src="https://blog.sunggwanchoi.com/content/images/2024/03/south-korea.png" alt=""><figcaption></figcaption></figure>

국내에서는 아직까지 레드팀/공격자 시뮬레이션과 관련된 움직임이 없다. 컨설팅 업체들의 경우 종종 목표 기반 침투 테스트라던지 시나리오 기반 침투 테스트를 진행하기는 한다. 하지만 포괄적인 자산(외부망, 사내/내부망, 액티브 디렉토리, 소셜 엔지니어링, 모바일, OT, 등)을 대상으로 실제 공격자들의 TTP(피싱, 뷔싱, C2 프레임워크 사용, PostEx 툴링, EDR 우회, 지속성 유지, 페이로드 개발, 터널링을 통한 피벗, 횡적 이동)를 사용하며, 해외 기준으로 레드팀 서비스라고 부를 수 있는 프로젝트를 진행하는 업체는 아직까지는 없는 것 같다. (있으면 링크드인 메시지 제보 부탁드린다. 바로 이력서 넣겠습니다.)

이는 컨설팅 업체들의 기술력 부재라기 보다는, 레드팀과 관련된 개념 및 수요가 국내에 아직 존재하지 않기 때문이다. ISMS-P 인증 심사를 제외하면 오펜시브 시큐리티와 관련된 평가 체계, 인증 체계, 프레임워크, 법, 컴플라이언스는 없는 것으로 알고 있다. 대부분의 고객사들 또한 사이버 공격을 받아도 별 다른 금전적 영향이 없다보니 인증 심사를 제외한 실제 정보 보안에는 큰 신경을 쓰지 않고 있다. 그렇다보니 레드팀/공격자 시뮬레이션에 관련된 개념을 모르는 경우가 많고, 안다고 하더라도 해야한다는 강제성이 없기에 수요가 없다. ISMS를 제외하고 대부분의 매출이 취약점 연구, 제로데이 발견에 치중되어 있기에 레드팀은 커녕 모의해킹에도 신경쓰는 컨설팅 업체가 많지 않기도 하다.

하지만 이런 국내의 상황도 최근 조금씩 변화하고 있는 것 같다. 인하우스 레드팀을 만드는 대기업들이 하나 둘 씩 보이기 시작하고, 단순한 모의해킹을 넘어 침투테스트를 진행하는 컨설팅 업체들도 보이고 있다. 컨설팅 업체들은 아직까지 레드팀/공격자 시뮬레이션 부서를 만들거나 인력을 뽑고 있지는 않지만, 슬슬 관련된 사업들 생각하거나 구상하는 경우도 있는 것 같다. HTTPS, 클라우드의 도입, 오픈소스, 데브옵스, 정보보안, AI까지 쭉 보고 있노라면 국내 IT는 서구권에 비해 약 5년에서 10년 정도 뒤쳐진 것 같다는 생각을 했었다. 따라서 어떻게 보면 2010년 중반 서구권에서 레드팀이 도입된 이후 10년이 지난 지금, 국내에서도 레드팀 개념이 조금씩 퍼지게 되지 않을까 는 전망을 해본다.

## 마치며

이번 글에서는 레드팀/공격자 시뮬레이션과 관련된 전세계 트랜드, 평가 체계, 프레임워크, 법안에 대해서 살펴본 뒤, 이런 방안들이 왜 나왔고, 어떤 내용들이 들어가 있는지에 대해 알아봤다. 또한 현재 국내 상황과 앞으로의 미래에 대해서 간단하게 생각을 남겨봤다. 앞으로 어떻게 될지 모르겠지만, 적어도 오펜시브 시큐리티와 레드팀에 관련된 전세계적인 트렌드는 국내에 알려야한다고 생각해 글을 남긴다.

레드팀과 공격자 시뮬레이션, 그걸 넘어 정보보안의 봄이 올때까지,

Happy Hacking!

### 레퍼런스

* TIBER-EU: [https://www.ecb.europa.eu/pub/pdf/other/ecb.tiber\_eu\_framework.en.pdf](https://www.ecb.europa.eu/pub/pdf/other/ecb.tiber\_eu\_framework.en.pdf)
* CBEST: [https://www.bankofengland.co.uk/financial-stability/operational-resilience-of-the-financial-sector/cbest-threat-intelligence-led-assessments-implementation-guide](https://www.bankofengland.co.uk/financial-stability/operational-resilience-of-the-financial-sector/cbest-threat-intelligence-led-assessments-implementation-guide)
* FEER: [https://uploads-ssl.webflow.com/59d28ad983887e000196f803/5fecc27919c71c17d5a10851\_Financial Entities Ethical Red Teaming Framework.pdf](https://uploads-ssl.webflow.com/59d28ad983887e000196f803/5fecc27919c71c17d5a10851\_Financial%20Entities%20Ethical%20Red%20Teaming%20Framework.pdf)
* AASE: [https://abs.org.sg/docs/library/abs-red-team-adversarial-attack-simulation-exercises-guidelines-v1-06766a69f299c69658b7dff00006ed795.pdf](https://abs.org.sg/docs/library/abs-red-team-adversarial-attack-simulation-exercises-guidelines-v1-06766a69f299c69658b7dff00006ed795.pdf)
* DORA - Article 26: [https://www.digital-operational-resilience-act.com/Article\_26.html](https://www.digital-operational-resilience-act.com/Article\_26.html)
* How Red team testing frameworsk can enhance the cyber resilience of financial institutions: [https://www.bis.org/fsi/publ/insights21.pdf](https://www.bis.org/fsi/publ/insights21.pdf)
* IT World - DORA 관련 기사: [https://www.itworld.co.kr/news/173484](https://www.itworld.co.kr/news/173484)
