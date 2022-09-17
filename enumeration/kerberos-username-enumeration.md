---
description: Discovery (TA0007) + Account Discovery (T1087)
---

# 커버로스 유저 이름 정보수집

### 개념&#x20;

윈도우의 사용자 인증 프로토콜 중 하나인 커버로스 (Kerberos) 프로토콜은 특정 유저 이름이 존재하는지에 대한 요청에 존재한다/존재하지 않는다라는 응답을 보낸다. 더 정확하게 설명하자면, 공격자가 유저 이름만 가지고 `KRB_AS_REQ` (커버로스 Authentication Service 요청) 을 보내면 도메인 컨트롤러는 다음과 같이 응답한다.

* 유저 계정 존재 - `KRB5KDC_ERR_PREAUTH_REQUIRED` - 유저가 존재하지만 커버로스 인증 6단계 중 첫 2단계인 Pre-Authentication 이 필요하다는 에러를 반환한다.&#x20;
* 유저 계정 존재 X - `KRB5KDC_ERR_C_PRINCIPAL_UNKNOWN` - 액티브 디렉토리 내 존재하지 않는 Principal 이라는 에러를 반환한다.&#x20;

에러 메시지들의 차이 이용해 공격자들은 준비해놓은 유저 이름 리스트를 이용해서 어떤 유저 이름이 존재하는지 존재하지 않는지 확인할 수 있다 - 이를 커버로스 유저 이름 정보수집 (Kerberos Username Enumeration) 이라고 부른다.&#x20;

커버로스 유저 이름 정보수집을 통해 얻은 도메인 유저 이름들은 다음과 같은 공격에 사용될 수 있다.&#x20;

1. [비밀번호 스프레이](../credential-access/password-spraying.md) (Password Spraying)&#x20;
2. 비밀번호 브루트포스 (Password Bruteforcing) - 실무에서는 계정 잠금 정책 (Account Lockout Policy) 때문에 잘 사용하지 않는 편이다.&#x20;
3. [AS-REP Roasting](../credential-access/kerberos/as-rep-roasting.md)&#x20;
4. [커버로스팅](../credential-access/kerberos/kerberoasting.md) (Kerberoasting) - 이미 도메인 유저 맥락을 갖고 있다면 커버로스팅을 진행할 수 있다 &#x20;

### 실습&#x20;

1. 브루트포스 할 유저 이름 리스트를 구한다. 대부분의 액티브 디렉토리 유저 이름은 직원 이름 (ex. `s.choi@choi.local`) 인 경우가 많지만, 서비스 유저 계정 (`svc-scanner@choi.local`, `o365-mfa@choi.local`) 들의 경우 비슷비슷하게 설정해놓는 경우가 많아 브루트포스를 통해 발견할 수도 있다.&#x20;

수많은 유저 이름 리스트가 존재하지만, 실습에서는 간단한 리스트를 이용한다.&#x20;

```
wget https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt .
```

&#x20;2\. [Kerbrute](https://github.com/ropnop/kerbrute) 를 통해 브루트포스를 진행한다&#x20;

```
wget https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64 .
chmod +x ./kerbrute_linux_amd64
mv kerbrute_linux_amd64 kerbrute

./kerbrute userenum -d <domain> --dc <DC-IP> <userlist-file> | tee <output-filename> 
```

![](<../.gitbook/assets/image (4) (1) (1).png>)

3\. 알아챈 유저 이름 리스트를 갖고 Password Spraying, AS-REPRoasting, Kerberoasting 등을 진행한다.&#x20;

### 대응 방안&#x20;

* 원천적으로 커버로스 유저 이름 정보수집을 막는 방법은 없다 - 적어도 마이크로소프트사나 커버로스를 만든 MIT에서 커버로스 프로토콜 자체를 바꾸지 않는 한 없다.&#x20;
* 단, 특정 IP주소나 도메인 유저 맥락을 통해 짧은 시간 안에 커버로스 유저 이름 확인 요청을 보내는 공격자를 탐지할 수 있는 방법은 있다. 방어자들은 탐지 후 공격자를 특정하여 격리하는 식으로 이 공격에 대응할 수 있다.&#x20;





### 레퍼런스&#x20;

{% embed url="https://research.nccgroup.com/2015/06/10/username-enumeration-techniques-and-their-value/" %}

{% embed url="https://research.splunk.com/endpoint/d82d4af4-a0bd-11ec-9445-3e22fbd008af/" %}

{% embed url="https://github.com/attackdebris/kerberos_enum_userlists" %}
