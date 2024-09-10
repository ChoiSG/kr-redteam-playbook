# 타겟 발견

초기 정찰 단계에서는 OSINT와 외부망 정찰을 통해 타겟 기관의 외부망 자산, 클라우드 자산, (서브)도메인, 모바일 앱, 회사 조직도, 인원 현황, 이메일 주소 등을 수집한다. 초기 정찰을 진행하며 앞으로 공격할 타겟들을 정리하는 기술적인 작업을 공격 표면 탐색/발견 (Attack Surface Discovery) 혹은 타겟 발견(Target Discovery)이라고도 한다.

타겟 발견 단계에서 집중적으로 수집하는 타겟의 종류는 다음과 같다.

1. 도메인과 서브도메인
2. IP주소
3. 클라우드 서비스 프로바이더 소속 IP 주소
4. 주요 네트워크 서비스와 포트
5. 유저/직원 이메일 주소
6. 메일 서비스/서버 및 이메일 게이트웨이
7. SaaS 및 PaaS 사용 여부

이외 OSINT를 통해 다른 종류의 자산과 자원들을 수집한다.

이 페이지에서는 특정 기관의 이름이 주어졌을 때 타겟 발견 단계를 통해 외부적으로 공격 가능한 타겟들을 정리해본다. 외부망 모의해킹 시 대부분 특정 도메인 및 아이피 범위가 제공되지만, 이번에는 아무런 정보 없이 레드팀을 진행한다고 가정해본다.

### 방법론

다음은 간단한 타겟 발견 방법론이다.

1. 루트 도메인 수집
2. 루트 도메인 기반으로 서브도메인 수집
3. 루트 도메인 기반으로 클라우드 관련 간단 DNS 브루트포스 진행
4. 수집한 서브도메인 DNS Resolution으로 IP 주소 확보
5. 확보한 IP주소 클라우드 프로바이더 소속 확인
6. 수집한 서브도메인 기반 STO 확인
7. 웹서버 확인
8. 웹서버 스크린샷 수집
9. 메일 서버 및 이메일 보안 확인
10. 클라우드 관련 OSINT 진행
    1. S3 bucket
    2. Azure Blob Storage
    3. Dark Web Leaks
    4. 클라우드 기반 SVN -> API 및 Access Token 수집
11. OSINT 진행
    1. OSINT 페이지 참고
12. 분산 스캔(Distributed Scanning)을 이용한 포트 스캔 진행

최근 Attack Surface Management SaaS나 프레임워크들이 많아지면서 위 방법론들을 자동화하는 경우가 많이 있다. 하지만 자동화된 솔루션들은 여러모로 작전보안에 위험할때가 많이 있다. 또한, 타겟 발견을 수동적으로 진행하면서 어느정도의 추론/추리가 들어가기 때문에 100% 자동화 하는 것은 그렇게 추천하지 않는다.

### 루트 도메인

"고객사" 이름 하나만 주어졌을 때 가장 먼저 해야하는 것들은 해당 고객사가 어떤 루트 도메인들을 가지고 있나 살펴보는 것이다. 특히 작은 기업이 아니라 계열사들을 가지고 있는 경우 다양한 루트 도메인들을 알아낼 수 있다.

1. 검색 엔진 - "고객사" 이름을 쳤을 때 나오는 도메인들
2. TLS/SSL 인증서 - "고객사" 이름과 연관된 SSL 인증서들과 해당 인증서가 적용된 (서브)도메인들을 찾아본다

예를 들어 다음과 같은 루트 도메인들이 나왔다고 가정한다.

```
# cat domains.txt 
example.com 
examplecorp.com 
example.im 
examplebank.com 
```

### 도메인과 서브 도메인

루트 도메인을 기반으로 서브도메인을 OSINT를 통해 수집한다. 이를 자동화 하기 위한 툴로 amass, theHarvester, subfinder 등을 이용한다.

```
amass enum -d domain.com -passive -o amass-domain.com 
theHarvester -d domain.com -b all -f domain.com.json 
subfinder -d domain.com -rl 30 -t 10 -silent -active -o subfinder-domain.com 

# 예시) 루트 도메인을 돌며 커맨드 실행 
for i in `cat domains.txt`; do amass enum -d $i -passive -o amass-$i; done 
for i in `cat domains.txt`; do theHarvester -d $i -b all -f theHarvester-$i.json ; done
for i in `cat domains.txt`; do subfinder -d $i -rl 30 -t 10 -silent -o subfinder-$i | tee subfinder-$i ; done

# 파싱 후 서브도메인 수집 
cat amass* > tmp.txt
cat subfinder* >> tmp.txt 
jq -r '.hosts[]' theHarvester-*.json >> tmp.txt
cat tmp.txt | sort -fuV > subdomains-unresolved.txt
```

