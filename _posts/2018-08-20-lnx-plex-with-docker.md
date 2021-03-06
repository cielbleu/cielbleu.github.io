---
title: "Docker로 Plex Media Server 설치하기"
description: "Docker를 사용하여 Plex Media Server를 설치하는 방법 정리"
category: Linux
tags: [Arch, Linux, Docker, Plex, 리눅스, 도커]
toc: true
toc_sticky: true
date: 2018-08-20 00:00:00
lastmod: 2019-04-10 00:00:00
sitemap:
  changefreq: daily
---

[Arch Linux](https://archlinux.org)에서 [Docker](https://www.docker.com/)를 사용하여 [Plex Media Server](https://www.plex.tv/)를 설치하는 방법에 대한 정리입니다.  

![Plex Logo](/assets/images/plex_logo.svg){:height="50%" width="50%"}  



# Docker로 Plex Media Server 설치하기  

Docker에 대해서 알고 계신가요?  
필자는 Docker라는 단어를 꽤 오래전에 듣긴 했지만, 관심을 가진 것은 최근입니다.  
아직 Docker에 대해서 잘 모르신다면 먼저 검색을 통해 Docker가 어떤 물건인지 알아보시길 권장하고 싶습니다.  
이 포스트에서는 Docker를 사용하여 Plex Media Server를 설치하는 방법에 대해서 간략히 정리하였습니다.  



## 1. Docker 설치  

Docker는 기본적으로 Linux에서 동작합니다.  
Windows 또는 Mac에서는 Docker for Windows 또는 Docker for Mac을 설치하세요.  
필자는 Arch Linux에서 Docker를 설치하였지만, Docker의 특성상 다른 배포판에서도 간단히 설치할 수 있습니다.  

- Arch Linux
```bash
[계정@localhost ~]$ > sudo pacman -Syu docker docker-compose
```

- Ubuntu
```bash
[계정@localhost ~]$ > sudo apt install docker docker-compose
```


조금 더 편하게 Docker를 사용하기 위해 현재 로그인한 사용자를 docker 그룹에 넣어줍니다.  
```bash
[계정@localhost ~]$ > sudo usermod -aG docker $USER
```


재부팅시 자동으로 실행되도록 서비스를 등록합니다.  
- Arch Linux
```bash
[계정@localhost ~]$ > sudo systemctl enable docker.service //서비스 활성화
[계정@localhost ~]$ > sudo systemctl start docker.service //서비스 시작
[계정@localhost ~]$ > docker info //서비스 확인
[계정@localhost ~]$ > docker version //Docker 클라이언트 및 서버 정보 확인
```



## 2. Docker로 Plex Media Server 실행하기  

Docker로 컨테이너를 실행하는 간단한 방법은 터미널에서 docker run 명령어를 사용하는 것입니다.  
설정이 복잡하지 않을 때는 docker run 명령어로 실행하는 것이 그다지 문제가 되지 않지만, 설정이 복잡해질 경우에는 명령어가 굉장히 복잡해지기 때문에 현실적으로 사용하기가 어렵습니다.  
이 포스트에서는 조금 더 편리한 사용을 위해 YAML 방식의 설정 파일을 이용하는 docker-compose를 사용하여 컨테이너를 실행할 예정입니다.  

### 설정  

우선 홈디렉토리에 Docker 및 Plex Media Server의 설정을 저장할 디렉토리를 아래와 같이 생성합니다.  
```bash
[계정@localhost ~]$ > mkdir ~/docker
[계정@localhost ~]$ > cd docker
[계정@localhost ~/docker]$ > mkdir plex
[계정@localhost ~/docker]$ > cd plex
[계정@localhost ~/docker/plex]$ > mkdir config
```


docker-compose.yml 파일을 다음과 같이 생성합니다.  
자신의 환경에 맞게 수정하세요.  
```bash
[계정@localhost ~]$ > cd ~/docker
[계정@localhost ~/docker]$ > vim docker-compose.yml
```

```yml
version: '2'
services:
  plex:
    container_name: plex
    image: plexinc/pms-docker
    restart: unless-stopped
    networks:
     - UDN_Service
    ports:
     - 32400:32400/tcp
     - 3005:3005/tcp
     - 8324:8324/tcp
     - 32469:32469/tcp
     - 1900:1900/udp
     - 32410:32410/udp
     - 32412:32412/udp
     - 32413:32413/udp
     - 32414:32414/udp
    volumes:
     - /storage/public:/data
     - /home/계정/docker/plex/config:/config
    environment:
     - PLEX_UID=1000
     - PLEX_GID=100
     - TZ=Asia/Seoul
     - VERSION=latest
     - PLEX_CLAIM=claim-TB4R~~~~~와 같은 claim token
     - ADVERTISE_IP=http://외부에 공개할 IP 주소 또는 URL:32400/
    hostname: 사용할 Plex Media Server 이름

networks:
  UDN_Database:
    external:
      name: UDN_Database
  UDN_Service:
    external:
      name: UDN_Service
```

**networks:**  
docker-compose가 생성하는 기본 네트워크가 아닌 별도의 네트워크를 사용하겠다는 의미입니다.  
이 항목을 설정하지 않으면 docker-compose가 생성하는 기본 네트워크를 사용하게 되는데, docker-compose.yml 파일이 있는 `디렉토리명_default`와 같은 이름으로 생성됩니다.  
docker-compose.yml에 지정된 모든 Docker 컨테이너는 별도 설정이 없을 경우 기본적으로 docker-compose가 생성한 기본 네트워크에 연결됩니다.  
```bash
[계정@localhost ~/docker]$ > docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
3640e59420fe        UDN_Database        bridge              local
e38fb7c9fe3c        UDN_Service         bridge              local
4398feaef58e        bridge              bridge              local
f780747b759b        docker_default      bridge              local
4756bb8ee162        host                host                local
0bedac919e79        none                null                local
```

**networks:**에서 지정한 네트워크는 docker-compose.yml 파일에 설정되어 있어야 합니다.  
Database용 네트워크(UDN_Database)와 서비스용 네트워크(UDN_Service)로 각각 설정하였고, Plex Media Server 컨테이너는 UDN_Service 네트워크를 사용하도록 지정하였습니다.  
```yml
networks:
  UDN_Database:
    external:
      name: UDN_Database
  UDN_Service:
    external:
      name: UDN_Service
```

Docker 네트워크를 생성하는 방법은 아래를 참고하세요.  
```bash
[계정@localhost ~/docker]$ > docker network create --driver bridge --subnet 10.10.10.0/24 --gateway 10.10.10.1 UDN_Database
[계정@localhost ~/docker]$ > docker network create --driver bridge --subnet 10.10.20.0/24 --gateway 10.10.20.1 UDN_Service
```

**volumes:**   
Docker 컨테이너(Plex Media Server)와 Host PC의 디렉토리 연결을 위한 항목입니다.  

`/storage/public:/data`는 컨텐츠가 저장되어 있는 Host PC의 `/storage/public` 디렉토리를 Docker 컨테이너(Plex Media Server)의 `/data` 디렉토리에 연결한다는 의미입니다.  
이 설정을 사용해 Host PC의 컨텐츠를 Docker 컨테이너(Plex Media Servder)를 통해 서비스 할 수 있습니다.  
`/storage/public`을 자신의 환경에 맞게 변경하세요.  

`/home/계정/docker/plex/config:/config`는 Host PC의 `/home/계정/docker/plex/config` 디렉토리를 Docker 컨테이너(Plex Media Server)의 `/config` 디렉토리에 연결한다는 의미입니다.  
Docker 컨테이너(Plex Media Server)의 `/config` 디렉토리는 Plex Media Server의 설정이 저장되는 디렉토리입니다.  
이 설정을 사용해 Docker 컨테이너(Plex Media Server)의 설정을 Host PC에 영구적으로 저장할 수 있습니다.  
설정하지 않아도 무방하나 이런 경우 컨테이너를 삭제하면 설정이 초기화됩니다.  
Plex Media Server의 설정을 계속 유지하기 위해 컨테이너 내부가 아닌 Host PC에 설정을 저장하는 것이 좋습니다.    
`/home/계정/docker/plex/config`를 자신의 환경에 맞게 변경하세요.  

**environment:**  
Docker 컨테이너(Plex Media Server)의 환경변수를 설정하기 위한 항목입니다.  

environment에서 중요한 항목은 `PLEX_CLAIM`과 `ADVERTISE_IP`입니다.  
`PLEX_CLAIM` 코드는 [https://plex.tv/claim](https://plex.tv/claim)에 접속하여 받을 수 있습니다.  
> *시간제한이 있으므로 유의하세요*  

`ADVERTISE_IP`는 외부에서 Plex Media Server에 접속할 수 있도록 IP 또는 URL을 설정하는 항목입니다.  
`ADVERTISE_IP`를 설정하지 않으면 Plex Media Server가 실행되는 동일 네트워크에서만 접속할 수 있습니다.  

### 실행  

docker-compose.yml 파일을 생성한 후 다음과 같은 명령어로 컨테이너를 실행합니다.  
```bash
[계정@localhost ~]$ > cd ~/docker
[계정@localhost ~/docker]$ > docker-compose up -d plex
```

docker-compose.yml에서 plex라는 이름을 가진 컨테이너를 백그라운드(-d)로 실행(up)하라는 의미입니다.  
`-d` 옵션을 주지 않으면 터미널 창을 닫을 때 Docker 컨테이너(Plex Media Server)가 같이 종료됩니다.  
