# Archlinux 自动解密磁盘

## 前言

由于我的Archlinux使用了LUKS进行磁盘加密，所以在启动系统时，必须输入两次密码，一次是加密磁盘的密码，另一次是用户密码

为了减少每次进入输入系统输入两次密码的麻烦，可以设置开机后自动解锁磁盘

## 加密磁盘

加密磁盘操作具体可以参照[Archlinux基础系统安装](https://fate96.github.io/arch_general_install/#%E5%8A%A0%E5%AF%86)

## 开机磁盘解密

### 配置开机钩子

开机解密，需要配置开机的钩子(Hook)，根据解密钩子的方式可以分为`encrypt hook`和`sd-encrypt hook`，其中后者特性更多，因此采用后者进行解密。编辑`/etc/mkinitcpio.conf`文件 ，将`udev`更换为`systemd`，并添加`sd-vconsole`和`sd-encrypt`

修改前

```conf
HOOKS=(base udev autodetect keyboard modconf block  filesystems fsck)
```

修改后为

```conf
HOOKS=(base systemd autodetect keyboard sd-vconsole modconf block sd-encrypt filesystems fsck)
```

在FILES=()这一栏中添加开机解密密钥的文件所在的位置

```conf
FILES=(/path/to/keyfile)
```

配置完后，重新生成mkinitcpio配置文件

```bash
mkinitcpio -P
```

### 配置内核参数

开机解密磁盘需要告诉内核加密磁盘所在的位置，所以需要配置对应的内核参数，编辑`/etc/default/grub`文件，在`GRUB_CMDLINE_LINUX_DEFAULT`或者`GRUB_CMDLINE_LINUX`参数栏中增加需要添加的内核参数，前者在每次开机都会进行加载，而后者在紧急启动进入系统时则不会加载，所以建议添加在前者栏中

- rd.luks.uuid=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX，使用此标志指定要在启动时解密的设备的UUID

- rd.luks.name=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX=name，指定加密磁盘的UUID和解密后的磁盘名，使用了该内核参数，则可以省略上面的uuid参数

- rd.luks.key=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX=/path/to/keyfile，使用keyfile进行系统解密，并传入keyfile的文件位置

- rd.luks.options=options，设置解密参数，可选设置。如果是SSD，可以使用`discard`参数提供`TRIM`功能

- **root=*/dev/mapper/cryptroot***: 指定解密设备，**该参数不管是encrypt还是sd-crypt，都必须要设置**

其中UUID获取指令如下，**获取的是解密前的磁盘UUID**，而不是解密后的磁盘UUID

```bash
lsblk -f

└─nvme0n1p2   crypto 2           a0ea985c-f2c0-4c6c-af66-2bc10a158b0a
  └─cryptroot ext4   1.0         96055a51-9138-431a-8976-845ca1d09e20 
```

上述查询结果中，`a0ea985c-f2c0-4c6c-af66-2bc10a158b0a`才是我们需要的UUID参数

**注意：这和休眠设置的UUID参数不同，休眠需要的是解密后的UUID**

如下示例:

```conf
GRUB_CMDLINE_LINUX_DEFAULT="rd.luks.name=a0ea985c-f2c0-4c6c-af66-2bc10a158b0a=cryptroot rd.luks.options=timeout=10s,discard rd.luks.key=a0ea985c-f2c0-4c6c-af66-2bc10a158b0a=/etc/mykeyfile root=/dev/mapper/cryptroot"
```

修改后，重新生成grub的配置文件

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

至此，重启开机便可以自动使用密钥进行解锁，而需要手动输入密码进行解锁磁盘了。

## 参考资料

[dm-crypt/System configuration](https://wiki.archlinux.org/title/Dm-crypt/System_configuration)

[让系统更安全 - 系统分区加密 (Btrfs on LUKS) 操作实录](https://nwn.moe/posts/btrfs-on-luks)
