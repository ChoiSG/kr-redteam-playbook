# 자산 정보 수집

자산 정보 수집 (Asset Enumeration, Public Asset Enumeration, Initial Reconnaissance, Publicly-facing Inventory Management, etc.) 은 타겟 기관의 인터넷에 연결된 호스트들을 찾고 맵핑하는 단계를 일컫는다. 외부망 모의해커에게 "고객사" 라는 이름 하나만 주어졌을 때, 이 고객사가 운영중인 인터넷에 연결된 자산 (워크스테이션, 서버, 클라우드 자산, etc) 을 찾는 단계가 바로 자산 정보 수집 단계다.

수집하는 정보들 중 일부는 다음과 같다:

* 도메인과 서브도메인 정보 수집
* IP 주소 범위 (IP Range) 를 통해 범위를 알아낸 뒤 간단한 스캔
  * WHOIS
  * Autonomous System Number (ASN)
  * NetblockTool
* 자산 검색 엔진/인터넷 취약점 스캐너등을 이용한 정보 수집
  * Shodan
  * Censys.io
  * (이외 수많은 비슷비슷한 서비스들...)

### 도메인과 서브도메인

"고객사" 이름 하나만 주어졌을 때 관련된 자산을 찾기 가장 편한 방법은 바로 도메인과 서브도메인을 이용해 호스트들을 찾는 것이다. 도메인과 관련된 서브 도메인들은 다음과 같은 방법으로 찾을 수 있다.

1. 검색 엔진 - "고객사" 이름을 쳤을 때 어떤 도메인들이 나오는가?
2. TLS/SSL 인증서 - "고객사" 이름과 비슷한 인증서들이 등록되거나 파기된 레코드가 존재하는가?
3. 브루트포스 - \*.고객사.com 도메인이 존재하는지 알아보기 위해 "\*" 자리에 도메인 이름에 자주 사용되는 단어들(vpn, info, internal, github, etc.)을 넣어 호스트가 반응 하는 지 등을 알아본다.

\#3번 도메인 브루트포스는 타겟 호스트와 네트워크적 연결이 되고, 공격의 의도가 있다고 판단될 수 있기 때문에 "사이버 공격"으로 간주될 수 있다. 따라서 외부망 모의해킹 등의 이용 허락이 없다면 절로 사용해서는 안된다.

#### 실습

도메인 정보 수집을 수동으로 해도 되지만, 시간이 너무 오래 걸린다. 따라서 실습에서는 amass 와 theHarvester 두 개의 툴을 이용해 서브도메인들을 알아보자.

Amass는 `-passive` 플래그를 사용할 경우 오로지 OSINT만 이용해 타겟 호스트에 접근하지 않고 서브도메인을 정보 수집하는 도구다. 유의할 점은 타겟 호스트에 접근하지 않기 때문에 반환된 서브도메인을 사용하고 있는 호스트가 실제로 아직까지 작동하고 있는지 아닌지 모른다는 것이다. 이는 추가 스캔이나 추후 알아볼 스크린샷을 찍는 기법등으로 알아낼 수 있다.

```
# amass enum -passive -d cafe.naver.com -o naver-subdomain.txt

< ... > 
sports.news.cafe.naver.com
ws-aicall.store.cafe.naver.com
naverapp.m.cafe.naver.com
bulletin.nexon.game.cafe.naver.com
travelsearch-api.cafe.naver.com
m.search.cafe.naver.com
admin.commentbox.cafe.naver.com
< ... >
```

theHarvester 또한 다양한 OSINT와 인터넷 포트 스캐너 서비스들 (Shodan, Censys, Security Trails) 등을 이용해 호스트 뿐만 아니라 전반적인 OSINT를 진행해주는 툴이다. theHarvester 의 진정한 위력은 바로 서비스나 검색 엔진들의 유료 API키가 있을 때 발휘된다. API 키만 주면 알아서 API 요청, 반환, 그리고 파싱을 진행한다. 이번 실습은 본인이 돈이 없기 때문에 API키는 지정하지 않고 진행한다.

