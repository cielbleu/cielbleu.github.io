---
title: "Linux 서버 RAID HDD 자동 절전 설정하기"
description: "Linux 서버에 구성한 RAID HDD의 자동 절전을 설정하는 방법 정리"
category: Linux
tags: [Arch, Linux, RAID, 절전]
toc: true
toc_sticky: true
date: 2019-02-25 00:00:00
lastmod: 2019-02-25 00:00:00
sitemap:
  changefreq: daily
---

[Arch Linux](https://archlinux.org)에서 RAID를 구성한 경우 특정시간 이후 자동 절전 상태로 설정하는 방법에 대한 정리입니다.  

![Arch Linux](/assets/images/arch_linux_logo.svg)



# Linux 서버에 구성한 RAID HDD의 자동 절전 설정하기  

개인적인 목적으로 가정에 Linux 서버를 구성하여 운영하다 보면 자연스럽게 전기요금에 대해 고민을 하게 됩니다.  
특히 NAS 용도로 사용하기 위해 다수의 HDD를 장착한 경우 별도의 HDD 절전 설정 없이 하루종일 서버를 가동하면 생각보다 많은 전기요금이 나오는 것을 경험할 수 있습니다.  
  
예를 들어 가동시 평균 10W의 전력을 소모하는 HDD 4개로 RAID 5를 구성하면 한 달 동안 (10W x 4(개)) x 24(시간) x 30(일) = 28,800Wh = 28.8kWh만큼의 전력을 소비합니다.  
여기에 CPU 등이 소모하는 전력까지 더하면 아무리 저전력을 지원하는 부품으로 서버를 구성하더라도 서버 한 대를 한 달 동안 계속 가동할 경우 4~50kWh의 전력을 소모하게 됩니다.  
  
2~3인 가구의 한 달 평균 소비전력이 3~400kWh인 것을 감안하면 서버 한 대가 가정 전체 사용량의 10%가 넘는 전력을 사용하는 셈입니다.  
  
이번 포스트에서는 서버의 소비전력을 줄이기 위해 특정시간 이후 RAID HDD의 자동 절전 설정에 대해 정리해 보았습니다.  
  
  
## 1. RAID란?  

RAID는 Redundant Array Inexpensive Disk 또는 Redundant Array Independant Disk의 약자이며 초기에는 폐기하기에는 아깝고, 그렇다고 독립적으로 사용하기에는 성능이 부족한 디스크를 하나로 묶어 재활용한다는 개념이었습니다.  
요즘은 다수의 독립적인 디스크를 모아 고성능 및 고가용성을 구현하기 위한 개념으로 이해하는 것이 일반적입니다.  
  
일반적으로 기업에서는 고가용성을 위해 HDD 3개로 RAID 1 with spare를 많이 사용합니다.  
서버의 로컬디스크에는 OS 및 소프트웨어만 설치하고 데이터는 대부분 SAN이나 iSCSI 같은 별도 Storage를 연결하여 사용합니다.  
  
참고로 RAID는 무정지 시스템을 위해 사용하는 것이지 데이터 백업을 위한 것이 아닙니다. RAID 1로 미러링을 구성했다고 하더라도 랜섬웨어나 바이러스로 인해 데이터가 날아가면 백업 이외에는 방법이 없습니다.  
  
하지만 가정에서 대용량 데이터를 백업하기 위해 장비를 구성하는 비용도 만만치 않기 때문에, 개인서버에 Software RAID를 구성해서 최소한 HDD 고장으로 인한 데이터 유실은 어느 정도 방지하는 것도 좋은 방법입니다.  
  
필자의 개인서버는 Linux 기반에 철저히 폐쇄적인 환경이기 때문에 랜섬웨어나 바이러스로 데이터 손상보다는 HDD 고장으로 인한 데이터 유실을 방지하기 위해 Software RAID 5로 구성하였습니다.  
Linux 서버에서 Software RAID는 mdadm을 사용하여 구성하세요.  
  
  
## 2. 사전준비  

1. Linux 기반 운영체제  
2. Python3 >= 3.4.2  
3. hdparm >= 9.43  
  
이 포스트에서 사용한 운영체제는 Arch Linux이지만 다른 배포판도 아무런 문제 없이 사용할 수 있습니다.  
단, Systemd 기반이므로 Systemd 기반이 아닌 Linux 배포판에서는 사용이 어려울 수 있습니다.  
  
Python은 대부분의 리눅스 배포판에 기본 설치되는 것이 보통입니다.  
Python3 버전은 다음과 같이 확인합니다.  
```bash
[계정@localhost ~]$ > python3 -V
Python 3.7.2
```
  
hdparm 버전은 다음과 같이 확인합니다.  
```bash
[계정@localhost ~]$ > hdparm -V
hdparm v9.58
```
  
아직 hdparm이 설치되어 있지 않다면 각 배포판의 설치방법을 참고하여 설치하세요.  
```bash
//Arch Linux
[계정@localhost ~]$ > sudo pacman -Syu hdparm
```
  
  
## 3. raid-sleep  

### 다운로드  

[GitHub](https://github.com/thomask77/raid-sleep)에 접속하여 raid-sleep을 다운로드 받습니다.  
`Clone or download` 버튼을 클릭한 후 `Download ZIP` 버튼을 클릭하세요.  
다운로드 후 [이전 포스트](https://blog.knowledgebox.online/linux/lnx-using-scp/)를 참고하여 개인서버에 다운로드 받은 파일을 업로드 하세요.  
서버에 Git이 설치되어 있다면 `git clone https://github.com/thomask77/raid-sleep.git`로 다운로드 받는 것이 가장 빠르고 간편합니다.  
```bash
[계정@localhost ~/git]$ > git clone https://github.com/thomask77/raid-sleep.git
Cloning into 'raid-sleep'...
remote: Enumerating objects: 1, done.
remote: Counting objects: 100% (1/1), done.
remote: Total 39 (delta 0), reused 0 (delta 0), pack-reused 38
Unpacking objects: 100% (39/39), done.
[계정@localhost ~/git]$ > cd raid-sleep
[계정@localhost ~/git/raid-sleep]$ > ls -al
total 72
drwxr-xr-x 3 계정 users  4096 2019-02-25 16:07 ./
drwxr-xr-x 4 계정 users  4096 2019-02-25 16:07 ../
drwxr-xr-x 8 계정 users  4096 2019-02-25 16:07 .git/
-rw-r--r-- 1 계정 users    11 2019-02-25 16:07 .gitignore
-rw-r--r-- 1 계정 users 35147 2019-02-25 16:07 COPYING.TXT
-rw-r--r-- 1 계정 users   746 2019-02-25 16:07 README.md
-rwxr-xr-x 1 계정 users   389 2019-02-25 16:07 install.sh*
-rwxr-xr-x 1 계정 users  3422 2019-02-25 16:07 raid-sleep*
-rw-r--r-- 1 계정 users   338 2019-02-25 16:07 raid-sleep.conf
-rw-r--r-- 1 계정 users   278 2019-02-25 16:07 raid-sleep.service
```
  
  
### 설정  

먼저 raid-sleep.conf에 raid-sleep을 적용할 HDD를 입력해줘야 합니다.  
raid-sleep.conf는 다음과 같이 구성되어 있습니다.  
```vim
# Disks to monitor
#
# Example:
#
# DISKS=\
#  /dev/disk/by-id/ata-WDC_WD60EFRX-68TGBN1_WD-WX11D153XFP6 \
#  /dev/disk/by-id/ata-WDC_WD60EFRX-68TGBN1_WD-WX11D153XHSF \
#  /dev/disk/by-id/ata-WDC_WD60EFRX-68TGBN1_WD-WX31DC45THHF \
#  /dev/disk/by-id/ata-WDC_WD60EFRX-68TGBN1_WD-WX31DC46J9UN
#

# Standby timeout in seconds
#
TIMEOUT=1800
```
`DISKS` 부분과 TIMEOUT=1800 부분을 자신의 환경에 맞게 수정해야 합니다.  
  
`DISKS`는 find /dev/disk/by-id로 자신의 환경을 확인한 후 수정하세요.
```bash
[계정@localhost ~]$ > find /dev/disk/by-id
/dev/disk/by-id/
/dev/disk/by-id/md-name-localhost.localdomain:0
/dev/disk/by-id/md-uuid-98ddd4e5:30e839a0:6438b7f7:45fd1a8e
/dev/disk/by-id/ata-HGST_HDN724030ALE640_PK2234P9K9U78Y
/dev/disk/by-id/wwn-0x5000cca248eebc94
/dev/disk/by-id/ata-Samsung_SSD_750_EVO_120GB_S3J0NB0HB38383F-part3
/dev/disk/by-id/wwn-0x5002538d41836899-part3
/dev/disk/by-id/ata-Samsung_SSD_750_EVO_120GB_S3J0NB0HB38383F-part1
/dev/disk/by-id/wwn-0x5002538d41836899-part1
/dev/disk/by-id/wwn-0x5002538d41836899-part2
/dev/disk/by-id/ata-Samsung_SSD_750_EVO_120GB_S3J0NB0HB38383F-part2
/dev/disk/by-id/ata-Samsung_SSD_750_EVO_120GB_S3J0NB0HB38383F
/dev/disk/by-id/wwn-0x5002538d41836899
/dev/disk/by-id/usb-HP_iLO_Internal_SD-CARD_000002660A01-0:0-part1
/dev/disk/by-id/usb-HP_iLO_Internal_SD-CARD_000002660A01-0:0
/dev/disk/by-id/wwn-0x5000cca248eeb9be
/dev/disk/by-id/ata-HGST_HDN724030ALE640_PK1234P9K9TGVX
/dev/disk/by-id/wwn-0x5000cca248eebbfe
/dev/disk/by-id/ata-HGST_HDN724030ALE640_PK2234P9K9U2EY
/dev/disk/by-id/ata-HGST_HDN724030ALE640_PK2234P9K98SBY
/dev/disk/by-id/wwn-0x5000cca248ee7eb6
```
필자의 경우 ata-HGST~~가 RAID로 구성한 HDD이므로 raid-sleep.conf를 이것으로 수정하고, TIMEOUT은 10분(TIMEOUT=600)으로 수정하였습니다.(DISKS 부분의 주석을 삭제하는 것을 잊지 마세요.)  
```vim
# Disks to monitor
#
# Example:
#
 DISKS=\
  /dev/disk/by-id/ata-HGST_HDN724030ALE640_PK1234P9K9TGVX \
  /dev/disk/by-id/ata-HGST_HDN724030ALE640_PK2234P9K98SBY \
  /dev/disk/by-id/ata-HGST_HDN724030ALE640_PK2234P9K9U2EY \
  /dev/disk/by-id/ata-HGST_HDN724030ALE640_PK2234P9K9U78Y
#

# Standby timeout in seconds
#
TIMEOUT=600
```
  
  
### 설치  

설치는 아주 간단합니다.  
설정 완료 후 아래의 명령어로 설치하세요.  
```bash
[계정@localhost ~/git/raid-sleep]$ > sudo ./install.sh
```
  
정상적으로 설치가 되었는지 아래의 명령어로 확인해 보세요.  
```bash
[계정@localhost ~]$ > sudo systemctl status raid-sleep
● raid-sleep.service - Power down RAID disks after a specified timeout
   Loaded: loaded (/etc/systemd/system/raid-sleep.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2019-02-25 16:46:13 KST; 13min ago
 Main PID: 9021 (python3)
    Tasks: 1 (limit: 4656)
   Memory: 9.7M
   CGroup: /system.slice/raid-sleep.service
           └─9021 python3 /usr/local/sbin/raid-sleep --timeout 600 /dev/disk/by-id/ata-HGST_HDN724030ALE640_PK1234P9K9T>

 2월 25 16:46:13 DataHub systemd[1]: Started Power down RAID disks after a specified timeout.
 2월 25 16:46:13 DataHub raid-sleep[9021]: Monitoring /dev/sdd, /dev/sdb, /dev/sdc, /dev/sda. Timeout = 0:10:00
 2월 25 16:56:13 DataHub raid-sleep[9021]: Powering down after 0:10:00
 2월 25 16:58:27 DataHub raid-sleep[9021]: Waking up after 0:12:14 of inactivity
 ```
  
HDD가 사용중일 때에는 다음과 같이 표시됩니다.  
```bash
[계정@localhost ~]$ > sudo hdparm -C /dev/sdc

/dev/sdc:
 drive state is:  active/idle
```

HDD가 절전 상태일 경우에는 다음과 같이 표시됩니다.  
```bash
[계정@localhost ~]$ > sudo hdparm -C /dev/sdc

/dev/sdc:
 drive state is:  standby
```
