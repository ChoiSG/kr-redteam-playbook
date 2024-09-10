---
description: MITRE ATTACK - A0001
---

# 개념

초기 침투는 공격자들이 타겟 네트워크에 진입해 초기 발판을 구축하는 단계를 일컫는다. 초기 침투 공격을 거쳐 계정 정보를 획득하거나, 외부와 연결된 자산을 장악하거나, 내부망에 C2 비컨/에이전트를 배포해 C2 커뮤니케이션을 구축한다.&#x20;

MITRE ATTACK의 초기 침투 방법들은 [여기서 확인이 가능](https://attack.mitre.org/tactics/TA0001/)하다. 개인적으로 생각해본 초기 침투 공격은 다음과 같은 것들이 있다.&#x20;

1. 피싱 - 스피어피싱, 첨부파일, 템플릿 인젝션, Follina, MitM, 등
2. 비밀번호 스프레이 공격 (Password Spraying)
3. 크레덴셜 스터핑 공격 (Credential Stuffing)
4. 공급망 공격 (Supply Chain attack)
5. 공개 익스플로잇을 통한 외부 자산 장악 (Public Exploit)
6. SaaS, Cloud 등의 외부 서비스 악용 (Abusing External Services)
7. 제로데이 사용 (Zeroday)&#x20;
8. 내부자 포섭&#x20;
9. 물리 침투 후 로그 디바이스 설치 &#x20;

이외에도 MITRE ATTACK 프레임워크 기준으로 여러가지가 있지만, 이 프로젝트에서 모든 TTP를 구현할 수는 없다. 예를 들어 제로데이는 실습 하기엔 시간이 너무 오래 걸리고, 물리 침투 후 로그 디바이스 설치 등은 불법이라 실습이 불가능하다.&#x20;

따라서 이 플레이북에서는 안전하고 도덕적으로 실습이 가능한 초기 침투 방식에 대해서 알아본 뒤, 대응 방안을 알아본다.&#x20;

### 레퍼런스&#x20;

{% embed url="https://attack.mitre.org/tactics/TA0001/" %}

{% embed url="https://mgeeky.tech/warcon-2022-modern-initial-access-and-evasion-tactics/" %}

{% embed url="https://posts.specterops.io/hang-fire-challenging-our-mental-model-of-initial-access-513c71878767" %}
