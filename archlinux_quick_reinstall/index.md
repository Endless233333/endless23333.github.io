# 我的 archlinux 快速重装与配置


我习惯于隔断时间整理文件并重装电脑, 以下是我重装的过程, 用以之后方便复制粘贴, 主要参考了 [archlinux wiki](https://wiki.archlinux.org/title/Installation_guide) 和 [archlinux 简明指南](https://arch.icekylin.online/)。

## 安装部分
### 无线网络连接
```
iwctl
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect wifi-name
exit
```

### 更新系统时间
`timedatectl`

### 分区
```
mkfs.ext4 /dev/nvme1n1p3
mkswap /dev/nvme1n1p2
mkfs.fat -F 32 /dev/nvme1n1p1
mount /dev/nvme1n1p3 /mnt
mount --mkdir /dev/nvme1n1p1 /mnt/boot
swapon /dev/nvme1n1p2
```

### 安装
`pacstrap -K /mnt base linux linux-firmware base-devel networkmanager vim vi sudo zsh zsh-completions openssh intel-ucode`

### 基础配置
生成 fstab: `genfstab -U /mnt > /mnt/etc/fstab`

chroot: `arch-chroot /mnt`

写入 hostname 例如 XXX: `vim /etc/hostname`

编辑 hosts: `vim /etc/hosts`

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   XXX.localdomain XXX
```

时区: `ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`

生成 adjtime: `hwclock --systohc`

改密码: `passwd`

### 语言
`vim /etc/locale.gen`

去掉下面两个前面的井号:
```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
```

`locale-gen`

`echo 'LANG=en_US.UTF-8'  > /etc/locale.conf`

### 引导程序
`bootctl --path=/boot install`

`vim /boot/loader/loader.conf`

```
default	arch.conf
timeout 0
console-mode keep
editor	no
```

`cat /etc/fstab`

`vim /boot/loader/entries/arch.conf`

```
title	Arch Linux
linux	/vmlinuz-linux
initrd	/initramfs-linux.img
options	root=UUID=... rw ibt=off nowatchdog
```

### 基础安装结束
`exit`

`umount -R /mnt`

`reboot`

### 后续安装
`useradd -m -G wheel -s /bin/bash endless`

`passwd endless`

`EDITOR=vim visudo`

去掉 `#%wheel ALL=(ALL:ALL) ALL` 前的井号

`vim /etc/pacman.conf`
去掉 `[multilib]` 对应两行前的井号

`systemctl enable --now NetworkManager.service`

`pacman -Syyu`

```sh
pacman -S xfce4 xfce4-goodies lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings \
sof-firmware alsa-firmware alsa-ucm-conf pavucontrol pulseaudio pulseaudio-alsa pulseaudio-bluetooth \
adobe-source-han-serif-cn-fonts wqy-zenhei noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra \
network-manager-applet nm-connection-editor \
mesa lib32-mesa vulkan-intel lib32-vulkan-intel \
blueman bluez bluez-utils \
gvfs gvfs-mtp udiskie \
fcitx5-im fcitx5-chinese-addons fcitx5-material-color fcitx5-pinyin-zhwiki fcitx5-configtool \
firefox flameshot syncthing \
fd fzf mtr p7zip ntfs-3g scrcpy tmux bat bottom neofetch ripgrep tldr wget tree zip \
git
```

```sh
pacman -S gimp libreoffice-still chromium man-db man-pages obs-studio okular \
docker docker-compose jdk-openjdk jdk17-openjdk jdk11-openjdk jdk8-openjdk hugo wireshark-qt php linux-headers
```

```
systemctl enable lightdm.service
systemctl enable bluetooth.service
systemctl enable systemd-boot-update.service
```

```
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
```

`yay -S yay-bin ttf-intel-one-mono drawio-desktop-bin visual-studio-code-bin vmware-workstation mycli goldendict-ng`

## 配置部分
### docker
`sudo usermod -aG docker endless`

### wireshark
`sudo usermod -a -G wireshark $USER`

### VMware
```
sudo systemctl start vmware-networks-configuration.service
sudo systemctl enable vmware-networks.service vmware-usbarbitrator.service
modprobe -a vmw_vmci vmmon
```

MC60H-DWHD5-H80U9-6V85M-8280D

### firefox
```
about:config
browser.newtabpage.activity-stream.improvesearch.handoffToAwesomebar
```

### fcitx5
`vim /etc/environment`

```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

Input Method 加上 pinyin
Addons 的 Classic User Interface 中的 Theme 改成 Material-Color-Red

