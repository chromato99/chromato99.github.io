---
title: Arch Linux 설치
author: chromato99
date: 2022-03-27 2:00:00 +0900
media_subpath: /assets/img/posts/2022-03-27-Arch-Linux-설치/
categories: [Linux, Installation and Configuration]
tags: [linux, arch linux, installation, boot, secure boot]
---

![arch_logo](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e8/Archlinux-logo-standard-version.png/2880px-Archlinux-logo-standard-version.png)

[아치 리눅스](https://wiki.archlinux.org/title/Arch_Linux)는 리눅스 배포판 가운데 하나로 미니멀리즘과 빠른 업데이트로 유명하다.

평소 리눅스를 공부하고자 리눅스를 메인 운영체제로 사용하기 위해 노력하는데, 최근 아치 리눅스에 정착해 사용중이다.

다만 리눅스를 사용하다보면 이래저래 뻘짓을 하다보면 파일이 꼬이는 등의 문제가 생겨 클린 설치를 할 일이 생기는데 그럴때 참고하기위해 정리한 내용을 블로그 포스트로 써보고자 한다.

> 아치 리눅스의 공식 설치 가이드는 [아치위키](https://wiki.archlinux.org)의 공식 [Inastallation Guide](https://wiki.archlinux.org/title/Installation_guide) 하나이므로 이를 중점적으로 참고하였다.
{: .prompt-info }

> 개인 참고용의 성격이 강하기 때문에 최대한 이유 등을 설명하기 위해 노력하겠지만 세부 설정 등이 글쓴이의 취향이 반영되어 있을 가능성이 높다.
{: .prompt-info }

## 설치전 준비

### 설치용 미디어 다운받기

아치 리눅스 설치용 미디어는 아치리눅스 [공식 홈페이지](https://archlinux.org) 위의 메뉴에서 [다운로드](https://archlinux.org/download/) 페이지에서 다운 받을 수 있다.

토렌트로 다운받아도 되고 아래 HTTP 다운로드 링크에서 다운받을 수 있다. 글쓴이의 경우 한국 서버 주소에서 다운받는 편이다.

### 설치용 미디어 만들기 (Windows)

윈도우인 경우 [Rufus](https://rufus.ie/ko/)를 사용해 설치용 usb를 만드는것이 편하다.

최신 버전의 Rufus를 다운 받은뒤 장치에서 사용할 usb(포맷되니 주의!!)를 선택하고 선택 버튼을 눌러 위에서 다운받은 iso파일을 선택해준다.

볼륨 레이블 이름은 적당히 입력하고 나머지는 기본으로 두고 시작 버튼을 누르면 된다!

딱히 상관은 없지만 파티션 형식은 기본값인 MBR이 아닌 GPT(좀더 최신이다)로 선택해도 된다. 다만 GPT로 선택하면 UEFI 환경에서만 부팅 가능하다.

![rufus_img](/rufus_img.webp){: width="50%" height="50%" }

### 설치용 미디어 만들기 (Linux)

이미 리눅스 환경이라면 대부분 설치용 미디어를 만드는 법을 알고 있을 것이라 생각하지만 간단하게 서술해 보겠다.

본인이 GNOME같은 데스크탑을 사용중이고 GNOME Image Writer 같은게 설치되어 있다면 이를 사용해도 된다.

그것이 아니라면 터미널에서 명령어를 입력해도 된다. 아치위키의 [USB flash installation medium](https://wiki.archlinux.org/title/USB_flash_installation_medium) 문서를 참고했다.

우선 아래 명령어를 입력해 USB drive의 경로를 확인한다.

```shell
lsblk
```

그리고 해당 USB를 부팅용 USB로 만들기 위해서는 내용을 모두 지워줘야 하는 관계로 [wipefs](https://wiki.archlinux.org/title/Device_file#wipefs) 명령을 입력한다.

> 디바이스 경로 (/dev/sdx)에서 x는 자신의 디바이스 경로로 입력해 주면 된다. 
{: .prompt-info }

> 아래 명령어를 입력하면 USB의 모든 내용이 지워지므로 주의!!
{: .prompt-danger }


```shell
wipefs --all /dev/sdx
```

구체적인 경로는 자신의 USB 장치에 맞게 입력해야 한다.

그리고 마지막으로 아래 명령어를 입력하면 된다.

```shell
cat ./archlinux-2022.03.01-x86_64.iso > /dev/sdx
```

혹시나 위의 명령이 permission denied가 뜰때는 아래처럼 입력하면 된다.

```shell
sudo sh -c 'cat ./archlinux-2022.03.01-x86_64.iso > /dev/sdx'
```

만약 cat 명령이 아닌 cp나 dd를 사용하고 싶다면 [USB flash installation medium](https://wiki.archlinux.org/title/USB_flash_installation_medium) 문서를 참고하자.

USB 드라이브가 제대로 완성됬는지 확인하고 싶다면 [fdisk](https://wiki.archlinux.org/title/Fdisk) 명령어로 확인해볼 수 있다.

```shell
fdisk -l
```

만들어진 USB 드라이브에 EFI 파티션이 만들어 졌다면 크게 문제 없을 것이다.

### Secure Boot 끄기 (optional)

윈도우 11이 Secure Boot이 기본값으로 설정하게끔 하면서 많은 PC들은 Secure Boot이 켜져있는것이 기본값이 되고 있다.

하지만 아치 리눅스 설치 미디어는 Secure Boot으로 부팅하는것을 지원하지 않기 때문에 자신의 컴퓨터가 Secure Boot이 켜져있다면 Bios 설정 창에 들어가 꺼주어야 한다.

일단 설치 과정에서 임시로 Secure Boot을 끄는 것으로 다시 Secure Boot을 아치 리눅스와 함께 설정하는 것은 밑에서 다시 언급할 것이다.

## 설치

이제 부팅을 할 차례이다!! USB 설치 미디어를 삽입하고 부팅을 해보도록 하자.

부팅이 완료되면 검은 화면의 터미널창이 반겨줄 것이다. 역시 미니멀리즘 배포판에게 GUI는 사치이다. 이제 차근차근 설치 단계를 밟아보도록 하자. 

> 아치위키에서는 부팅후 키보드 레이아웃을 설정하라고 하지만 기본 설정인 US 키맵에서 한글키보드는 크게 문제가 없으니 생략했다.
{: .prompt-info }

### 네트워크 연결

처음 연결하면 네트워크가 연결 되어있는지 확인해야 한다. 아래 명령으로 네트워크 인터페이스가 제대로 인식되는지 확인할 수 있다.

```shell
ip link
```

아마 유선네트워크라면 자동으로 연결될 것이다. 아래 명령어로 제대로 네트워크에 접속되는지 확인할 수 있다.

```shell
ping archlinux.org
```

만약 와이파이를 사용해야하는 노트북등의 사용자라면 아래 설명을 따라 해주면 된다.

### Wi-Fi 연결 (optional)

와이파이 연결은 [iwctl](https://wiki.archlinux.org/title/Iwd#iwctl) 명령을 사용해 할 수 있다. 아래 과정대로 해주면 된다.

```shell
# iwctl 프롬프트 창으로 바뀔것이다.
iwctl

# 명령어들을 확인할 수 있다.
help

# Wi-Fi 디바이스들을 확인 할 수 있다.
device list

# 주변 네트워크를 스캔한다.
station <device name> scan

# 주변 네트워크 리스트를 출력한다.
station <device name> get-networks

# Wi-Fi 네트워크에 연결한다.
station <device name> connect <SSID>

# iwctl 종료
exit
```

> 위의 명령에서 <> 부분에는 자신에게 맞게 입력하면 된다.
{: .prompt-info }

### System clock 업데이트

인터넷에 연결된것을 확인했으면 시스템 시간을 정확하게 업데이트 해줄 수 있다.

```shell
timedatectl set-ntp true
```

설정 상태 확인은 아래 명령어로 할 수 있다.

```shell
timedatectl status
```

리눅스 시스템 시간에 관련된 좀더 자세한 정보는 [system time](https://wiki.archlinux.org/title/System_time) 문서에서 확인할 수 있다.

### System encryption 설정 (optional)

시스템 전체를 암호화 하여 아치 리눅스를 설치하고자 하면 현재 단계에서 설정하면 된다.

암호화에 대한 설명은 아치위키 [dm-crypt](https://wiki.archlinux.org/title/Dm-crypt) 문서를 참고하면 된다.

### Partition 설정

이제 파티션을 설정해 줘야 한다.

이전에 [EFI 파티션 설정](https://chromato99.com/posts/%EB%A6%AC%EB%88%85%EC%8A%A4-%EB%93%80%EC%96%BC%EB%B6%80%ED%8C%85%EC%9D%84-%EC%9C%84%ED%95%B4-%EC%9C%88%EB%8F%84%EC%9A%B0-EFI-%EC%8B%9C%EC%8A%A4%ED%85%9C-%ED%8C%8C%ED%8B%B0%EC%85%98(ESP)-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0/)에 대한 포스트를 쓴적이 있는데 이와 비슷하다.

위에 링크에 들어가면 cfdisk로 파티션 설정하는 방법에 대한 설명을 할 수 있다.

이번에는 fdisk명령으로 설정해 보려 한다.

[아치위키](https://wiki.archlinux.org/title/Installation_guide#Example_layouts)의 설명에 따르면 아래 표처럼 파티션을 설정하는것을 제안하고 있다.

| Mount point          | Partiton                  | Partition Type        | Suggested size          |
|:---------------------|:--------------------------|:----------------------|------------------------:|
| /mnt/boot            | /dev/efi_system_partition | EFI system partition  | At least 300 MiB        |
| [SWAP]               | /dev/swap_partition       | Linux swap            | More than 512 MiB       |
| /mnt                 | /dev/root_partition       | Linux x86-64 root (/) | Remainder of the device |

아래 명령을 통해 [fdisk](https://wiki.archlinux.org/title/Fdisk) 명령으로 파티션을 설정 할 수 있다.

```shell
# 현재 인식되는 저장장치 및 파티션 표시
fdisk -l

# fdisk 명령 모드로 진입
fdisk /dev/the_dist_to_be_partitioned

# 명령어 확인
m
```

```
# 아래 과정으로 파티션 설정과정 수행

전체 디스크 포맷을 위해 g 입력 (전체 데이터를 지우고 싶지 않으면 생략 ex: 듀얼부팅)
또는 d를 입력해 불필요한 파티션 삭제
첫 파티션 생성을 위해 n 입력
첫번째 파티션이므로 엔터
용량 입력으로 EFI 파티션은 보통 512MB로 설정하므로 +512M 입력
파일 시스템 변경을 위해 t입력
첫번째 파티션을 변경하는 것이므로 1입력
L 로 파티션 타입 번호를 확인
EFI 시스템 파티션이므로 1입력

위와 비슷한 방법으로 swap 파티션 설정
나머지는 그래로 진행하되 용량은 원하는 용량 입력
파티션 타입은 linux swap 파티션이므로 19 입력
(swap 파티션 용량은 과거에는 RAM 용량의 두배를 설정하는게 보통이었지만 현재는 원하는 값으로 해도 무방 심지어 설정 안해도 된다.)

마지막으로 위의 방법으로 root 파티션 생성
남은 용량 전체를 설정하고 싶으면 용량을 입력하지 않고 엔터
w로 파티션 설정 저장
q로 fdisk명령 종료
```

### Partition 포맷

이제 파티션 설정을 모두 끝냈으니 파티션 들을 포맷해줘야 한다.
리눅스에서는 [mkfs](https://wiki.archlinux.org/title/File_systems#Create_a_file_system) 명령으로 파티션을 원하는 타입으로 포맷할 수 있다.

우선 아래 명령으로 EFI 시스템 파티션을 포맷할 수 있다.

```shell
mkfs.fat -F 32 /dev/efi_system_partition
```

다음으로 아래 명령으로 linux swap 파티션을 포맷 할 수 있다.

```shell
mkswap /dev/swap_partition
```

마지막으로 아래 명령으로 ext4 형식으로 루트 파티션을 포맷 할 수 있다.

```shell
mkfs.ext4 /dev/root_partition -L arch_os 
```

> -L은 파티션 라벨을 추가하는 옵션이다.
{: .prompt-info }

### Mount 파일 시스템

이제 파티션 설정이 끝났다!!

파티션을 마운트하여 접근 할 수 있도록 해야한다.

우선 루트 파티션을 아래 명령으로 마운트 한다.

```shell
mount /dev/root_partition /mnt
```

다음으로 EFI 시스템 파티션을 루트 파티션 아래에 마운트 해준다.

```shell
cd /mnt
mkdir boot
mount /dev/efi_system_partition /mnt/boot
```

참고로 EFI 시스템 파티션을 마운트하는 위치는 몇가지 다른 옵션이 있는데 관심 있다면 관련 [문서](https://wiki.archlinux.org/title/EFI_system_partition#Mount_the_partition)를 참고하면 된다.

마지막으로 swap 파티션을 마운트 해준다.

```shell
swapon /dev/swap_partition
```

### Mirrorlist 설정

아치 리눅스를 패키지를 여러 [미러서버](https://wiki.archlinux.org/title/Mirrors)들을 통해 다운받을 수 있다. 설치용 미디어 환경에서는 인터넷 연결이 확이되는 순간 20여개의 미러서버들을 속도순으로 정렬해준다.

하지만 우리나라에서 사용하기에는 모두 해외에 서버가있어 속도가 느리므로 수동으로 설정하면 조금더 빠르게 패키지들을 설치할 수 있다. 따라서 아래 설명을 따라 미러서버를 설정해주는 것을 추천한다.

우선 [Pacman Mirrorlist Generator](https://archlinux.org/mirrorlist/)에 들어가면 한국 서버들의 리스트를 확인할 수 있다.

미러서버 설정은 매우 간단한데 위의 링크에서 찾은 서버 목록을 `/etc/pacman.d/mirrorlist` 파일에 입력해주면 된다.

> 서버 리스트에는 Server 앞에 # 주석이 적혀있는데 주석은 지워줘야 한다!
{: .prompt-info }

### Base 패키지 및 kernel 설치

드디어 가장 메인인 아치 리눅스 커널과 기본 패키지를 설치할 차례이다!!

[pacstrap](https://man.archlinux.org/man/pacstrap.8) 명령어를 통해 매우 기본적인 아치 리눅스 커널과 기본 패키지를 설치 할 수 있다.

```shell
pacstrap /mnt base linux linux-firmware
```

다만 위의 명령어는 미니멀리즘한 배포판 답게 매우 기본적인 기능밖에 없다. 따라서 [아치위키](https://wiki.archlinux.org/title/Installation_guide#Install_essential_packages) 설명을 참고하여 글쓴이는 좀더 추가적인 패키지들도 같이 설치하고자 한다.

또한 리눅스 커널은 기본 커널이 아닌 linux-zen 커널을 설치할 것이다. 이는 조금 개조된 리눅스 커널로 이전에 valve의 게임 최적화 패치를 빠르게 업데이트 하기도 하였고 [waydroid](https://wiki.archlinux.org/title/Waydroid)와 같은 프로그램을 쓰기위한 기능들도 일부 포함되어 있기 때문이다.

커널 종류들은 [kernel](https://wiki.archlinux.org/title/Kernel) 문서를 참고하면 좋다.

따라서 글쓴이는 아래와 같이 커널과 기본 패키지들을 설치하였다.

```shell
pacstrap /mnt base linux-zen linux-zen-headers linux-firmware base-devel vim networkmanager man-db man-pages texinfo dosfstools mdadm lvm2
```

이 이상의 패키지는 chroot 이후에 설치할 예정이다.

### Fstab

다음으로 fstab 파일을 생성해야 한다. [fstab](https://wiki.archlinux.org/title/Fstab) 이란 파일 시스템 테이블로 리눅스의 파일 시스템에 대한 정보를 저장하는 파일이다.

아래 명령으로 현재 마운트된 파일 시스템을 기반으로 테이블을 생성할 수 있다.

```shell
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot

이제 설치한 시스템으로 진입할 수 있다!!
이래 명령어를 입력한다.

```shell
arch-chroot /mnt
```

### Time zone

새로 설치한 시스템을 위한 설정을 몇가지 해줘야 한다. 

우선 아래 명령으로 시간대 설정을 할 수 있다.

```shell
ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime
```

다음으로 아래 명령으로 하드웨어 시간도 동기화 해줄 수 있다.

```shell
hwclock --systohc
```

좀더 추가적인 정보는 [timezone](https://wiki.archlinux.org/title/System_time#Time_zone) 문서와 [이전 포스트](https://chromato99.com/posts/%EB%93%80%EC%96%BC%EB%B6%80%ED%8C%85-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-%EC%8B%9C%EA%B0%84%EC%9D%B4-%EC%9D%B4%EC%83%81%ED%95%98%EA%B2%8C-%EB%82%98%EC%98%A4%EB%8A%94-%EB%AC%B8%EC%A0%9C/) 참고하자.

### Localizaion

다음으로 언어 설정을 해주어야 하는데 `/etc/locale.gen` 파일에서 사용할 언어들의 주석을 제거해주면 된다.

글쓴이의 경우 `en_US.UTF-8 UTF-8`와 `ko_KR.UTF-8 UTF-8` 두개의 주석을 제거했다.

파일 편집이 끝나면 다음 명령을 입력한다.

```shell
locale-gen
```

마지막으로 `/etc/locale.conf` 파일을 아래처럼 작성한다.

```
LANG=en_US.UTF-8
```

> 글쓴이의 경우 리눅스 시스템 기본 언어를 한국어로 설정하면 충돌 등의 문제가 우려되 영어로 설정하였다.
{: .prompt-info }

### Network 설정

네트워크 설정 또한 해주어야 하는데 `/etc/hostname` 파일을 생성해 원하는 호스트네임을 적어주면 된다.

```
myhostname
```

또한 위의 pacstrap 명령으로 설치한 [NetworkManager](https://wiki.archlinux.org/title/NetworkManager)가 자동 실행되도록 아래 명령을 입력한다.

```shell
systemctl enable NetworkManager.service
```

### 비밀번호 설정

아래 명령으로 root 계정의 비밀번호를 설정할 수 있다.

```shell
passwd
```

### 사용자 계정 추가

아치위키의 [관련 문서](https://wiki.archlinux.org/title/General_recommendations#Users_and_groups)에 따르면 root 계정만을 사용하는 것은 보안상 좋지 않다고 하므로 사용자 계정을 추가해주어야 한다.

```shell
useradd -m -g users -G wheel -s /bin/bash username
```

> wheel 그룹에 사용자를 추가하는 이유는 sudo 명령을 사용가능하게 하기 위함이다.
{: .prompt-info }

사용자 계정 비밀번호는 아래 명령으로 설정한다.

```shell
passwd user
```

### Sudo 설정

기본 설정으로는 [sudo](https://wiki.archlinux.org/title/Sudo) 명령을 사용할 수 없으므로 sudo 사용을 설정해주어야 한다.

아래 명령을 입력하면 vi 편집기 모드로 sudo 설정 파일을 수정할 수 있다.

```shell
visudo
```

sudo 설정파일에서 아래로 내리다 보면 `## Uncomment to allow members of group wheel to execute any command`라고 적힌 부분이 있는데 해당부분 아래의 주석을 아래처럼 제거해주면 된다.

```
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL
```

### Microcode (optional)

CPU 제조사들은 CPU의 보안 이슈가 발생하면 이에대한 소프트웨어 패치를 [microcode](https://wiki.archlinux.org/title/Microcode)라는 이름으로 배포한다.

이를 설치하지 않아도 무방하지만 보안을 조금더 신경쓰고 싶다면 아래 명령으로 설치하도록 하자.

```shell
pacman -S amd-ucode # AMD CPU인 경우
pacman -S intel-ucode # Intel CPU인 경우
```

이러한 패키지의 로딩은 부트로더에서 설정하므로 아래 부트로더 항목에서 설명했다.

### Boot loader 설정

아직 부팅을 위한 부트로더를 설정하지 않았다. systemd에 기본으로 포함되어 있는 [systemd-boot](https://wiki.archlinux.org/title/Systemd-boot)로 부트로더를 설정할 수 있다.

아래 명령으로 EFI 시스템 파티션에 systemd-boot를 설치할 수 있다.

```shell
bootctl install
```

다음으로 `/boot/loader/loader.conf`를 설정해 주어야 한다.

글쓴이의 경우 아치위키에 나와있는대로 아래와 같이 설정하였다.

```
default  arch.conf
timeout  4
console-mode max
editor   no
```

다음으로 부팅시의 진입점을 설정해 주어야 한다. `/boot/loader/entries/arch.conf` 파일을 생성해 아래와 같이 입력해 준다.

> 위의 microcode를 설치했다면 ucode 부분을 꼭 추가하도록 한다.
{: .prompt-info }


```
title   Arch Linux
linux   /vmlinuz-linux-zen
initrd  /amd-ucode.img # 위의 microcode를 설치했을 때로 intel인 경우 amd를 intel로
initrd  /initramfs-linux-zen.img
options root="LABEL=arch_os" rw
```

또한 추가적으로 `/boot/loader/entries/arch-fallback.conf` 파일을 생성해 아래와 같이 입력해 준다.

```
title   Arch Linux (fallback initramfs)
linux   /vmlinuz-linux-zen
initrd  /amd-ucode.img # 위의 microcode를 설치했을 때만 intel인 경우 amd를 intel로
initrd  /initramfs-linux-zen-fallback.img
options root="LABEL=arch_os" rw
```

> 만약 듀얼부팅 환경이라면 윈도우 부팅 매니저는 수동으로 추가하지 않아도 자동으로 추가해 준다!
{: .prompt-info }

### Initramfs (optional)

보통의 경우에는 `mkinitcpio`명령이 커널을 설치할 때 자동으로 실행되기 때문에 필요없지만 LVM, system encryption, RAID 등의 설정을 할때는 `mkinitcpio.conf`파일을 수정하기 때문에 새로 initramfs를 생성해줘야 한다.

initramfs 생성은 아래 명령을 실행하면 된다.

```shell
mkinitcpio -P
```

### 마운트 해제 및 재부팅

이제야 기본적인(?) 아치 리눅스 운영체제 설치가 끝났다.

아래 명령으로 시스템 환경에서 나갈수 있다.

```shell
exit
```

다음으로 마운트 했던 저장장치들을 모두 해제한다.

```shell
umount -lR /mnt
```

이제 설치 미디어가 아닌 설치된 아치 리눅스로 재부팅 할 수 있다!

```shell
reboot
```

이제 아치 리눅스 설치용 USB 드라이브는 분리해도 된다.

아치 리눅스에 오신걸 환영합니다!!!

## 추가 권장 패키지 설치

열심히 설치한 아치 리눅스로 부팅하였지만 아직 GUI도 없고 인터넷 브라우저와 같은 필수적인 응용프로그램이 설치되어있지 않다.

따라서 아주 기본적인 몇가지만 설치하는 방법을 설명하고자 한다.
우선 여러 패키지들을 다운받으려면 인터넷이 필요하니 인터넷 연결을 확인 한다.

```shell
ping -c 3 archlinux.org
```

### Wi-Fi 연결 (optional)

유선 네트워크를 사용할 수 없는 환경이라면 Wi-Fi에 연결해야 한다. 위의 설치에서 [Network Manager](https://wiki.archlinux.org/title/NetworkManager)를 설치했으니 아래 명령어로 Wi-Fi에 연결 할 수 있다.

```shell
nmtui
```

명령어를 입력하면 터미널 기반의 UI로 편하게 Wi-Fi에 연결 할 수 있다. 방향키로 커서를 이동할 수 있고 `Activate connection`을 선택하면 주변 Wi-Fi를 확인할 수 있고 `Activate`로 연결할 수 있다.

### 패키지 업데이트

우선 다른 패키지를 다운받기 전에 설치된 패키지들을 모두 없데이트 해준다. 아치 리눅스 기본 패키지 매니저인 [pacman](https://wiki.archlinux.org/title/Pacman)에서는 아래 명령어로 업데이트를 할 수 있다.

```shell
sudo pacman -Syu
```

### 그래픽 드라이버 설치

현재의 CLI 환경에서도 많은 작업을 할 수는 있겠지만 아무래도 그래픽 환경이 있는게 편하다. 이런 그래픽 환경을 사용하기 위해서는 그래픽 카드를 잘 작동 시킬수 있도록 그래픽 드라이버를 먼저 설치해 주어야 한다.

그래픽 드라이버에 대한 설명은 [Intel Graphics](https://wiki.archlinux.org/title/Intel_graphics), [AMDGPU](https://wiki.archlinux.org/title/AMDGPU), [Nvidia](https://wiki.archlinux.org/title/NVIDIA)에서 확인할 수 있다.

설치 명령을 정리하면 아래와 같다.

```shell
# Intel
sudo pacman -S mesa xf86-video-intel vulkan-intel

# AMD
sudo pacman -S mesa xf86-video-amdgpu vulkan-radeon

# Nvidia
sudo pacman -S nvidia

# Nvidia with custom kernel
sudo pacman -S nvidia-dkms
```

추가로 커널 모드 설정을 해주어야 하는데 `/etc/mkinitcpio.conf`파일을 수정해주어야 한다. 자세한 설명은 [kernel mode setting](https://wiki.archlinux.org/title/Kernel_mode_setting) 문서를 확인하면 된다.

간단하게 정리하면 아래와 같다.

```
# Intel
MODULES=(... i915 ...)

# AMD
MODULES=(... amdgpu ...)

# Nvidia
MODULES=(... nvidia nvidia_modeset nvidia_uvm nvidia_drm ...)
```

파일 수정이 끝난 뒤에는 initramfs를 아래 명령어로 다시 생성해 주어야 한다.

```shell
mkinitcpio -P
```

Nvidia의 경우 커널 파라미터도 설정해 주어야 하는데 `/boot/loader/entries/arch.conf`파일에서 `options`를 아래와 같이 파라미터를 추가해준다.

```
options root="LABEL=arch_os" rw nvidia-drm.modeset=1
```

또한 Nvidia의 경우 initramfs도 업데이트 해줘야 하는데 [pacman hook](https://wiki.archlinux.org/title/Pacman#Hooks)을 사용하면 이를 자동화할 수 있다.

이를 위해서는 `/etc/pacman.d/hooks/nvidia.hook`경로에 파일을 만들고 아래 내용을 저장해주면 된다.

```
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia-dkms
Target=linux-zen
# Change the linux part above and in the Exec line if a different kernel is used

[Action]
Description=Update Nvidia module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case $trg in linux-zen) exit 0; esac; done; /usr/bin/mkinitcpio -P'
```

> 위 내용에서 linux-zen과 nvidia-dkms는 자신이 설치한 패키지 이름으로 설정해 줘야 한다.
{: .prompt-info }

### 데스크탑 환경 설치

[데스크탑 환경(Desktop Environment)](https://wiki.archlinux.org/title/Desktop_environment)란 리눅스에서 GUI 환경 패키지 세트쯤 된다고 생각하면 된다.

아치위키 문서에서도 확인할 수 있듯이 리눅스에는 매우 다양한 데스크탑 환경이 있는데 유명한것으로는 [GNOME](https://wiki.archlinux.org/title/GNOME), [KDE](https://wiki.archlinux.org/title/KDE), [Xfce](https://wiki.archlinux.org/title/Xfce)가 있다.

여기서는 글쓴이가 개인적으로 좋아하는 한국에서는 그놈이라 쓰는 GNOME을 기준으로 설명한다. 우선 gnome 패키지를 설치해 준다.

```shell
sudo pacman -S gnome
```

위 명령어를 입력하면 다양한 GNOEM 소프트웨어들을 선택하라 하는데 그냥 엔터를 누르면 기본값으로 설치된다.

설치를 완료하였다면 재부팅시 자동으로 실행되게 해야하는데 [systemctl](https://wiki.archlinux.org/title/Systemd#Basic_systemctl_usage) 명령을 사용한다.

```shell
sudo systemctl enable gdm
```

### 한글 설정

아치 리눅스는 기본적으로 서방에서 많이 사용하다보니 한글 폰트나 한글 입력기가 기본적으로 설치되어 있지 않다. 따라서 몇가지 설정을 해주어야 하는데 자세한건 [Localization/Korean](https://wiki.archlinux.org/title/Localization/Korean) 문서를 확인하면 된다.

아래 설명은 글쓴이의 취향에 기반해 있다.

```shell
# 폰트 설치
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra

# 한글 입력기 설치
sudo pacman -S ibus-hangul
```

입력기를 설치 후에는 관련 설정또한 해주어야 하는데 `/etc/environment`파일을 아래와 같이 입력해주면 된다.

```
GTK_IM_MODULE=ibus
QT_IM_MODULE=ibus
XMODIFIERS=@im=ibus
```

### AUR helper 설치

아치 리눅스에는 [AUR(Arch User Repository)](https://wiki.archlinux.org/title/Arch_User_Repository)라는 아치 유저들이 관리하고 있는 유저 레포지토리가 있다. 어디까지나 '유저' 레포지토리인 만큼 악성코드가 있다거나 문제가 있는 패키지가 있던 경우도 있으나 유명하고 활성화된 패키지들 또한 많다.

그리고 악성코드의 경우 'PKGBUILD'를 꼼꼼히 확인하는 등의 약간의 수고를 들이면 충분히 안전하게 사용할 수 있다. 다만 이러한 AUR을 그냥 사용하기엔 약간의 불편함이 있어 이를 도와주는 [AUR helper](https://wiki.archlinux.org/title/AUR_helpers)를 설치하는 방법에 대해 설명하고자 한다.

아래 방법은 AUR 패키지의 메뉴얼 한 설치 방법이기도 하므로 수동 설치가 관심 있으면 참고해도 좋다. 여기서는 가장 유명한 AUR helper인 [yay](https://github.com/Jguer/yay)를 설치한다.

yay는 AUR에 업로드 되어 있으므로 우선 이 패키지 파일을 다운받아야 하므로 [git](https://wiki.archlinux.org/title/Git)을 설치하고 다운받아 준다.

```shell
# git 설치
sudo pacman -S git

# git으로 yay 가져오기
git clone https://aur.archlinux.org/yay.git
```

이제 다운이 다 됬으므로 아래 명령어들을 입력해 설치해준다.

```shell
# yay 디렉토리 진입
cd yay

# 패키지 설치
makepkg -risc
```

### Secure boot 설정 (optional)

위의 부팅전에 secure boot 설정을 껐지만 앞으로 다시 켜서 사용하고 싶을 경우 secure boot 설정을 해주어야 한다.

[Secure boot](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot) 설명 문서에 여러 방법들이 나와있는데 이중 구글에 검색해봤을때 가장 유명했던 [shim](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#shim)을 기준으로 설명하고자 한다.

shim은 AUR에 업로드되어 있으므로 아래 명령으로 설치해준다. 추가로 [efibootmgr](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface#efibootmgr) 또한 필요하므로 설치해준다.

```shell
yay shim-signed
yay -S efibootmgr
```

yay등의 AUR helper로 설치한 뒤에도 몇가지 파일들을 복사 해주어야 하는데 아래 명령어들을 입력해주면 된다.

```shell
# 우선 BOOTx64.EFI를 grubx64.efi로 이름을 바꿔준다.
mv /boot/EFI/BOOT/BOOTx64.EFI /boot/EFI/BOOT/grubx64.efi

# 두개의 파일을 복사해 준다.
cp /usr/share/shim-signed/shimx64.efi /boot/EFI/BOOT/BOOTx64.EFI
cp /usr/share/shim-signed/mmx64.efi /boot/EFI/BOOT/

# shim으로 부팅될수 있도록 부팅 진입점을 새로 efibootmgr 명령을 사용해 생성해준다.
efibootmgr --verbose --disk /dev/sdX --part Y --create --label "Shim" --loader /EFI/BOOT/BOOTx64.EFI
```

> efibootmgr 명령에서 `--disk /dev/sdX` 부분은 자신의 저장장치 경로에 맞게 설정해주고(파티션 번호 제외) `--part Y` 부분에서 Y대신 파티션 번호를 설정해준다.
{: .prompt-info }

이제 모든 설치가 끝났으니 재부팅해주면 된다!

다만 shim은 부트 이미지들에 해쉬값을 생성해주어야 하는데 이는 재부팅 과정에서 진행할 수 있다. 

재부팅을 하게되면 `MokManager`라는 것이 실행되게 되는데 여기서 `Enroll hash`를 선택해준뒤 `/boot/EFI/BOOT/grubx64.efi`파일을 선택해 해쉬를 생성해준다. 또한 `vmlinuz-linux-zen` 파일을 찾아 한번더 해쉬를 생성해주면 된다.

> 이러한 해쉬 생성은 shim이나 커널이 업데이트 되어 파일이 변경되었을때 다시 해주면 된다.
{: .prompt-info }

이것으로 Secure Boot까지 기본적인 설정들은 모두 마치게 된다!!

## 추가로 참고할 자료

이 이상의 설정은 이미 위에서 설정한것도 몇가지 있지만 아치위키의 [General recommendations](https://wiki.archlinux.org/title/General_recommendations) 문서를 참고하면 좋다. 또한 아치 리눅스에서 사용할만한 응용프로그램에 대해서는 [List of applications](https://wiki.archlinux.org/title/List_of_applications) 문서도 참고하면 좋다.

아치 리눅스에대한 많은 정보는 [아치위키](https://wiki.archlinux.org)에 정말 잘 정리되어 있으니 뭔가 문제가 생기거나 하면 꼭 확인해 보는것이 좋다.
