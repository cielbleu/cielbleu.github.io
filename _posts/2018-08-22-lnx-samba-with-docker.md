---
title: "Docker로 Samba Server 설치하기"
description: "Docker를 사용하여 Samba Server를 설치하는 방법 정리"
category: Linux
tags: [Linux, Docker, Samba, 리눅스, 도커, 삼바]
toc: true
toc_sticky: true
date: 2018-08-22 00:00:00
lastmod: 2018-12-19 00:00:00
sitemap:
  changefreq: daily
---

[Archlinux](https://archlinux.org)에서 [Docker](https://www.docker.com/)를 사용하여 [Samba Server](https://www.samba.org/)를 설치하는 방법에 대한 정리입니다.  
![Samba Logo](/assets/images/samba_logo.svg){:height="50%" width="50%"}  



# Docker로 Samba Server 설치하기  

[이전 포스트](https://blog.knowledgebox.online/linux/lnx-plex-with-docker)에 이어 이번 포스트에서는 Docker를 사용하여 Samba Server를 설치하는 방법에 대해서 간략히 정리하였습니다.  

## 1. Docker 설치  

[이전 포스트](https://blog.knowledgebox.online/linux/lnx-plex-with-docker)에서 Docker를 어떻게 설치하는지 정리했습니다.  
아직 Docker를 설치하지 않았다면 [이전 포스트](https://blog.knowledgebox.online/linux/lnx-plex-with-docker)를 참고하여 먼저 Docker를 설치하세요.  



## 2. Docker로 Samba Server 실행하기  

[이전 포스트](https://blog.knowledgebox.online/linux/lnx-plex-with-docker)와 마찬가지로 Samba 컨테이너 역시 docker-compose를 사용해 실행합니다.  

### 설정  

[이전 포스트](https://blog.knowledgebox.online/linux/lnx-plex-with-docker)에서 docker-compose용 YAML 설정 파일과 Plex Media Server의 설정을 Host PC에 저장하기 위해 홈디렉토리에 docker 디렉토리 및 하위 디렉토리를 생성하였습니다.  
이번 포스트의 Samba Server 컨테이너도 설정 및 데이터를 Host PC에 저장하기 위해 Host PC에 디렉토리를 아래와 같이 생성합니다.  
```bash
[계정@localhost ~]$ > cd ~/docker
[계정@localhost ~/docker]$ > mkdir samba
```

docker-compose.yml 파일에 다음과 같이 추가합니다.  
자신의 환경에 맞게 수정하세요.  
```bash
[계정@localhost ~]$ > cd ~/docker
[계정@localhost ~/docker]$ > vim docker-compose.yml
```

```yml
  samba:
    container_name: samba
    image: dperson/samba
    restart: unless-stopped
    ports:
     - 139:139/tcp
     - 445:445/tcp
    volumes:
     - /storage:/mnt
     - /home/계정/docker/samba:/etc/samba
    environment:
     - TZ=Asia/Seoul
     - USER=Samba 사용자 ID;Samba 사용자 비밀번호;Samba 사용자 UID(Option);Samba 사용자 그룹(Option)
```

**volumes:**  
Docker 컨테이너(Samba Server)와 Host PC의 디렉토리 연결을 위한 항목입니다.  

`/storage:/mnt`는 Host PC의 `/storage` 디렉토리를 Samba Server에서 Windows와 같은 Samba Client에게 공유디렉토리로 제공하기 위해 Docker 컨테이너(Samba Server)의 `/mnt` 디렉토리에 연결한다는 의미입니다.  
이 설정을 사용해 Host PC의 디렉토리를 Docker 컨테이너(Samba Server)를 통해 공유디렉토리로 서비스 할 수 있습니다.  
`/storage`를 자신의 환경에 맞게 변경하세요.  

`/home/계정/docker/samba:/etc/samba`는 Host PC의 `/home/계정/docker/samba` 디렉토리를 Docker 컨테이너(Samba Server)의 `/etc/samba` 디렉토리에 연결한다는 의미입니다.  
Docker 컨테이너(Samba Server)의 `/etc/samba` 디렉토리는 Samba Server의 설정이 저장되는 디렉토리입니다.  
이번 포스트에서는 설정을 저장하기 위함이 아니라 설정 파일을 불러오기 위한 용도로 사용합니다.  
기존에 사용하던 `smb.conf` 파일을 Host PC의 `/home/계정/docker/samba` 디렉토리에 복사하세요.  
만약 기존에 사용하던 `smb.conf` 파일이 없다면 [여기](https://github.com/zentyal/samba/blob/master/examples/smb.conf.default)에서 샘플을 다운로드 하세요.  
필자는 기존에 사용하던 `smb.conf` 파일에서 공유디렉토리 설정만 Docker 컨테이너(Samba Server)에 맞게 수정하였습니다.  
```
[storage]
    path=/mnt  # **volumes:**에서 설정한 Docker 컨테이너(Samba Server) 공유디렉토리
    valid users=smb_test  # 아래 **environment:**에서 설정한 사용자 계정
    writable=yes
    directory mask=0755
    create mask=0644
```

`/home/계정/docker/samba`및 `smb.conf`를 자신의 환경에 맞게 변경하세요.  

**environment:**  
Docker 컨테이너(Samba Server)의 환경변수를 설정하기 위한 항목입니다.  

`USER=Samba 사용자 ID;Samba 사용자 비밀번호;Samba 사용자 UID(Option);Samba 사용자 그룹(Option)`는 Docker 컨테이너(Samba Server)에 Samba 계정을 생성하기 위한 환경변수입니다.  
필자는 `USER=smb_test;Passwd0821;1000;users`과 같이 설정했으며 의미는 아래와 같습니다.  
1. smb_test라는 Linux 계정을 Docker 컨테이너(Samba Server)에 생성
2. smb_test 계정의 UID를 1000으로 설정
3. smb_test 계정을 users 그룹에 포함
4. smb_test 계정의 Samba 비밀번호를 Passwd0821로 설정(smbpasswd -a smb_test)

> *비밀번호에 특수문자(예를 들면 $)가 포함되어 있을 경우 비밀번호 등록이 제대로 되지 않을 수 있습니다.*  
> *필자의 경우 비밀번호를 Passwd0821$$으로 설정했을 때 비밀번호가 제대로 등록되지 않았습니다.*  

### 실행  

docker-compose.yml 파일을 수정한 후 다음과 같은 명령어로 컨테이너를 실행합니다.
```bash
[계정@localhost ~]$ > cd ~/docker
[계정@localhost ~/docker]$ > docker-compose up -d samba
```

docker-compose.yml에서 samba라는 이름을 가진 컨테이너를 백그라운드(-d)로 실행(up)하라는 의미입니다.  
`-d` 옵션을 주지 않으면 터미널 창을 닫을 때 Docker 컨테이너(Samba Server)가 같이 종료됩니다.  
만약 docker-compose.yml 파일에 설정된 모든 컨테이너를 실행하고자 한다면 `docker-compose up -d`와 같이 실행하면 됩니다.  
