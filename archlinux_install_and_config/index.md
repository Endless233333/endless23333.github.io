# 我的 Arch Linux 安装和基础配置过程


本文记录我的 Arch Linux 安装和基础配置的过程, 桌面为 xfce4, boot loader 为 systemd-boot, 文件系统为 ext4, intel cpu + 核显。

## 安装部分
### 联网
有线网络能自动联网, 无线网络使用 `iwctl` 进行连接:
```sh
iwctl
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect wifi-name
exit
```

可以使用 `ping` 检查网络连接。

联网之后可以通过 `passwd` 设置安装环境中 root 的密码, 然后通过其他机器 ssh 去连, 方便复制粘贴。

### 更新系统时间
使用 `timedatectl` 确保系统时间是准确的。

### 硬盘分区与格式
使用 `fdisk -l` 列出当前分区情况。

分区步骤:
```sh
fdisk /dev/nvme0n1
n
(enter)
(enter)
+512M
n
(enter)
+16G
n
(enter)
(enter)
t
1
1
t
2
19
w
```

格式化:
```sh
mkfs.ext4 /dev/nvme0n1p3
mkswap /dev/nvme0n1p2
mkfs.fat -F 32 /dev/nvme0n1p1
```

### 挂载
按如下顺序挂载:
```sh
mount /dev/nvme0n1p3 /mnt
mount --mkdir /dev/nvme0n1p1 /mnt/boot
swapon /dev/nvme0n1p2
```

### 安装系统和必要的功能性软件
```
pacstrap -K /mnt base linux linux-firmware base-devel networkmanager vim vi sudo openssh zsh intel-ucode
```

### 生成 fstab 文件
```sh
genfstab -U /mnt > /mnt/etc/fstab
```

### change root
使用 `arch-chroot /mnt` 把系统环境切换到新系统下。

### 设置时区
```sh
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### 设置主机名
在 `/etc/hostname` 中写入主机名, 例如 myarch.

在 `vim /etc/hosts` 中写入以下内容:
```
127.0.0.1   localhost
::1         localhost
127.0.1.1   myarch.localdomain myarch
```

### 硬件时间设置
```sh
hwclock --systohc
```

### 设置 Locale
编辑 `/etc/locale.gen`, 去掉 `en_US.UTF-8 UTF-8` 和 `zh_CN.UTF-8 UTF-8` 前的井号。

然后使用 `locale-gen` 生成 `locale`.

`echo 'LANG=en_US.UTF-8'  > /etc/locale.conf` 设定 LANG 变量。

### 设置 root 用户密码
`passwd` 然后输两遍密码。

### 安装引导程序
安装 `systemd-boot`:
```sh
bootctl --path=/boot install
```

编辑 `/boot/loader/loader.conf` 配置启动选单:
```
default	arch.conf
timeout 0
console-mode keep
editor	no
```

使用 `cat /etc/fstab` 查看根分区的 UUID.

编辑 `/boot/loader/entries/arch.conf` 增加启动选项:
```
title	Arch Linux
linux	/vmlinuz-linux
initrd  /intel-ucode.img
initrd	/initramfs-linux.img
options	root=UUID=... rw ibt=off nowatchdog
```

### 准备非 root 用户
创建用户:
```sh
useradd -m -G wheel -s /bin/bash XXX
```

设置密码:
```sh
passwd XXX
```

编辑 `sudoers`:
```
EDITOR=vim visudo
```

去掉 `#%wheel ALL=(ALL:ALL) ALL` 前的井号。

### 重启
```sh
exit
umount -R /mnt
reboot
```

### 联网
```sh
systemctl enable --now NetworkManager
nmcli device wifi connect 名 password 密
```

联网后可以通过 `systemctl start sshd` 开启 ssh 服务。

### 开启 32 位支持库
编辑 `/etc/pacman.conf` 去掉 `[multilib]` 及其下一行前的井号。

`pacman -Syyu` 更新一下。

### 一大波安装
```sh
pacman -S \
zsh-autosuggestions zsh-completions zsh-history-substring-search zsh-syntax-highlighting zsh-theme-powerlevel10k \
xfce4 xfce4-goodies lightdm lightdm-gtk-greeter gvfs gvfs-mtp udiskie sof-firmware alsa-firmware alsa-ucm-conf pavucontrol pulseaudio pulseaudio-alsa pulseaudio-bluetooth network-manager-applet nm-connection-editor \
adobe-source-han-serif-cn-fonts wqy-zenhei noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra \
mesa lib32-mesa vulkan-intel lib32-vulkan-intel \
blueman bluez bluez-utils \
fcitx5-im fcitx5-chinese-addons fcitx5-material-color fcitx5-pinyin-zhwiki fcitx5-configtool \
firefox chromium flameshot okular steam libreoffice-still libreoffice-still-zh-cn man-db man-pages syncthing fd fzf git
```

### yay 安装
使用非 root 用户:
```sh
git clone https://aur.archlinux.org/yay-bin.git
pacman -R yay-bin
cd yay-bin
makepkg -si
```

常用软件:
```sh
yay -S yay-bin clash-verge-rev-bin ttf-intel-one-mono visual-studio-code-bin
```

### 启动一些服务
```sh
systemctl enable lightdm.service bluetooth.service systemd-boot-update.service
```

## 配置部分

### fcitx5
编辑 `/etc/environment` 加入以下内容:
```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

fcitx5 设置:
- Input Method 加上 pinyin
- Addons -> Classic User Interface -> Theme -> Material-Color-Red

### firefox
地址栏输入 `about:config`, 然后将 `browser.newtabpage.activity-stream.improvesearch.handoffToAwesomebar` 改成 false.

### libreoffice
tools -> options -> language settings -> user interface 选中文

### xfce4
中文化, 在 ~/.xinitrc 或 ~/.xprofile 里加上：
```
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:en_US
```

xfce4 设置:
Appearance
- style: Adwaita-dark
- icons: elementary
- fonts:
  - default font: IntelOne Mono Regular
  - dpi: 120

Desktop
桌面
icon: default icon 去掉前三个

Panel
- panel1-items
  - whisker menu
    - commands 勾上 restart, shut down, suspend
  - window buttons
  - separator
  - network monitor
    - update interval: 0.5
    - show values as bits
    - present data as: Bars and values
  - system load monitor
    - update interval: 1000
    - 只留下 cpu 和 mem
  - mount devices
  - separator
  - stautus tray plugin
  - pulseaudio plugin
  - notification plugin
  - separator
  - clock
    - clock option:
    - layout: date only
    - font: 12
    - format: custom format %m-%d
  - clock
    - clock option:
    - layout: time only
    - font: 16
    - format: 14:38
- panel2-items
  - 前两个 launcher 上移一个，中间加各种 launcher

Window Manager
- keyboard
  - Window operations menu 改成 super+space
  - 改下 tile window to the ...

Xfce Screensaver
- 时间改成 60

Terminal
- Appearance
  - 字体: IntelOne Mono Regular 14
  - background: Transparent Background 0.9
- Colors
  - Background Color: 黑色上面那个

Keyboard
- Application Shortcuts
  - xflock4 super+l
  - flameshot gui shift+super+s
  - xfce4-appfinder 改成 alt+space

Power Manager
- General
  - when power button is pressed: Ask
- Display
  - 连接电源时拉满

Session and Startup
- Application Autostart
  - 打开 Clipman
  - 添加 thunar 命令是 thunar --daemon

