---
title: "Arch Linux 설치 A to Z"
description: "Arch Linux(아치 리눅스)를 설치방법 가이드"
category: Linux
tags: [Arch, Linux, 아치리눅스, 설치 가이드]
toc: true
toc_sticky: true
date: 2019-03-06 00:00:00
lastmod: 2019-03-06 00:00:00
sitemap:
  changefreq: daily
---

[Arch Linux](https://archlinux.org)를 설치하는 방법에 대한 정리입니다.  

![Arch Linux](/assets/images/arch_linux_logo.svg)



# Arch Linux 설치 가이드  
ls /sys/firmware/efi

lsblk

gfdisk /dev/sda
gfdisk /dev/sdb
gfdisk /dev/sdc
gfdisk /dev/sdd

mkfs.vfat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/sda3
mkfs.ext4 /dev/sdb1
mkfs.ext4 /dev/sdc1
mkfs.ext4 /dev/sdd1

tune2fs -L "System" -m 0 /dev/sda2
tune2fs -L "Home" -m 0 /dev/sda3
tune2fs -L "Storage1" -m 0 /dev/sdb1
tune2fs -L "Storage2" -m 0 /dev/sdc1
tune2fs -L "Storage3" -m 0 /dev/sdd1

mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
mkdir /mnt/home
mount /dev/sda3 /mnt/home
mkdir /mnt/storage1
mkdir /mnt/storage2
mkdir /mnt/storage3
mount /dev/sdb1 /mnt/storage1
mount /dev/sdc1 /mnt/storage2
mount /dev/sdd1 /mnt/storage3

nano /etc/pacman.d/mirrorlist
Server = http://mirror.premi.st/archlinux/$repo/os/$arch

pacstrap /mnt base base-devel

genfstab -U /mnt >> /mnt/etc/fstab

arch-chroot /mnt

passwd
New password:
Retype new password:
password: password updated successfully

nano /etc/locale.gen
en_US.UTF-8
kr_KR.UTF-8

locale-gen
Generating locales...
  en_US.UTF-8... done
  ko_KR.UTF-8... done
Genetation complete.

[cielbleu@DataHub ~]$ > cat /etc/locale.conf
```vim
# 영어로 LANG 설정
# 만약 한국어로 UTF-8 설정을 하려면 ko_KR.UTF-8로 설정
LANG="en_US.UTF-8"

# 기본 정렬 방식 유지하도록 설정('.' 파일이 디렉토리 목록의 처음에 표시됨)
LC_COLLATE="C"

#  YYYY년 MM월 DD일 (토) 오후 HH시 MM분 SS초 ("date +%c" 명령어로 시험) 형식으로 날짜 표시하기
LC_TIME="ko_KR.UTF-8"

LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="ko_KR.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="ko_KR.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
```

echo ArchBox > /etc/hostname

ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime

hwclock --systohc --utc

useradd -m -g users -G users,wheel -s /bin/bash cielbleu

passwd cielbleu
New password:
Retype new password:
password: password updated successfully

sudo vim /etc/group
users:x:985 > users:x:100으로 변경

pacman -S vim
resolving dependencies...
looking for conflicting package...

Packages (3) gpm-1.20.7.r27.g1fd1941-1 vim-runtime-8.1.0996-1 vim-8.1.0996-1

Total Download Size:   7.05MiB
Total Installed Size: 32.29MiB

:: Proceed with installation? [Y/n]

visudo
```vim
##
## User privilege specification
##
root ALL=(ALL) ALL

## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL

## Same thing without a password
# %wheel ALL=(ALL) NOPASSWD: ALL
```

```bash
[계정] bootctl --path=/boot install
Created "/boot/EFI".
Created "/boot/EFI/systemd".
Created "/boot/EFI/BOOT".
Created "/boot/loader".
Created "/boot/loader/entries".
Copied "/usr/lib/systemd/boot/efi/systemd-bootx64.efi" to "/boot/EFI/systemd/systemd-bootx64.efi".
Copied "/usr/lib/systemd/boot/efi/systemd-bootx64.efi" to "/boot/EFI/BOOT/BOOTX64.EFI".
Created EFI boot entry "Linux Boot Manager".
[계정]
```

```bash
[계정] vim /boot/loader/loader.conf
```
```vim
timeout 3
#HiDPI일 경우 1로 변경
console-mode 1
default ArchBox
editor 1
```

```bash
[계정] pacman -S intel-ucode 또는 amd-ucode
resolving dependencies...
looking for conflicting package...

Packages (1) intel-ucode-20180807.a-1

Total Download Size:  1.30MiB
Total Installed Size: 1.67MiB

:: Proceed with installation? [Y/n]

[계정] vim /boot/loader/entries/ArchBox.conf
```
```vim
title ArchBox
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
```
```bash
[계정] echo “options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/sda2) rw”>> /boot/loader/entries/arch.conf  
```

exit

umount -R /mnt

reboot

```bash
[계정] sudo systemctl start dhcpcd

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:
    #1) Respect the privacy of others.
	#2) Think before you type.
	#3) With great power comes great responsibility.
[sudo] password for cielbleu:
```

ping -c 3 8.8.8.8

sudo pacman -Syu

```bash
[계정] sudo pacman -Syu xorg-server gnome
....
Enter a selection (default=all):
:: Starting full system upgrade...
resolving dependencies...
:: There are 2 providers available for libgl:
:: Repository extra
   1) libglvnd   2) nvidia-340xx-utils

Enter a numner (default=1):  //1 입력

[계정] sudo systemctl enable gdm
```

VGA 드라이버
```bash
[계정] lspci | grep -e VGA -e 3D
00:02.0 VGA compatible controller: Intel Corporation Xeon E3-1200 v2/3rd Gen Core processor Graphics Controller (rev 09)

[계정] sudo pacman -S mesa xf86-video-intel vulkan-intel
[계정] sudo pacman -S mesa xf86-video-amdgpu vulkan-radeon libva-mesa-driver mesa-vdpau
```

reboot

```bash
[계정] sudo systemctl enable NetworkManager
[sudo] password for cielbleu:

[계정] sudo systemcl start NetworkManager
```


```bash
[계정] sudo vim /etc/pacman.conf
```
//아래 2개 주석 제거 구문은 Pacui를 사용할 때 구문임
Color 주석 제거
VerbosePkgListㄴ 주석 제거

pamac 설치
firefox 설치 후 설치 패키지 다운로드
cd Download/pamac-aur
makepkg -sic

pamac 환경설정(스크린샷)

Tweak tools 설치
pamac에서 검색 후 설치

한글 폰트 설치
pamac에서 Noto Sans CJK KR 검색 후 설치

폰트 변경
Gnome Tweak Tools > Fonts에서 변경(스크린샷)
Noto Sans CJK KR DemiLight 10으로 변경
Antialiasing은 subpixel로 변경
Windows에서 Center New Windows 활성화
Keyboard & Mouse > Additional Layout Options에서 한글/한자 설정(스크린샷)

Gnome shell integration 애드온 설치(Firefox)
pamac에서 chrome-gn
ome-shell 검색 후 설치

https://extensions.gnome.org 접속 후 애드온 설치
User Themes
Applications Menu
Native window placement
Alternatetab
Places status indicator
Workspace indicator
Openweather

테마 설치
pamac에서 arc 검색 후 설치(arc-gtk-theme arc-icon-theme arc-kde)

한글입력기 설치
pamac에서 nimf 검색 후 설치
https://extensions.gnome.org/extension/615/appindicator-support/ 설치
How to enable Nimf on systems using systemd v233 or later

    Create a file and write the following lines to ~/.config/environment.d/50-nimf.conf

      GTK_IM_MODULE=nimf
      QT4_IM_MODULE=nimf
      QT_IM_MODULE=nimf
      XMODIFIERS=@im=nimf

    Then append the following line to ~/.xprofile

      export $(/usr/lib/systemd/user-environment-generators/30-systemd-environment-d-generator)



xbindkeys 및 xautomation 설치(제외)

.vim 및 .vimrc 복사
.dircolor 복사
.bashrc 파일 수정(새로 만든 내용 복사하여 정리)

sshd_config 설정
sudo systemctl enable sshd
sudo systemctl start sshd

비디오 섬네일
ffmpegthumbnailer, gst-libav, gst-plugins-ugly 설치

Wayland mouse mapper 설치
git clone https://github.com/mathportillo/wayland-mouse-mapper
sudo cp mousemapper.sh /usr/bin/mousemapper
sudo cp mousemapper.service /usr/lib/systemd/system
sudo systemctl enable mousemapper.service
