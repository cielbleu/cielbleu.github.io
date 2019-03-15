---
title: "Wayland 마우스 버튼 매핑"
description: "Wayland 디스플레이 환경에서 특정 마우스 버튼을 다른 키로 매핑하는 방법"
category: Linux
tags: [Arch, Linux, Wayland, 마우스 키 매핑, xbindkeys]
toc: true
toc_sticky: true
date: 2019-03-14 00:00:00
lastmod: 2019-03-14 00:00:00
sitemap:
  changefreq: daily
---

[Arch Linux](https://archlinux.org)에서 디스플레이 서버를 X Window System(X11)이 아닌 Wayland를 사용할 때 특정 마우스 버튼을 키보드의 다른 키로 매핑하는 방법에 대한 정리입니다.  

![Arch Linux](/assets/images/arch_linux_logo.svg)



# Wayland 디스플레이 서버에서 마우스 키 매핑 설정  

현재까지 리눅스에서는 GUI를 구현하기 위해 30년 가까이 된 낡은 코드에 이것저것 기능을 더한(포니에 블랙박스 달고, 네비게이션 달고, HUD 달아서 쓰는 상황???) X Window System을 사용하고 있습니다.  
하지만 누더기 상태나 다름 없는 X Window System 코드를 계속 유지보수하고 개선하는 것은 한계가 있다는 공감대가 형성되어 새로운 디스플레이 서버를 만드는 프로젝트들이 시작되었습니다.  
  
이 포스트에서 다루고 있는 Wayland도 이런 프로젝트의 결과물입니다.  
이외에도 현재는 중단된 상태지만 [Ubuntu](https://www.ubuntu,com)를 배포하고 있는 캐노니컬에서 Mir라는 디스플레이 서버 프로젝트를 진행하기도 했습니다. (근데 또 이걸 Fork해서 만드는 사람들도 있다고 합니다.)  
  
Wayland 프로젝트가 시작된지는 꽤 오래되었지만 아직도 대세는 X Window System입니다.  
하지만 리눅스 양대 데스크탑 환경인 Gnome과 KDE에서 Wayland를 적극 지원하고 있어 조만간 Wayland가 X Window System을 제치고 대세가 될 것 같습니다.  

본 필자도 이번에 새롭게 아치 리눅스를 설치하면서 디스플레이 서버를 Wayland로 바꿨는데, 몇 가지 소소한 문제점을 제외하면 실사용에 크게 문제가 되지는 않는 상황입니다.  
  
이번 포스트에서는 Wayland 기반에서 몇 가지 문제점 중 하나인 마우스 키 매핑에 대해 정리해 보았습니다.  
  
  
## 1. Wayland란?  
