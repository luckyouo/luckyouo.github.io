# Yadm的基本使用

## 前言

在linux系统下，配置文件一般都是以`.`符号开头的，因此也被称为 dotfles。github有他人分享的配置，我们可以从中找到适合自己的配置文件

为了方便管理自己系统系统的配置文件，可以使用工具来对这些 dotfiles 进行统一管理。根据 arch wiki，常用的管理方式主要分为两种，软链接和git跟踪点文件，以下是 github 仓库中用的较多的工具

- [stow](https://www.gnu.org/software/stow/): GNU产品，使用软链进行管理，arch的farseerfc大佬就是使用该工具进行管理，具体介绍可以参照他的博客[使用GNU stow 管理你的点文件](https://farseerfc.me/using-gnu-stow-to-manage-your-dotfiles.html)
  
- git bare: 使用git bare裸仓库进行管理

- [yadm](https://yadm.io/): Yet Another Dotfiles Manager，一个git的wrapper，底层仍然是使用git来进行管理，方法使用和git没有区别，可以对特定文件加密

- [chezmoi](https://www.chezmoi.io/): 和git用法差不多，但是会将需要跟踪的点文件冗余存储在chezmoi特定的本地仓库，可以对特定文件加密

为了保持和 git 的使用习惯，最后选择yadm作为点文件的管理工具，yadm默认相对位置为自己的家目录($HOME)

## 安装

- Archlinux
  
```bash
sudo pacman -S yadm
```

- Ubuntu/Debian

```bash
sudo apt install -y yadm
```

- OSX

```bash
brew install yadm
```

## 仓库初始化

### 初始化本地仓库

如果一开始没有远程的 dotfiles 仓库，则需要先初始化本地仓库

```bash
yadm init
yadm add <important file>
yadm commit
```

再将本地仓库与远程仓库进行同步

```bash
yadm remote add origin <url>
yadm push -u origin main
```

### 同步远程仓库

若一开始就有远程仓库，或使用他人的 yadm 的远程仓库时，则将远程仓库克隆下来即可

```bash
yadm clone <url>
yadm status
```

## 基本使用

### 添加需要跟踪的配置文件

使用 add 方法，将 dotfile 添加至 yadm 跟踪 list 当中

```bash
yadm add /path/to/dotfile
```

### 跟踪列表查看

使用 list 方法，查看当前路径下的跟踪文件，使用 `-a` 参数(Optional)，可以查看所有跟踪文件

```bash
yadm list <-a>
```

### 推送至远程仓库

使用commit和push方法，将变化文件推送至远程仓库

```bash
git commit -m "commit word"
git push -u origin main
```

## 文件加密和解密

在`$HOME/.config/yadm/encrypt`文件中设置需要加密的文件，支持正则表达式，默认文件的起始地址为`%HOME`目录下，`$HOME/.config/yadm/encrypt`参考格式如下

```config
.ssh/*.key
```

yadm支持gpg的对称加密和非对称加密，默认加密方式为gpg的对称加密，加密后的文件存储于 `$HOME/.local/share/yadm/archive` 文件当中，

```bash
yadm add $HOME/.config/yadm/encrypt
yadm add $HOME/.local/share/yadm/archive
```

也可以通过 `yadm config yadm.gpg-recipient <recipient-address>` 指令指定使用非对称加密

```bash
yadm config yadm.gpg-recipient <recipient-address>
```

chezmoi 不同的是，yadm 需要将文件手动进行解密，若使用对称加密，则按提示输入密码既可进行解密

注：系统没有 gunpg 则，则无法解密

```bash
yadm decrypt
```

## 引导程序

yadm 支持在初始化仓库时，自动调用 Bootstrap 程序执行初始化，该文件默认位置为`$HOME/.config/yadm/bootstrap`，且该文件必须为可执行程序

在初始化时，如果系统本身就含有 yadm 仓库对应的点文件时，yadm 默认是不会处理该文件，即不会覆盖，需要用户自己手动处理该冲突，对于引导程序，除了初始化执行时，也可以手动执行引导程序，来进行引导程序的调试

```bash
yadm bootstrap
```

## 参考资料

[Archlinux Dotfiles](https://wiki.archlinux.org/title/Dotfiles)

[Yet Another Dotfiles Manager](https://yadm.io/docs/bootstrap#)

[使用Yadm来管理我的配置](https://www.escapelife.site/posts/696ce25d.html)

[使用GNU stow 管理你的点文件](https://farseerfc.me/using-gnu-stow-to-manage-your-dotfiles.html)
