# Archlinux 桌面系统安装

## 前言

作为主力系统，自然需要桌面系统(DE)。主流DE有KDE和GNOME等，根据个人习惯选择系统。本文主要介绍KDE桌面系统的安装

## 联网

由于桌面系统还没有安装完成，所以采用iwd和dhcpcd进行网络连接，首先启动iwd和dhcpcd，在使用iwd连接无限网络。iwd参照基础系统安装教程

```code
systemctl start iwd
systemctl start dhcpcd
```

## 准备普通用户

由于桌面系统需要普通用户，故先添加普通用户，并配置相应的权限

```code
useradd -m -G wheel -s /bin/bash username
passwd username
```

配置sudo权限

```code
EDITOR=vim visudo
```

找到下面该行，并取消注释

```sudoers
#%wheel ALL=(ALL) ALL
```

## 安装KDE环境

KDE安装有三个包组，plasma-meta、plasma和plasma-desktop。其中plasma和plasma-meta区别为，kde后续增加软件包时，plasma-meta会自动安装，而plasma不会，剩下plasma-desktop只有kde能跑起来最少的软件。推荐使用plasma-meta，防止后续缺少组件。konsole和dolphin分别为kde的常用终端和文件管理

```code
pacman -S plasma-meta konsole dolphin
```

## 启用DM

kde默认自带sddm，可以根据需要更换

```code
systemctl enable sddm
```

## 启用网络管理组件

kde和gnome桌面系统一般采用NetworkManager进行网络管理

```code
systemctl enable NetworkManager
```

## 重启进入桌面系统

自此，基本的桌面系统已经安装完成，可以重启进入桌面系统

```code
reboot
```

## AUR Helper安装

`yay`和`paru`是较为常用的aur helper组件，根据个人习惯进行选择

**注：由于安装需要从github拉取文件，需要配置代理才可以进行**

```code
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

## 输入法安装

facitx5为社区推荐输入法，安装facitx5

```code
sudo pacman -S fcitx5-im
sudo pacman -S fcitx5-chinese-addons fcitx5-pinyin-zhwiki fcitx5-material-colo
```

配置环境，konsole和dolphin需要环境变量的支持才可以使用输入法，使用`EDITOR=vim sudoedit /etc/environment`进行添加环境变量

```code
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
```

## 配置默认编辑器

arch默认编辑器为vi，一般使用vim，使用`EDITOR=vim sudoedit /etc/profile`配置环境变量

```code
export EDITOR='vim'
```

## 蓝牙配置

安装并配置蓝牙

```code
sudo pacman -S bluez bluez-utils pulseaudio-bluetooth pavucontrol pulseaudio-alsa
yay -S bluez-firmware
sudo systemctl enable bluetooth.service
sudo systemctl start bluetooth.service
pulseaudio -k
pulseaudio --start
sudo usermod -a -G lp $USER
```

设置蓝牙自启，编辑`/etc/bluetooth/main.conf`文件，更改AutoEnable的值为`true`

```code
sudo vim /etc/bluetooth/main.conf 
```

## KDE安装完成

自此，arch的KDE系统基本完成安装。

KDE是可以根据自己的需要进行高度定制化，可以根据自己的需要进行定制化

## 参考资料

[Archlinux Installation guide](https://wiki.archlinux.org/title/installation_guide)

[Arch Linux 安装使用教程 - ArchTutorial - Arch Linux Studio](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/)

[2021 Archlinux双系统安装教程（超详细）](https://zhuanlan.zhihu.com/p/138951848)

[Arch Linux Monthly Install: January 2022](https://www.youtube.com/watch?v=7btEUHjECAo&t=2362s)
