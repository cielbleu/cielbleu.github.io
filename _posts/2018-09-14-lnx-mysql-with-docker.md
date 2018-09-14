---
title: "Docker로 Mysql 설치하기"
description: "Docker를 사용하여 Mysql을 설치하는 방법 정리"
category: Linux
tags: [Linux, Docker, Mysql]
toc: true
toc_sticky: true
---

[Archlinux](https://archlinux.org)에서 [Docker](https://www.docker.com/)를 사용하여 [Mysql](https://www.mysql.com/)을 설치하는 방법에 대한 정리입니다.  

![Mysql Logo](/assets/images/mysql_logo.svg)



# Docker로 Mysql 설치하기  

이번 포스트에서는 Docker를 사용하여 [Mysql](https://www.mysql.com/)을 설치하는 방법에 대해서 간략히 정리하였습니다.  



## 1. Docker 설치  

[이전 포스트](https://cielbleu.github.io/linux/lnx-plex-with-docker)에서 Docker를 어떻게 설치하는지 정리했습니다.  
아직 Docker를 설치하지 않았다면 [이전 포스트](https://cielbleu.github.io/linux/lnx-plex-with-docker)를 참고하여 먼저 Docker를 설치하세요.  



## 2. Docker로 Mysql 실행하기  

[이전 포스트](https://cielbleu.github.io/linux/lnx-plex-with-docker)와 마찬가지로 Mysql 컨테이너 역시 docker-compose를 사용해 실행합니다.  

### 설정  

[이전 포스트](https://cielbleu.github.io/linux/lnx-plex-with-docker)에서 docker-compose용 YAML 설정 파일과 Plex Media Server의 설정을 Host PC에 저장하기 위해 홈디렉토리에 docker 디렉토리 및 하위 디렉토리를 생성하였습니다.  
이번 포스트의 Mysql 컨테이너도 설정 및 데이터를 Host PC에 저장하기 위해 Host PC에 디렉토리를 아래와 같이 생성합니다.  
```bash
$ cd ~/docker
$ mkdir mysql
$ mkdir mysql/config
$ mkdir mysql/data
```

docker-compose.yml 파일에 다음과 같이 추가합니다.  
자신의 환경에 맞게 수정하세요.  
```bash
$ cd ~/docker
$ vim docker-compose.yml
```

```yml
  mysql:
    container_name: mysql
    image: mysql:latest
    restart: always
    ports:
     - 33060(외부에 공개할 포트):3306
    volumes:
     - /home/계정/docker/mysql/data:/var/lib/mysql
     - /home/계정/docker/mysql/config:/etc/mysql/conf.d
    environment:
     - MYSQL_ROOT_PASSWORD=PassW@rd!! (Mysql 관리자 비밀번호, 관리자 계정은 root)
     - MYSQL_USER=tomcat (Mysql 사용자 계정)
     - MYSQL_PASSWORD=PassW@rd!! (Mysql 사용자 계정 비밀번호)
    user: 1000:100
```

**ports:**  
Host PC의 네트워크 포트를 Docker 컨테이너(Mysql)의 특정 포트와 연결하기 위한 항목입니다.  

3306번 포트는 Mysql이 사용하는 네트워크 포트인데, 이 포트를 Host PC의 33060번 포트와 연결한다는 의미입니다.  
외부 또는 다른 Docker 컨테이너에서 Mysql에 접속하기 위해서는 3306번 포트가 아닌 33060번 포트로 접속하여야 합니다.  

**volumes:**   
Docker 컨테이너(Mysql)와 Host PC의 디렉토리 연결을 위한 항목입니다.  

`/home/계정/docker/mysql/data:/var/lib/mysql`는 Host PC의 `/home/계정/docker/mysql/data` 디렉토리를 Docker 컨테이너(Mysql)의 `/var/lib/mysql` 디렉토리에 연결한다는 의미입니다.  
Docker 컨테이너(Mysql)의 `/var/lib/mysql` 디렉토리는 Mysql의 데이터가 저장되는 디렉토리입니다.  
이 설정을 사용해 Mysql의 데이터를 Host PC에 영구적으로 저장할 수 있습니다.  
설정하지 않아도 무방하나 이런 경우 컨테이너를 삭제하면 데이터가 초기화됩니다.   
`/home/계정/docker/mysql/data`를 자신의 환경에 맞게 변경하세요.  

`/home/계정/docker/mysql/config:/etc/mysql/conf.d`는 별도로 수정한 Mysql 설정 파일을 사용하고자 할 경우 이외에는 설정하지 않아도 됩니다.  

**environment:**  
Docker 컨테이너(Mysql)의 환경변수를 설정하기 위한 항목입니다.  
- MYSQL_ROOT_PASSWORD
- MYSQL_DATABASE
- MYSQL_USER
- MYSQL_PASSWORD
- MYSQL_ALLOW_EMPTY_PASSWORD
- MYSQL_RANDOM_ROOT_PASSWORD
- MYSQL_ONETIME_PASSWORD

이 포스트에서는 위와 같은 환경변수 항목 중 `MYSQL_ROOT_PASSWORD`와 `MYSQL_USER`, `MYSQL_PASSWORD`만 사용합니다.  
`MYSQL_ROOT_PASSWORD`는 Mysql의 관리자 비밀번호를 지정하는 변수입니다.    
`MYSQL_USER`과 `MYSQL_PASSWORD`는 Mysql의 사용자 계정 및 비밀번호를 지정하는 변수입니다.  
일부 특수문자(예를 들면 $)는 제대로 인식을 하지 못하므로 주의하세요.  

**user: 1000:100**  
`user: 1000:100`은 Docker 컨테이너(MySQL)를 실행할 때 uid와 gid를 지정하는 설정입니다.  
이 설정이 없으면 **volumes:**에서 지정한 `/home/계정/docker/mysql/data` 디렉토리의 소유자 및 그룹이 999라는 숫자로 표시됩니다.  
만약 Host PC에 uid가 999번인 계정이 있다면 소유자가 그 계정으로 표시됩니다.  
이 포스트에서는 Host PC에 gid가 999번인 그룹이 adm이므로 adm으로 그룹이 표시됩니다.  

```bash
[계정@localhost ~/docker]$ > ls -al mysql/data
total 179208
.
.
.
-rw-r--r-- 1      999 adm       1112 2018-09-14 12:44 client-cert.pem
-rw------- 1      999 adm       1680 2018-09-14 12:44 client-key.pem
-rw-r----- 1      999 adm       6994 2018-09-14 12:44 ib_buffer_pool
-rw-r----- 1      999 adm   50331648 2018-09-14 12:44 ib_logfile0
-rw-r----- 1      999 adm   50331648 2018-09-14 12:44 ib_logfile1
-rw-r----- 1      999 adm   12582912 2018-09-14 12:44 ibdata1
-rw-r----- 1      999 adm   12582912 2018-09-14 12:44 ibtmp1
drwxr-x--- 2      999 adm       4096 2018-09-14 12:44 mysql/
-rw-r----- 1      999 adm   31457280 2018-09-14 12:44 mysql.ibd
.
.
.
[계정@localhost ~/docker]$ > 
```

실제로 Docker 컨테이너(Mysql)에 접속하여 확인해 보면 다음과 같습니다.  
`/etc/passwd`에 Mysql에서 사용하는 mysql 계정이 uid 999번으로 생성되어 있습니다.  
마찬가지로 `/etc/group`에 mysql 그룹이 gid 999번으로 생성되어 있습니다.  

```bash
[계정@localhost ~/docker]$ > docker exec -it mysql bash

//여기부터는 Docker 컨테이너 내부
root@a25cc8b24be5:/# cat /etc/passwd
.
.
.
games:x:60:
users:x:100:
nogroup:x:65534:
mysql:x:999:
root@a25cc8b24be5:/# cat /etc/group
.
.
.
staff:x:50:
games:x:60:
users:x:100:
nogroup:x:65534:
mysql:x:999:
root@a25cc8b24be5:/#
```

Database를 초기화하는 과정에서 데이터 저장 위치인 `/var/lib/mysql`의 소유자 및 그룹이 mysql 계정(uid 999)과 mysql 그룹(gid 999)으로 변경됩니다.  
그런데 `/var/lib/mysql`는 Host PC의 `/home/계정/docker/mysql/data`와 연결되어 있기 때문에 Host PC의 해당 디렉토리의 소유자 및 그룹이 999로 변경됩니다.  
따라서 Host PC에서 디렉토리를 조회해보면 Host PC의 uid와 gid를 사용하여 표시하므로 소유자는 999(Jost PC에는 999번 uid가 없음), 그룹은 엉뚱하게 adm(Host PC에서 999번 gid는 adm임)으로 표시되는 것입니다.  
Host PC의 uid와 gid는 각자의 환경에 따라 다르므로 이 포스트와 다르게 표시될 수 있습니다.  

### 실행  

docker-compose.yml 파일을 생성한 후 다음과 같은 명령어로 컨테이너를 실행합니다.  
```bash
$ cd ~/docker
$ docker-compose up -d mysql
```

docker-compose.yml에서 mysql이라는 이름을 가진 컨테이너를 백그라운드(-d)로 실행(up)하라는 의미입니다.  
`-d` 옵션을 주지 않으면 터미널 창을 닫을 때 Docker 컨테이너가 같이 종료됩니다.  
