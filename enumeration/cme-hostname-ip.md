# CME - 호스트이름과 IP주소

내부망 모의해킹 중 도메인 유저를 장악했다면, 액티브 디렉토리내의 모든 호스트들의 리스트를 알아낼 수 있다. 블러드하운드로 LDAP 데이터를 뽑아와도 되지만, 이때 LDAP 데이터는 오래된 데이터들도 있기 때문에 거기서 얻는 호스트 리스트는 불완전한 경우가 많다.&#x20;

CME를 이용해 현재 액티브 디렉토리내에 DNS 레코드가 있고, IP주소가 제대로 설정되어 있는 모든 호스트들을 빠르게 뽑는 방법이 있다.&#x20;

또한, 고랭 툴인 MapCidr를 이용해 호스트 리스트를 기반으로 타겟 CIDR 까지 만들어낼 수 있다.&#x20;

```
go install -v github.com/projectdiscovery/mapcidr/cmd/mapcidr@latest
```

### 타겟 CIDR

```
cme ldap dc01.choi.local -u Administrator -p 'Password123!' -M get-network
cat /root/.cme/logs/choi.local_network_2023-02-28_232638.log | ~/go/bin/mapcidr -aa -silent | ~/go/bin/mapcidr -a -silent
```

### 타겟 호스트이름 + IP 주소

```
cme ldap dc01.choi.local -u Administrator -p 'Password123!' -M get-network -o ALL=true
```
