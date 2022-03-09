# Aria2的基本配置


## 前言

最近节点也不知道怎么了，从谷歌云盘下载东西时，经常因为断流而导致下载中断，导致我的大文件几乎就下不下来，网上找了下解决方法，最后选择aria2，一个支持断点续传的软件

aria2 是一个轻量级的多协议和多源命令行下载实用程序。它支持 HTTP/HTTPS、FTP、BitTorrent 和 Metalink。 aria2 可以通过内置的 JSON-RPC 和 XML-RPC 接口进行操作。

## 安装

aria2 可以在Arch的官方仓库找到

```bash
sudo pacman -S aria2
```

同时 [archlinuxcn](https://github.com/archlinuxcn/repo) 源和 [AUR](https://aur.archlinux.org/packages/aria2-fast) 仓库都存在对原版 aria2 补丁的 aria2-fast，原版设置了16线程的上线，该补丁版本最多可以设置为128线程

```bash
yay -S aria2-fast
```

## 配置

aria2 默认配置文件位于 `$HOME/.aria2/aria2.conf` ，具体配置内容可以参考[arch wiki](https://wiki.archlinux.org/title/aria2) 或者 [P3TERX](https://github.com/P3TERX/aria2.conf) 大佬的

**注意：若开启了 session,需要手动创建对应的 aria2.session 文件，否则 daemon 启动不了**

## 设置daemon

一般情况下，aria2 下载完成后便会退出，因此我们可以使用守护单元，在开机后自动在后台保持允许即可

用户级别的 systemd unit 位于 `$HOME/.config/systemd/usr` 目录下，系统级别的在 `/etc/systemd/system` 目录下，根据自己情况选择。本教程采用用户级别

编辑对应目下的aria2.service，配置内容如下

```conf
[Unit]
Description=aria2 Daemon

[Service]
Type=simple
ExecStart=/usr/bin/aria2c --conf-path=${HOME}/.config/aria2/aria2.conf

[Install]
WantedBy=default.target
```

接着设置后台自启并立即启动

```bash
systemctl enable --user now aria2.service
```

## 浏览器配置

AriaNG 是较为主流的 aria2 前端，其中有第三方在此基础之上实现的浏览器插件，比如 [Chrome](https://chrome.google.com/webstore/detail/aria2-for-chrome/mpkodccbngfoacfalldjimigbofkhgjn)和 [Firefox](https://addons.mozilla.org/zh-CN/firefox/addon/aria2-integration/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)

根据配置文件的设置的RPC服务器的密码，在插件中配置好相应的 `rpc-secret`，同时可以在扩展选项中设置对浏览器的下载进行拦截，统一交给 aria2 服务进行下载

插件设置内容如下图所示

![AriaNG](/images/aria2/AriaNG.png)
<!-- {{ < image src="/images/aria2/AriaNG.png" > }} -->

至此，完成了 aria2 的基本设置

## 参考资料

[Archlinux aria2](https://wiki.archlinux.org/title/aria2)

[Aria2 前端面板 ( GUI、WebUI ) AriaNg 使用教程](https://p3terx.com/archives/aria2-frontend-ariang-tutorial.html)

[Arch Linux 的 Aria2 食用指南](https://blog.allwens.work/archlinuxAria2/)
