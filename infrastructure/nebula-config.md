# 네뷸라 설정

시작하기 앞서 이 페이지에 나오는 모든 정보 및 과정은 [이 레퍼런스](https://notes.huskyhacks.dev/blog/red-team-infrastructure-done-right)를 상당히 많이 참고했다는 점을 밝힌다. HuskyHacks 라는 분이 올린 블로그인데, 책임감 있는 레드팀 인프라 구축에 상당히 도움이 되는 블로그다. All credits goes to HuskyHacks, I do not claim any of this as my work. I'm simply practicing/mimicking his work to get some experience on creating a redteam infra.&#x20;



### 네뷸라 설치

네뷸라 등대서버, 리다이렉터에 먼저 네뷸라를 설치한다

```
# 등대 서버 
sudo apt update -y 
wget https://github.com/slackhq/nebula/releases/download/v1.5.2/nebula-linux-amd64.tar.gz -O nebula.tar.gz
tar -xvf nebula.tar.gz 


# 리다이렉터 
sudo apt update -y && sudo apt install -y socat 
wget https://github.com/slackhq/nebula/releases/download/v1.5.2/nebula-linux-amd64.tar.gz -O nebula.tar.gz
tar -xvf nebula.tar.gz 
```

이후 팀 서버에도 네뷸라를 설치하고 설정을 시작한다. 팀 서버에서는 네뷸라에서 가장 중요한 Certificate Authority 를 만든 뒤, 각 노드들에게 부여할 인증서 파일들을 생성한다.

```
# 팀 서버 
mkdir ~/redteam/nebula 
cd ~/redteam/nebula 
wget https://github.com/slackhq/nebula/releases/download/v1.5.2/nebula-linux-amd64.tar.gz -O nebula.tar.gz
tar -xvf nebula.tar.gz 

./nebula-cert ca -name "Korea MBTI Health, LLC"
./nebula-cert sign -name "lighthouse" -ip "192.168.100.1/24"
./nebula-cert sign -name "teamserver" -ip "192.168.100.2/24" -groups "teamservers"   
./nebula-cert sign -name "redirector" -ip "192.168.100.3/24" -groups "redirectors"
```

### 네뷸라 설정

이제 각 노드들이 사용할 CA의 인증서, 노드 인증서, 그리고 노드 키들을 만들었으니, 설정 파일을 만들어보자. 각 네뷸라 노드들의 설정 파일들은 다음과 같다:

* lighthouse-conf.yml
* redirector-conf.yml
* teamserver-conf.yml

설정 파일들의 방화벽 설정은 언제든지 바꿔도 되지만, AWS의 방화벽이 우선순위가 더 높기 때문에 네뷸라 방화벽을 바꿀때에는 AWS의 방화벽/Security Group도 바꿔야한다. 더 자세한 설정을 위해선 공식 [네뷸라 설정 파일](https://github.com/slackhq/nebula/blob/master/examples/config.yml)을 참고한다.

모든 설정파일안의 등대 서버 공인 아이피는 꼭 본인의 등대 서버 아이피로 바꿔주자.

**lighthouse-conf.yml**

```
pki:
  ca: /home/ubuntu/ca.crt
  cert: /home/ubuntu/lighthouse.crt
  key: /home/ubuntu/lighthouse.key

static_host_map:
  "192.168.100.1": ["<등대서버-공인-아이피>:4242"]

lighthouse:
  am_lighthouse: true

listen:
  host: 0.0.0.0
  port: 4242

punchy:
  punch: true

tun:
  disabled: false
  dev: nebula1
  drop_local_broadcast: false
  drop_multicast: false
  tx_queue: 500
  mtu: 1300
  routes:
  unsafe_routes:

logging:
  level: info
  format: text

firewall:
  conntrack:
    tcp_timeout: 12m
    udp_timeout: 3m
    default_timeout: 10m
    max_connections: 100000

  outbound:
    - port: any
      proto: any
      host: any

  inbound:
    - port: any
      proto: icmp
      host: any

    - port: 22
      proto: any
      cidr: 192.168.100.0/24
```

**redirector-conf.yml**

```
pki:
  ca: /home/ubuntu/ca.crt
  cert: /home/ubuntu/redirector.crt
  key: /home/ubuntu/redirector.key

static_host_map:
  "192.168.100.1": ["<등대서버-공인-아이피>:4242"]

lighthouse:
  am_lighthouse: false
  interval: 60
  hosts:
    - "192.168.100.1"

listen:
  host: 0.0.0.0
  port: 4242

punchy:
  punch: true

tun:
  disabled: false
  dev: nebula1
  drop_local_broadcast: false
  drop_multicast: false
  tx_queue: 500
  mtu: 1300
  routes:
  unsafe_routes:

logging:
  level: info
  format: text

firewall:
  conntrack:
    tcp_timeout: 12m
    udp_timeout: 3m
    default_timeout: 10m
    max_connections: 100000

  outbound:
    - port: any
      proto: any
      host: any

  inbound:
    - port: any
      proto: icmp
      host: any

    - port: 80
      proto: any
      host: any

    - port: 443
      proto: any
      host: any

    - port: 22
      proto: any
      cidr: 192.168.100.0/24
```

**teamserver-conf.yml**

팀 서버의 포트 80과 포트 443은 리스너로 사용될 예정이다. 따라서 인바운드 80/443을 열어두지만, 오로지 같은 네뷸라 네트워크에서만 포트 80과 443이 연결 가능하도록 방화벽을 설정한다. &#x20;

```
pki:
  ca: /root/redteam/nebula/ca.crt
  cert: /root/redteam/nebula/teamserver.crt
  key: /root/redteam/nebula/teamserver.key

static_host_map:
  "192.168.100.1": ["<등대서버-공인-아이피>:4242"]

lighthouse:
  am_lighthouse: false
  interval: 60
  hosts:
    - "192.168.100.1"

listen:
  host: 0.0.0.0
  port: 4242

punchy:
  punch: true

tun:
  disabled: false
  dev: nebula1
  drop_local_broadcast: false
  drop_multicast: false
  tx_queue: 500
  mtu: 1300
  routes:
  unsafe_routes:

logging:
  level: info
  format: text

firewall:
  conntrack:
    tcp_timeout: 12m
    udp_timeout: 3m
    default_timeout: 10m
    max_connections: 100000

  outbound:
    - port: any
      proto: any
      host: any

  inbound:
    - port: any
      proto: icmp
      host: any

    - port: 80
      proto: any
      cidr: 192.168.100.0/24

    - port: 443
      proto: any
      cidr: 192.168.100.0/24
```

### 네뷸라 실행

설정 파일들과 인증서들이 만들어졌다면 `ca.crt`, `.crt`, `.key`, `-conf.yml` 파일들을 각 서버들로 옮긴다.&#x20;

```
# 등대 서버로 필요한 파일 scp 

┌──(root㉿kali)-[~/redteam/nebula]
└─# scp -i /root/redteam/sshkeys/aws-lighthouse-ssh ca.crt lighthouse.crt lighthouse.key lighthouse-conf.yml ubuntu@lighthouse:/home/ubuntu 
ca.crt 
lighthouse.crt 
lighthouse.key 
lighthouse-conf.yml

# 리다이렉터 서버로 필요한 파일 scp 

┌──(root㉿kali)-[~/redteam/nebula]
└─# scp -i /root/redteam/sshkeys/aws-redirector-ssh ca.crt redirector.crt redirector.key redirector-conf.yml ubuntu@redirector:/home/ubuntu 
ca.crt 
redirector.crt 
redirector.key
redirector-conf.yml
```

#### 등대 서버 - 네뷸라 실행

```
# 등대 서버 네뷸라 실행 
ssh -i aws-lighthouse-ssh ubuntu@lighthouse 

ubuntu@ip-172-31-1-5:~$ sudo ./nebula -config lighthouse-conf.yml 
INFO[0000] Firewall rule added                           firewallRule="map[caName: caSha: direction:outgoing endPort:0 groups:[] host:any ip: proto:0 startPort:0]"
INFO[0000] Firewall rule added                           firewallRule="map[caName: caSha: direction:incoming endPort:0 groups:[] host:any ip: proto:1 startPort:0]"
INFO[0000] Firewall rule added                           firewallRule="map[caName: caSha: direction:incoming endPort:22 groups:[] host: ip:192.168.100.0/24 proto:0 startPort:22]"
INFO[0000] Firewall started                              firewallHash=20074ab6410f3a8b00148fa81600962b20f990645774609aa538d99b64d8a672
INFO[0000] Main HostMap created                          network=192.168.100.1/24 preferredRanges="[]"
INFO[0000] UDP hole punching enabled                    
INFO[0000] Nebula interface is active                    build=1.5.2 interface=nebula1 network=192.168.100.1/24 udpAddr="0.0.0.0:4242"
```

#### 리다이렉터 - 네뷸라 실행

등대 서버와 교신 후 Handshake 를 하는 것을 확인할 수 있다.

```
# 리다이렉터 네뷸라 실행 
ssh -i aws-redirector-ssh ubuntu@redirector

ubuntu@ip-172-31-1-253:~$ sudo ./nebula -config redirector-conf.yml

< ... > 
INFO[0000] Nebula interface is active                    build=1.5.2 interface=nebula1 network=192.168.100.3/24 udpAddr="0.0.0.0:4242"
INFO[0000] Handshake message sent                        handshake="map[stage:1 style:ix_psk0]" initiatorIndex=2753302351 udpAddrs="[<등대-서버-아이피>:4242]" vpnIp=192.168.100.1
INFO[0000] Handshake message received                    certName=lighthouse durationNs=2339487 fingerprint=989b008898ef3df8b245743424d71d9f4da9b44ca93743b094e6786abe0e0ff0 handshake="map[stage:2 style:ix_psk0]" initiatorIndex=2753302351 issuer=bc0264c734878d756850b4d997f6d76bfebc58e5770f9ca6ab6beadc5a2ec7b1 remoteIndex=2753302351 responderIndex=3140316740 sentCachedPackets=1 udpAddr="<등대-서버-아이피>:4242" vpnIp=192.168.100.1
```

#### 팀 서버 - 네뷸라 실행

```
# 팀 서버 네뷸라 실행 

┌──(root㉿kali)-[~/redteam/nebula]
└─# ./nebula -config teamserver-conf.yml                                              

< ... > 
INFO[0000] Nebula interface is active                    build=1.5.2 interface=nebula1 network=192.168.100.2/24 udpAddr="0.0.0.0:4242"
INFO[0000] Handshake message sent                        handshake="map[stage:1 style:ix_psk0]" initiatorIndex=3897418042 udpAddrs="[<등대-서버-아이피>:4242]" vpnIp=192.168.100.1
INFO[0000] Handshake message received                    certName=lighthouse durationNs=43484001 fingerprint=989b008898ef3df8b245743424d71d9f4da9b44ca93743b094e6786abe0e0ff0 handshake="map[stage:2 style:ix_psk0]" initiatorIndex=3897418042 issuer=bc0264c734878d756850b4d997f6d76bfebc58e5770f9ca6ab6beadc5a2ec7b1 remoteIndex=3897418042 responderIndex=1618900374 sentCachedPackets=1 udpAddr="<등대-서버-아이피>:4242" vpnIp=192.168.100.1
```

### 네뷸라 구축 확인

`ip a` 등의 명령어로 네뷸라 VPN이 구축된 것을 확인할 수 있다. 예를 들어 굳이 공인 아이피주소를 사용할 필요 없이, 네뷸라 아이피주소를 이용해 SSH를 할수도 있다.&#x20;

```
┌──(root㉿kali)-[/opt]
└─# ssh -i ~/redteam/sshkeys/aws-lighthouse-ssh ubuntu@192.168.100.1
Welcome to Ubuntu 22.04 LTS (GNU/Linux 5.15.0-1004-aws x86_64)

< ... > 

Last login: Tue Jun  7 22:13:36 2022 from 192.168.100.2
ubuntu@ip-172-31-1-5:~$ whoami
ubuntu
```

### 레퍼런스

[네뷸라 설정 파일](https://github.com/slackhq/nebula/blob/master/examples/config.yml)

[HuskyHacks - 제대로된 레드팀 인프라](https://notes.huskyhacks.dev/blog/red-team-infrastructure-done-right)
