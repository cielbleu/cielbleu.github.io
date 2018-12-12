---
title: "Linux 서버와 파일 주고받기"
description: "SCP를 이용하여 Linux 서버와 파일을 주고받는 방법 정리"
category: Linux
tags: [Linux, SSH, SCP]
toc: true
toc_sticky: true
---

SSH로 접속한 원격지 Linux 서버와 파일을 주고받는 방법에 대한 정리입니다.  

![SSH](/assets/images/Document_text_security.svg)



# Linux 서버와 파일 주고받기  

원격지에 위치한 Linux 서버에 SSH로 접속하여 작업을 하다보면 내 PC의 파일을 Linux 서버에 복사하거나, Linux 서버의 파일을 내 PC로 복사해야 하는 상황이 종종 발생합니다.  
이런 경우 전통적으로 FTP(File Transfer Protocol) 서버를 구축해 사용하였으나, FTP 서버를 별도로 구축해야 하는 점 및 기본설정으로 사용할 때 보안상 취약점 노출 등의 이유로 관리목적의 파일 전송을 위한 용도로는 권장하지 않고 있습니다. (FTP 서버는 다중사용자 대상으로 파일 다운로드 서비스 제공을 위한 용도로만 사용하는 것이 좋습니다)  
  
이번 포스트에서는 이런 경우에 유용하게 사용할 수 있는 SCP를 이용한 파일 전송에 대해 정리해 보았습니다.  


## 1. SCP란?  

SCP(Secure Copy Protocol)는 로컬 호스트와 원격 호스트 간 또는 두 원격 호스트 간에 파일을 안전하게 전송하는 수단입니다.  
SSH(Secure Shell) 기반으로 동작하며, 일반적으로 "SCP"라고 하면 Secure Copy Protocol과 프로그램 자체를 모두 의미한다고 할 수 있습니다.  
자세한 정보는 [Wikipedia](https://en.wikipedia.org/wiki/Secure_copy)를 참조하세요.  



## 2. SCP 서버 준비하기  

원격지에 위치한 Linux 서버에 SSH로 접속할 수 있는 환경이 구성되어 있다면 별도의 작업은 필요 없습니다.  
SCP는 SSH(Secure Shell) 프로토콜 기반으로 동작하기 때문에 SSH 서버가 실행되고 있고 접속할 수 있으면 됩니다.  



## 3. SCP 클라이언트 준비하기  

Windows 환경에서 SSH 접속을 위해 [putty](https://www.putty.org/)를 사용하고 있다면 SCP 클라이언트로 사용할 수 있는 pscp.exe가 같이 설치되어 있을 것입니다.  
다만 pscp.exe는 명령줄 인터페이스만 제공하므로 사용하기가 어렵습니다.  
  
이 포스트에서는 Windows용 GUI를 제공하면서 Free SCP 클라이언트인 [WinSCP](https://winscp.net/eng/docs/lang:ko)를 사용하였습니다. 특히 한국어 언어팩을 제공하기 때문에 추천하지 않을 이유가 하나도 없는 클라이언트입니다.  
  
우선 [WinSCP 다운로드 페이지](https://winscp.net/eng/download.php)에 접속하여 소프트웨어를 다운로드 받습니다.  
설치하지 않고 사용할 수 있는 Portable용 다운로드도 제공하고 있으니 참고하세요. (이 포스트에서는 Portable용을 다운로드 받아 사용하였습니다)  

그리고 한국어로 사용하고자 할 경우 [WinSCP-5.13.5 버전 언어팩](https://winscp.net/eng/translations.php?v=5.13.5.8967&lang=0412&isinstalled=0&utm_source=winscp&utm_medium=app&utm_campaign=5.13.5)을 다운로드 받아 WinSCP 폴더에 압축을 풀어주세요.  
만약 WinSCP의 버전이 5.13.5가 아니면 WinSCP 설정화면에서 제공하는 언어팩 다운로드 링크를 사용하세요.  
- WinSCP.exe : GUI 클라이언트  
- WinSCP.com : CUI 클라이언트(주로 *.bat 또는 *.cmd 같은 배치파일에서 원격지 서버에 파일을 전송하고자 할 때 사용)  
- WinSCP.ko : 한국어 언어팩  
- WinSCP.ini : 설정파일(WinSCP.exe를 실행하면 생성됨)  
![WinSCP #1](/assets/images/WinSCP_1.png)


### 설정  

WinSCP.exe를 실행하면 로그인할 서버를 선택하는 사이트 관리자 화면이 자동으로 실행됩니다.  
좌측 하단의 `도구 → 설정`을 클릭하여 입맛에 맞게 설정합니다.  
기본설정을 그대로 사용해도 괜찮습니다.  
이 포스트에서는 폰트와 Putty/터미널 클라이언트 경로만 변경하였습니다.  
![WinSCP #2](/assets/images/WinSCP_2.png)

설정이 완료되면 사이트 관리자 화면에서 접속하고자 하는 사이트를 등록합니다.  
1. 파일 프로토콜 : SCP 선택  
2. 호스트 이름 : 접속하고자 하는 원격지 Linux 서버의 공인 IP 또는 URL  
3. 포트 번호 : 접속하고자 하는 원격지 Linux 서버의 SSH 포트  
4. 사용자 이름 및 비밀번호 : 설정하지 않아도 됩니다.  
![WinSCP #3](/assets/images/WinSCP_3.png)


### 접속  
처음 접속하면 이 서버를 신뢰할 것인지 묻는 경고창이 반겨줍니다.  
![WinSCP #4](/assets/images/WinSCP_4.png)
  
`예` 버튼을 누르고 계속 진행합니다.  
사용자 이름과 비밀번호를 입력합니다.  
![WinSCP #5](/assets/images/WinSCP_5.png)
  
정상적으로 접속이 되면 내 PC와 원격지 Linux 서버 간에 파일을 주고받을 수 있습니다. (Drag & Drop)  
![WinSCP #6](/assets/images/WinSCP_6.png)
  
이 외에도 디렉토리 비교, 동기화, 터미널 실행 등 여러 가지 기능이 있습니다.  
