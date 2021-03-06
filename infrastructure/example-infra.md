# 예시 인프라

다음은 이 플레이북에서 쓰일 레드팀 인프라다. 개념증명과 교육용으로만 쓰일 것이기 때문에 최대한 간단하게 만들었다.&#x20;

![](../.gitbook/assets/홈랩-구현도.drawio\(2\).png)

앞서 "개념" 페이지에서 알아봤던 레드팀의 리소스들 (팀 서버, 도메인, 리다이렉터, 네뷸라)과 타겟 공간의 `choi.local` 과 `pci.choi.local` 도메인 또한 준비했다.&#x20;

### 도덕적인 보안 연구&#x20;

안전하고 도덕적인 보안 연구를 위해 모든 리소스들은 본인 소유의 리소스들로만 준비했다. 예를 들어 메인 호스트안에 있는 팀 서버, `choi.local`, `pci.choi.local`, AWS 안의 리다이렉터, 네뷸라 등대서버, 집 공유기와 방화벽, 그리고 공격자 도메인까지, 모두 내가 소유하고 만든 리소스들이며, 본인의 집 IP주소를 제외하고는 인터넷과는 모두 연결이 차단되어 있는 리소스들이다.&#x20;

다음 페이지들에서는 실제로 팀 서버, 도메인, 네뷸라, 리다이렉터 서버등을 어떻게 구현하는지에 대해서 알아본다.&#x20;

