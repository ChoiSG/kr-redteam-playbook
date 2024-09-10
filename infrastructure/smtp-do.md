# SMTP Gophish (Digital Ocean + Postfix )

이 페이지는 아래의 레퍼런스의 글들을 참고한 뒤 재구성한 페이지다. 모든 명령어와 코드는 레퍼런스 섹션을 참고했다.

**프로젝트 리포:** [https://github.com/ChoiSG/RTPSourceCodes/tree/main/iac/smtp-terraform/configs](https://github.com/ChoiSG/RTPSourceCodes/tree/main/iac/smtp-terraform/configs)

## 개요

메일 인프라는 다양한 방법으로 구성할 수 있지만, 대부분 다음과 같은 요소들을 사용한다.

* 도메인
  * 이메일용 도메인 1개
  * 피싱 페이지용 도메인 1개
* 이메일 서버
  * 메일 발송 서버 - Gophish 등
  * 메일 서버 (Postfix) 혹은 메일 서비스 프로바이더 (Amazon SES, Mailgun, O365)
* 클라우드 서비스 프로바이더
  * 도메인 레지스트라 - NameCheap, Godaddy, Route53, Cloudflare 등
  * 클라우드 - AWS, Azure, Digital Ocean, Linode, 등

이번 실습에서는 이메일용 + 피싱 페이지용 도메인을 합쳐 1개만 구입한 뒤 사용한다. 이메일 같은 경우는 이메일 서비스 프로바이더 (Amazon SES, Mailgun, O365, 등)을 사용하는 것이 일반적이긴 하나, 교육 목적으로 메일 서버를 직접 구축한 뒤 Postfix 등의 메일 서비스를 직접 설치 및 설정한다. 클라우드 서비스 프로바이더는 NameCheap + Digital Ocean 을 사용한다. 물론, 다른 서비스를 사용해도 상관없다.

## 실습 - 도메인

도메인 구입은 따로 설명하지 않는다. 사용하고자 하는 DNS 프로바이더에 접속한 뒤 도메인을 구입한다. 구입할 때 `Domain Privacy` - 도메인 프라이버시를 꼭 같이 구입한다. 도메인 프라이버시를 적용하지 않으면 도메인을 구입할 때 사용한 Registrant 및 다른 개인 정보들이 Whois 나 다른 서비스들을 통해 유출될 가능성이 있다. 이는 작전 보안 실패로 이어진다.

도메인과 관련된 중요한 정보들은 다음 페이지에서 확인한다 - https://www.레드팀.com/infrastructure/domain-categorization-reputation

* 도메인 이름
* 만료된 도메인
* 도메인 분류
* 도메인 신뢰도

도메인 레지스트라에서 도메인을 구입했다면 네임 서버를 클라우드 프로바이더로 설정한다. 예를 들어 이 실습에서는 NameCheap + DigitalOcean 을 사용하고 있기 때문에, NameCheap 으로 가서 도메인의 네임 서버를 DO의 네임서버 3개로 지정해주면 된다.

<figure><img src="../../.gitbook/assets/p1.PNG" alt=""><figcaption></figcaption></figure>

## 실습 - 메일 서버 (mail.domain.com)

클라우드 프로바이더로 가서 서버를 2대 생성한다. 고피시 서버는 1기가 이상의 메모리를 가지도록 설정한다.

* OS: 우분투 22.10
* 종류: Basic (Plan Selected)
* CPU: Regular + SSD
* 메일 서버 가격: 4달러 / 달
* 고피시 서버 가격: 6달러 / 달 (1gb 메모리 - 필수!)
* 인증 방식: SSH 키
* 호스트이름: mail (메일 릴레이 서버), login (gophish 서버)

Droplet (드롭렛)을 생성한 뒤 드롭렛의 이름을 사용할 DNS A 레코드로 바꿔준다. 예를 들어 이 메일 서버의 이름은 `mail.domain.com` 이 될 것이니 그렇게 바꿔주면 된다. 이는 추후 도메인 PTR 레코드를 설정하는데 사용될 것이나, 지금은 크게 신경쓰지 않아도 된다.

<figure><img src="../../.gitbook/assets/p3.PNG" alt=""><figcaption></figcaption></figure>

## 메일 서버 설정

메일 서버에는 다음의 서비스/개념들을 설치한 뒤 설정한다.

* 이메일 서비스 - Postfix
* SPF - 없음 (postfix-policyd-spf-python 패키지는 들어오는 메일의 SPF를 체크하지만, 피싱 이메일 서버는 들어오는 이메일들까지 세세하게 검증할 필요가 없기 때문에 패스한다)
* DKIM - Opendkim, Opendkim-tools
* Certbot - Certbot
* `/etc/postfix/master.cf` - 아래 Postfix 헤더 설정을 적용
* `/etc/postfix/header_checks` - 메일 발송 서버의 헤더들을 검열
* `/etc/postfix/main.cf` - Postfix 서비스에 관련된 일반적인 설정
* `/etc/opendkim.conf` - DKIM을 설정하는데 필요한 기본 설정
* `/etc/opendkim/keys/<domain>/mail.txt` - DKIM 관련 DNS TXT 레코드에 들어갈 내용
* `/etc/opendkim/keys/<domain>/mail.private` - DKIM에 사용될 비공개 키

설정할 것이 많아보이지만, 막상 해보면 또 그렇지도 않다. SPF, DKIM, DMARC 등에 관련된 내용은 추후 다른 페이지에서 설명한다.

이번 실습에서는 먼저 수동적으로 모든 것을 설정한다. 추후 다른 페이지에서 Terraform과 Ansible을 이용한 IoC를 제작해 설정을 자동화한다.

1. 먼저 패키지들을 설치한다.

```
export DEBIAN_FRONTEND=noninteractive; apt update && apt-get -y -qq install socat postfix opendkim opendkim-tools certbot

hostnamectl set-hostname mail 
```

2. 이후 postfix의 `main.cf` 와 기본적인 설정들을 `postconf` 를 통해 설정해준다. 주의할 점은 SMTP 릴레이를 허용하는 화이트리스트인 `mynetworks` 와 메일을 받을 때 최종 목적지를 설정하는 `mydestination` 이다.

```
myhostname="mail.koreambtihealth.com"
domain="koreambtihealth.com"
ip=`curl http://ipinfo.io/ip`
gophiship="64.227.18.187"

echo $domain > /etc/mailname 
echo $ip $domain > /etc/hosts 

postconf -e myhostname=$myhostname 
postconf -e milter_protocol=2
postconf -e milter_default_action=accept
postconf -e smtpd_milters=inet:localhost:12345
postconf -e non_smtpd_milters=inet:localhost:12345
postconf -e mydestination="$domain, $myhostname, localhost.localdomain, localhost"
postconf -e mynetworks="127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 $gophiship"
```

3. OpenDKIM 을 설정한다

```
mkdir -p /etc/opendkim/keys/$domain 
cd /etc/opendkim/keys/$domain 

# Create dkim TXT record 
opendkim-genkey -t -s mail -d $domain 
cat mail.txt | tr -d '\n" ' | grep -o 'v=DKIM1.*' | cut -d ';' -f 1-5 | tr '\t' ' ' | tr -s ' ' | tr -d ')' > /root/dkim.txt 
chown opendkim:opendkim mail.private 

# Configure necessary files - KeyTable, SigningTable, default/opendkim, TrustedHosts 
echo mail._domainkey.$domain $domain:mail:/etc/opendkim/keys/$domain/mail.private > /etc/opendkim/KeyTable 
echo *@$domain mail._domainkey.$domain > /etc/opendkim/SigningTable 
echo SOCKET=\"inet:12345@localhost\" >> /etc/default/opendkim 
echo $gophiship > /etc/opendkim/TrustedHosts 
echo *.$domain >> /etc/opendkim/TrustedHosts 
echo localhost >> /etc/opendkim/TrustedHosts
echo 127.0.0.1 >> /etc/opendkim/TrustedHosts
```

4. 필요 설정 파일들을 덮어씌운 뒤, 리부팅한다.

* 각 사용되는 설정파일들은 프로젝트 리포를 확인한다

[https://github.com/ChoiSG/RTPSourceCodes/tree/main/iac/smtp-terraform/configs](https://github.com/ChoiSG/RTPSourceCodes/tree/main/iac/smtp-terraform/configs)

```
mv /opt/RTPSourceCodes/ioc/phishing/configs/header_checks /etc/postfix/header_checks 
mv /opt/RTPSourceCodes/ioc/phishing/configs/master.cf /etc/postfix/master.cf
mv /opt/RTPSourceCodes/ioc/phishing/configs/opendkim.conf /etc/opendkim.conf 
```

### DNS 설정

DNS 프로바이더로 가서 SPF, DKIM, DMARC, rDNS 설정을 해준다. 이 실습에서는 DO를 사용한다.

* SPF - 메일 서버의 아이피 주소를 사용한다
* DMARC - 아래의 DMARC 레코드를 그대로 쓴다
* DKIM - `/root/dkim.txt` 에 저장해놨던 문자열을 사용한다
* rDNS - 프로바이더마다 다르지만, DO의 경우 드롭렛의 이름을 FQDN으로 바꾸면 알아서 설정된다.

```
TYPE RECORD HOSTNAME VALUE 
(phish) A login <gophish 서버 IP 주소> 
(mail) A mail <mail서버 IP주소> 
(mail) MX <domain.com> mail.<domain.com>
(spf) TXT "@" v=spf1 a mx ip4:<mail서버 IP> ~all  (redirector) (ttl=60)
(dmarc) TXT _dmarc v=DMARC1; p=reject (ttl=60)
(dkim) TXT mail._domainkey <cat /root/dkim.txt> (ttl=600) 
(rDNS/PTR) < rename droplet with FQDN of the mail/smtp server > 
```

## 중간 점검

여기까지 진행했다면 중간 점검을 통해 메일 서버와 SPF, DKIM, DMARC 등의 이메일 관련 TXT 레코드가 잘 생성되었는지 확인한다. 한번에 확인할 수 있는 가장 빠르고 효율적인 방법은 `mail-tester.com` 을 사용하는 것이다. 방문한 뒤 이메일 주소를 받고, 해당 이메일 주소로 이메일을 보내본다.

현재 이메일 서버는 Gophish 서버에서만 접속 + 발송 가능하게끔 설정해놨기 때문에 gophish 서버에 접속 한 뒤, 이메일 서버를 사용해본다.

```
// login == gophish 서버다 
└─# ssh -i id_rsa root@login.koreambtihealth.com

root@login:~# telnet mail.koreambtihealth.com 25
Trying 143.198.179.236...
Connected to mail.koreambtihealth.com.
Escape character is '^]'.
220 mail.koreambtihealth.com ESMTP Postfix (Ubuntu)
helo koreambtihealth.com
250 mail.koreambtihealth.com
mail from: contact@koreambtihealth.com
250 2.1.0 Ok
rcpt to: test-rcwa6a0at@srv1.mail-tester.com
250 2.1.5 Ok
data
354 End data with <CR><LF>.<CR><LF>
to: test-rcwa6a0at@srv1.mail-tester.com <test-rcwa6a0at@srv1.mail-tester.com>
from: contact <contact@koreambtihealth.com>

Hello mail tester, this is a testo thing!
.
250 2.0.0 Ok: queued as 170167C58D
quit
221 2.0.0 Bye
Connection closed by foreign host.
```

<figure><img src="../../.gitbook/assets/p4-good-score.PNG" alt=""><figcaption></figcaption></figure>

점수가 높게 나오건 낮게 나오건 일단 중요한 섹션은 `You're Properly Authenticated` 이 부분이다. 이 부분의 SPF, DKIM, DMARC, rDNS 섹션에 모두 체크마크가 있다면 성공한 것이다. 만약 없다면 DNS 레코드 및 DKIM 문자열 등을 다시 한 번 체크한다.

전반적인 테스트가 아니라 부분별로 설정을 확인하고 싶다면 mxtoolbox 및 nslookup 등을 이용한다.

SPF, DKIM, DMARC 확인

* https://mxtoolbox.com/SuperTool.aspx?action=spf
* https://mxtoolbox.com/SuperTool.aspx?action=dkim
* https://mxtoolbox.com/SuperTool.aspx?action=dmarc

Nslookup 으로 DNS 레코드 확인

```
nslookup -type=MX domain.com
nslookup -type=A mail.domain.com 
nslookup -type=TXT domain.com
nslookup -type=TXT mail._domainkey.domain.com
nslookup -type=TXT _dmarc.domain.com 
```

## Gophish 서버 설정

메일 서버의 설정이 끝났다면 고피시 서버를 설정한다. 고피시 서버에는 악용을 막기 위한 잘 알려진 IOC 들이 있기 때문에 서버를 설치하기 전 IOC 부터 먼저 제거한다. 이 부분은 Sprocketsecurity 사의 블로그(https://www.sprocketsecurity.com/resources/never-had-a-bad-day-phishing-how-to-set-up-gophish-to-evade-security-controls)를 참고했다.

```
apt update -y ; apt install golang-go gcc -y 
cd /opt 
git clone https://github.com/gophish/gophish.git
cd ./gophish 

# Opsec changes 
find . -type f -name "config.go" -exec sed -i 's/const ServerName = "gophish"/const ServerName = "IGNORE"/g' {} + 
find . -type f -name "campaign.go" -exec sed -i 's/const RecipientParameter = "rid"/const RecipientParameter = "keyname"/g' {} + 
find . -type f -exec sed -i 's/X-Gophish-Contact/X-Contact/g; s/X-Gophish-Signature/X-Signature/g' {} +

go build 
```

로컬 포트 포워딩을 한 뒤 고피시를 실행한다. 고피시 실행 뒤 나오는 비밀번호를 이용해 호스트에서 `https://localhost:3333` 로 접속한다.

```
ssh -i id_rsa root@login.koreambtihealth.com -L 3333:127.0.0.1:3333
./gophish 
https://localhost:3333
```

## Gophish 설정

Sending Profile, Landing Page, Email Template, Users & Group을 모두 설정해준 뒤, Campaign을 진행한다.

1. Sending Profile

```
Name: sending-profile-test
SMTP From: contact@domain.com 
Host: mail.domain.com:25
Username/Password: empty since we have open relay for our gophish server
Email Headers: 
- X-Mailer: Microsoft office outlook, build 17.551210
- Date: Sun, 02 Apr 2023 18:30:00 -0500
```

2. Landing Pages

```
Name: landing-page-test
HTML: test 
Capture Submitted Data 
Capture Passwords 
Redirect to: http://www.google.com 
```

3. Email Templates

```
Name: template-test
Envelope Sender: noreply <contact@koreambtihealth.com>
Subject: Testo Fish for Red Team Playbook!
Text: Hello world, this is a testo! {{.URL}}
Add Tracking Image 
```

4. Users & Group

```
Group Name: group-test
<add emails and stuffs> 
```

5. Campaign 을 시작한다.

```
Name: Campaign-test
Email Template: template-test
Landing Page: test
URL: http://login.domain.com (http://login.koreambtihealth.com)
Sending Profile: sending-profile-test
Groups: groups-test
```

<figure><img src="../../.gitbook/assets/p5-campaign.PNG" alt=""><figcaption></figcaption></figure>

## 실행

고피시를 통해 캠페인을 진행하면 왠만한 이메일 프로바이더의 스팸 필터를 거쳐 타겟의 이메일 inbox 안으로 이메일이 도착한 것을 볼 수 있다. 텍스트 기반의 이메일도 나쁘지 않지만, 현업에서 피싱 이메일 시뮬레이션을 할때는 HTML이나 잘 알려진 타사들의 이메일을 쓰기도 하니, 이렇게 연습해보는 것도 나쁘지 않다.

<figure><img src="../../.gitbook/assets/p6-kokoa-1.PNG" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/p7-kokoa-2.PNG" alt=""><figcaption></figcaption></figure>

HTML에 들어가는 모든 링크는 고피시 링크 `{{.URL}}` 로 바꿔 테스트 페이지로 리다이렉트 시키거나 Evilginx2 페이지로 리다이렉트 시킬 수 있다. 예를 들어 위의 비밀번호 재설정을 누르면 Evilginx2 에서 준비해둔 리버스 프록시 페이지로 유저를 리다이렉트 시켜 [aitm.md](../../initial-access/aitm.md "mention") 을 실행하는 식이다.

## 마치며

개요에서도 언급했듯 이 페이지에서는 메일 서버와 고피시 서버를 설정하는 법에 대해서 다뤄봤다. 추후 테라폼이나 엔서블을 이용한 자동화나, Evilginx2와 고피시를 같이 이용하는 방법에 대해서는 추후 다른 페이지에서 다룬다.

참고로 위 방법을 이용해 실제 사이버 공격을 실행한 스키디가 있다면 행운을 빈다. 일부러 쉽게 잡힐 수 있는 IOC 2개 정도를 지우지 않았기 때문에 일반적인 이메일 게이트웨이를 사용하는 블루팀이라면 24시간 내에 본인을 찾아낼 것이다.

### Reference

* https://www.ired.team/offensive-security/red-team-infrastructure/automating-red-team-infrastructure-with-terraform
* https://github.dev/b1gbroth3r/red-team-infrastructure-example
* https://www.mail-tester.com/
* https://www.sprocketsecurity.com/resources/never-had-a-bad-day-phishing-how-to-set-up-gophish-to-evade-security-controls
* https://www.mogozobo.com/?p=3685
