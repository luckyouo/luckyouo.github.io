# Archlinux 的基础系统安装流程

## 前言

第一次使用linux系统，查询linux各个版本的区别，最后看上了arch的aur，故选择arch作为主力linux系统(人的生命在于折腾，折腾就完事了)

这篇主要记录了基本系统的安装，不包含桌面系统

## EFI模式验证

首先确认安装模式是否为EFI模式

```code
ls /sys/firmware/efi/efivars
```

## 系统时钟校正

```bash
timedatectl set-ntp true
```

## 进行网络连接

arch系统网络连接方式主要有两种，有线连接和无线连接

### 有线连接

可以使用手机线与电脑进行连接，进行网络共享

### 无线连接

arch live系统默认为开启iwd,使用iwd进行无线网络连接

```code
iwctl
device list
station wlan0 get-networks
station waln0 connect WIFI-NAME
```

首先进入iwctl模式，查询当前设备无线网卡设备名,比如wlan0），再使用网卡查询当前wifi网络情况，最后对目标WIFI-NAME进行连接，提示输入密码进行确认

## 数据分区及格式化

### 分区

常见分区工具有cfdisk,gdisk等。为了方便，可以使用GUI的cfdisk进行分区

1. 为了防止某些异常，优先对efi进行分区，大小500M～1G即可
2. 剩下分区方式根据个人习惯进行。

- 比以将home分区单独分区，缺点是home分区和其他分区大小需要控制好，防止某分区空间爆满而需要对文件系统挪动，
  
- 将剩余分区大小划分为一个分区，缺点是重装系统麻烦

为了节省存储空间，故采用第二种方式。分区完成后，对主分区进行加密(Optional)

### 加密(可选)

为了保证自己的数据安全，可以对主要分区内容进行加密，防止他人挂载直接进入系统读取文件。

1. 使用LUKS对主分区进行加密，没有特殊要求采用默认参数即可

- 默认使用手动输入密码对文件进行加密
  
```code
cryptsetup -y -v luksFormat /path/to/device
```

- 使用keyfile对文件进行加密

    首先生成所需要的keyfile文件，根据生成keyfile文件的类型不同，可以分为密码方式和随机字符或二进制。

    为了简便，采用密码对文件进行加密，并将密码制作为keyfile,用作开机自动解密。

```code
echo -n 'your_passphrase' > /etc/keyfile
chown root:root /etc/keyfile; chmod 400 /etc/keyfile
```

2. 解密分区

    cryptroot 为解密后对应的文件名，可以自定义

```code
cryptsetup open /path/to/device/ cryptroot
```

### 格式化

efi分区采用mkfs.fat进行格式化。主分区根据不同的文件类型特性进行选择格式化，比如mkfs.ext4格式化为ext4,mkfs.btrfs格式化为btrfs，其中ext4速度整体快于btrfs,而btrfs存在快照等特性。

```code
mkfs.fat -F 32 /path/to/efi
mkfs.ext4 /path/to/device
```

## 挂载分区，并安装基本软件

先挂载/mnt分区，在创建/mnt/boot分区，并挂载(也可以创建/mnt/efi分区并进行挂载，二者不同之处可以查看archwiki)

```code
mount /dev/mapper/cryptroot /mnt
mount /path/to/efi /mnt/boot
```

其中cryptroot为解密后分区所对应的名字，挂载时需要选择解密后的分区名

使用pacstarp在/mnt分区中安装基本软件，并生成分区文件，最后进入安装系统(根据电脑CPU类型安装所需要的微码，intel的cpu安装intel-ucode，amd的安装amd-ucode)

```code
pacstarp /mnt base base-devel linux linux-headers linux-firmware  sudo amd-ucode
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
```

## 更改镜像源并启用32位库

由于国外镜像站网速限制，可以使用国内教育镜像站进行加速，镜像配置文件为/etc/pacman.d/mirrorlist，可以设置多个镜像站

```code
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
```

由于部分软件需要32位库，比如wine,而默认32位库是注释掉了，需要取消注释才能启用，不需要32位软件的可以不进行该操作。pacman的默认库文件位/etc/pacman.conf，取消注释multilib库

```code
[multilib]
Server = /etc/pacman.d/mirrorlist
```

## 修改密码

修改root的密码

```code
passwd
```

## 基础软件安装

安装基础软件，比如网络，编辑器等

```code
pacman -S networkmanager vim git wget curl dhcpcd iwd bash-completion dialog wpa_supplicant
```

## 时区和区域设置

使用软链设置时区，比如设置位亚洲的上海时区，并同步

```code
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

区域设置，对应文件位/etc/locale.gen，编辑并取消所需要的区域设置，并生成对应文件

```code
vim /etc/locale.gen             # 取消en_US.UTF-8和zh_CN.UTF-8的注销
locale-gen
```

向/etc/locale.conf导入本地语言设置，防止乱码

```code
echo 'LANG=en_US.UTF-8'  > /etc/locale.conf
```

## 主机名设置

编辑/etc/hostname，设置主机名，比如arch，并根据主机名设置host文件

```code
echo arch > /etc/hostname
cat > /etc/hosts <<EOF
127.0.0.1 localhost
::1 localhost
127.0.1.1 arch.localdomain arch
EOF
```

## 安装引导文件

引导文件有grub和system-boot，system-boot需要手动写入启动文件，grub指令可以直接生成，较为方便，故采用grub(若efi分区挂载为/mnt/efi，则将efi-directory更改为/efi)

```code
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

## 解密参数配置(可选)

如果对磁盘设置了加密，必须设置一些hook和内核参数来告诉内核加密磁盘的位置

具体操作可以参见[Archlinux自动解密磁盘](https://fate96.github.io/arch_auto_decrypt/)

## 完成安装

基本系统已经完成安装，退出系统，并取消文件挂载，重启既可以进入安装好的系统

```code
exit
umount -a
reboot
```

## 参考资料

[Archlinux Installation guide](https://wiki.archlinux.org/title/installation_guide)

[dm-crypt/Encrypting an entire system](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system)

[Arch Linux 安装使用教程 - ArchTutorial - Arch Linux Studio](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/)

[Arch Linux 搭建 java 开发环境](https://teaper.dev/)

[2021 Archlinux双系统安装教程（超详细）](https://zhuanlan.zhihu.com/p/138951848)

[Arch Linux Monthly Install: January 2022](https://www.youtube.com/watch?v=7btEUHjECAo&t=2362s)

