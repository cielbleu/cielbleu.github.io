---
title: "DDNS(Dynamic DNS) 자동 업데이트 설정하기"
description: "Linux 서버에서 DDNS 자동 업데이트 설정하기"
category: Linux
tags: [Linux, DDNS, 개인서버, DNSZi]
toc: true
toc_sticky: true
date: 2019-03-06 00:00:00
lastmod: 2019-03-06 00:00:00
sitemap:
  changefreq: daily
---

[Archlinux](https://archlinux.org)를 사용하여 구성한 서버에서 DDNS 자동 업데이트를 설정하는 방법에 대한 정리입니다.  

![Arch Linux](/assets/images/arch_linux_logo.svg)



# Linux 서버에서 DDNS 자동 업데이트 설정하기  

개인적인 목적으로 가정에 Linux 서버 또는 NAS를 구성하여 사용하다 보면 몇 가지 고민거리가 생깁니다.  
예를 들어  
1. 외부에서 서버에 접속하고 싶은데 IP가 바뀌면 매번 확인하기 힘들다. 돈 더 내고 고정 IP 서비스를 신청할까?  
2. IP를 외우기가 힘들다. naver.com처럼 접속할 수는 없을까?  
  
보통 이런 경우에 공유기에서 제공하는 DDNS를 사용하면 문제가 해결됩니다.  
하지만 한 가지 아쉬운 점이 있는데...개인도메인을 보유하고 있을 경우 개인도메인을 사용하지 못한다는 것입니다.  
공유기에 따라 다르지만, 우리나라에서 많이 사용하는 iptime 공유기는 도메인이 고정되어 있습니다. (＊＊＊.iptime.org)  
  
이번 포스트에서는 개인도메인을 DDNS에 등록하고 특정시간마다 자동으로 업데이트 하는 방법에 대해 정리해 보았습니다.  
  
  
## 1. DDNS란?  

도메인 네임 시스템(Domain Name Sysem, DNS)은 인터넷에 연결된 호스트의 도메인을 호스트의 네트워크 주소(IP)로 변환하거나 그 반대의 변환을 해주는 서비스입니다.  
상호간 변환을 위한 서비스인만큼 가정용 인터넷 회선처럼 IP가 종종 변경되는 환경에서는 사용하기 어렵습니다.  
DNS에 대한 자세한 내용은 [위키백과](https://ko.wikipedia.org/wiki/%EB%8F%84%EB%A9%94%EC%9D%B8_%EB%84%A4%EC%9E%84_%EC%8B%9C%EC%8A%A4%ED%85%9C)를 참고하세요.  
  
Dynamic DNS(DDNS)는 유동적으로 IP가 변동되는 상황에서 서버를 운용해야 하는 경우에 발생하는 DNS 문제점을 극복하기 위해 고안된 것입니다.  
  
  
## 2. 사전준비  

### 도메인 구입  

만약 아직 개인도메인이 없다면 도메인을 먼저 준비해야 합니다.  
도메인은 [가비아](https://www.gabia.com/)나 [후이즈](https://domain.whois.co.kr/), [GoDaddy](https://kr.godaddy.com/) 등의 업체에서 구입할 수 있습니다.  
업체별로 가격이나 혜택이 조금씩 다르므로 여러 업체를 비교해 보신 후 구입하는 것이 좋습니다.  
  
도메인을 구입했다면 네임서버를 DDNS 업체에서 제공하는 네임서버로 변경해 줍니다.  
이 포스트에서는 [DNSZi](https://dnszi.com/)를 사용할 예정이므로 DNSZi에서 제공하는 네임서버로 변경하였습니다.  
![Name Server](/assets/images/name_server.PNG)  
  
  
### DDNS 서비스 가입 및 도메인 등록  

도메인 구입과 마찬가지로 DDNS 또한 유/무료로 서비스를 제공하는 업체가 많습니다.  
이 포스트에서는 무료로 DDNS 서비스를 제공하고 Linux 서버에서 별도의 소프트웨어 설치 없이 DDNS 자동 업데이트를 지원하는 [DNSZi](https://dnszi.com/)를 사용할 예정입니다.  
  
DNSZi 또는 다른 DDNS 서비스 업체에 가입하고 구입한 개인도메인을 등록해 주세요.  
그리고 DDNS 업체에서 제공하는 네임서버 정보를 도메인을 관리하는(이 포스트에서는 도메인을 구입한 GoDaddy에서 관리합니다.) 업체의 홈페이지에 접속하여 개인도메인의 네임서버 정보를 변경해 주세요.  
  
  
## 3. DDNS 설정  

개인도메인 구입과 등록이 완료되었으므로 DDNS 자동 업데이트를 위한 설정을 위해 DNSZi 사이트에 접속하여 필요한 설정을 해줍니다.  
  
### A 레코드 등록  

DNSZi에 접속한 다음 도메인 목록에서 자신의 도메인을 선택하면 도메인을 관리할 수 있습니다.  
상단의 탭에서 **호스트 IP 관리(A 레코드)**를 선택하세요.  
  
아래 이미지처럼 붉은색 네모 부분에 자신의 환경에 맞게 등록해 줍니다.  
  
|   항 목   |             설  명            |                    비 고                    |
|:---------:|:-----------------------------:|:-------------------------------------------:|
|  A 레코드 | 등록하고자 하는 호스트명 입력 | 생략 가능                                   |
|  IP 주소  | 자신의 현재 공인IP 주소 입력  | 잘 모르면 10.10.10.10과 같은 임의의 IP 입력 |
| DDNS 설정 | ○로 변경                     | 필수                                        |
|  관리메모 | A레코드에 대한 메모 입력      | 생략 가능                                   |
  
자신의 현재 공인 IP 주소를 잘 모른다면 10.10.10.10과 같이 대충 입력하세요. 어차피 DDNS 자동 업데이트를 통해 자동으로 업데이트 하려고 하는 것이니까요.  
DDNS 설정은 반드시 ○로 변경해야 DDNS 자동 업데이트 기능을 사용할 수 있습니다.  
  
![DNSZi A Record](/assets/images/dnszi_a_conf.png)  
>A 레코드는 Address Mapping Records를 의미하는데 주어진 호스트에 대한 IP 주소(IPv4)를 알려줍니다.  
>IPv6에서는 A 레코드 대신 AAAA 레코드를 사용합니다.  
  
### CNAME 등록  

필자의 경우 개인서버와 별개로 [GitHub](https://github.com)에서 제공하는 기능을 사용하여 [개인블로그](https://blog.knowledgebox.online)를 운영하고 있는데 개인블로그도 개인도메인으로 사용하기 위해 CNAME을 추가로 등록했습니다.  
상단의 탭에서 **CNAME 관리(별명)**을 선택하세요.  
  
아래 이미지처럼 붉은색 네모 부분에 자신의 환경에 맞게 등록해 줍니다.  
필자의 경우 https://cielbleu.github.io라는 GitHub에서 제공하는 개인블로그 URL을 https://blog.knowledgebox.online으로 연결해서 사용하고자 CNAME을 등록했습니다.  
  
|     항 목     |                 설  명                 |   비 고   |
|:-------------:|:--------------------------------------:|:---------:|
|    CNAME 값   | 별칭으로 등록하고자 하는 호스트명 입력 | 필수      |
| 목적지 도메인 | CNAME과 연결하고자 하는 도메인 입력    | 필수      |
|    관리메모   | CNAME에 대한 메모 입력                 | 생략 가능 |
  
![DNSZi CNAME](/assets/images/dnszi_cname_conf.png)  
>CNAME은 Canonical Name인데 의미 그대로 별칭을 만드는데 사용합니다.  
>일반적으로 CNAME 레코드는 도메인을 외부 도메인과 연결하는데 유용합니다.  
  
### DDNS 등록  

마지막으로 DDNS 자동 업데이트를 위해 인증키를 설정해 줍니다.  

상단의 탭에서 **고급 관리**를 선택하세요.  
  
아래 이미지처럼 붉은색 네모 부분에서 `인증키 생성` 버튼을 클릭하여 인증키를 생성한 다음 `인증키 저장` 버튼을 클릭하여 생성된 인증키를 저장해 주세요.  
이 인증키는 개인서버에서 DDNS 자동 업데이트시 필요합니다.  
  
![DNSZi Advanced](/assets/images/dnszi_advanced_conf.png)  
  
  
## 4. DDNS 자동 업데이트 설정  

DNSZi에서는 리눅스 서버에서의 DDNS 자동 업데이트 방법을 [Cron](https://ko.wikipedia.org/wiki/Cron)을 사용하는 것으로 설명하고 있습니다.  
하지만 Arch Linux에서는 Cron이 기본적으로 설치되어 있지 않고, 요즘은 Cron보다는 Systemd로 통합되는 추세이므로 이 포스트에서는 Systemd를 사용하는 것으로 설명합니다.  
  
우선 Systemd로 DDNS 자동 업데이트를 사용하기 위해서 2개의 파일을 만들어야 합니다.  
각각 `.timer`와 `.service`입니다.  

### .timer 생성  

사용하기 편한 편집기를 사용해 `/etc/systemd/system/` 디렉토리에 `.timer` 파일을 생성합니다.  
```bash
[계정@localhost ~]$ > sudo vim /etc/systemd/system/public_ip_check.timer
```
root가 아닌 일반 계정을 사용하는 경우엔 /etc 디렉토리에 파일을 생성하기 위해서 sudo를 사용해야 합니다.  

```vim
# Public IP Check
[Unit]
Description = Run public_ip_check.service every 15 minutes

[Timer]
# Time to wait after booting before we run first time
OnBootSec = 5min
OnCalendar = *-*-* *:0/15:00

[Install]
WantedBy = default.target
```
**[Timer]**  
.timer 파일에서 가장 중요한 부분입니다.  
`OnBootSec = 5min`은 부팅 후 5분 후에 실행하라는 의미입니다.  
`OnCalendar = *-*-* *:0/15:00`는 매 15분마다 실행하라는 의미입니다.  
  
| 항목 | Year | - | Month | - |  Day |   |  Hour  | : |  MInute  | : | Second |
|:----:|:----:|:-:|:-----:|:-:|:----:|:-:|:------:|:-:|:--------:|:-:|:------:|
| 예시 |   *  | - |   *   | - |   *  |   |    *   | : |   0/15   | : |   00   |
| 설명 | 매년 |   |  매월 |   | 매일 |   | 매시간 |   | 15분마다 |   |        |
  
### .service 생성  

사용하기 편한 편집기를 사용해 `/etc/systemd/system/` 디렉토리에 `.service` 파일을 생성합니다.  
```bash
[계정@localhost ~]$ > sudo vim /etc/systemd/system/public_ip_check.service
```

```vim
# Public IP Check
[Unit]
Description = Public IP check for knowledgebox.online

[Service]
Type = simple
ExecStart = /usr/local/bin/curl -s 'http://ddns.dnszi.com/set.html?user=DNSZi 계정&auth=인증키&domain=개인도메인&record='
```
**[Service]**  
.service 파일에서 가장 중요한 부분입니다.  
`ExecStart = /usr/local/bin/curl -s` 는 curl을 진행 상태나 에러 정보를 보여주지 않는 Slient 모드로 실행하라는 의미입니다.  
`'http://ddns.dnszi.com/set.html?user=DNSZi 계정&auth=인증키&domain=개인도메인&record='` 이 부분이  DNSZi 사이트에 공인 IP 주소를 업데이트 하는 부분입니다.  
  
DNSZi 계정과 인증키, 개인도메인 부분을 자신의 상황에 맞게 변경하세요.  
  
`.service` 파일은 먼저 생성한 `.timer` 파일과 동일한 이름으로 생성하는 것이 좋습니다.  
만약 `.timer` 파일과 다른 이름으로 생성하고자 한다면 `timer` 파일의 **[Timer]** 부분에 아래와 같이 `Unit` 항목을 추가해야 합니다.  
```vim
 [Timer]
 # Time to wait after booting before we run first time
 OnBootSec = 5min
 OnCalendar = *-*-* *:0/15:00
 Unit = myService.service
```
  
### Systemd 서비스 등록  

`.timer`와 `.service` 파일을 생성했지만 실제로 동작하지는 않습니다.  
실제로 동작하도록 하기 위해서 Systemd에 `활성화(enable)`하고 `시작(start)`해줘야 합니다.  

```bash
//활성화
[계정@localhost ~]$ > sudo systemctl enable public_ip_check.timer

//시작
[계정@localhost ~]$ > sudo systemctl start public_ip_check.timer

//등록된 모든 타이머 확인
[계정@localhost ~]$ > sudo systemctl list-timers --all
NEXT                         LEFT          LAST                         PASSED       UNIT                         ACTIV>
Wed 2019-03-06 18:15:00 KST  14min left    Wed 2019-03-06 18:00:20 KST  19s ago      public_ip_check.timer        publi>
Thu 2019-03-07 00:00:00 KST  5h 59min left Wed 2019-03-06 00:00:07 KST  18h ago      logrotate.timer              logro>
Thu 2019-03-07 00:00:00 KST  5h 59min left Wed 2019-03-06 15:27:32 KST  2h 33min ago man-db.timer                 man-d>
Thu 2019-03-07 00:00:00 KST  5h 59min left Wed 2019-03-06 00:00:07 KST  18h ago      shadow.timer                 shado>
Thu 2019-03-07 17:54:23 KST  23h left      Wed 2019-03-06 17:54:23 KST  6min ago     systemd-tmpfiles-clean.timer syste>
Mon 2019-03-11 00:00:00 KST  4 days left   Mon 2019-03-04 00:00:07 KST  2 days ago   fstrim.timer                 fstri>
n/a                          n/a           n/a                          n/a          mdadm-last-resort@md0.timer  mdadm>

7 timers listed.
lines 1-10/10 (END)

//상태 확인
[계정@localhost ~]$ > sudo systemctl status public_ip_check.timer
● public_ip_check.timer - Run public_ip_check.service every 15 minutes
   Loaded: loaded (/etc/systemd/system/public_ip_check.timer; enabled; vendor preset: disabled)
   Active: active (waiting) since Wed 2019-03-06 17:39:09 KST; 21min ago
  Trigger: Wed 2019-03-06 18:15:00 KST; 14min left

 3월 06 17:39:09 DataHub systemd[1]: Started Run public_ip_check.service every 15 minutes.
```
