---
title: "Docker로 OpenConnect SSL VPN 서버 구축하기"
description: "Docker를 사용하여 OpenConnect SSL VPN 서버를 구축하는 방법 정리"
category: Linux
tags: [Linux, Docker, VPN, OpenConnect]
toc: true
toc_sticky: true
---

[Archlinux](https://archlinux.org)에서 [Docker](https://www.docker.com/)를 사용하여 [OpenConnect SSL VPN](https://ocserv.gitlab.io/www/index.html) 서버를 구축하는 방법에 대한 정리입니다.  

![VPN](/assets/images/vpn.svg)



# Docker로 OpenConnect SSL VPN 서버 구축하기  

중국 인터넷 환경에 대한 준비 없이 중국에 출장 또는 여행을 가게 되면 굉장히 답답한 상황을 맞이하게 됩니다.  
대표적으로 카카오톡이 잘 안 되거나, 구글 검색 또는 지메일을 사용하지 못하거나, 유튜브 또는 페이스북에 접속이 되지 않는 것인데, 이는 `황금방패` 또는 `만리방화벽(The Great Firewall of China, GFW)`으로 불리는 중국 공산당의 `자국민 정보 검열 시스템` 때문에 발생하는 현상입니다. (로밍 핸드폰은 GFW에 영향을 받지 않음)  
자세한 정보는 [나무위키](https://namu.wiki/w/%ED%99%A9%EA%B8%88%EB%B0%A9%ED%8C%A8)를 참조하세요.  
  
이런 검열을 우회하기 위한 수단으로 VPN을 많이 사용했으나, 2017년 하반기부터는 기존에 차단이 되지 않았던 OpenVPN이나 IKEv2 방식의 VPN도 대부분 차단되고 있으며, 유료로 서비스하는 VPN도 일부 차단되고 있습니다.  
현재는 SSL 방식의 VPN이 그나마 안정적으로 검열을 우회할 수 있는 방안입니다.  
  
이번 포스트에서는 Docker를 사용하여 OpenConnect SSL VPN 서버인 [OCServ](https://ocserv.gitlab.io/www/index.html)를 설치하는 방법에 대해서 간략히 정리해 보았습니다.  



## 1. Docker 설치  

[이전 포스트](https://cielbleu.github.io/linux/lnx-plex-with-docker)에서 Docker를 어떻게 설치하는지 정리했습니다.  
아직 Docker를 설치하지 않았다면 [이전 포스트](https://cielbleu.github.io/linux/lnx-plex-with-docker)를 참고하여 먼저 Docker를 설치하세요.  



## 2. Docker로 OpenConnect SSL VPN 서버 실행하기  

[이전 포스트](https://cielbleu.github.io/linux/lnx-plex-with-docker)와 마찬가지로 OpenConnect SSL VPN 컨테이너 역시 docker-compose를 사용해 실행합니다.  

### 설정  

[이전 포스트](https://cielbleu.github.io/linux/lnx-plex-with-docker)에서 docker-compose용 YAML 설정 파일과 Plex Media Server의 설정을 Host PC에 저장하기 위해 홈디렉토리에 docker 디렉토리 및 하위 디렉토리를 생성하였습니다.  
이번 포스트의 OpenConnect SSL VPN 서버 컨테이너는 설정 및 데이터를 Host PC에 저장할 필요가 없기 때문에 Host PC에 디렉토리를 생성하지는 않습니다.  
  
docker-compose.yml 파일에 다음과 같이 추가합니다.  
자신의 환경에 맞게 수정하세요.  
```bash
$ cd ~/docker
$ vim docker-compose.yml
```

```yml
  ocserv:
    container_name: ocserv
    image: vimagick/ocserv
    restart: always
    ports:
     - 443(외부에 공개할 포트):443/tcp
     - 443:443/udp
    environment:
	 - VPN_DOMAIN=ocserv
	 - VPN_NETWORK=10.20.30.0 (VPN 클라이언트에 할당할 ip 대역)
	 - VPN_NETMASK=255.255.255.0 (vpn 클라이언트에 할당할 subnet mask)
	 - LAN_NETWORK=192.168.0.0 (vpn 예외 네트워크 ip 대역)
	 - LAN_NETMASK=255.255.0.0 (vpn 예외 네트워크 subnet mask)
	 - VPN_USERNAME=vpnuser (vpn 클라이언트에서 로그인 ID, 추가 생성 가능)
	 - VPN_PASSWORD=PassW@rd (vpn 클라이언트 로그인 비밀번호, 인증서의 비밀번호로도 사용됨, 추가 생성 가능)
    cap_add:
	 - NET_ADMIN
```

**ports:**  
Host PC의 네트워크 포트를 Docker 컨테이너(OCServ)의 특정 포트와 연결하기 위한 항목입니다.  

TCP/UDP 443번 포트는 OCServ에서 사용하는 네트워크 포트인데, 이 포트를 Host PC의 443번 포트와 연결한다는 의미입니다.  
Host PC의 포트를 443이 아닌 다른 포트로 변경해도 됩니다.  
이 포스트에서는 Host PC와 Docker 컨테이너(OCServ)의 포트를 동일하게 443번 포트를 사용하는 것으로 설정했습니다.  

**environment:**  
Docker 컨테이너(OCServ)의 환경변수를 설정하기 위한 항목입니다.  

`VPN_NETWORK` 및 `VPN_NETMASK`는 vpn 클라이언트에 할당할 ip 대역입니다.  
노트북이나 스마트폰으로 OpenConnect SSL VPN 서버에 접속하면 여기에 설정한 ip 대역을 할당해 줍니다.  
`LAN_NETWORK` 및 `LAN_NETMASK`는 vpn 연결을 사용하지 않을 예외 네트워크 대역 설정입니다.  
접속하고자 하는 목적지 ip가 여기에 설정한 ip 대역에 해당한다면 vpn 연결을 사용하지 말라는 의미입니다.  
이 포스트에서는 C Class 사설 ip 대역 전체에 대해 vpn 연결을 사용하지 않도록 설정했습니다.  
`VPN_USERNAME` 및 `VPN_PASSWORD`는 vpn 클라이언트에서 접속할 때 사용하는 ID 와 Password입니다.  
또한 `VPN_PASSWORD`는 인증서 비밀번호로도 사용됩니다.    

**cap_add**  
Docker 컨테이너(OCServ)에 기능을 추가하기 위한 항목입니다.  
비슷한 항목으로 Docker 컨테이너의 기능을 제거하기 위해 cap_drop를 사용합니다.  
  
`NET_ADMIN`은 Docker 컨테이너(OCServ)에 root 권한 없이 네트워크 관련한 설정을 할 수 있도록 권한을 부여합니다.  
비슷한 용도로 cap_add가 아닌 privileged를 사용하기도 합니다.  
docker-compose.yml 파일에 cap_add를 빼고 privileged: true를 추가하면 됩니다.  
```yml
  ocserv:
    container_name: ocserv
    image: vimagick/ocserv
    restart: always
    privileged: true
    ports:
     - 443:443/tcp
     - 443:443/udp
    environment:
	 - VPN_DOMAIN=ocserv
	 - VPN_NETWORK=10.20.30.0
	 - VPN_NETMASK=255.255.255.0
	 - LAN_NETWORK=192.168.0.0
	 - LAN_NETMASK=255.255.0.0
	 - VPN_USERNAME=vpnuser
	 - VPN_PASSWORD=PassW@rd
```

### 실행  

docker-compose.yml 파일을 생성한 후 다음과 같은 명령어로 컨테이너를 실행합니다.  
```bash
$ cd ~/docker
$ docker-compose up -d ocserv
```

docker-compose.yml에서 ocserv라는 이름을 가진 컨테이너를 백그라운드(-d)로 실행(up)하라는 의미입니다.  
`-d` 옵션을 주지 않으면 터미널 창을 닫을 때 Docker 컨테이너가 같이 종료됩니다.  



## 3. OpenConnect SSL VPN 클라이언트 설치하기  

OpenConnect SSL VPN 서버에 접속하기 위해 Cisco의 AnyConnect 앱을 사용합니다.  
Apple AppStore나 Google Play에서 AnyConnect를 검색하여 설치하세요.  
노트북이나 데스크탑에서는 [OpenConnect-GUI for Windows](https://github.com/openconnect/openconnect-gui/releases)를 설치하세요.  
![OpenConnect-GUI](/assets/images/OpenConnectGUI.PNG)

### 설정  

AnyConnect 앱을 사용해 OpenConnect SSL VPN 서버에 접속하는 방법은 두 가지가 있습니다.  
하나는 OpenConnect SSL VPN 서버에서 발급한 인증서를 사용하는 방식이고, 또 다른 하나는 일반적인 ID & PW를 사용하는 방법입니다.  

**인증서 방식**  
인증서 방식으로 사용하기 위해서는 OpenConnect SSL VPN 서버에서 인증서를 발급받아 AnyConnect 앱에 설치해야 합니다.  

아래의 순서대로 인증서를 발급받아 AnyConnect 앱에 인증서를 설치하세요.  
1. Docker 컨테이너(OCServ)를 실행하고 있는 서버에 SSH로 접속합니다.  
2. 다음의 명령어를 입력해 Docker 컨테이너(OCServ)로부터 인증서 발급 및 복사합니다.  
```bash
$ cd ~/docker
$ mkdir vpn_cert
$ docker cp ocserv:/etc/ocserv/certs/client.p12 vpn_cert (ocserv 컨테이너의 /etc/ocserv/certs/client.p12 파일을 /home/자신의 계정/docker/vpn_cert/ 디렉토리에 복사)
```
3. client.p12 파일을 서버에서 자신의 PC로 복사하거나 서버에서 직접 자신의 이메일로 client.12를 보냅니다. (SCP를 사용해 복사하면 편리합니다)  
4. client.p12 파일을 자신의 이메일로 보냅니다.  
5. 스마트폰에서 자신의 이메일에 접속한 후 client.p12 파일을 다운로드 받습니다.  
6. 스마트폰에 인증서를 설치합니다.  
7. `AnyConnect 앱 → 진단 → 인증서 관리 → 가져오기`를 선택해 인증서를 가져옵니다.  

**ID & PW 방식**  
ID & PW 방식으로 사용하기 위해서는 Docker 컨테이너(OCServ)로 접속해서 계정을 추가로 생성해야 합니다.  
혼자만 사용할 예정이면 계정을 추가할 필요 없이 docker-compose.yml에 설정한 ID와 PW를 사용해도 됩니다.  

1. Docker 컨테이너(OCServ)를 실행하고 있는 서버에 SSH로 접속합니다.  
2. 다음의 명령어를 입력해 Docker 컨테이너(OCServ)의 셸을 실행합니다.  
```bash
$ docker exec -it ocserv sh
```
3. Docker 컨테이너(OCServ)의 셸에서 다음의 명령어로 계정을 생성합니다.  
```bash
# ocpasswd -c ocpasswd vpnuser2 (계정 생성)
# cat /etc/ocserv/ocpasswd (계정이 정상적으로 생성되었는지 확인)
vpnuser:*:$1$VvUApR6k$s8nsL5M1OwunM0h.5IgWY/
vpnuser2:*:$1$Clk6bTSk$nHFpEURdeA3wqS2dsLKpD1
```
4. exit를 입력하여 Docker 컨테이너(OCServ)의 셸을 빠져나갑니다.  

인증서 또는 ID & PW 준비가 완료되면 AnyConnect 앱에서 접속정보를 설정합니다.  

1. AnyConnect 앱에서 `연결`을 선택합니다.  
![AnyConnect #1](/assets/images/AnyConnectClient1.png)

2. `새 VPN 연결 추가`를 선택합니다.  
![AnyConnect #2](/assets/images/AnyConnectClient2.png)

3. `설명`과 `서버 주소`를 입력하고 `고급 설정`을 선택합니다.  
`서버 주소`에는 Docker 컨테이너(OCServ)가 실행중인 서버의 공인 IP 또는 도메인을 입력합니다.  
이 포스트에서는 Iptime 공유기 하단에 Docker 컨테이너 서버가 연결되어 있으므로 Iptime 공유기에 설정한 ddns 도메인(xxxxx.iptime.org)을 입력했습니다.  
![AnyConnect #3](/assets/images/AnyConnectClient3.png)

4. `고급 설정`에서 `인증서`를 선택하고 설치한 인증서를 선택합니다. ID & PW 방식으로 사용하고자 하는 경우에는 설정하지 않아도 됩니다.  

5. 마지막으로 `AnyConnect 앱 → 설정 → 신뢰할 수 없는 서버 차단` 항목의 체크박스를 해제하세요.  
자체적으로 발급한 인증서를 사용하기 때문에 체크박스를 해제하지 않으면 접속이 되지 않습니다.   
보안상 문제가 될 수 있는 부분이지만 혼자 사용할 VPN 서버이기 때문에 이 포스트에서는 이에 대한 해결책은 포함하지 않았습니다.  
![AnyConnect #4](/assets/images/AnyConnectClient4.png)

### 결과  
최근 중국에 출장을 갈 일이 있어 실제 테스트를 해볼 수 있었습니다.  
테스트 결과 정상적으로 구글, 유튜브 등 중국에서 접근이 차단된 사이트에 접속할 수 있었습니다.  
