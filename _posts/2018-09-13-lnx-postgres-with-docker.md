---
title: "Docker로 PostgreSQL 및 pgAdmin 설치하기"
description: "Docker를 사용하여 PostgreSQL 및 pgAdmin을 설치하는 방법 정리"
category: Linux
tags: [Arch, Linux, Docker, PostgreSQL, pgAdmin, 리눅스, 도커]
toc: true
toc_sticky: true
date: 2018-09-13 00:00:00
lastmod: 2019-04-10 00:00:00
sitemap:
  changefreq: daily
---

[Arch Linux](https://archlinux.org)에서 [Docker](https://www.docker.com/)를 사용하여 [PostgreSQL](https://www.postgresql.org/) 및 관리 도구인 [pgAdmin](https://www.pgadmin.org/)를 설치하는 방법에 대한 정리입니다.  

![PostgreSQL Logo](/assets/images/postgresql_logo.svg){:height="50%" width="50%"}  



# Docker로 PostgreSQL 및 pgAdmin 설치하기  

이번 포스트에서는 Docker를 사용하여 Open Source Database인 [PostgreSQL](https://www.postgresql.org/)과 관리 도구인 [pgAdmin](https://www.pgadmin.org/)를 설치하는 방법에 대해서 간략히 정리해 보았습니다.  



## 1. Docker 설치  

[이전 포스트](https://blog.knowledgebox.online/linux/lnx-plex-with-docker)에서 Docker를 어떻게 설치하는지 정리했습니다.  
아직 Docker를 설치하지 않았다면 [이전 포스트](https://blog.knowledgebox.online/linux/lnx-plex-with-docker)를 참고하여 먼저 Docker를 설치하세요.  



## 2. Docker로 PostgreSQL 실행하기  

[이전 포스트](https://blog.knowledgebox.online/linux/lnx-plex-with-docker)와 마찬가지로 PostgreSQL 컨테이너 역시 docker-compose를 사용해 실행합니다.  

### 설정  

[이전 포스트](https://blog.knowledgebox.online/linux/lnx-plex-with-docker)에서 docker-compose용 YAML 설정 파일과 Plex Media Server의 설정을 Host PC에 저장하기 위해 홈디렉토리에 docker 디렉토리 및 하위 디렉토리를 생성하였습니다.  
이번 포스트의 PostgreSQL 컨테이너도 설정 및 데이터를 Host PC에 저장하기 위해 Host PC에 디렉토리를 아래와 같이 생성합니다.  
```bash
[계정@localhost ~]$ > cd ~/docker
[계정@localhost ~/docker]$ > mkdir pgsql
[계정@localhost ~/docker]$ > mkdir pgsql/config
[계정@localhost ~/docker]$ > mkdir pgsql/pgdata
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
  postgres:
    container_name: postgres
    image: postgres:latest
    restart: always
    networks:
     - UDN_Database
    ports:
     - 15432(외부에 공개할 포트):5432
    volumes:
     - /home/계정/docker/pgsql/pgdata:/var/lib/postgresql/data
     - /home/계정/docker/pgsql/config/postgres.conf:/etc/postgresql/postgres.conf
    environment:
     - POSTGRES_USER=master (PostgreSQL 관리자로 사용할 계정(기본값은 postgres))
     - POSTGRES_PASSWORD=PassW@rd!! (PostgreSQL 관리자로 사용할 계정의 비밀번호)
    user: 1000:100

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
Database용 네트워크(UDN_Database)와 서비스용 네트워크(UDN_Service)로 각각 설정하였고, PostgreSQL과 pgAdmin 컨테이너는 UDN_Database 네트워크를 사용하도록 지정하였습니다.  
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
Host PC의 네트워크 포트를 Docker 컨테이너(PostgreSQL)의 특정 포트와 연결하기 위한 항목입니다.  

5432번 포트는 PostgreSQL이 사용하는 네트워크 포트인데, 이 포트를 Host PC의 15432번 포트와 연결한다는 의미입니다.  
외부 또는 다른 Docker 컨테이너에서 PostgreSQL에 접속하기 위해서는 5432번 포트가 아닌 15432번 포트로 접속하여야 합니다.  

**volumes:**   
Docker 컨테이너(PostgreSQL)와 Host PC의 디렉토리 연결을 위한 항목입니다.  

`/home/계정/docker/pgsql/pgdata:/var/lib/postgresql/data`는 Host PC의 `/home/계정/docker/pgsql/pgdata` 디렉토리를 Docker 컨테이너(PostgreSQL)의 `/var/lib/postgresql/data` 디렉토리에 연결한다는 의미입니다.  
Docker 컨테이너(PostgreSQL)의 `/var/lib/postgresql/data` 디렉토리는 PostgreSQL의 데이터가 저장되는 디렉토리입니다.  
이 설정을 사용해 PostgreSQL의 데이터를 Host PC에 영구적으로 저장할 수 있습니다.  
설정하지 않아도 무방하나 이런 경우 컨테이너를 삭제하면 데이터가 초기화됩니다.   
`/home/계정/docker/pgsql/pgdata`를 자신의 환경에 맞게 변경하세요.  

`/home/계정/docker/pgsql/config/postgres.conf:/etc/postgresql/postgres.conf`는 별도로 수정한 PostgreSQL 설정 파일을 사용하고자 할 경우 이외에는 설정하지 않아도 됩니다.  

**environment:**  
Docker 컨테이너(PostgreSQL)의 환경변수를 설정하기 위한 항목입니다.  
- POSTGRES_PASSWORD
- POSTGRES_USER
- PGDATA
- POSTGRES_DB
- POSTGRES_INITDB_ARGS
- POSTGRES_INITDB_WALDIR

이 포스트에서는 위와 같은 환경변수 항목 중 `POSTGRES_PASSWORD`와 `POSTGRES_USER`만 사용합니다.  
`POSTGRES_USER`는 PostgreSQL의 관리자 계정을 지정하는 변수입니다. 이 환경변수를 사용하지 않으면 기본값인 postgres로 설정됩니다.  
`POSTGRES_PASSWORD`는 PostgreSQL의 관리자 계정 비밀번호를 지정하는 변수입니다.  
일부 특수문자(예를 들면 $)는 제대로 인식을 하지 못하므로 주의하세요.  

**user: 1000:100**  
`user: 1000:100`은 Docker 컨테이너(PostgreSQL)를 실행할 때 uid와 gid를 지정하는 설정입니다.  
이 설정이 없으면 **volumes:**에서 지정한 `home/계정/docker/pgsql/pgdata` 디렉토리의 소유자가 999라는 숫자로 표시됩니다.  
만약 Host PC에 uid가 999번인 계정이 있다면 소유자가 그 계정으로 표시됩니다.  

```bash
[계정@localhost ~/docker]$ > ls -al pgsql/pgdata/
total 128
drwx------ 19 999 users  4096 2018-09-13 14:42 ./
drwxr-xr-x  4 999 users  4096 2018-09-13 12:47 ../
-rw-------  1 999 users     3 2018-09-13 12:52 PG_VERSION
drwx------  6 999 users  4096 2018-09-13 13:06 base/
drwx------  2 999 users  4096 2018-09-13 14:42 global/
drwx------  2 999 users  4096 2018-09-13 12:52 pg_commit_ts/
drwx------  2 999 users  4096 2018-09-13 12:52 pg_dynshmem/
-rw-------  1 999 users  4537 2018-09-13 12:52 pg_hba.conf
-rw-------  1 999 users  1636 2018-09-13 12:52 pg_ident.conf
.
.
.
[계정@localhost ~/docker]$ >
```

실제로 Docker 컨테이너(PostgreSQL)에 접속하여 확인해 보면 다음과 같습니다.  
`/etc/passwd`에 POSTGRES_USER 환경변수로 설정한 계정이 uid 999번으로 생성되어 있습니다.  

```bash
[계정@localhost ~/docker]$ > docker exec -it postgres bash

//여기부터는 Docker 컨테이너 내부
I have no name!@460a58a59084:/$ cat /etc/passwd
.
.
.
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/bin/false
master:x:999:999::/var/lib/postgresql:/bin/bash
Debian-exim:x:101:101::/var/spool/exim4:/bin/false
I have no name!@460a58a59084:/$
```

Database를 초기화하는 과정에서 데이터 저장 위치인 `/var/lib/postgresql/data`의 소유자가 POSTGRES_USER 환경변수에 설정한 계정으로 변경됩니다.  
그런데 `/var/lib/postgresql/data`는 Host PC의 `/home/계정/docker/pgsql/pgdata`와 연결되어 있기 때문에 Host PC의 해당 디렉토리의 소유자가 999로 변경됩니다.  
따라서 Host PC에서 디렉토리를 조회해보면 Host PC의 uid와 gid를 사용하여 표시하므로 소유자가 999(Host PC에는 999번 uid가 없음)로 표시되는 것입니다.  
Host PC의 uid와 gid는 각자의 환경에 따라 다르므로 이 포스트와 다르게 표시될 수 있습니다.  

이 문제 해결을 위해 [공식사이트](https://hub.docker.com/_/postgres/)에서는 3가지 방법을 제시하는데, 이 포스트에서는 3번째 방법을 사용하였습니다.  

```bash
//기본 postgres 계정으로 컨테이너를 생성하여 Database 초기화까지 진행한 후 컨테이너 삭제
[계정@localhost ~/docker]$ > docker run -it --rm -v /home/계정/docker/pgsql/pgdata:/var/lib/postgresql/data postgres

//소유자 및 그룹을 각각 1000번과 100번으로 변경하고 컨테이너 삭제
//uid 1000번은 Host PC의 필자 계정의 uid입니다.
//gid 100번은 Host PC의 users 그룹 gid입니다.
[계정@localhost ~/docker]$ > docker run -it --rm -v /home/계정/docker/pgsql/pgdata:/var/lib/postgresql/data bash chown -R 1000:100 /var/lib/postgresql/data

//소유자 변경 확인 후 실제 컨테이너 실행
[계정@localhost ~/docker]$ > docker-compose up -d postgres
```

### 실행  

docker-compose.yml 파일을 생성한 후 다음과 같은 명령어로 컨테이너를 실행합니다.  
```bash
[계정@localhost ~]$ > cd ~/docker
[계정@localhost ~/docker]$ > docker-compose up -d postgres
```

docker-compose.yml에서 postgres라는 이름을 가진 컨테이너를 백그라운드(-d)로 실행(up)하라는 의미입니다.  
`-d` 옵션을 주지 않으면 터미널 창을 닫을 때 Docker 컨테이너가 같이 종료됩니다.  



## 3. Docker로 pgAdmin 실행하기  
PostgreSQL 관리 도구는 여러 가지가 있지만 이 포스트에서는 pgAdmin을 사용합니다.  
[공식사이트](https://www.pgadmin.org/)에서 설치파일을 다운로드하여 사용할 수 있지만, 이 포스트에서는 Docker 컨테이너 방식을 사용합니다.  

### 설정  

pgAdmin은 딱히 Host PC의 디렉토리와 연결해서 사용할 필요가 없으므로 설정을 저장할 디렉토리는 생성하지 않습니다.  
단, https 프로토콜로 사용하기를 원한다면 Host PC의 디렉토리에 인증서를 저장할 필요가 있습니다.  
이 포스트에서는 http 프로토콜 사용으로 한정합니다.  

docker-compose.yml 파일에 다음과 같이 추가합니다.  
자신의 환경에 맞게 수정하세요.  
```bash
[계정@localhost ~]$ > cd ~/docker
[계정@localhost ~/docker]$ > vim docker-compose.yml
```

```yml
version: '2'

services:
  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4
    restart: always
    networks:
     - UDN_Database
    ports:
     - 8088:(외부에 공개할 포트):80
    environment:
     - PGADMIN_DEFAULT_EMAIL=my@mail.com(pgAdmin 최초 접속시 사용할 이메일)
     - PGADMIN_DEFAULT_PASSWORD=PassW@rd!!(pgAdmin 최초 접속시 사용할 비밀번호)

networks:
  UDN_Database:
    external:
      name: UDN_Database
  UDN_Service:
    external:
      name: UDN_Service
```

**ports:**  
Host PC의 네트워크 포트를 Docker 컨테이너(pgAdmin)의 특정 포트와 연결하기 위한 항목입니다.  

80번 포트는 pgAdmin이 사용하는 네트워크 포트인데, 이 포트를 Host PC의 8088번 포트와 연결한다는 의미입니다.  
외부에서 pgAdmin에 접속하기 위해서는 80번 포트가 아닌 http://host ip or url:8088로 접속하여야 합니다.  

**environment:**  
Docker 컨테이너(pgAdmin)의 환경변수를 설정하기 위한 항목입니다.  

`PGADMIN_DEFAULT_EMAIL`는 pgAdmin에 최초로 로그인할 때 사용할 ID입니다. 자신이 사용하는 이메일을 등록하면 됩니다.  
계속 이 ID를 사용해도 되고 로그인 이후 다른 ID를 생성하여 사용해도 됩니다.  
`PGADMIN_DEFAULT_PASSWORD`는 pgAdmin에 최초로 로그인할 때 사용할 비밀번호입니다.    
일부 특수문자(예를 들면 $)는 제대로 인식을 하지 못하므로 주의하세요.  

### 실행  

docker-compose.yml 파일을 생성한 후 다음과 같은 명령어로 컨테이너를 실행합니다.  
```bash
[계정@localhost ~]$ > cd ~/docker
[계정@localhost ~/docker]$ > docker-compose up -d pgadmin
```

docker-compose.yml에서 pgadmin이라는 이름을 가진 컨테이너를 백그라운드(-d)로 실행(up)하라는 의미입니다.  
`-d` 옵션을 주지 않으면 터미널 창을 닫을 때 Docker 컨테이너가 같이 종료됩니다.  

### 접속  
pgAdmin은 웹브라우저로 접속합니다.  
웹브라우저에 http://host pc ip 또는 url:8088을 입력하고 접속합니다.  
로그인 윈도우에 초기 ID와 비밀번호를 입력합니다.  
>초기 ID: PGADMIN_DEFAULT_EMAIL=my@mail.com  
>초기 비밀번호: PGADMIN_DEFAULT_PASSWORD=PassW@rd!!  

**로그인**  
![pgAdmin_Login](/assets/images/pgadmin_login.png)

아래 이미지와 같은 화면이 나오면 로그인에 성공한 것입니다.  
**초기화면**  
![pgAdmin_Welcome](/assets/images/pgadmin_welcome.png)
