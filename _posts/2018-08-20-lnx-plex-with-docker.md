---
title: "Docker로 Plex Midia Server 설치하기"
description: "Docker를 사용하여 Plex Media Server를 설치하는 방법 정리"
category: Linux
tags: [Linux, Docker, Plex, 도커]
---

**Archlinux**(https://archlinux.org)에서 Docker(https://www.docker.com/)를 사용하여 Plex Media Server(https://www.plex.tv/)를 설치하는 방법에 대한 정리입니다.  
<center>![Plex Logo](/assets/images/plex.jpeg){: width="576" height="324"}</center>

## Docker로 Plex Media Server 설치하기  
  
Docker에 대해서 알고 계신가요?  
필자는 Docker라는 단어를 꽤 오래전에 듣긴 했지만 관심을 가진 것은 최근입니다.  
아직 Docker에 대해서 잘 모르신다면 먼저 검색을 통해 Docker가 어떤 물건인지 알아보시길 권장하고 싶습니다.  
본 문서에서는 Docker를 사용하여 Plex Media Server를 설치하는 방법에 대해서 간략히 정리해 보았습니다.  


# 1. Docker 설치  
Docker는 기본적으로 Linux에서 동작합니다.  
Windows 또는 Mac에서는 Docker for Windows 또는 Docker for Mac을 설치하세요.  
필자는 ArchLinux에서 Docker를 설치하였지만 Docker의 특성상 다른 배포판에서도 간단히 설치할 수 있습니다.  

- ArchLinux
```bash
$ sudo pacman -Syu docker docker-compose
```

- Ubuntu
```bash
$ sudo apt install docker docker-compose
```

조금 더 편하게 Docker를 사용하기 위해 현재 로그인한 사용자를 docker 그룹에 넣어줍니다.
```bash
$ sudo usermod -aG docker $USER
```

재부팅시 자동으로 실행되도록 서비스를 등록합니다.
- ArchLinux
```bash
$ sudo systemctl enable docker.service //서비스 활성화
$ sudo systemctl start docker.service //서비스 시작
$ docker info //서비스 확인
$ docker version //Docker 클라이언트 및 서버 정보 확인
```

# 2. Docker로 Plex Media Server 실행하기  
Docker로 컨테이너를 실행하는 간단한 방법은 터미널에서 docker run 명령어를 사용하는 것입니다.  
하지만 이 방법은 매번 명령어를 입력해야 합니다.  
설정이 복잡하지 않을 때에는 그다지 문제가 되지 않지만, 설정이 복잡해질 경우에는 명령어가 굉장히 복잡해집니다.  
필자는 YAML 방식의 설정파일을 이용하는 Docker-Compose를 사용하여 컨테이너를 실행하겠습니다.  
우선 홈디렉토리에 폴더(docker)를 만들고 Plex Media Server의 설정을 저장할 폴더를 아래와 같이 만들었습니다.  
```bash
$ mkdir docker
$ cd docker
$ mkdir plex
$ cd plex
$ mkdir config
$ cd ..
```


Docker-Compose 파일을 다음과 같이 생성합니다.  
자신의 환경에 맞게 적당히 수정하세요.  
```bash
$ vim docker-compose.yml
```

- docker-compose.yml 설정
```vim
version: '2'

services:

  plex:
    container_name: plex
    image: plexinc/pms-docker
    restart: unless-stopped
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
```
**volumes:** 하위의 항목은 Plex Media Server 컨테이너에서 연결할 Host PC의 폴더를 설정합니다.  
`/storage/public:/data`는 컨텐츠가 저장되어 있는 Host PC의 폴더를 Plex Media Server 컨테이너와 연결해 주는 설정입니다.  
Host PC의 `/storage/public` 폴더를 Plex Media Server 컨테이너의 `/data` 폴더에 연결해 준다는 의미입니다.  
`/storage/public`을 자신의 환경에 맞게 변경하세요.  

`/home/계정/docker/plex/config:/config`는 Plex Media Server의 설정이 저장되는 폴더를 Host PC의 폴더에 연결해 주는 설정입니다.  
설정을 하지 않아도 무방하나 이런 경우 컨테이너를 삭제하면 설정이 초기화됩니다.  
Plex Media Server의 설정을 계속 유지하기 위해 컨테이너가 아닌 Host PC의 특정 폴더에 설정을 저장하는 것을 추천합니다.  
`/home/계정/docker/plex/config`를 자신의 환경에 맞게 변경하세요.  

**environment:** 하위의 항목에서는 환경변수를 설정합니다.  
environment에서 중요한 항목은 `PLEX_CLAIM`과 `ADVIRTISE_IP`입니다.  
PLEX_CLAIM 코드는 (https://plex.tv/claim)에 접속하여 받을 수 있습니다.  
*시간제한이 있으므로 유의하세요*
ADVIRTISE_IP는 외부에서 Plex Media Server에 접속하기 위해 IP 또는 URL을 설정하는 항목입니다.  
ADVERTISE_IP를 설정하지 않으면 Plex Media Server가 실행되는 동일 네트워크에서만 접속할 수 있습니다.  

docker-compose.yml 파일을 만들었으면 다음과 같은 명령어로 컨테이너를 실행합니다.
```bash
$ cd ~/docker
$ docker-compose up -d plex
```
docker-compose.yml에서 plex라는 이름을 가진 컨테이너를 데몬(-d)으로 실행(up)하라는 의미입니다.  
`-d` 옵션을 주지 않으면 터미널 창을 닫을 때 컨테이너가 같이 종료됩니다.  
