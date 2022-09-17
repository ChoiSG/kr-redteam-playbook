# Active Directory Certificate Services (ADCS)

> 이 페이지와 하위 페이지들에 있는 ADCS와 관련된 개념 및 공격 방식은 모두 SpecterOps 사의 Will Schroeder 분과 Lee Christensen 분이 2021년에 작성하신 "[Ceritfied Pre-Owned](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf)" 라는 백서를 기반으로 만들어졌습니다. 더 자세하게 알아보고 싶으신 분들은 레퍼런스 섹션의 백서를 참고해주시기 바랍니다.

### 개념&#x20;

Active Directory Certificate Services (이하 ADCS) 는 마이크로소프트 사의 액티브 디렉토리 공개 키 기반구조 (Public Key Infrastructure; PKI) 구현 방법이자 서비스이다. ADCS는 액티브 디렉토리내에서 다음과 같은 기능을 갖는다:&#x20;

* 공개 키 암호화&#x20;
* 디지털 인증서 발급 및 관리&#x20;
* 사용자 인증&#x20;
* 서버 인증&#x20;
* 파일 시스템 암호화&#x20;

이 중 내부망 침투테스터들에게 중요하고 악용 가능한 기능은 바로 사용자 인증이다. 액티브 디렉토리에서 가장 많이 사용되는 사용자 인증 방식에는 크게 세 가지가 있다.&#x20;

* NTLM 사용자 인증: LM 혹은 NT 해시 등을 이용
* Kerberos (커버로스) 인증: TGT, TGS 등의 티켓들을 이용&#x20;
* ADCS 인증: 공개 키를 기반으로 한 인증서/서명서 (Certificates) 를 이용&#x20;

NTLM 인증 방식이 relay 가 가능하고, 커버로스과 관련된 다양한 공격 방식 (커버로스팅, AS-REPRoasting, Delegation 공격 등)이 있는 것 처럼, ADCS의 사용자 인증 방식 또한 다양한 방식으로 공격할 수 있다. 공격자들은 ADCS를 통해 권한 상승 공격과 도메인/계정 지속성 공격을 할 수 있지만, 레드팀 플레이북에서는 권한 상승 공격만 정리해놓는다.&#x20;

### 용어 정리&#x20;

ADCS와 관련된 공격 방법을 알아보기 전 ADCS 용어들을 정리한다.&#x20;

* **CA / 인증기관 (Certificate Authority):** 공개 키 기반구조의 인증서를 발급 및 관리하는 주체 (액티브 디렉토리에서는 "서버")&#x20;
* **인증서 (Certificate):** 공개 키 암호 알고리즘이 적용된 인증서. 인터넷에서는 TLS/SSL, 전자상거래, 전자 서명등을 위해서 사용된다. 액티브 디렉토리에서는 사용자 인증, 코드 서명, 파일 시스템 암호화, 서버 인증 등에 사용된다. `X.509` 포멧을 이용한다.&#x20;
* **인증서 양식 (Certificate Template):** 발급한 인증서들의 양식. 특정한 설정들과 정책들로 이뤄져있다.&#x20;
* **인증서 서명 요청 (Certificate Signing Request):** 인증서를 발급 받기 위한 요청. 대부분 CA로 보낸다.&#x20;
* **주체 (Subject):** 인증서를 발급 받는 주체&#x20;
* **SAN (Subject Alternative Name):** 주체의 또 다른 이름 (alias)&#x20;
* **EKU (Extended Key Usages):** 인증서 용도 (코드 서명, 파일 시스템 암호화, 이메일 암호화, 클라이언트 인증, 서버 인증, 스마트카드 로그인, 등.)&#x20;
* **ESC 1 \~ 8 (Escalation (Attack) 1\~8):** "Certified Pre-Owned" 백서에서 Will Schroeder분과 Lee Christensen 이 발견한 ADCS 권한 상승 공격 방식 1\~8&#x20;

### 실무 연관

액티브 디렉토리를 사용하는 회사라면 매우 높은 확률로 ADCS가 설정되어 있는 경우가 많다. 그리고 또한 안타깝게도 매우 높은 확률로 CA 서버가 잘못 설정 되어있거나, ADCS 서비스가 잘못 설정 되어 있거나, 인증서 양식이 잘못 설정 되어 있거나, 셋 다 잘못 설정(...) 되어 있는 경우가 많다.&#x20;

CA 서버, ADCS 서비스, 인증서 양식 중 하나라도 잘못 설정 되어 있다면 높은 확률로 도메인 내 그 어떤 유저라도 몇 분 사이에 도메인 관리자 권한을 획득한 뒤 도메인 장악을 할 수 있게 된다. 따라서 안전한 내부망 네트워크를 구축하기 위해 ADCS 공격 방식과 대응 방식에 대해 알아본다.&#x20;

레드팀 플레이북에서는 실무에서 많이 볼 수 있는 ESC1, ESC6, ESC8 에 대해서 알아본다.&#x20;

\<TODO: 링크>&#x20;



### 레퍼런스&#x20;

{% embed url="https://posts.specterops.io/certified-pre-owned-d95910965cd2" %}

{% embed url="https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf" %}

