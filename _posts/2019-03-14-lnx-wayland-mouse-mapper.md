---
title: "Wayland 마우스 버튼 매핑"
description: "Wayland 디스플레이 환경에서 특정 마우스 버튼을 다른 키로 매핑하는 방법"
category: Linux
tags: [Arch, Linux, Wayland, 아치 리눅스, 마우스 키 매핑, xbindkeys]
toc: true
toc_sticky: true
date: 2019-03-14 00:00:00
lastmod: 2019-03-14 00:00:00
sitemap:
  changefreq: daily
---

[Arch Linux](https://archlinux.org)에서 디스플레이 서버를 X Window System(X11)이 아닌 Wayland를 사용할 때 특정 마우스 버튼을 키보드의 다른 키로 매핑하는 방법에 대한 정리입니다.  

![X Window System](/assets/images/XOrg_Logo.svg){:height="50%" width="50%"}![Wayland](/assets/images/Wayland_Logo.svg){:height="50%" width="50%"}  



# Wayland 디스플레이 서버에서 마우스 키 매핑 설정  

현재까지 리눅스에서는 GUI를 구현하기 위해 30년 가까이 된 낡은 코드에 이것저것 기능을 더한(포니에 블랙박스 달고, 내비게이션 달고, HUD 달아서 쓰는 상황???) X Window System을 사용하고 있습니다.  
하지만 누더기 상태나 다름없는 X Window System 코드를 계속 유지보수하고 개선하는 것은 한계가 있다는 공감대가 형성되어 새로운 디스플레이 서버를 만드는 프로젝트들이 시작되었습니다.  
  
이 포스트에서 다루고 있는 Wayland도 이런 프로젝트의 결과물입니다.  
이외에도 현재는 중단된 상태지만 [Ubuntu](https://www.ubuntu,com)를 배포하고 있는 캐노니컬에서 Mir라는 디스플레이 서버 프로젝트를 진행하기도 했습니다. (근데 또 이걸 Fork 해서 개발하고 있는 사람들도 있다고 합니다.)  
  
Wayland 프로젝트가 시작된 지는 꽤 오래되었지만, 아직도 대세는 X Window System입니다.  
하지만 리눅스 양대 데스크탑 환경인 Gnome과 KDE에서 Wayland를 적극적으로 지원하고 있어 조만간 Wayland가 X Window System을 제치고 대세가 될 것 같습니다.  

본 필자도 이번에 새롭게 아치 리눅스를 설치하면서 디스플레이 서버를 Wayland로 바꿨는데, 몇 가지 소소한 문제점을 제외하면 실사용에 크게 문제가 되지는 않는 상황입니다.  
  
이번 포스트에서는 Wayland 기반에서 몇 가지 문제점 중 하나인 마우스 키 매핑에 대해 정리해 보았습니다.  
  
  
## 1. Wayland란?  

Wayland에 대해 잘 설명해 놓은 블로그가 있기에 링크로 대체합니다.  
[NEMO-UX 블로그](https://nemoux00.wordpress.com/2013/08/28/wayland-1-%EC%86%8C%EA%B0%9C/)를 참고하세요.  
  
  
## 2. 마우스 키 매핑  

요즈음 출시되는 마우스는 사용상 편의를 위해 다양한 기능을 제공하는 버튼이 포함된 경우가 많습니다.  
Forward와 Back 버튼이 대표적인데 이것 외에도 감도 조정, 제스쳐, 무한 휠 등 제조사별로 다양한 기능을 제공하는 여분의 버튼이 포함되어 있습니다.  
  
다만 이러한 기능을 100% 사용하려면 Microsoft Windows를 사용해야 한다는 제약사항이 있습니다.  
Microsoft Windows에서는 제조사에서 제공한 소프트웨어를 설치하면 마우스 버튼의 기능을 변경할 수 있도록 UI를 제공하고 있습니다.  
![Logitech Option](/assets/images/logitech_option.png){:height="50%" width="50%"}  
  
하지만 아쉽게도 Linux용 소프트웨어를 제공하는 제조사는 아직까지 없습니다.  
대신 몇몇 오픈소스로 제한적으로나마 이러한 기능을 제공하는 [소프트웨어](https://github.com/libratbag/piper)가 있습니다.  
![Piper](/assets/images/piper_buttonpage.png){:height="50%" width="50%"}  
문제는 아직까지는 X Window System 기반으로 동작하고 있기에 Wayland에서는 정상적으로 동작하지 않는 상황입니다.  
  
필자의 경우 X Window System을 디스플레이 서버로 사용할 때에는 마우스의 특정 버튼을 키보드의 다른 키로(예를 들어 Alt + F4) 매핑하기 위해 xautomation과 xbindkeys를 설치하여 사용했습니다.  
문제는 xautomation과 xbindkeys 역시 Wayland에서 제대로 동작하지 않는다는 것입니다.  
  
다행히 비슷한 문제로 고민을 하는 사람들이 있었고 [mathportillo](https://github.com/mathportillo/wayland-mouse-mapper)라는 GitHub 사용자가 systemd 서비스로 사용할 수 있도록 스크립트를 공개하였습니다.  
  
다운로드 후 테스트를 해 본 결과 지정한 마우스 버튼 클릭 이벤트 발생시 키보드의 특정 키조합이 발생한 것처럼 넘겨주는 부분이 동작하지 않았습니다. (사실 이 부분이 핵심인데...)  
systemd 로그를 확인한 결과 지정한 마우스 버튼 클릭시 특정 키조합이 발생은 하지만, 키보드에서 키조합이 발생한 것처럼 넘겨주는 부분이 제대로 동작하지 않았습니다.  
  
필자는 bash 쉘 스크립트나 프로그래밍에 대해 잘 알지 못하기 때문에 이에 대한 원인을 찾지는 못하였고, 스크립트에 키보드 하드웨어 변수로 지정된 부분을 강제로 제 환경에 맞게 지정하는 것으로 마무리를 지었습니다.  
  
bash 쉘 스크립트를 잘 아시는 분이 조금 더 나은 해결방법을 제시해 주시면 좋겠습니다.  
  
### Prerequisites  

우선 이 스크립트를 사용하기 위해서 필요한 소프트웨어가 설치되어 있어야 합니다.  
아래 2가지 소프트웨어 설치 여부를 확인하고 아직 설치되어 있지 않다면 설치합니다.  
- libinput
- evemu

필자가 사용하는 Arch Linux에서는 libinput은 설치되어 있었고, evemu는 설치되어 있지 않아 추가로 설치했습니다.  
```bash
[계정@localhost ~]$ > sudo pacman -Syu evemu
```
  
### 스크립트 다운로드  

스크립트에 필요한 소프트웨어를 설치한 후 Github에서 소프트웨어를 다운로드 합니다.  
여러 방법이 있지만 역시 가장 간편한 방법은 git으로 직접 다운로드 받는 방법입니다.  
git이 설치되어 있지 않다면 git을 먼저 설치하세요.  
```bash
//git 설치
[계정@localhost ~]$ > sudo pacman -Syu git

//스크립트 다운로드
[계정@localhost ~]$ > mkdir git
[계정@localhost ~]$ > cd git
[계정@localhost ~/git]$ > git clone https://github.com/mathportillo/wayland-mouse-mapper
```
  
### 스크립트 설정  

스크립트 다운로드 후 자신의 목적에 맞게 수정을 해야 합니다.  
개발자는 다음과 같이 초기 설정을 해놨습니다.  
```vim
# COMMANDS MAP
BTN_EXTRA=(KEY_LEFTMETA KEY_PAGEUP)
BTN_SIDE=(KEY_LEFTMETA KEY_PAGEDOWN)
```
이 부분을 자신이 사용하고자 하는 목적대로 수정해야 합니다.  
필자는 Logitech G500s 게이밍 마우스를 사용하고 있는데 EXTRA 버튼을 누르면 활성화된 창을 닫도록 (ALT + F4) 키를 누른 것처럼 사용하기 위해 아래와 같이 설정하였습니다.  
```vim
# COMMANDS MAP
BTN_EXTRA=(KEY_LEFTALT KEY_F4)
```
  
설정을 완료했지만, 테스트를 해 본 결과 위에서 문제점으로 기술했다시피 실제 키보드에서 (ALT + F4) 키를 누른 것처럼 동작하지 않는 문제점이 있었습니다.  
스크립트에서 아래 부분의 코드인데... `device=$1`에서 `$1`에 키보드 하드웨어가 설정되어야 하는데 이 부분이 잘 동작하지 않는 것 같습니다.  
```vim
function pressCommand(){
    device=$1; button=$2; movement=$3
    var=$button[@]
    command=${!var}

    if [ ${movement} = ${pressed} ]; then
        for key in ${command}; do
            pressKey ${device} ${key} 1
        done
    else
        for key in ${command}; do
            pressKey ${device} ${key} 0
        done | tac
    fi
}
```
  
필자는 프로그래밍에 문외한이다 보니 이에 대한 원인을 찾지는 못하였고, 직접 키보드 하드웨어를 지정해주는 방식으로 문제점을 해결하였습니다.  
`sudo libinput list-devices`로 조회했을 때 필자의 키보드 하드웨어는 `/dev/input/event4`였는데 이를 스크립트에 직접 지정했습니다.  
```vim
function pressCommand(){
    device=/dev/input/event4; button=$2; movement=$3
    var=$button[@]
    command=${!var}

    if [ ${movement} = ${pressed} ]; then
        for key in ${command}; do
            pressKey ${device} ${key} 1
        done
    else
        for key in ${command}; do
            pressKey ${device} ${key} 0
        done | tac
    fi
}
```
  
### systemd 서비스 등록  

부팅시에 이 스크립트가 자동으로 실행되도록 아래와 같이 systemd 서비스로 등록해 줍니다.  
```bash
//쉘 스크립트 복사
[계정@localhost ~/git/wayland-mouse-mapper]$ > sudo cp mousemapper.sh /usr/bin/mousemapper

//systemd 서비스 등록
[계정@localhost ~/git/wayland-mouse-mapper]$ > sudo cp mousemapper.service /usr/lib/systemd/system

//systemd mousemapper.service 활성화
[계정@localhost ~/git/wayland-mouse-mapper]$ > sudo systemctl enable mousemapper.service
```
  
systemd 서비스로 등록한 이후 설정을 변경할 사항이 발생하면 다음과 같이 진행하면 됩니다.  
```bash
//vim 또는 선호하는 에디터로 설정 수정
[계정@localhost ~]$ > sudo vim /usr/bin/mousemapper

//설정 수정 후 mousemapper.service 재실행
[계정@localhost ~]$ > sudo systemctl restart mousemapper.service
```
