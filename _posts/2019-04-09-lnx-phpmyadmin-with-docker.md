---
title: "Docker로 phpMyAdmin 설치하기"
description: "Docker를 사용하여 phpMyAdmin을 설치하는 방법 정리"
category: Linux
tags: [Arch, Linux, Docker, Mysql, 리눅스, 도커, phpMyAdmin]
toc: true
toc_sticky: true
date: 2019-04-09 00:00:00
lastmod: 2019-04-09 00:00:00
sitemap:
  changefreq: daily
---

[Arch Linux](https://archlinux.org)에서 [Docker](https://www.docker.com/)를 사용하여 [phpMyAdmin](https://www.phpmyadmin.net/)을 설치하는 방법에 대한 정리입니다.  

![Mysql Logo](/assets/images/mysql_logo.svg){:height="50%" width="50%"}  



# Docker로 phpMyAdmin 설치하기  

이번 포스트에서는 Docker를 사용하여 [phpMyAdmin](https://www.phpmyadmin.net/)을 설치하는 방법에 대해서 간략히 정리하였습니다.  



## 1. Docker 설치  

[이전 포스트](https://blog.knowledgebox.online/linux/lnx-plex-with-docker)에서 Docker를 어떻게 설치하는지 정리했습니다.  
아직 Docker를 설치하지 않았다면 [이전 포스트](https://blog.knowledgebox.online/linux/lnx-plex-with-docker)를 참고하여 먼저 Docker를 설치하세요.  



## 2. Docker로 phpMyAdmin 실행하기  

[이전 포스트](https://blog.knowledgebox.online/linux/lnx-plex-with-docker)와 마찬가지로 phpMyAdmin 컨테이너 역시 docker-compose를 사용해 실행합니다.  

### 설정  

[이전 포스트](https://blog.knowledgebox.online/linux/lnx-plex-with-docker)에서 docker-compose용 YAML 설정 파일과 Plex Media Server의 설정을 Host PC에 저장하기 위해 홈디렉토리에 docker 디렉토리 및 하위 디렉토리를 생성하였습니다.  
이번 포스트의 phpMyAdmin 컨테이너도 설정 및 데이터를 Host PC에 저장하기 위해 Host PC에 디렉토리를 아래와 같이 생성합니다.  
```bash
[계정@localhost ~]$ > cd ~/docker
[계정@localhost ~/docker]$ > mkdir phpmyadmin
```

docker-compose.yml 파일에 다음과 같이 추가합니다.  
자신의 환경에 맞게 수정하세요.  
```bash
[계정@localhost ~]$ > cd ~/docker
[계정@localhost ~/docker]$ > vim docker-compose.yml
```

```yml
version: '2'

services:
  phpmyadmin:
    container_name: phpmyadmin
    image: phpmyadmin/phpmyadmin
    restart: always
    networks:
     - UDN_Database
    ports:
     - 10240(외부에 공개할 포트):80
    volumes:
     - /home/계정/docker/phpmyadmin/config.user.inc.php:/etc/phpmyadmin/config/config.user.inc.php
    environment:
     - PMA_HOSTS=mysql,mysql(Prod),mysql(Dev)
     - PMA_PORTS=,33060,33061

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
Database용 네트워크(UDN_Database)와 서비스용 네트워크(UDN_Service)로 각각 설정하였고, Mysql과 phpMyAdmin 컨테이너는 UDN_Database 네트워크를 사용하도록 지정하였습니다.  
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

**ports:**  
Host PC의 네트워크 포트를 Docker 컨테이너(phpMyAdmin)의 특정 포트와 연결하기 위한 항목입니다.  
80번 포트는 phpMyAdmin이 사용하는 네트워크 포트인데, 이 포트를 Host PC의 10240번 포트와 연결한다는 의미입니다.  
외부 또는 다른 Docker 컨테이너에서 phpMyAdmin에 접속하기 위해서는 80번 포트가 아닌 10240번 포트로 접속하여야 합니다.  

**volumes:**   
Docker 컨테이너(phpMyAdmin)와 Host PC의 디렉토리 연결을 위한 항목입니다.  

`/home/계정/docker/phpmyadmin/config.user.inc.php:/etc/phpmyadmin/config/config.user.inc.php`는 Host PC의 `/home/계정/docker/phpmyadmin/config.user.inc.php` 디렉토리를 Docker 컨테이너(phpMyAdmin)의 `/etc/phpmyadmin/config/config.user.inc.php` 디렉토리에 연결한다는 의미입니다.  
Docker 컨테이너(phpMyAdmin)의 `/etc/phpmyadmin/config/config.user.inc.php` 디렉토리는 phpMyAdmin의 설정이 저장되는 디렉토리입니다.  
이 설정을 사용해 phpMyAdmin의 설정을 Host PC에 영구적으로 저장할 수 있습니다.  
설정하지 않아도 무방하나 이런 경우 컨테이너를 삭제하면 데이터가 초기화됩니다.   
`/home/계정/docker/phpmyadmin/config.user.inc.php`를 자신의 환경에 맞게 변경하세요.  

**environment:**  
Docker 컨테이너(phpMyAdmin)의 환경변수를 설정하기 위한 항목입니다.  
- PMA_ARBITRARY //로그인 화면에 접속할 서버를 직접 입력할 수 있는 폼을 제공합니다.
- PMA_HOST //접속할 서버를 특정합니다. (로그인 화면에서 접속할 서버를 선택할 수 없음)
- PMA_VERBOSE
- PMA_PORT //접속할 포트를 특정합니다. (로그인 화면에서 접속할 포트를 선택할 수 없음)
- PMA_HOSTS //로그인 화면에 접속할 서버를 선택할 수 있는 드랍다운 폼을 제공합니다.
- PMA_VERBOSES
- PMA_PORTS  // 로그인 화면에 접속할 포트를 선택할 수 있는 드랍다운 폼을 제공합니다.
- PMA_USER  //서버에 접속할 사용자 계정을 특정합니다. (로그인 화면에서 사용자 계정을 입력할 수 없음)
- PMA_PASSWORD  //서버에 접속할 사용자 비밀번호를 특정합니다. (로그인 화면에서 사용자 비밀번호를 입력할 수 없음)
- PMA_ABSOLUTE_URI

이 포스트에서는 위와 같은 환경변수 항목 중 `PMA_HOSTS`와 `PMA_PORTS`만 사용합니다.  
`PMA_HOSTS`는 phpMyAdmin으로 관리할 서버가 여러 개일 때 각각의 서버를 지정해 놓으면 로그인 화면에서 접속할 서버를 선택할 수 있는 드랍다운 폼을 제공합니다.  
`PMA_PORTS`는 `PMA_HOSTS`에서 지정한 서버의 포트가 각각 다를 때 각각의 서버 포트를 지정해 놓으면 로그인 화면에서 접속할 서버를 선택하는 드랍다운 폼에 포트까지 지정되어 표시됩니다.  
모두 동일한 포트를 사용한다면 이 옵션을 빼도 됩니다.  
![phpMyAdmin_PMA_HOST](/assets/images/phpmyadmin_pma_host.png)  


### 실행  

docker-compose.yml 파일을 생성한 후 다음과 같은 명령어로 컨테이너를 실행합니다.  
```bash
[계정@localhost ~]$ > cd ~/docker
[계정@localhost ~/docker]$ > docker-compose up -d phpmyadmin
```

docker-compose.yml에서 phpmyadmin이라는 이름을 가진 컨테이너를 백그라운드(-d)로 실행(up)하라는 의미입니다.  
`-d` 옵션을 주지 않으면 터미널 창을 닫을 때 Docker 컨테이너가 같이 종료됩니다.  
