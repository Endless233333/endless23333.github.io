# Archlinux 安装记录与常用软件


本文记录了我又双叒叕重装 archlinux 的过程和一些常用软件, 主要参考 [ArchWiki](https://wiki.archlinux.org/) 和 [archlinux 简明指南](https://arch.icekylin.online/)。

## 安装
进入安装环境的过程略过。

### 联网
有线网络一般能自动联网, 无线网络使用 `iwctl` 进行连接:
```bash
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

### 硬盘分区、格式化、挂载
由于是重装, 所以硬盘不需要重新分区, 最终分区如下:
```
Device            Start        End    Sectors  Size Type
/dev/nvme0n1p1     2048    1050623    1048576  512M EFI System
/dev/nvme0n1p2  1050624   34605055   33554432   16G Linux swap
/dev/nvme0n1p3 34605056 1953523711 1918918656  915G Linux filesystem
```

格式化:
```bash
mkfs.ext4 /dev/nvme0n1p3
mkswap /dev/nvme0n1p2
mkfs.fat -F 32 /dev/nvme0n1p1
```

按如下顺序挂载:
```bash
mount /dev/nvme0n1p3 /mnt
mount --mkdir /dev/nvme0n1p1 /mnt/boot
swapon /dev/nvme0n1p2
```

### 安装系统和必要的功能性软件
```
pacstrap -K /mnt base linux linux-firmware base-devel networkmanager vim sudo openssh zsh intel-ucode
```

### 生成 fstab 文件
```bash
genfstab -U /mnt > /mnt/etc/fstab
```

### change root
使用 `arch-chroot /mnt` 把系统环境切换到新系统下。

### 设置时区
```bash
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
```bash
hwclock --systohc
```

### 设置 Locale
编辑 `/etc/locale.gen`, 去掉 `en_US.UTF-8 UTF-8` 和 `zh_CN.UTF-8 UTF-8` 前的井号。

然后使用 `locale-gen` 生成 locale.

`echo 'LANG=en_US.UTF-8'  > /etc/locale.conf` 设定 LANG 变量。

### 设置 root 用户密码
```bash
passwd
```

### 安装引导程序
安装 `systemd-boot`:
```bash
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

`root=UUID=` 后接跟分区的 UUID.

### 准备非 root 用户
创建用户:
```bash
useradd -m -G wheel -s /bin/bash XXX
```

设置密码:
```bash
passwd XXX
```

编辑 `sudoers`:
```
EDITOR=vim visudo
```

去掉 `#%wheel ALL=(ALL:ALL) ALL` 前的井号。

### 重启
```bash
exit
umount -R /mnt
reboot
```

### 联网
```bash
systemctl enable --now NetworkManager
nmcli device wifi connect 名 password 密
```

联网后可以通过 `systemctl start sshd` 开启 ssh 服务。

### 开启 32 位支持库
编辑 `/etc/pacman.conf` 去掉 `[multilib]` 及其下一行前的井号。

`pacman -Syyu` 更新一下。

### 一大波安装
```bash
# zsh 插件
pacman -S zsh-autosuggestions zsh-completions zsh-history-substring-search zsh-syntax-highlighting
# xfce4 桌面相关
pacman -S xfce4 xfce4-goodies lightdm lightdm-gtk-greeter gvfs gvfs-mtp udiskie sof-firmware alsa-firmware alsa-ucm-conf pavucontrol pulseaudio pulseaudio-alsa pulseaudio-bluetooth network-manager-applet nm-connection-editor
# 字体
pacman -S adobe-source-han-serif-cn-fonts wqy-zenhei noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra
# intel 核显驱动
pacman -S mesa lib32-mesa vulkan-intel lib32-vulkan-intel
# 蓝牙
pacman -S blueman bluez bluez-utils
# 输入法
pacman -S fcitx5-im fcitx5-chinese-addons fcitx5-material-color fcitx5-pinyin-zhwiki fcitx5-configtool
# 常用软件
pacman -S firefox  flameshot  man-db man-pages syncthing fd fzf git
```

### yay 安装
使用非 root 用户:
```bash
git clone https://aur.archlinux.org/yay-bin.git
pacman -R yay-bin
cd yay-bin
makepkg -si
```

常用软件:
```bash
yay -S clash-verge-rev-bin ttf-intel-one-mono visual-studio-code-bin zsh-theme-powerlevel10k
```

### 启动一些服务
```bash
systemctl enable lightdm.service bluetooth.service systemd-boot-update.service
```

### fcitx5 配置
编辑 `/etc/environment` 加入以下内容:
```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

fcitx5 设置的 Input Method 加上 pinyin.

### 完成安装
至此基本安装结束, 重启之后即可进入桌面环境。

xfce4 中文化需要在 `~/.xprofile` 里加上：
```
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:en_US
```

## 常用软件
无需额外配置的软件直接在下面列出, 需要额外配置的在对应的标题下。

**Packages**
- `aria2`
- `beebeep` 局域网通信
- `bottom` 系统监控
- `dbeaver` 数据库管理
- `dosfstools` dosfat 文件系统工具
- `fping` ping 多个主机
- `geckodriver` firefox 驱动
- `gimp` 绘图
- `gucharmap` 字符查看
- `hugo` 一个 cms
- `inetutils` ftp, rlogin, rsh, telnet 客户端和服务端
- `ipcalc` ip 计算器
- `juk` 音乐播放器
- `kdeconnect` 连手机
- `mdcat` 终端看 markdown
- `mtr` traceroute+ping
- `neofetch` 看系统信息
- `nmap`
- `obs-studio` 录屏
- `p7zip` 7z
- `php`
- `poke` 交互式二进制文件编辑器
- `scrcpy` 连安卓手机
- `tmux` 终端复用
- `tree`
- `wget`
- `z` 智能跳转插件
- `zellij` 终端复用

**AUR**
- `bruno-bin` 接口测试工具
- `drawio-desktop-bin` 绘图
- `fluent-reader-bin` rss 客户端
- `goldendict-ng` 字典
- `localsend-bin` 局域网传文件
- `mycli` mysql 客户端
- `python-selenium` 网页自动化操作
- `toolong` 查看特别长的文件

### docker
`sudo pacman -S docker`

加入到 docker 组: `sudo usermod -aG docker $USER`

使用前需启动 docker 服务。

### java
`sudo pacman -S jdk-openjdk jdk17-openjdk jdk11-openjdk jdk8-openjdk`

使用 `archlinux-java` 切换 java 环境。

### thunderbird
雷鸟邮件客户端。

`sudo pacman -S thunderbird thunderbird-i18n-zh-cn systray-x-common`

qq 邮箱导入时密码要使用授权码。

### tldr
`sudo pacman -S tldr`

使用之前更新以下: `tldr --update`

### tlp
tlp 电源管理

```bash
sudo pacman -S tlp tlp-rdw
yay -S tlpui
sudo systemctl enable tlp.service
sudo systemctl enable NetworkManager-dispatcher.service
sudo systemctl mask systemd-rfkill.service
sudo systemctl mask systemd-rfkill.socket
```

### wireshark
`sudo pacman -S wireshark-qt`

加入到 wireshark 组: `sudo usermod -aG wireshark $USER`

### 虚拟机
#### virtualbox
`sudo pacman -S virtualbox virtualbox-guest-iso` 选择 virtualbox-host-modules-arch

virtualbox-guest-iso 在 `/usr/lib/virtualbox/additions/VBoxGuestAdditions.iso`

加载模块: `sudo modprobe vboxdrv vboxnetadp vboxnetflt`

加入到 vboxusers 组: `sudo usermod -aG vboxusers $USER`

#### vmware
```bash
sudo pacman -S linux-headers
yay vmware-workstation
sudo systemctl start vmware-networks-configuration.service
sudo systemctl enable vmware-networks.service vmware-usbarbitrator.service
sudo modprobe -a vmw_vmci vmmon
```