```
# theHarvester -d cafe.naver.com -b all -f cafe-naver-com-theHarvester

< ... > 
[*] Hosts found: 53
---------------------
253am.cafe.naver.com
bridge.cafe.naver.com:223.130.195.199
chat.cafe.naver.com
downapi.cafe.naver.com:223.130.192.249, 223.130.192.250
g.cafe.naver.com:210.89.168.65, 210.89.168.33
golda.cafe.naver.com:223.130.192.249, 223.130.192.250
< ... >
```

### IP 주소 범위 (IP Ranges)

어느정도 기업 역사가 길거나 규모가 큰 대기업들은 특정 공인 IP 주소 범위를 할당 받는다. 중소기업이나 중견 기업등의 규모가 작은 회사들은 할당 받지 않고, 클라우드나 VPS 등의 인터넷 자산을 사용하는 경우가 많다. 물론 대기업들 또한 2022년 기준으로 인터넷에 연결되는 자산들은 DMZ + 공인 IP 주소 범위를 사용하지 않고 그냥 클라우드에 올리는 경우도 많다.

전통적인 공인 IP 주소 범위는 WHOIS 나 Autonomous System Number (ASN) 을 검색한 뒤 찾아보면 알아낼 수 있다. WHOIS는 전세계에 있는 도메인 등록기관에 등록된 도메인 이름 혹은 회사 이름과 관련된 정보를 반환해준다.

```
# Asia Pacific 에 있는 마이크로소프트사의 정보 
└─# whois -h whois.apnic.net Microsoft                                                            
% [whois.apnic.net]                                                                                
% Whois data copyright terms    http://www.apnic.net/db/dbcopyright.html                          
                                                                                                  
% Information related to '58.246.69.164 - 58.246.69.167'                                          
                                                                                                  
% Abuse contact for '58.246.69.164 - 58.246.69.167' is 'hqs-ipabuse@chinaunicom.cn'               
                                                                                                  
inetnum:        58.246.69.164 - 58.246.69.167                                                     
netname:        Microsoft                                                                          
country:        cn                                                                                
descr:          Microsoft (China) Co., Ltd. 

< ... > 
```

예를 들어 위 예시의 경우 마이크로소프트사의 중국 관련 공인 IP 주소 중 일부가 `58.246.69.164 - 58.246.69.167` 레인지에 있음을 알 수 있다. 마이크로소프트사에 할당된 중국 IP주소가 4개 밖에 없는 것이 아니라 반환된 결과가 너무 길어 잘라냈다.

### 자산 검색 엔진 / 인터넷 포트 스캐너

경계선 보안 (Parameter Security)가 중요시 되면서 2010년대 초반 이후 다양한 인터넷 포트 스캐너 + 취약점 스캐너들이 만들어졌고, 이를 인덱싱한 서비스들이 많아졌다. 초반에는 IoT와 OT 중심으로 하다가 이제는 전반적인 자산 검색엔진이 된 Shodan, 그리고 포트 및 취약점 중심의 Censys.io 등이 있다.

인터넷에 찾아보면 가끔씩 "이상한 곳에서 포트 스캐닝 공격이 들어와요. 저 이제 해킹 당하는건가요?" 와 비슷한 질문들을 볼 수 있다. 대부분 그냥 Shodan/Censys 와 비슷한 서비스들에서 사용하는 포트 스캐너들이다.

이런 서비스들을 이용해 특정 기관의 어떤 호스트들이 있는지, 그리고 그 호스트들은 어떤 포트와 네트워크 서비스를 사용하고 있는지 정보 수집을 할 수 있다.

서비스들이 너무 많고 다양해 실습은 생략한다. 만약 각 서비스들에 방문해 유료 API키를 사고, API 사용법을 익히고, CLI 프로그램을 다운받아 사용하는게 귀찮다면, 위에서 살펴본 theHarvester 와 같은 툴들을 이용해 이런 서비스 들을 사용하는 것을 자동화하는 것도 좋다. 실제로 버그바운티 헌터들이 많이 사용하는 방법이기도 하다.

### 레퍼런스

{% embed url="https://github.com/NetSPI/NetblockTool" %}
