---
title: 리눅스 로그인 화면(GDM)에서 Auto Suspend 끄기
author: chromato99
date: 2022-06-12 10:00:00 +0900
media_subpath: /assets/img/posts/2022-06-12-리눅스-로그인-화면(GDM)에서-auto-suspend-끄기/
categories: [Linux, Installation and Configuration]
tags: [linux, configuration, autosuspend, gdm, gnome]
---

리눅스로 세팅해둔 데스크탑을 원격 접속하기 위해 wake on lan 설정을 해두었는데 settings에서 auto suspend를 off 해도 컴퓨터가 절전모드로 들어가는 문제가 있었다.

이를 어떻게 해결하면 될지 알아보고자 한다.

## Auto Suspend

Auto Suspend 란 자동 절전모드로 컴퓨터를 일정시간 이상 사용하지 않으면 컴퓨터를 일시 중단해서 전력 소모량을 줄이는 기능이다.

보통 Gnome의 경우 오른쪽 위에 메뉴창에서 Settings를 선택하면 설정창을 열 수 있다.

여기서 Power 탭을 눌러 auto suspend 관련 설정을 바꿀 수 있다. 하지만 이는 Gnome 데스크탑 환경이 로그인 후 본격적으로 켜졌을때의 설정을 한것으로 그 이전에 그래픽 로그인 창을 띄워주는 디스플레이 매니저인 GDM 화면에서는 적용되지 않는다.

보통 컴퓨터를 처음 키면 이 GDM 창이 나오게 되는데 원격으로 부팅했을 때는 그래픽 환경에서 로그인을 할 수 없으므로 컴퓨터가 일정시간이 지나면 절전모드로 들어가게 된다. 따라서, 이 GDM에서의 절전모드 설정을 바꿔주려 한다.

## GDM Auto Suspend 끄기

GDM은 Settings창으로 설정 할 수 없기 때문에 터미널에서 명령어를 입력해줘야 한다.

[아치위키](https://wiki.archlinux.org/title/GDM#GDM_auto-suspend_(GNOME_3.28)) 내용에 따르면 아래 명령을 입력해주면 Auto Suspend를 아예 꺼버릴 수 있다고 한다.

```shell
sudo -u gdm dbus-launch gsettings \
  set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'
```

## Gnome Auto Suspend 설정을 가져오기

단순히 auto suspend를 꺼버리는게 아니라 Gnome Settings에서 설정한 auto suspend 설정(꺼지는 시간 등)을 가져오고 싶을 수도 있다.

그럴 때에는 아래 명령을 입력해서 Gnome Settings 설정을 가져와 적용할 수 있다고 한다.

```shell
IFS=$'\n'; \
  for x in $(sudo -u username gsettings list-recursively org.gnome.settings-daemon.plugins.power); \
  do eval "sudo -u gdm dbus-launch gsettings set $x"; done; \
  unset IFS
```

> username은 설정을 가져오고 싶은 리눅스 유저명을 입력해 주면 된다.
{: .prompt-info }

## Reference

<https://wiki.archlinux.org/title/GDM>