### 서브도메인 테이크오버 (STO) 체크

DNS Resolution을 진행하기 전 간단하게 STO체크를 한다.

```
# dnsreaper (https://github.com/punk-security/dnsReaper.git)
python3 main.py file --filename subdomains-unresolved.txt 
```

### DNS Resolution과 아이피주소

서브도메인들을 모두 찾아냈다면 DNS resolution을 통해 아이피주소를 알아낸다.

```
# dnsx (https://github.com/projectdiscovery/dnsx)
dnsx -l subdomains-unresolved.txt -resp -a -cname -mx -txt -t 30 -rl 30 -silent -o dnsx-resolved.txt 

# 아이피주소만 가져오기 
cat dnsx-resolved.txt | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' | sort -fuV > rawips.txt 

# 서브도메인만 가져오기 
cat dnsx-resolved.txt | cut -d ' ' -f 1 | sort -fuV > subdomains-resolved.txt 
```

### DMZ 및 클라우드 확인

확보한 아이피주소들이 on-prem DMZ/데이터센터에 있는지 혹은 클라우드 자산인지 확인한다. 그 뒤, 서브도메인 + 아이피주소 + 클라우드 프로바이더 리스트를 생성한다.

```
# ip2provider (https://github.com/oldrho/ip2provider)
cat rawips.txt | python3 ip2provider.py | tee ipcloud.txt 

# 서브도메인, 아이피주소, 클라우드 프로바이더 체크 스크립트 (subipcloud.sh)
====================================
#!/bin/bash
if [ "$#" -ne 2 ]; then
    echo "Usage: $0 <ip2provider_file> <dnsx_file>"
    exit 1
fi

ip2provider_file="$1"
dnsx_file="$2"

while IFS= read -r line; do
    ip=$(echo $line | awk '{print $1}')
    grep -F "[$ip]" "$dnsx_file" | while read -r match; do
        subdomain=$(echo $match | cut -d' ' -f1)
        echo "$subdomain $ip ${line#* }" >> output.txt
    done
done < "$ip2provider_file"

echo '[+] output.txt created'
====================================

./subipcloud.sh ipcloud.txt dnsx-resolved.txt 
```

### 웹서버 확인 & 스크린샷

웹서버 빠른 확인

```
# httpx (https://github.com/projectdiscovery/httpx) 
cat subdomains-resolved.txt | httpx -silent -title -follow-redirects -status-code -delay 400ms -t 10 -rl 30 -random-agent -o httpx-alive-web.txt 
```

웹서버 스크린샷

```
# httpx 이용해도 되지만 gowitness 사용 
mkdir gowitness 

gowitness file -f subdomains-resolved.txt --delay 5 -P ./gowitness --timeout 10 --user-agent 'Mozilla/5.0 (Macintosh; Intel Mac OS X 11_15_7) AppleWebKit/557.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/517.36'
```

### Azure, O365 확인

```
https://login.microsoftonline.com/getuserrealm.srf?login=test@domain.com&xml=1
```

* O365 사용여부 확인
* Managed = AAD 사용중, PHS 혹은 PTA
* Federated = ADFS 사용중, AuthURL을 통해서 ADFS 서버 및 SSO 프로바이더 (OKTA, 등) 확인

### 이메일 서버 보안 확인

DNSX에서 수집한 MX와 TXT 레코드를 자세히 살펴본다.

* SPF, DKIM, DMARC 등의 레코드를 살펴보며 어느 도메인이 스푸핑 가능한 도메인인지 확인한다. 그 뒤, 수동적으로 체크하거나 서비스(https://dmarc-tester.com/)들을 이용해 스푸핑을 확인한다.
* 스푸핑이 가능한 도메인들은 나중에 소셜엔지니어링에 쓰면 좋으니 기록해둔다.
* MX레코드와 SPF를 통해 어떤 메일 서비스를 쓰고 있는지 확인한다.
  * O365, Amazon SES, Google Apps, Proofpoint, mailchimp, mandrill 등
* 이메일 보안 및 게이트웨이를 살펴본다

### 유저 이메일 확인

OSINT와 다크웹 서비스들을 이용해 유저들의 이메일을 확인한다.

* 깃헙리포 + gitleaks -> 이메일
* 다크웹 서비스
* 파일 메타데이터 (FOCA, 등)
* 링크드인

타겟 기관이 o365를 사용한다면 해당 이메일이 존재하는지 확인한다. REDACTED

### 분산 스캔을 통한 포트 스캔

TODO - 따로 페이지 생성

### S3 Bucket 및 Azure Blob Storage

```
# cloud_enum.py (https://github.com/initstring/cloud_enum)
python3 cloud_enum.py -k domain.com --disable-gcp | tee cloud_enum-domain.com 
```
