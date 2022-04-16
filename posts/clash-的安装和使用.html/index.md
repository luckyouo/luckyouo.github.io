# Clash 安装与使用

## 前言

Clash 是一个跨平台、支持 SS/V2ray 等协议、基于规则的网络代理软件

Clash 只是一个内核，为开源软件，而 Clash-premium 是闭源软件，后者多了scripts，rule-set和tun(windows只能使用该功能实现全局代理)

从clash的内核衍生了众多版本，用的最多的有 Clash for windows (简称 cfw )，Open Clash，Clashx。其中cfw不仅支持windows，还支持macos和linux，open Clash只支持路由器的openwrt系统，Clashx只支持macos

## 安装

在 arch 的官方仓库当中有根据 clash 内核打包好的组件，aur社区有打包好的 cfw 和 clash-user。其中cfw有ui界面，与 windows 上设置没有什么区别，而 clash-user 自带 clash 用户，可以根据用户的 uid 来设置防止流量回环

由于 linux 可以使用 tproxy 实现全局代理，可以直接使用官方仓库的 clash (或者 aur 的 clash-user ），来减少 clash 的资源占用( cfw 的 gui 需要占用资源)

```bash
sudo pacman -S clash
```

使用`yacd`面板对Clash管理和流量监控

```bash
yay -S yacd
```

## 文件配置

clash 默认的配置文件位于用户目录下，而 clash-user 的配置位于`/etc/clash`目录下

配置中主要参数解释：

- port: http(s)代理端口
  
- socks-port: socks5代理端口

- redir-port: redir tcp代理端口

- tproxy-port: tproxy udp(tcp)代理端口

- mode: 代理方式，有全局代理(gloable)、规则代理(rule)和直连(direct)。其rle根据配置所提供的规则流量代理

- external-controller: clash提供的api控制地址

- external-ui: 外部控制ui地址

- profile: clash部分存储设置模块

  - store-selected: 策略组节点选择后，是否需要存储选择记录，默认不存储(false)

  - store-fake-ip: 设置为 true 是，将 fake-ip 与真实 dns 的 ip 对应记录存储在本地，以达到持久存储 dns 记录，加快 dns 解析速度
  
- dns: clash的dns模块
  
  - enable: 设置为 true 时开启 clash 内置的dns模块

  - listen: dns 的监听端口

  - enhanced-mode: clash 提供两种 dns 查询模式，一种为正常 redir 转发模式，另一种为 fake-ip 模式，每次应用 dns 请求时，clash 将返回一个 fake 的 ip 地址，来达到加速建立 tcp 连接需求，但也会导致得到的 ip 不是真实的 ip，从而对某些网络调试带来麻烦
  
  - nameserver: dns 查询服务器，可以设置为 doh，tls 等dns查询方式。具体可以根根据个人习惯设置

  - fake-ip-filter: 过滤不使用 fake-ip 查来询dns的地址

- proxies: 代理节点设置模块，请根据官方要求进行设置

- proxy-groups: clash 的策略组，与 surge 类似。包含 relay(轮询)，url-test(ping最低优先)，load-balance(负载均衡)，fallback(按节点顺序优先使用可用节点)，select(手动选择节点)

- proxy-providers: 节点提供模块，一次可以提供多个级诶但，clash的`1.9.0`中增加了filter用来过滤节点

  - type: 节点组提供的类型，可选 http (远程提供)和 file (本地文件提供)
  
  - filter: 节点过滤方式，支持正则

- rules: 规则模块，以下是官方提供的说明，可以根据需要进行规则设置
  - DOMAIN: 规则会匹配与请求完全相同的地址
  - DOMAIN-SUFFIX: 规则会匹配与请求与主域名相同的地址，比如，`google.com`匹配`www.google.com`, `mail.google.com`和`google.com`本身，但不会匹配`content-google.com`
  - DOMAIN-KEYWORD: 规则会匹配包含相应关键字的域名，除了 "apple" 会匹配 `www.apple.com`，"app" 同样也会匹配到
  - GEOIP: 规则会匹配相应国家和地区的 IP 地址
  - IP-CIDR: 规则会匹配规则范围内请求的 IP 地址
  - IP-CIDR6: 规则会匹配规则范围内请求的 IPv6 地址
  - SRC-IP-CIDR: 规则会匹配源 IP 地址
  - SRC-PORT: 规则会匹配源端口地址
  - DST-PORT: 规则会匹配目的地端口地址
  - PROCESS-NAME: 规则会匹配这个进程名的程序
  - MATCH: 将其余数据包路由到策略。此规则是必需的

以下为本人根据 clash 提供的官方配置进行更改的自用配置。dns 设置为 fake-ip 模式，dns 服务器均采用 doh，防止国内而 dns 污染，并使用 yacd 模块。因为 clash 开源版不支持rule-set模块(clash-premium支持)，所以不能像 surge 一样简洁书写规则，嫌规则冗长的可以使用 clash-premium 闭源软件书写，其他模块书写规则一致

```yaml
mixed-port: 7890
allow-lan: true
mode: Rule
log-level: warning
external-controller: 127.0.0.1:9090
socks-port: 7891
redir-port: 7892
tproxy-port: 7893
ipv6: false
external-ui: '/usr/share/yacd'
profile:
  store-selected: false
  store-fake-ip: true
dns:
  enable: true
  listen: 0.0.0.0:1053
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  nameserver:
    - https://223.5.5.5/dns-query
    - https://doh.pub/dns-query
    #- 114.114.114.114
    #- 223.5.5.5
  fallback:
    - https://1.1.1.1/dns-query
    - https://1.0.0.1/dns-query
    - https://8.8.8.8/dns-query
  fallback-filter:
    geoip: true
    geoip-code: CN
  fake-ip-filter:
    - '*.lan'
    - '*.localhost'
    - '*.local'
    - 'lens.l.google.com'
    - 'stun.l.google.com'
    - '*.gitbook.io'


proxy-providers:
  HK_LOW:
    type: file
    path: ./cordcloud.yaml
    interval: 3600
    filter: '长沙联通转香港|广东移动转香港'
    health-check:
      enable: true
      url: http://wifi.vivo.com.cn/generate_204
      interval: 600

  HK:
    type: file
    path: ./cordcloud.yaml
    interval: 3600
    filter: '深港专线'
    health-check:
      enable: true
      url: http://wifi.vivo.com.cn/generate_204
      interval: 600

  TW:
    type: file
    path: ./cordcloud.yaml
    interval: 3600
    filter: '台湾'
    health-check:
      enable: true
      url: http://wifi.vivo.com.cn/generate_204
      interval: 600

  JP:
    type: file
    path: ./cordcloud.yaml
    interval: 3600
    filter: '日本'
    health-check:
      enable: true
      url: http://wifi.vivo.com.cn/generate_204
      interval: 600

  SG:
    type: file
    path: ./cordcloud.yaml
    interval: 3600
    filter: '新加坡'
    health-check:
      enable: true
      url: http://wifi.vivo.com.cn/generate_204
      interval: 600

  US:
    type: file
    path: ./cordcloud.yaml
    interval: 3600
    filter: '美国'
    health-check:
      enable: true
      url: http://wifi.vivo.com.cn/generate_204
      interval: 600


proxy-groups:

  - name: Final
    type: select
    proxies:
      - Proxies
      - DIRECT

  - name: Proxies
    type: select
    proxies:
      - load_balance
      - hk_urltest
      - tw_urltest
      - sg_urltest
      - us_urltest

  - name: StreamSE
    type: select
    proxies:
      - load_balance
      - hk_urltest
      - tw_urltest
      - sg_urltest
      - us_urltest

  - name: Google
    type: select
    proxies:
      - hk_fallback
      - hk_urltest
      - tw_urltest
      - sg_urltest
      - Proxies

  - name: Telegram
    type: select
    proxies:
      - sg_urltest
      - hk_urltest
      - tw_urltest
      - Proxies


  - name: load_balance
    type: load-balance
    url: http://wifi.vivo.com.cn/generate_204
    interval: 300
    proxies:
    use:
      - HK_LOW

  - name: hk_urltest
    type: url-test
    url: http://wifi.vivo.com.cn/generate_204
    interval: 300
    proxies:
    use:
      - HK_LOW

  - name: hk_fallback
    type: fallback
    url: http://wifi.vivo.com.cn/generate_204
    interval: 300
    proxies:
    use:
      - HK

  - name: us_urltest
    type: url-test
    url: http://wifi.vivo.com.cn/generate_204
    interval: 300
    proxies:
    use:
      - US

  - name: tw_urltest
    type: url-test
    url: http://wifi.vivo.com.cn/generate_204
    interval: 300
    proxies:
    use:
      - TW

  - name: sg_urltest
    type: url-test
    url: http://wifi.vivo.com.cn/generate_204
    interval: 300
    proxies:
    use:
      - TW

  - name: jp_urltest
    type: url-test
    url: http://wifi.vivo.com.cn/generate_204
    interval: 300
    proxies:
    use:
      - JP



rules:
  - DOMAIN-SUFFIX,smtp,DIRECT
  - DOMAIN-KEYWORD,aria2,DIRECT


  - DOMAIN-KEYWORD,www.google.,Google
  - DOMAIN,lens.l.google.com,Google
  - DOMAIN-SUFFIX,translate.googleapis.com,Google


  - DOMAIN-SUFFIX,ampproject.org,Proxies
  - DOMAIN-SUFFIX,appspot.com,Proxies
  - DOMAIN-SUFFIX,blogger.com,Proxies
  - DOMAIN-SUFFIX,getoutline.org,Proxies
  - DOMAIN-SUFFIX,gvt0.com,Proxies
  - DOMAIN-SUFFIX,gvt1.com,Proxies
  - DOMAIN-SUFFIX,gvt3.com,Proxies
  - DOMAIN-SUFFIX,xn--ngstr-lra8j.com,Proxies
  - DOMAIN-KEYWORD,google,Proxies
  - DOMAIN-KEYWORD,blogspot,Proxies
  - DOMAIN-SUFFIX,onedrive.live.com,Proxies
  - DOMAIN-SUFFIX,xboxlive.com,Proxies
  - DOMAIN-SUFFIX,cdninstagram.com,Proxies
  - DOMAIN-SUFFIX,fb.com,Proxies
  - DOMAIN-SUFFIX,fb.me,Proxies
  - DOMAIN-SUFFIX,fbaddins.com,Proxies
  - DOMAIN-SUFFIX,fbcdn.net,Proxies
  - DOMAIN-SUFFIX,fbsbx.com,Proxies
  - DOMAIN-SUFFIX,fbworkmail.com,Proxies
  - DOMAIN-SUFFIX,instagram.com,Proxies
  - DOMAIN-SUFFIX,m.me,Proxies
  - DOMAIN-SUFFIX,messenger.com,Proxies
  - DOMAIN-SUFFIX,oculus.com,Proxies
  - DOMAIN-SUFFIX,oculuscdn.com,Proxies
  - DOMAIN-SUFFIX,rocksdb.org,Proxies
  - DOMAIN-SUFFIX,whatsapp.com,Proxies
  - DOMAIN-SUFFIX,whatsapp.net,Proxies
  - DOMAIN-KEYWORD,facebook,Proxies
  - IP-CIDR,3.123.36.126/32,Proxies,no-resolve
  - IP-CIDR,35.157.215.84/32,Proxies,no-resolve
  - IP-CIDR,35.157.217.255/32,Proxies,no-resolve
  - IP-CIDR,52.58.209.134/32,Proxies,no-resolve
  - IP-CIDR,54.93.124.31/32,Proxies,no-resolve
  - IP-CIDR,54.162.243.80/32,Proxies,no-resolve
  - IP-CIDR,54.173.34.141/32,Proxies,no-resolve
  - IP-CIDR,54.235.23.242/32,Proxies,no-resolve
  - IP-CIDR,169.45.248.118/32,Proxies,no-resolve
  - DOMAIN-SUFFIX,pscp.tv,Proxies
  - DOMAIN-SUFFIX,periscope.tv,Proxies
  - DOMAIN-SUFFIX,t.co,Proxies
  - DOMAIN-SUFFIX,twimg.co,Proxies
  - DOMAIN-SUFFIX,twimg.com,Proxies
  - DOMAIN-SUFFIX,twitpic.com,Proxies
  - DOMAIN-SUFFIX,vine.co,Proxies
  - DOMAIN-KEYWORD,twitter,Proxies
  - DOMAIN-SUFFIX,t.me,Proxies
  - DOMAIN-SUFFIX,tdesktop.com,Proxies
  - DOMAIN-SUFFIX,telegra.ph,Proxies
  - DOMAIN-SUFFIX,telegram.me,Proxies
  - DOMAIN-SUFFIX,telegram.org,Proxies
  - IP-CIDR,91.108.4.0/22,Proxies,no-resolve
  - IP-CIDR,91.108.8.0/22,Proxies,no-resolve
  - IP-CIDR,91.108.12.0/22,Proxies,no-resolve
  - IP-CIDR,91.108.16.0/22,Proxies,no-resolve
  - IP-CIDR,91.108.56.0/22,Proxies,no-resolve
  - IP-CIDR,149.154.160.0/20,Proxies,no-resolve
  - IP-CIDR6,2001:b28:f23d::/48,Proxies,no-resolve
  - IP-CIDR6,2001:b28:f23f::/48,Proxies,no-resolve
  - IP-CIDR6,2001:67c:4e8::/48,Proxies,no-resolve
  - DOMAIN-SUFFIX,line.me,Proxies
  - DOMAIN-SUFFIX,line-apps.com,Proxies
  - DOMAIN-SUFFIX,line-scdn.net,Proxies
  - DOMAIN-SUFFIX,naver.jp,Proxies
  - IP-CIDR,103.2.30.0/23,Proxies,no-resolve
  - IP-CIDR,125.209.208.0/20,Proxies,no-resolve
  - IP-CIDR,147.92.128.0/17,Proxies,no-resolve
  - IP-CIDR,203.104.144.0/21,Proxies,no-resolve
  - DOMAIN-SUFFIX,4shared.com,Proxies
  - DOMAIN-SUFFIX,520cc.cc,Proxies
  - DOMAIN-SUFFIX,881903.com,Proxies
  - DOMAIN-SUFFIX,9cache.com,Proxies
  - DOMAIN-SUFFIX,9gag.com,Proxies
  - DOMAIN-SUFFIX,abc.com,Proxies
  - DOMAIN-SUFFIX,abc.net.au,Proxies
  - DOMAIN-SUFFIX,abebooks.com,Proxies
  - DOMAIN-SUFFIX,amazon.co.jp,Proxies
  - DOMAIN-SUFFIX,apigee.com,Proxies
  - DOMAIN-SUFFIX,apk-dl.com,Proxies
  - DOMAIN-SUFFIX,apkfind.com,Proxies
  - DOMAIN-SUFFIX,apkmirror.com,Proxies
  - DOMAIN-SUFFIX,apkmonk.com,Proxies
  - DOMAIN-SUFFIX,apkpure.com,Proxies
  - DOMAIN-SUFFIX,aptoide.com,Proxies
  - DOMAIN-SUFFIX,archive.is,Proxies
  - DOMAIN-SUFFIX,archive.org,Proxies
  - DOMAIN-SUFFIX,arte.tv,Proxies
  - DOMAIN-SUFFIX,artstation.com,Proxies
  - DOMAIN-SUFFIX,arukas.io,Proxies
  - DOMAIN-SUFFIX,ask.com,Proxies
  - DOMAIN-SUFFIX,avg.com,Proxies
  - DOMAIN-SUFFIX,avgle.com,Proxies
  - DOMAIN-SUFFIX,badoo.com,Proxies
  - DOMAIN-SUFFIX,bandwagonhost.com,Proxies
  - DOMAIN-SUFFIX,bbc.com,Proxies
  - DOMAIN-SUFFIX,behance.net,Proxies
  - DOMAIN-SUFFIX,bibox.com,Proxies
  - DOMAIN-SUFFIX,biggo.com.tw,Proxies
  - DOMAIN-SUFFIX,binance.com,Proxies
  - DOMAIN-SUFFIX,bitcointalk.org,Proxies
  - DOMAIN-SUFFIX,bitfinex.com,Proxies
  - DOMAIN-SUFFIX,bitmex.com,Proxies
  - DOMAIN-SUFFIX,bit-z.com,Proxies
  - DOMAIN-SUFFIX,bloglovin.com,Proxies
  - DOMAIN-SUFFIX,bloomberg.cn,Proxies
  - DOMAIN-SUFFIX,bloomberg.com,Proxies
  - DOMAIN-SUFFIX,blubrry.com,Proxies
  - DOMAIN-SUFFIX,book.com.tw,Proxies
  - DOMAIN-SUFFIX,booklive.jp,Proxies
  - DOMAIN-SUFFIX,books.com.tw,Proxies
  - DOMAIN-SUFFIX,boslife.net,Proxies
  - DOMAIN-SUFFIX,box.com,Proxies
  - DOMAIN-SUFFIX,businessinsider.com,Proxies
  - DOMAIN-SUFFIX,bwh1.net,Proxies
  - DOMAIN-SUFFIX,castbox.fm,Proxies
  - DOMAIN-SUFFIX,cbc.ca,Proxies
  - DOMAIN-SUFFIX,cdw.com,Proxies
  - DOMAIN-SUFFIX,change.org,Proxies
  - DOMAIN-SUFFIX,channelnewsasia.com,Proxies
  - DOMAIN-SUFFIX,ck101.com,Proxies
  - DOMAIN-SUFFIX,clarionproject.org,Proxies
  - DOMAIN-SUFFIX,clyp.it,Proxies
  - DOMAIN-SUFFIX,cna.com.tw,Proxies
  - DOMAIN-SUFFIX,comparitech.com,Proxies
  - DOMAIN-SUFFIX,conoha.jp,Proxies
  - DOMAIN-SUFFIX,crucial.com,Proxies
  - DOMAIN-SUFFIX,cts.com.tw,Proxies
  - DOMAIN-SUFFIX,cw.com.tw,Proxies
  - DOMAIN-SUFFIX,cyberctm.com,Proxies
  - DOMAIN-SUFFIX,dailymotion.com,Proxies
  - DOMAIN-SUFFIX,dailyview.tw,Proxies
  - DOMAIN-SUFFIX,daum.net,Proxies
  - DOMAIN-SUFFIX,daumcdn.net,Proxies
  - DOMAIN-SUFFIX,dcard.tw,Proxies
  - DOMAIN-SUFFIX,deepdiscount.com,Proxies
  - DOMAIN-SUFFIX,depositphotos.com,Proxies
  - DOMAIN-SUFFIX,deviantart.com,Proxies
  - DOMAIN-SUFFIX,disconnect.me,Proxies
  - DOMAIN-SUFFIX,discordapp.com,Proxies
  - DOMAIN-SUFFIX,discordapp.net,Proxies
  - DOMAIN-SUFFIX,disqus.com,Proxies
  - DOMAIN-SUFFIX,dlercloud.com,Proxies
  - DOMAIN-SUFFIX,dns2go.com,Proxies
  - DOMAIN-SUFFIX,dowjones.com,Proxies
  - DOMAIN-SUFFIX,dropbox.com,Proxies
  - DOMAIN-SUFFIX,dropboxusercontent.com,Proxies
  - DOMAIN-SUFFIX,duckduckgo.com,Proxies
  - DOMAIN-SUFFIX,dw.com,Proxies
  - DOMAIN-SUFFIX,dynu.com,Proxies
  - DOMAIN-SUFFIX,earthcam.com,Proxies
  - DOMAIN-SUFFIX,ebookservice.tw,Proxies
  - DOMAIN-SUFFIX,economist.com,Proxies
  - DOMAIN-SUFFIX,edgecastcdn.net,Proxies
  - DOMAIN-SUFFIX,edu,Proxies
  - DOMAIN-SUFFIX,elpais.com,Proxies
  - DOMAIN-SUFFIX,enanyang.my,Proxies
  - DOMAIN-SUFFIX,encyclopedia.com,Proxies
  - DOMAIN-SUFFIX,esoir.be,Proxies
  - DOMAIN-SUFFIX,etherscan.io,Proxies
  - DOMAIN-SUFFIX,euronews.com,Proxies
  - DOMAIN-SUFFIX,evozi.com,Proxies
  - DOMAIN-SUFFIX,feedly.com,Proxies
  - DOMAIN-SUFFIX,firech.at,Proxies
  - DOMAIN-SUFFIX,flickr.com,Proxies
  - DOMAIN-SUFFIX,flitto.com,Proxies
  - DOMAIN-SUFFIX,foreignpolicy.com,Proxies
  - DOMAIN-SUFFIX,freebrowser.org,Proxies
  - DOMAIN-SUFFIX,freewechat.com,Proxies
  - DOMAIN-SUFFIX,freeweibo.com,Proxies
  - DOMAIN-SUFFIX,friday.tw,Proxies
  - DOMAIN-SUFFIX,ftchinese.com,Proxies
  - DOMAIN-SUFFIX,ftimg.net,Proxies
  - DOMAIN-SUFFIX,gate.io,Proxies
  - DOMAIN-SUFFIX,getlantern.org,Proxies
  - DOMAIN-SUFFIX,getsync.com,Proxies
  - DOMAIN-SUFFIX,globalvoices.org,Proxies
  - DOMAIN-SUFFIX,goo.ne.jp,Proxies
  - DOMAIN-SUFFIX,goodreads.com,Proxies
  - DOMAIN-SUFFIX,gov,Proxies
  - DOMAIN-SUFFIX,gov.tw,Proxies
  - DOMAIN-SUFFIX,greatfire.org,Proxies
  - DOMAIN-SUFFIX,gumroad.com,Proxies
  - DOMAIN-SUFFIX,hbg.com,Proxies
  - DOMAIN-SUFFIX,heroku.com,Proxies
  - DOMAIN-SUFFIX,hightail.com,Proxies
  - DOMAIN-SUFFIX,hk01.com,Proxies
  - DOMAIN-SUFFIX,hkbf.org,Proxies
  - DOMAIN-SUFFIX,hkbookcity.com,Proxies
  - DOMAIN-SUFFIX,hkej.com,Proxies
  - DOMAIN-SUFFIX,hket.com,Proxies
  - DOMAIN-SUFFIX,hkgolden.com,Proxies
  - DOMAIN-SUFFIX,hootsuite.com,Proxies
  - DOMAIN-SUFFIX,hudson.org,Proxies
  - DOMAIN-SUFFIX,hyread.com.tw,Proxies
  - DOMAIN-SUFFIX,ibtimes.com,Proxies
  - DOMAIN-SUFFIX,i-cable.com,Proxies
  - DOMAIN-SUFFIX,icij.org,Proxies
  - DOMAIN-SUFFIX,icoco.com,Proxies
  - DOMAIN-SUFFIX,imgur.com,Proxies
  - DOMAIN-SUFFIX,initiummall.com,Proxies
  - DOMAIN-SUFFIX,insecam.org,Proxies
  - DOMAIN-SUFFIX,ipfs.io,Proxies
  - DOMAIN-SUFFIX,issuu.com,Proxies
  - DOMAIN-SUFFIX,istockphoto.com,Proxies
  - DOMAIN-SUFFIX,japantimes.co.jp,Proxies
  - DOMAIN-SUFFIX,jiji.com,Proxies
  - DOMAIN-SUFFIX,jinx.com,Proxies
  - DOMAIN-SUFFIX,jkforum.net,Proxies
  - DOMAIN-SUFFIX,joinmastodon.org,Proxies
  - DOMAIN-SUFFIX,justmysocks.net,Proxies
  - DOMAIN-SUFFIX,justpaste.it,Proxies
  - DOMAIN-SUFFIX,kakao.com,Proxies
  - DOMAIN-SUFFIX,kakaocorp.com,Proxies
  - DOMAIN-SUFFIX,kik.com,Proxies
  - DOMAIN-SUFFIX,kobo.com,Proxies
  - DOMAIN-SUFFIX,kobobooks.com,Proxies
  - DOMAIN-SUFFIX,kodingen.com,Proxies
  - DOMAIN-SUFFIX,lemonde.fr,Proxies
  - DOMAIN-SUFFIX,lepoint.fr,Proxies
  - DOMAIN-SUFFIX,lihkg.com,Proxies
  - DOMAIN-SUFFIX,listennotes.com,Proxies
  - DOMAIN-SUFFIX,livestream.com,Proxies
  - DOMAIN-SUFFIX,logmein.com,Proxies
  - DOMAIN-SUFFIX,mail.ru,Proxies
  - DOMAIN-SUFFIX,mailchimp.com,Proxies
  - DOMAIN-SUFFIX,marc.info,Proxies
  - DOMAIN-SUFFIX,matters.news,Proxies
  - DOMAIN-SUFFIX,maying.co,Proxies
  - DOMAIN-SUFFIX,medium.com,Proxies
  - DOMAIN-SUFFIX,mega.nz,Proxies
  - DOMAIN-SUFFIX,mil,Proxies
  - DOMAIN-SUFFIX,mingpao.com,Proxies
  - DOMAIN-SUFFIX,mobile01.com,Proxies
  - DOMAIN-SUFFIX,myspace.com,Proxies
  - DOMAIN-SUFFIX,myspacecdn.com,Proxies
  - DOMAIN-SUFFIX,nanyang.com,Proxies
  - DOMAIN-SUFFIX,naver.com,Proxies
  - DOMAIN-SUFFIX,neowin.net,Proxies
  - DOMAIN-SUFFIX,newstapa.org,Proxies
  - DOMAIN-SUFFIX,nexitally.com,Proxies
  - DOMAIN-SUFFIX,nhk.or.jp,Proxies
  - DOMAIN-SUFFIX,nicovideo.jp,Proxies
  - DOMAIN-SUFFIX,nii.ac.jp,Proxies
  - DOMAIN-SUFFIX,nikkei.com,Proxies
  - DOMAIN-SUFFIX,nofile.io,Proxies
  - DOMAIN-SUFFIX,now.com,Proxies
  - DOMAIN-SUFFIX,nrk.no,Proxies
  - DOMAIN-SUFFIX,nyt.com,Proxies
  - DOMAIN-SUFFIX,nytchina.com,Proxies
  - DOMAIN-SUFFIX,nytcn.me,Proxies
  - DOMAIN-SUFFIX,nytco.com,Proxies
  - DOMAIN-SUFFIX,nytimes.com,Proxies
  - DOMAIN-SUFFIX,nytimg.com,Proxies
  - DOMAIN-SUFFIX,nytlog.com,Proxies
  - DOMAIN-SUFFIX,nytstyle.com,Proxies
  - DOMAIN-SUFFIX,ok.ru,Proxies
  - DOMAIN-SUFFIX,okex.com,Proxies
  - DOMAIN-SUFFIX,on.cc,Proxies
  - DOMAIN-SUFFIX,orientaldaily.com.my,Proxies
  - DOMAIN-SUFFIX,overcast.fm,Proxies
  - DOMAIN-SUFFIX,paltalk.com,Proxies
  - DOMAIN-SUFFIX,pao-pao.net,Proxies
  - DOMAIN-SUFFIX,parsevideo.com,Proxies
  - DOMAIN-SUFFIX,pbxes.com,Proxies
  - DOMAIN-SUFFIX,pcdvd.com.tw,Proxies
  - DOMAIN-SUFFIX,pchome.com.tw,Proxies
  - DOMAIN-SUFFIX,pcloud.com,Proxies
  - DOMAIN-SUFFIX,picacomic.com,Proxies
  - DOMAIN-SUFFIX,pinimg.com,Proxies
  - DOMAIN-SUFFIX,pixiv.net,Proxies
  - DOMAIN-SUFFIX,player.fm,Proxies
  - DOMAIN-SUFFIX,plurk.com,Proxies
  - DOMAIN-SUFFIX,po18.tw,Proxies
  - DOMAIN-SUFFIX,potato.im,Proxies
  - DOMAIN-SUFFIX,potatso.com,Proxies
  - DOMAIN-SUFFIX,prism-break.org,Proxies
  - DOMAIN-SUFFIX,proxifier.com,Proxies
  - DOMAIN-SUFFIX,pt.im,Proxies
  - DOMAIN-SUFFIX,pts.org.tw,Proxies
  - DOMAIN-SUFFIX,pubu.com.tw,Proxies
  - DOMAIN-SUFFIX,pubu.tw,Proxies
  - DOMAIN-SUFFIX,pureapk.com,Proxies
  - DOMAIN-SUFFIX,quora.com,Proxies
  - DOMAIN-SUFFIX,quoracdn.net,Proxies
  - DOMAIN-SUFFIX,rakuten.co.jp,Proxies
  - DOMAIN-SUFFIX,readingtimes.com.tw,Proxies
  - DOMAIN-SUFFIX,readmoo.com,Proxies
  - DOMAIN-SUFFIX,redbubble.com,Proxies
  - DOMAIN-SUFFIX,reddit.com,Proxies
  - DOMAIN-SUFFIX,redditmedia.com,Proxies
  - DOMAIN-SUFFIX,resilio.com,Proxies
  - DOMAIN-SUFFIX,reuters.com,Proxies
  - DOMAIN-SUFFIX,reutersmedia.net,Proxies
  - DOMAIN-SUFFIX,rfi.fr,Proxies
  - DOMAIN-SUFFIX,rixcloud.com,Proxies
  - DOMAIN-SUFFIX,roadshow.hk,Proxies
  - DOMAIN-SUFFIX,scmp.com,Proxies
  - DOMAIN-SUFFIX,scribd.com,Proxies
  - DOMAIN-SUFFIX,seatguru.com,Proxies
  - DOMAIN-SUFFIX,shadowsocks.org,Proxies
  - DOMAIN-SUFFIX,shopee.tw,Proxies
  - DOMAIN-SUFFIX,slideshare.net,Proxies
  - DOMAIN-SUFFIX,softfamous.com,Proxies
  - DOMAIN-SUFFIX,soundcloud.com,Proxies
  - DOMAIN-SUFFIX,ssrcloud.org,Proxies
  - DOMAIN-SUFFIX,startpage.com,Proxies
  - DOMAIN-SUFFIX,steamcommunity.com,Proxies
  - DOMAIN-SUFFIX,steemit.com,Proxies
  - DOMAIN-SUFFIX,steemitwallet.com,Proxies
  - DOMAIN-SUFFIX,t66y.com,Proxies
  - DOMAIN-SUFFIX,tapatalk.com,Proxies
  - DOMAIN-SUFFIX,teco-hk.org,Proxies
  - DOMAIN-SUFFIX,teco-mo.org,Proxies
  - DOMAIN-SUFFIX,teddysun.com,Proxies
  - DOMAIN-SUFFIX,textnow.me,Proxies
  - DOMAIN-SUFFIX,theguardian.com,Proxies
  - DOMAIN-SUFFIX,theinitium.com,Proxies
  - DOMAIN-SUFFIX,thetvdb.com,Proxies
  - DOMAIN-SUFFIX,tineye.com,Proxies
  - DOMAIN-SUFFIX,torproject.org,Proxies
  - DOMAIN-SUFFIX,tumblr.com,Proxies
  - DOMAIN-SUFFIX,turbobit.net,Proxies
  - DOMAIN-SUFFIX,tutanota.com,Proxies
  - DOMAIN-SUFFIX,tvboxnow.com,Proxies
  - DOMAIN-SUFFIX,udn.com,Proxies
  - DOMAIN-SUFFIX,unseen.is,Proxies
  - DOMAIN-SUFFIX,upmedia.mg,Proxies
  - DOMAIN-SUFFIX,uptodown.com,Proxies
  - DOMAIN-SUFFIX,urbandictionary.com,Proxies
  - DOMAIN-SUFFIX,ustream.tv,Proxies
  - DOMAIN-SUFFIX,uwants.com,Proxies
  - DOMAIN-SUFFIX,v2ray.com,Proxies
  - DOMAIN-SUFFIX,viber.com,Proxies
  - DOMAIN-SUFFIX,videopress.com,Proxies
  - DOMAIN-SUFFIX,vimeo.com,Proxies
  - DOMAIN-SUFFIX,voachinese.com,Proxies
  - DOMAIN-SUFFIX,voanews.com,Proxies
  - DOMAIN-SUFFIX,voxer.com,Proxies
  - DOMAIN-SUFFIX,vzw.com,Proxies
  - DOMAIN-SUFFIX,w3schools.com,Proxies
  - DOMAIN-SUFFIX,washingtonpost.com,Proxies
  - DOMAIN-SUFFIX,wattpad.com,Proxies
  - DOMAIN-SUFFIX,whoer.net,Proxies
  - DOMAIN-SUFFIX,wikimapia.org,Proxies
  - DOMAIN-SUFFIX,wikipedia.org,Proxies
  - DOMAIN-SUFFIX,wikiquote.org,Proxies
  - DOMAIN-SUFFIX,wikiwand.com,Proxies
  - DOMAIN-SUFFIX,winudf.com,Proxies
  - DOMAIN-SUFFIX,wire.com,Proxies
  - DOMAIN-SUFFIX,wordpress.com,Proxies
  - DOMAIN-SUFFIX,workflow.is,Proxies
  - DOMAIN-SUFFIX,worldcat.org,Proxies
  - DOMAIN-SUFFIX,wsj.com,Proxies
  - DOMAIN-SUFFIX,wsj.net,Proxies
  - DOMAIN-SUFFIX,xhamster.com,Proxies
  - DOMAIN-SUFFIX,xn--90wwvt03e.com,Proxies
  - DOMAIN-SUFFIX,xn--i2ru8q2qg.com,Proxies
  - DOMAIN-SUFFIX,xnxx.com,Proxies
  - DOMAIN-SUFFIX,xvideos.com,Proxies
  - DOMAIN-SUFFIX,yahoo.com,Proxies
  - DOMAIN-SUFFIX,yandex.ru,Proxies
  - DOMAIN-SUFFIX,ycombinator.com,Proxies
  - DOMAIN-SUFFIX,yesasia.com,Proxies
  - DOMAIN-SUFFIX,yes-news.com,Proxies
  - DOMAIN-SUFFIX,yomiuri.co.jp,Proxies
  - DOMAIN-SUFFIX,you-get.org,Proxies
  - DOMAIN-SUFFIX,zaobao.com,Proxies
  - DOMAIN-SUFFIX,zb.com,Proxies
  - DOMAIN-SUFFIX,zello.com,Proxies
  - DOMAIN-SUFFIX,zeronet.io,Proxies
  - DOMAIN-SUFFIX,zoom.us,Proxies
  - DOMAIN-KEYWORD,github,Proxies
  - DOMAIN-KEYWORD,jav,Proxies
  - DOMAIN-KEYWORD,pinterest,Proxies
  - DOMAIN-KEYWORD,porn,Proxies
  - DOMAIN-KEYWORD,wikileaks,Proxies
  - DOMAIN-SUFFIX,apartmentratings.com,Proxies
  - DOMAIN-SUFFIX,apartments.com,Proxies
  - DOMAIN-SUFFIX,bankmobilevibe.com,Proxies
  - DOMAIN-SUFFIX,bing.com,Proxies
  - DOMAIN-SUFFIX,booktopia.com.au,Proxies
  - DOMAIN-SUFFIX,cccat.io,Proxies
  - DOMAIN-SUFFIX,centauro.com.br,Proxies
  - DOMAIN-SUFFIX,clearsurance.com,Proxies
  - DOMAIN-SUFFIX,costco.com,Proxies
  - DOMAIN-SUFFIX,crackle.com,Proxies
  - DOMAIN-SUFFIX,depositphotos.cn,Proxies
  - DOMAIN-SUFFIX,dish.com,Proxies
  - DOMAIN-SUFFIX,dmm.co.jp,Proxies
  - DOMAIN-SUFFIX,dmm.com,Proxies
  - DOMAIN-SUFFIX,dnvod.tv,Proxies
  - DOMAIN-SUFFIX,esurance.com,Proxies
  - DOMAIN-SUFFIX,extmatrix.com,Proxies
  - DOMAIN-SUFFIX,fastpic.ru,Proxies
  - DOMAIN-SUFFIX,flipboard.com,Proxies
  - DOMAIN-SUFFIX,fnac.be,Proxies
  - DOMAIN-SUFFIX,fnac.com,Proxies
  - DOMAIN-SUFFIX,funkyimg.com,Proxies
  - DOMAIN-SUFFIX,fxnetworks.com,Proxies
  - DOMAIN-SUFFIX,gettyimages.com,Proxies
  - DOMAIN-SUFFIX,go.com,Proxies
  - DOMAIN-SUFFIX,here.com,Proxies
  - DOMAIN-SUFFIX,jcpenney.com,Proxies
  - DOMAIN-SUFFIX,jiehua.tv,Proxies
  - DOMAIN-SUFFIX,mailfence.com,Proxies
  - DOMAIN-SUFFIX,nationwide.com,Proxies
  - DOMAIN-SUFFIX,nbc.com,Proxies
  - DOMAIN-SUFFIX,nexon.com,Proxies
  - DOMAIN-SUFFIX,nordstrom.com,Proxies
  - DOMAIN-SUFFIX,nordstromimage.com,Proxies
  - DOMAIN-SUFFIX,nordstromrack.com,Proxies
  - DOMAIN-SUFFIX,superpages.com,Proxies
  - DOMAIN-SUFFIX,target.com,Proxies
  - DOMAIN-SUFFIX,thinkgeek.com,Proxies
  - DOMAIN-SUFFIX,tracfone.com,Proxies
  - DOMAIN-SUFFIX,unity3d.com,Proxies
  - DOMAIN-SUFFIX,uploader.jp,Proxies
  - DOMAIN-SUFFIX,vevo.com,Proxies
  - DOMAIN-SUFFIX,viu.tv,Proxies
  - DOMAIN-SUFFIX,vk.com,Proxies
  - DOMAIN-SUFFIX,vsco.co,Proxies
  - DOMAIN-SUFFIX,xfinity.com,Proxies
  - DOMAIN-SUFFIX,zattoo.com,Proxies
  - DOMAIN,testflight.apple.com,Proxies
  - DOMAIN-SUFFIX,appsto.re,Proxies
  - DOMAIN,books.itunes.apple.com,Proxies
  - DOMAIN,hls.itunes.apple.com,Proxies
  - DOMAIN,apps.apple.com,Proxies
  - DOMAIN,itunes.apple.com,Proxies
  - DOMAIN,api-glb-sea.smoot.apple.com,Proxies
  - DOMAIN,lookup-api.apple.com,Proxies
  - DOMAIN-SUFFIX,abc.xyz,Proxies
  - DOMAIN-SUFFIX,android.com,Proxies
  - DOMAIN-SUFFIX,androidify.com,Proxies
  - DOMAIN-SUFFIX,dialogflow.com,Proxies
  - DOMAIN-SUFFIX,autodraw.com,Proxies
  - DOMAIN-SUFFIX,capitalg.com,Proxies
  - DOMAIN-SUFFIX,certificate-transparency.org,Proxies
  - DOMAIN-SUFFIX,chrome.com,Proxies
  - DOMAIN-SUFFIX,chromeexperiments.com,Proxies
  - DOMAIN-SUFFIX,chromestatus.com,Proxies
  - DOMAIN-SUFFIX,chromium.org,Proxies
  - DOMAIN-SUFFIX,creativelab5.com,Proxies
  - DOMAIN-SUFFIX,debug.com,Proxies
  - DOMAIN-SUFFIX,deepmind.com,Proxies
  - DOMAIN-SUFFIX,firebaseio.com,Proxies
  - DOMAIN-SUFFIX,getmdl.io,Proxies
  - DOMAIN-SUFFIX,ggpht.com,Proxies
  - DOMAIN-SUFFIX,gmail.com,Proxies
  - DOMAIN-SUFFIX,gmodules.com,Proxies
  - DOMAIN-SUFFIX,godoc.org,Proxies
  - DOMAIN-SUFFIX,golang.org,Proxies
  - DOMAIN-SUFFIX,gstatic.com,Proxies
  - DOMAIN-SUFFIX,gv.com,Proxies
  - DOMAIN-SUFFIX,gwtproject.org,Proxies
  - DOMAIN-SUFFIX,itasoftware.com,Proxies
  - DOMAIN-SUFFIX,madewithcode.com,Proxies
  - DOMAIN-SUFFIX,material.io,Proxies
  - DOMAIN-SUFFIX,polymer-project.org,Proxies
  - DOMAIN-SUFFIX,admin.recaptcha.net,Proxies
  - DOMAIN-SUFFIX,recaptcha.net,Proxies
  - DOMAIN-SUFFIX,shattered.io,Proxies
  - DOMAIN-SUFFIX,synergyse.com,Proxies
  - DOMAIN-SUFFIX,tensorflow.org,Proxies
  - DOMAIN-SUFFIX,tfhub.dev,Proxies
  - DOMAIN-SUFFIX,tiltbrush.com,Proxies
  - DOMAIN-SUFFIX,waveprotocol.org,Proxies
  - DOMAIN-SUFFIX,waymo.com,Proxies
  - DOMAIN-SUFFIX,webmproject.org,Proxies
  - DOMAIN-SUFFIX,webrtc.org,Proxies
  - DOMAIN-SUFFIX,whatbrowser.org,Proxies
  - DOMAIN-SUFFIX,widevine.com,Proxies
  - DOMAIN-SUFFIX,x.company,Proxies
  - DOMAIN-SUFFIX,youtu.be,Proxies
  - DOMAIN-SUFFIX,yt.be,Proxies
  - DOMAIN-SUFFIX,ytimg.com,Proxies
  - DOMAIN-SUFFIX,1drv.com,Proxies
  - DOMAIN-SUFFIX,1drv.ms,Proxies
  - DOMAIN-SUFFIX,blob.core.windows.net,Proxies
  - DOMAIN-SUFFIX,livefilestore.com,Proxies
  - DOMAIN-SUFFIX,onedrive.com,Proxies
  - DOMAIN-SUFFIX,storage.live.com,Proxies
  - DOMAIN-SUFFIX,storage.msn.com,Proxies
  - DOMAIN,oneclient.sfx.ms,Proxies
  - DOMAIN-SUFFIX,0rz.tw,Proxies
  - DOMAIN-SUFFIX,4bluestones.biz,Proxies
  - DOMAIN-SUFFIX,9bis.net,Proxies
  - DOMAIN-SUFFIX,allconnected.co,Proxies
  - DOMAIN-SUFFIX,aol.com,Proxies
  - DOMAIN-SUFFIX,bcc.com.tw,Proxies
  - DOMAIN-SUFFIX,bit.ly,Proxies
  - DOMAIN-SUFFIX,bitshare.com,Proxies
  - DOMAIN-SUFFIX,blog.jp,Proxies
  - DOMAIN-SUFFIX,blogimg.jp,Proxies
  - DOMAIN-SUFFIX,blogtd.org,Proxies
  - DOMAIN-SUFFIX,broadcast.co.nz,Proxies
  - DOMAIN-SUFFIX,camfrog.com,Proxies
  - DOMAIN-SUFFIX,cfos.de,Proxies
  - DOMAIN-SUFFIX,citypopulation.de,Proxies
  - DOMAIN-SUFFIX,cloudfront.net,Proxies
  - DOMAIN-SUFFIX,ctitv.com.tw,Proxies
  - DOMAIN-SUFFIX,cuhk.edu.hk,Proxies
  - DOMAIN-SUFFIX,cusu.hk,Proxies
  - DOMAIN-SUFFIX,discord.gg,Proxies
  - DOMAIN-SUFFIX,discuss.com.hk,Proxies
  - DOMAIN-SUFFIX,dropboxapi.com,Proxies
  - DOMAIN-SUFFIX,duolingo.cn,Proxies
  - DOMAIN-SUFFIX,edditstatic.com,Proxies
  - DOMAIN-SUFFIX,flickriver.com,Proxies
  - DOMAIN-SUFFIX,focustaiwan.tw,Proxies
  - DOMAIN-SUFFIX,free.fr,Proxies
  - DOMAIN-SUFFIX,gigacircle.com,Proxies
  - DOMAIN-SUFFIX,hk-pub.com,Proxies
  - DOMAIN-SUFFIX,hosting.co.uk,Proxies
  - DOMAIN-SUFFIX,hwcdn.net,Proxies
  - DOMAIN-SUFFIX,ifixit.com,Proxies
  - DOMAIN-SUFFIX,iphone4hongkong.com,Proxies
  - DOMAIN-SUFFIX,iphonetaiwan.org,Proxies
  - DOMAIN-SUFFIX,iptvbin.com,Proxies
  - DOMAIN-SUFFIX,linksalpha.com,Proxies
  - DOMAIN-SUFFIX,manyvids.com,Proxies
  - DOMAIN-SUFFIX,myactimes.com,Proxies
  - DOMAIN-SUFFIX,newsblur.com,Proxies
  - DOMAIN-SUFFIX,now.im,Proxies
  - DOMAIN-SUFFIX,nowe.com,Proxies
  - DOMAIN-SUFFIX,redditlist.com,Proxies
  - DOMAIN-SUFFIX,s3.amazonaws.com,Proxies
  - DOMAIN-SUFFIX,signal.org,Proxies
  - DOMAIN-SUFFIX,smartmailcloud.com,Proxies
  - DOMAIN-SUFFIX,sparknotes.com,Proxies
  - DOMAIN-SUFFIX,streetvoice.com,Proxies
  - DOMAIN-SUFFIX,supertop.co,Proxies
  - DOMAIN-SUFFIX,tv.com,Proxies
  - DOMAIN-SUFFIX,typepad.com,Proxies
  - DOMAIN-SUFFIX,udnbkk.com,Proxies
  - DOMAIN-SUFFIX,urbanairship.com,Proxies
  - DOMAIN-SUFFIX,whispersystems.org,Proxies
  - DOMAIN-SUFFIX,wikia.com,Proxies
  - DOMAIN-SUFFIX,wn.com,Proxies
  - DOMAIN-SUFFIX,wolframalpha.com,Proxies
  - DOMAIN-SUFFIX,x-art.com,Proxies
  - DOMAIN-SUFFIX,yimg.com,Proxies
  - DOMAIN,api.steampowered.com,Proxies
  - DOMAIN,store.steampowered.com,StreamSE

  # Apple
  - DOMAIN-SUFFIX,aaplimg.com,DIRECT
  - DOMAIN-SUFFIX,apple.co,DIRECT
  - DOMAIN-SUFFIX,apple.com,DIRECT
  - DOMAIN-SUFFIX,apple-cloudkit.com,DIRECT
  - DOMAIN-SUFFIX,appstore.com,DIRECT
  - DOMAIN-SUFFIX,cdn-apple.com,DIRECT
  - DOMAIN-SUFFIX,crashlytics.com,DIRECT
  - DOMAIN-SUFFIX,icloud.com,DIRECT
  - DOMAIN-SUFFIX,icloud-content.com,DIRECT
  - DOMAIN-SUFFIX,me.com,DIRECT
  - DOMAIN-SUFFIX,mzstatic.com,DIRECT
  - DOMAIN,www-cdn.icloud.com.akadns.net,DIRECT

  - DOMAIN-SUFFIX,t.me,Telegram
  - DOMAIN-SUFFIX,tdesktop.com,Telegram
  - DOMAIN-SUFFIX,telegra.ph,Telegram
  - DOMAIN-SUFFIX,telegram.me,Telegram
  - DOMAIN-SUFFIX,telegram.org,Telegram
  - IP-CIDR,91.108.4.0/22,Telegram,no-resolve
  - IP-CIDR,91.108.8.0/22,Telegram,no-resolve
  - IP-CIDR,91.108.12.0/22,Telegram,no-resolve
  - IP-CIDR,91.108.16.0/22,Telegram,no-resolve
  - IP-CIDR,91.108.56.0/22,Telegram,no-resolve
  - IP-CIDR,149.154.160.0/20,Telegram,no-resolve
  - IP-CIDR6,2001:b28:f23d::/48,Telegram,no-resolve
  - IP-CIDR6,2001:b28:f23f::/48,Telegram,no-resolve
  - IP-CIDR6,2001:67c:4e8::/48,Telegram,no-resolve

  - DOMAIN-SUFFIX,googlevideo.com,StreamSE
  - DOMAIN-SUFFIX,youtube.com,StreamSE
  - DOMAIN,youtubei.googleapis.com,StreamSE

  - DOMAIN-SUFFIX,netflix.com,StreamSE
  - DOMAIN-SUFFIX,netflix.net,StreamSE
  - DOMAIN-SUFFIX,nflxext.com,StreamSE
  - DOMAIN-SUFFIX,nflximg.com,StreamSE
  - DOMAIN-SUFFIX,nflximg.net,StreamSE
  - DOMAIN-SUFFIX,nflxso.net,StreamSE
  - DOMAIN-SUFFIX,nflxvideo.net,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest0.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest1.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest2.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest3.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest4.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest5.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest6.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest7.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest8.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest9.com,StreamSE
  - IP-CIDR,23.246.0.0/18,StreamSE,no-resolve
  - IP-CIDR,37.77.184.0/21,StreamSE,no-resolve
  - IP-CIDR,45.57.0.0/17,StreamSE,no-resolve
  - IP-CIDR,64.120.128.0/17,StreamSE,no-resolve
  - IP-CIDR,66.197.128.0/17,StreamSE,no-resolve
  - IP-CIDR,108.175.32.0/20,StreamSE,no-resolve
  - IP-CIDR,192.173.64.0/18,StreamSE,no-resolve
  - IP-CIDR,198.38.96.0/19,StreamSE,no-resolve
  - IP-CIDR,198.45.48.0/20,StreamSE,no-resolve

  # StreamSE
  - DOMAIN-SUFFIX,deezer.com,StreamSE
  - DOMAIN-SUFFIX,dzcdn.net,StreamSE
  - DOMAIN-SUFFIX,kkbox.com,StreamSE
  - DOMAIN-SUFFIX,kkbox.com.tw,StreamSE
  - DOMAIN-SUFFIX,kfs.io,StreamSE
  - DOMAIN-SUFFIX,joox.com,StreamSE
  - DOMAIN-SUFFIX,pandora.com,StreamSE
  - DOMAIN-SUFFIX,p-cdn.us,StreamSE
  - DOMAIN-SUFFIX,sndcdn.com,StreamSE
  - DOMAIN-SUFFIX,soundcloud.com,StreamSE
  - DOMAIN-SUFFIX,pscdn.co,StreamSE
  - DOMAIN-SUFFIX,scdn.co,StreamSE
  - DOMAIN-SUFFIX,spotify.com,StreamSE
  - DOMAIN-SUFFIX,spoti.fi,StreamSE
  - DOMAIN-KEYWORD,spotify.com,StreamSE
  - DOMAIN-KEYWORD,-spotify-com,StreamSE
  - DOMAIN-SUFFIX,tidal.com,StreamSE
  - DOMAIN-SUFFIX,c4assets.com,StreamSE
  - DOMAIN-SUFFIX,channel4.com,Proxies
  - DOMAIN-SUFFIX,abema.io,StreamSE
  - DOMAIN-SUFFIX,ameba.jp,StreamSE
  - DOMAIN-SUFFIX,abema.tv,StreamSE
  - DOMAIN-SUFFIX,hayabusa.io,StreamSE
  - DOMAIN,abematv.akamaized.net,Proxies
  - DOMAIN,ds-linear-abematv.akamaized.net,StreamSE
  - DOMAIN,ds-vod-abematv.akamaized.net,StreamSE
  - DOMAIN,linear-abematv.akamaized.net,StreamSE
  - DOMAIN-SUFFIX,aiv-cdn.net,StreamSE
  - DOMAIN-SUFFIX,aiv-delivery.net,StreamSE
  - DOMAIN-SUFFIX,amazonvideo.com,StreamSE
  - DOMAIN-SUFFIX,primevideo.com,StreamSE
  - DOMAIN,avodmp4s3ww-a.akamaihd.net,StreamSE
  - DOMAIN,d25xi40x97liuc.cloudfront.net,Proxies
  - DOMAIN,dmqdd6hw24ucf.cloudfront.net,Proxies
  - DOMAIN,d22qjgkvxw22r6.cloudfront.net,StreamSE
  - DOMAIN,d1v5ir2lpwr8os.cloudfront.net,StreamSE
  - DOMAIN-KEYWORD,avoddashs,StreamSE
  - DOMAIN-SUFFIX,bahamut.com.tw,StreamSE
  - DOMAIN-SUFFIX,gamer.com.tw,StreamSE
  - DOMAIN,gamer-cds.cdn.hinet.net,StreamSE
  - DOMAIN,gamer2-cds.cdn.hinet.net,StreamSE
  - DOMAIN-SUFFIX,bbc.co.uk,StreamSE
  - DOMAIN-SUFFIX,bbci.co.uk,StreamSE
  - DOMAIN-KEYWORD,bbcfmt,StreamSE
  - DOMAIN-KEYWORD,uk-live,StreamSE
  - DOMAIN-SUFFIX,dazn.com,StreamSE
  - DOMAIN-SUFFIX,dazn-api.com,StreamSE
  - DOMAIN,d151l6v8er5bdm.cloudfront.net,StreamSE
  - DOMAIN-KEYWORD,voddazn,StreamSE
  - DOMAIN-SUFFIX,bamgrid.com,StreamSE
  - DOMAIN-SUFFIX,disney-plus.net,StreamSE
  - DOMAIN-SUFFIX,disneyplus.com,StreamSE
  - DOMAIN-SUFFIX,dssott.com,StreamSE
  - DOMAIN,cdn.registerdisney.go.com,StreamSE
  - DOMAIN-SUFFIX,encoretvb.com,StreamSE
  - DOMAIN,edge.api.brightcove.com,StreamSE
  - DOMAIN,bcbolt446c5271-a.akamaihd.net,StreamSE
  - DOMAIN-SUFFIX,fox.com,StreamSE
  - DOMAIN-SUFFIX,foxdcg.com,StreamSE
  - DOMAIN-SUFFIX,theplatform.com,StreamSE
  - DOMAIN-SUFFIX,uplynk.com,StreamSE
  - DOMAIN-SUFFIX,hbo.com,StreamSE
  - DOMAIN-SUFFIX,hbogo.com,StreamSE
  - DOMAIN-SUFFIX,hbonow.com,StreamSE
  - DOMAIN-SUFFIX,hbogoasia.com,StreamSE
  - DOMAIN-SUFFIX,hbogoasia.hk,StreamSE
  - DOMAIN,bcbolthboa-a.akamaihd.net,StreamSE
  - DOMAIN,players.brightcove.net,StreamSE
  - DOMAIN,s3-ap-southeast-1.amazonaws.com,StreamSE
  - DOMAIN,dai3fd1oh325y.cloudfront.net,StreamSE
  - DOMAIN,44wilhpljf.execute-api.ap-southeast-1.amazonaws.com,StreamSE
  - DOMAIN,hboasia1-i.akamaihd.net,StreamSE
  - DOMAIN,hboasia2-i.akamaihd.net,StreamSE
  - DOMAIN,hboasia3-i.akamaihd.net,StreamSE
  - DOMAIN,hboasia4-i.akamaihd.net,StreamSE
  - DOMAIN,hboasia5-i.akamaihd.net,StreamSE
  - DOMAIN,cf-images.ap-southeast-1.prod.boltdns.net,StreamSE
  - DOMAIN-SUFFIX,5itv.tv,StreamSE
  - DOMAIN-SUFFIX,ocnttv.com,StreamSE
  - DOMAIN-SUFFIX,hulu.com,StreamSE
  - DOMAIN-SUFFIX,huluim.com,StreamSE
  - DOMAIN-SUFFIX,hulustream.com,StreamSE
  - DOMAIN-SUFFIX,happyon.jp,StreamSE
  - DOMAIN-SUFFIX,hulu.jp,StreamSE
  - DOMAIN-SUFFIX,itv.com,StreamSE
  - DOMAIN-SUFFIX,itvstatic.com,StreamSE
  - DOMAIN,itvpnpmobile-a.akamaihd.net,StreamSE
  - DOMAIN-SUFFIX,kktv.com.tw,StreamSE
  - DOMAIN-SUFFIX,kktv.me,StreamSE
  - DOMAIN,kktv-theater.kk.stream,StreamSE
  - DOMAIN-SUFFIX,linetv.tw,StreamSE
  - DOMAIN,d3c7rimkq79yfu.cloudfront.net,StreamSE
  - DOMAIN-SUFFIX,litv.tv,StreamSE
  - DOMAIN,litvfreemobile-hichannel.cdn.hinet.net,StreamSE
  - DOMAIN-SUFFIX,channel5.com,StreamSE
  - DOMAIN-SUFFIX,my5.tv,StreamSE
  - DOMAIN,d349g9zuie06uo.cloudfront.net,StreamSE
  - DOMAIN-SUFFIX,mytvsuper.com,StreamSE
  - DOMAIN-SUFFIX,tvb.com,StreamSE
  - DOMAIN-SUFFIX,netflix.com,StreamSE
  - DOMAIN-SUFFIX,netflix.net,StreamSE
  - DOMAIN-SUFFIX,nflxext.com,StreamSE
  - DOMAIN-SUFFIX,nflximg.com,StreamSE
  - DOMAIN-SUFFIX,nflximg.net,StreamSE
  - DOMAIN-SUFFIX,nflxso.net,StreamSE
  - DOMAIN-SUFFIX,nflxvideo.net,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest0.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest1.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest2.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest3.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest4.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest5.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest6.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest7.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest8.com,StreamSE
  - DOMAIN-SUFFIX,netflixdnstest9.com,StreamSE
  - IP-CIDR,23.246.0.0/18,StreamSE,no-resolve
  - IP-CIDR,37.77.184.0/21,StreamSE,no-resolve
  - IP-CIDR,45.57.0.0/17,StreamSE,no-resolve
  - IP-CIDR,64.120.128.0/17,StreamSE,no-resolve
  - IP-CIDR,66.197.128.0/17,StreamSE,no-resolve
  - IP-CIDR,108.175.32.0/20,StreamSE,no-resolve
  - IP-CIDR,192.173.64.0/18,StreamSE,no-resolve
  - IP-CIDR,198.38.96.0/19,StreamSE,no-resolve
  - IP-CIDR,198.45.48.0/20,StreamSE,no-resolve
  - DOMAIN-SUFFIX,dmc.nico,StreamSE
  - DOMAIN-SUFFIX,nicovideo.jp,StreamSE
  - DOMAIN-SUFFIX,nimg.jp,StreamSE
  - DOMAIN-SUFFIX,socdm.com,StreamSE
  - DOMAIN-SUFFIX,pbs.org,StreamSE
  - DOMAIN-SUFFIX,phncdn.com,StreamSE
  - DOMAIN-SUFFIX,pornhub.com,StreamSE
  - DOMAIN-SUFFIX,pornhubpremium.com,StreamSE
  - DOMAIN-SUFFIX,skyking.com.tw,StreamSE
  - DOMAIN,hamifans.emome.net,StreamSE
  - DOMAIN-SUFFIX,twitch.tv,StreamSE
  - DOMAIN-SUFFIX,twitchcdn.net,StreamSE
  - DOMAIN-SUFFIX,ttvnw.net,StreamSE
  - DOMAIN-SUFFIX,jtvnw.net,StreamSE
  - DOMAIN-SUFFIX,viu.com,StreamSE
  - DOMAIN-SUFFIX,viu.tv,StreamSE
  - DOMAIN,api.viu.now.com,StreamSE
  - DOMAIN,d1k2us671qcoau.cloudfront.net,StreamSE
  - DOMAIN,d2anahhhmp1ffz.cloudfront.net,StreamSE
  - DOMAIN,dfp6rglgjqszk.cloudfront.net,StreamSE
  - DOMAIN-SUFFIX,googlevideo.com,StreamSE
  - DOMAIN-SUFFIX,youtube.com,StreamSE
  - DOMAIN,youtubei.googleapis.com,StreamSE

  # Bilibili
  - DOMAIN-SUFFIX,biliapi.com,DIRECT
  - DOMAIN-SUFFIX,biliapi.net,DIRECT
  - DOMAIN-SUFFIX,bilibili.com,DIRECT
  - DOMAIN-SUFFIX,bilibili.tv,DIRECT
  - DOMAIN-SUFFIX,bilivideo.com,DIRECT

  - DOMAIN-SUFFIX,local,DIRECT
  - IP-CIDR,192.168.0.0/16,DIRECT,no-resolve
  - IP-CIDR,10.0.0.0/8,DIRECT,no-resolve
  - IP-CIDR,172.16.0.0/12,DIRECT,no-resolve
  - IP-CIDR,127.0.0.0/8,DIRECT,no-resolve
  - IP-CIDR,100.64.0.0/10,DIRECT,no-resolve
  - IP-CIDR6,::1/128,DIRECT,no-resolve
  - IP-CIDR6,fc00::/7,DIRECT,no-resolve
  - IP-CIDR6,fe80::/10,DIRECT,no-resolve
  - IP-CIDR6,fd00::/8,DIRECT,no-resolve
  - DOMAIN,app.adjust.com,DIRECT
  - DOMAIN-SUFFIX,googletraveladservices.com,DIRECT
  - DOMAIN,dl.google.com,DIRECT
  - DOMAIN,mtalk.google.com,DIRECT
  - DOMAIN,livew.l.qq.com,DIRECT
  - DOMAIN,vd.l.qq.com,DIRECT
  - DOMAIN,analytics.strava.com,DIRECT
  - DOMAIN,msg.umeng.com,DIRECT
  - DOMAIN,msg.umengcloud.com,DIRECT
  - DOMAIN-SUFFIX,qhres.com,DIRECT
  - DOMAIN-SUFFIX,qhimg.com,DIRECT
  - DOMAIN-SUFFIX,akadns.net,DIRECT
  - DOMAIN-SUFFIX,alibaba.com,DIRECT
  - DOMAIN-SUFFIX,alicdn.com,DIRECT
  - DOMAIN-SUFFIX,alikunlun.com,DIRECT
  - DOMAIN-SUFFIX,alipay.com,DIRECT
  - DOMAIN-SUFFIX,amap.com,DIRECT
  - DOMAIN-SUFFIX,autonavi.com,DIRECT
  - DOMAIN-SUFFIX,dingtalk.com,DIRECT
  - DOMAIN-SUFFIX,mxhichina.com,DIRECT
  - DOMAIN-SUFFIX,soku.com,DIRECT
  - DOMAIN-SUFFIX,taobao.com,DIRECT
  - DOMAIN-SUFFIX,tmall.com,DIRECT
  - DOMAIN-SUFFIX,tmall.hk,DIRECT
  - DOMAIN-SUFFIX,ykimg.com,DIRECT
  - DOMAIN-SUFFIX,youku.com,DIRECT
  - DOMAIN-SUFFIX,xiami.com,DIRECT
  - DOMAIN-SUFFIX,xiami.net,DIRECT
  - DOMAIN-SUFFIX,aaplimg.com,DIRECT
  - DOMAIN-SUFFIX,apple.co,DIRECT
  - DOMAIN-SUFFIX,apple.com,DIRECT
  - DOMAIN-SUFFIX,apple-cloudkit.com,DIRECT
  - DOMAIN-SUFFIX,appstore.com,DIRECT
  - DOMAIN-SUFFIX,cdn-apple.com,DIRECT
  - DOMAIN-SUFFIX,crashlytics.com,DIRECT
  - DOMAIN-SUFFIX,icloud.com,DIRECT
  - DOMAIN-SUFFIX,icloud-content.com,DIRECT
  - DOMAIN-SUFFIX,me.com,DIRECT
  - DOMAIN-SUFFIX,mzstatic.com,DIRECT
  - DOMAIN,www-cdn.icloud.com.akadns.net,DIRECT
  - DOMAIN-SUFFIX,baidu.com,DIRECT
  - DOMAIN-SUFFIX,baidubcr.com,DIRECT
  - DOMAIN-SUFFIX,bdstatic.com,DIRECT
  - DOMAIN-SUFFIX,yunjiasu-cdn.net,DIRECT
  - DOMAIN-SUFFIX,acgvideo.com,DIRECT
  - DOMAIN-SUFFIX,hdslb.com,DIRECT
  - DOMAIN-SUFFIX,blizzard.com,DIRECT
  - DOMAIN-SUFFIX,battle.net,DIRECT
  - DOMAIN,blzddist1-a.akamaihd.net,DIRECT
  - DOMAIN-SUFFIX,feiliao.com,DIRECT
  - DOMAIN-SUFFIX,pstatp.com,DIRECT
  - DOMAIN-SUFFIX,snssdk.com,DIRECT
  - DOMAIN-SUFFIX,iesdouyin.com,DIRECT
  - DOMAIN-SUFFIX,toutiao.com,DIRECT
  - DOMAIN-SUFFIX,cctv.com,DIRECT
  - DOMAIN-SUFFIX,cctvpic.com,DIRECT
  - DOMAIN-SUFFIX,livechina.com,DIRECT
  - DOMAIN-SUFFIX,didialift.com,DIRECT
  - DOMAIN-SUFFIX,didiglobal.com,DIRECT
  - DOMAIN-SUFFIX,udache.com,DIRECT
  - DOMAIN-SUFFIX,343480.com,DIRECT
  - DOMAIN-SUFFIX,baduziyuan.com,DIRECT
  - DOMAIN-SUFFIX,com-hs-hkdy.com,DIRECT
  - DOMAIN-SUFFIX,czybjz.com,DIRECT
  - DOMAIN-SUFFIX,dandanzan.com,DIRECT
  - DOMAIN-SUFFIX,fjhps.com,DIRECT
  - DOMAIN-SUFFIX,kuyunbo.club,DIRECT
  - DOMAIN-SUFFIX,21cn.com,DIRECT
  - DOMAIN-SUFFIX,hitv.com,DIRECT
  - DOMAIN-SUFFIX,mgtv.com,DIRECT
  - DOMAIN-SUFFIX,iqiyi.com,DIRECT
  - DOMAIN-SUFFIX,iqiyipic.com,DIRECT
  - DOMAIN-SUFFIX,71.am.com,DIRECT
  - DOMAIN-SUFFIX,jd.com,DIRECT
  - DOMAIN-SUFFIX,jd.hk,DIRECT
  - DOMAIN-SUFFIX,jdpay.com,DIRECT
  - DOMAIN-SUFFIX,360buyimg.com,DIRECT
  - DOMAIN-SUFFIX,iciba.com,DIRECT
  - DOMAIN-SUFFIX,ksosoft.com,DIRECT
  - DOMAIN-SUFFIX,meitu.com,DIRECT
  - DOMAIN-SUFFIX,meitudata.com,DIRECT
  - DOMAIN-SUFFIX,meitustat.com,DIRECT
  - DOMAIN-SUFFIX,meipai.com,DIRECT
  - DOMAIN-SUFFIX,duokan.com,DIRECT
  - DOMAIN-SUFFIX,mi-img.com,DIRECT
  - DOMAIN-SUFFIX,miui.com,DIRECT
  - DOMAIN-SUFFIX,miwifi.com,DIRECT
  - DOMAIN-SUFFIX,xiaomi.com,DIRECT
  - DOMAIN-SUFFIX,microsoft.com,DIRECT
  - DOMAIN-SUFFIX,msecnd.net,DIRECT
  - DOMAIN-SUFFIX,office365.com,DIRECT
  - DOMAIN-SUFFIX,outlook.com,DIRECT
  - DOMAIN-SUFFIX,s-microsoft.com,DIRECT
  - DOMAIN-SUFFIX,visualstudio.com,DIRECT
  - DOMAIN-SUFFIX,windows.com,DIRECT
  - DOMAIN-SUFFIX,windowsupdate.com,DIRECT
  - DOMAIN,officecdn-microsoft-com.akamaized.net,DIRECT
  - DOMAIN-SUFFIX,163.com,DIRECT
  - DOMAIN-SUFFIX,126.net,DIRECT
  - DOMAIN-SUFFIX,127.net,DIRECT
  - DOMAIN-SUFFIX,163yun.com,DIRECT
  - DOMAIN-SUFFIX,lofter.com,DIRECT
  - DOMAIN-SUFFIX,netease.com,DIRECT
  - DOMAIN-SUFFIX,ydstatic.com,DIRECT
  - DOMAIN-SUFFIX,sina.com,DIRECT
  - DOMAIN-SUFFIX,weibo.com,DIRECT
  - DOMAIN-SUFFIX,weibocdn.com,DIRECT
  - DOMAIN-SUFFIX,sohu.com,DIRECT
  - DOMAIN-SUFFIX,sohucs.com,DIRECT
  - DOMAIN-SUFFIX,sohu-inc.com,DIRECT
  - DOMAIN-SUFFIX,v-56.com,DIRECT
  - DOMAIN-SUFFIX,sogo.com,DIRECT
  - DOMAIN-SUFFIX,sogou.com,DIRECT
  - DOMAIN-SUFFIX,sogoucdn.com,DIRECT
  - DOMAIN-SUFFIX,steampowered.com,DIRECT
  - DOMAIN-SUFFIX,steam-chat.com,DIRECT
  - DOMAIN-SUFFIX,steamgames.com,DIRECT
  - DOMAIN-SUFFIX,steamusercontent.com,DIRECT
  - DOMAIN-SUFFIX,steamcontent.com,DIRECT
  - DOMAIN-SUFFIX,steamstatic.com,DIRECT
  - DOMAIN-SUFFIX,steamcdn-a.akamaihd.net,DIRECT
  - DOMAIN-SUFFIX,steamstat.us,DIRECT
  - DOMAIN-SUFFIX,gtimg.com,DIRECT
  - DOMAIN-SUFFIX,idqqimg.com,DIRECT
  - DOMAIN-SUFFIX,igamecj.com,DIRECT
  - DOMAIN-SUFFIX,myapp.com,DIRECT
  - DOMAIN-SUFFIX,myqcloud.com,DIRECT
  - DOMAIN-SUFFIX,qq.com,DIRECT
  - DOMAIN-SUFFIX,tencent.com,DIRECT
  - DOMAIN-SUFFIX,tencent-cloud.net,DIRECT
  - DOMAIN-SUFFIX,jstucdn.com,DIRECT
  - DOMAIN-SUFFIX,zimuzu.io,DIRECT
  - DOMAIN-SUFFIX,zimuzu.tv,DIRECT
  - DOMAIN-SUFFIX,zmz2019.com,DIRECT
  - DOMAIN-SUFFIX,zmzapi.com,DIRECT
  - DOMAIN-SUFFIX,zmzapi.net,DIRECT
  - DOMAIN-SUFFIX,zmzfile.com,DIRECT
  - DOMAIN-SUFFIX,ccgslb.com,DIRECT
  - DOMAIN-SUFFIX,ccgslb.net,DIRECT
  - DOMAIN-SUFFIX,chinanetcenter.com,DIRECT
  - DOMAIN-SUFFIX,meixincdn.com,DIRECT
  - DOMAIN-SUFFIX,ourdvs.com,DIRECT
  - DOMAIN-SUFFIX,staticdn.net,DIRECT
  - DOMAIN-SUFFIX,wangsu.com,DIRECT
  - DOMAIN-SUFFIX,ipip.net,DIRECT
  - DOMAIN-SUFFIX,ip.la,DIRECT
  - DOMAIN-SUFFIX,ip-cdn.com,DIRECT
  - DOMAIN-SUFFIX,ipv6-test.com,DIRECT
  - DOMAIN-SUFFIX,test-ipv6.com,DIRECT
  - DOMAIN-SUFFIX,whatismyip.com,DIRECT
  - DOMAIN-SUFFIX,netspeedtestmaster.com,DIRECT
  - DOMAIN,speedtest.macpaw.com,DIRECT
  - DOMAIN-SUFFIX,awesome-hd.me,DIRECT
  - DOMAIN-SUFFIX,broadcasthe.net,DIRECT
  - DOMAIN-SUFFIX,chdbits.co,DIRECT
  - DOMAIN-SUFFIX,classix-unlimited.co.uk,DIRECT
  - DOMAIN-SUFFIX,empornium.me,DIRECT
  - DOMAIN-SUFFIX,gazellegames.net,DIRECT
  - DOMAIN-SUFFIX,hdchina.org,DIRECT
  - DOMAIN-SUFFIX,hdsky.me,DIRECT
  - DOMAIN-SUFFIX,icetorrent.org,DIRECT
  - DOMAIN-SUFFIX,jpopsuki.eu,DIRECT
  - DOMAIN-SUFFIX,keepfrds.com,DIRECT
  - DOMAIN-SUFFIX,madsrevolution.net,DIRECT
  - DOMAIN-SUFFIX,m-team.cc,DIRECT
  - DOMAIN-SUFFIX,nanyangpt.com,DIRECT
  - DOMAIN-SUFFIX,ncore.cc,DIRECT
  - DOMAIN-SUFFIX,open.cd,DIRECT
  - DOMAIN-SUFFIX,ourbits.club,DIRECT
  - DOMAIN-SUFFIX,passthepopcorn.me,DIRECT
  - DOMAIN-SUFFIX,privatehd.to,DIRECT
  - DOMAIN-SUFFIX,redacted.ch,DIRECT
  - DOMAIN-SUFFIX,springsunday.net,DIRECT
  - DOMAIN-SUFFIX,tjupt.org,DIRECT
  - DOMAIN-SUFFIX,totheglory.im,DIRECT
  - DOMAIN-SUFFIX,acm.org,DIRECT
  - DOMAIN-SUFFIX,acs.org,DIRECT
  - DOMAIN-SUFFIX,aip.org,DIRECT
  - DOMAIN-SUFFIX,ams.org,DIRECT
  - DOMAIN-SUFFIX,annualreviews.org,DIRECT
  - DOMAIN-SUFFIX,aps.org,DIRECT
  - DOMAIN-SUFFIX,ascelibrary.org,DIRECT
  - DOMAIN-SUFFIX,asm.org,DIRECT
  - DOMAIN-SUFFIX,asme.org,DIRECT
  - DOMAIN-SUFFIX,astm.org,DIRECT
  - DOMAIN-SUFFIX,bmj.com,DIRECT
  - DOMAIN-SUFFIX,cambridge.org,DIRECT
  - DOMAIN-SUFFIX,cas.org,DIRECT
  - DOMAIN-SUFFIX,clarivate.com,DIRECT
  - DOMAIN-SUFFIX,ebscohost.com,DIRECT
  - DOMAIN-SUFFIX,emerald.com,DIRECT
  - DOMAIN-SUFFIX,engineeringvillage.com,DIRECT
  - DOMAIN-SUFFIX,icevirtuallibrary.com,DIRECT
  - DOMAIN-SUFFIX,ieee.org,DIRECT
  - DOMAIN-SUFFIX,imf.org,DIRECT
  - DOMAIN-SUFFIX,iop.org,DIRECT
  - DOMAIN-SUFFIX,jamanetwork.com,DIRECT
  - DOMAIN-SUFFIX,jhu.edu,DIRECT
  - DOMAIN-SUFFIX,jstor.org,DIRECT
  - DOMAIN-SUFFIX,karger.com,DIRECT
  - DOMAIN-SUFFIX,libguides.com,DIRECT
  - DOMAIN-SUFFIX,madsrevolution.net,DIRECT
  - DOMAIN-SUFFIX,mpg.de,DIRECT
  - DOMAIN-SUFFIX,myilibrary.com,DIRECT
  - DOMAIN-SUFFIX,nature.com,DIRECT
  - DOMAIN-SUFFIX,oecd-ilibrary.org,DIRECT
  - DOMAIN-SUFFIX,osapublishing.org,DIRECT
  - DOMAIN-SUFFIX,oup.com,DIRECT
  - DOMAIN-SUFFIX,ovid.com,DIRECT
  - DOMAIN-SUFFIX,oxfordartonline.com,DIRECT
  - DOMAIN-SUFFIX,oxfordbibliographies.com,DIRECT
  - DOMAIN-SUFFIX,oxfordmusiconline.com,DIRECT
  - DOMAIN-SUFFIX,pnas.org,DIRECT
  - DOMAIN-SUFFIX,proquest.com,DIRECT
  - DOMAIN-SUFFIX,rsc.org,DIRECT
  - DOMAIN-SUFFIX,sagepub.com,DIRECT
  - DOMAIN-SUFFIX,sciencedirect.com,DIRECT
  - DOMAIN-SUFFIX,sciencemag.org,DIRECT
  - DOMAIN-SUFFIX,scopus.com,DIRECT
  - DOMAIN-SUFFIX,siam.org,DIRECT
  - DOMAIN-SUFFIX,spiedigitallibrary.org,DIRECT
  - DOMAIN-SUFFIX,springer.com,DIRECT
  - DOMAIN-SUFFIX,springerlink.com,DIRECT
  - DOMAIN-SUFFIX,tandfonline.com,DIRECT
  - DOMAIN-SUFFIX,un.org,DIRECT
  - DOMAIN-SUFFIX,uni-bielefeld.de,DIRECT
  - DOMAIN-SUFFIX,webofknowledge.com,DIRECT
  - DOMAIN-SUFFIX,westlaw.com,DIRECT
  - DOMAIN-SUFFIX,wiley.com,DIRECT
  - DOMAIN-SUFFIX,worldbank.org,DIRECT
  - DOMAIN-SUFFIX,worldscientific.com,DIRECT
  - DOMAIN-SUFFIX,cn,DIRECT
  - DOMAIN-SUFFIX,360in.com,DIRECT
  - DOMAIN-SUFFIX,51ym.me,DIRECT
  - DOMAIN-SUFFIX,8686c.com,DIRECT
  - DOMAIN-SUFFIX,abchina.com,DIRECT
  - DOMAIN-SUFFIX,accuweather.com,DIRECT
  - DOMAIN-SUFFIX,aicoinstorge.com,DIRECT
  - DOMAIN-SUFFIX,air-matters.com,DIRECT
  - DOMAIN-SUFFIX,air-matters.io,DIRECT
  - DOMAIN-SUFFIX,aixifan.com,DIRECT
  - DOMAIN-SUFFIX,amd.com,DIRECT
  - DOMAIN-SUFFIX,b612.net,DIRECT
  - DOMAIN-SUFFIX,bdatu.com,DIRECT
  - DOMAIN-SUFFIX,beitaichufang.com,DIRECT
  - DOMAIN-SUFFIX,bjango.com,DIRECT
  - DOMAIN-SUFFIX,booking.com,DIRECT
  - DOMAIN-SUFFIX,bstatic.com,DIRECT
  - DOMAIN-SUFFIX,cailianpress.com,DIRECT
  - DOMAIN-SUFFIX,camera360.com,DIRECT
  - DOMAIN-SUFFIX,chinaso.com,DIRECT
  - DOMAIN-SUFFIX,chua.pro,DIRECT
  - DOMAIN-SUFFIX,chuimg.com,DIRECT
  - DOMAIN-SUFFIX,chunyu.mobi,DIRECT
  - DOMAIN-SUFFIX,chushou.tv,DIRECT
  - DOMAIN-SUFFIX,cmbchina.com,DIRECT
  - DOMAIN-SUFFIX,cmbimg.com,DIRECT
  - DOMAIN-SUFFIX,ctrip.com,DIRECT
  - DOMAIN-SUFFIX,dfcfw.com,DIRECT
  - DOMAIN-SUFFIX,docschina.org,DIRECT
  - DOMAIN-SUFFIX,douban.com,DIRECT
  - DOMAIN-SUFFIX,doubanio.com,DIRECT
  - DOMAIN-SUFFIX,douyu.com,DIRECT
  - DOMAIN-SUFFIX,dxycdn.com,DIRECT
  - DOMAIN-SUFFIX,dytt8.net,DIRECT
  - DOMAIN-SUFFIX,eastmoney.com,DIRECT
  - DOMAIN-SUFFIX,eudic.net,DIRECT
  - DOMAIN-SUFFIX,feng.com,DIRECT
  - DOMAIN-SUFFIX,fengkongcloud.com,DIRECT
  - DOMAIN-SUFFIX,frdic.com,DIRECT
  - DOMAIN-SUFFIX,futu5.com,DIRECT
  - DOMAIN-SUFFIX,futunn.com,DIRECT
  - DOMAIN-SUFFIX,gandi.net,DIRECT
  - DOMAIN-SUFFIX,geilicdn.com,DIRECT
  - DOMAIN-SUFFIX,getpricetag.com,DIRECT
  - DOMAIN-SUFFIX,gifshow.com,DIRECT
  - DOMAIN-SUFFIX,godic.net,DIRECT
  - DOMAIN-SUFFIX,hicloud.com,DIRECT
  - DOMAIN-SUFFIX,hongxiu.com,DIRECT
  - DOMAIN-SUFFIX,hostbuf.com,DIRECT
  - DOMAIN-SUFFIX,huxiucdn.com,DIRECT
  - DOMAIN-SUFFIX,huya.com,DIRECT
  - DOMAIN-SUFFIX,infinitynewtab.com,DIRECT
  - DOMAIN-SUFFIX,ithome.com,DIRECT
  - DOMAIN-SUFFIX,java.com,DIRECT
  - DOMAIN-SUFFIX,jidian.im,DIRECT
  - DOMAIN-SUFFIX,kaiyanapp.com,DIRECT
  - DOMAIN-SUFFIX,kaspersky-labs.com,DIRECT
  - DOMAIN-SUFFIX,keepcdn.com,DIRECT
  - DOMAIN-SUFFIX,kkmh.com,DIRECT
  - DOMAIN-SUFFIX,licdn.com,DIRECT
  - DOMAIN-SUFFIX,linkedin.com,DIRECT
  - DOMAIN-SUFFIX,loli.net,DIRECT
  - DOMAIN-SUFFIX,luojilab.com,DIRECT
  - DOMAIN-SUFFIX,maoyan.com,DIRECT
  - DOMAIN-SUFFIX,maoyun.tv,DIRECT
  - DOMAIN-SUFFIX,meituan.com,DIRECT
  - DOMAIN-SUFFIX,meituan.net,DIRECT
  - DOMAIN-SUFFIX,mobike.com,DIRECT
  - DOMAIN-SUFFIX,moke.com,DIRECT
  - DOMAIN-SUFFIX,mubu.com,DIRECT
  - DOMAIN-SUFFIX,myzaker.com,DIRECT
  - DOMAIN-SUFFIX,nim-lang-cn.org,DIRECT
  - DOMAIN-SUFFIX,nvidia.com,DIRECT
  - DOMAIN-SUFFIX,oracle.com,DIRECT
  - DOMAIN-SUFFIX,paypal.com,DIRECT
  - DOMAIN-SUFFIX,paypalobjects.com,DIRECT
  - DOMAIN-SUFFIX,qdaily.com,DIRECT
  - DOMAIN-SUFFIX,qidian.com,DIRECT
  - DOMAIN-SUFFIX,qyer.com,DIRECT
  - DOMAIN-SUFFIX,qyerstatic.com,DIRECT
  - DOMAIN-SUFFIX,raychase.net,DIRECT
  - DOMAIN-SUFFIX,ronghub.com,DIRECT
  - DOMAIN-SUFFIX,ruguoapp.com,DIRECT
  - DOMAIN-SUFFIX,s-reader.com,DIRECT
  - DOMAIN-SUFFIX,sankuai.com,DIRECT
  - DOMAIN-SUFFIX,scomper.me,DIRECT
  - DOMAIN-SUFFIX,seafile.com,DIRECT
  - DOMAIN-SUFFIX,sm.ms,DIRECT
  - DOMAIN-SUFFIX,smzdm.com,DIRECT
  - DOMAIN-SUFFIX,snapdrop.net,DIRECT
  - DOMAIN-SUFFIX,snwx.com,DIRECT
  - DOMAIN-SUFFIX,sspai.com,DIRECT
  - DOMAIN-SUFFIX,takungpao.com,DIRECT
  - DOMAIN-SUFFIX,teamviewer.com,DIRECT
  - DOMAIN-SUFFIX,tianyancha.com,DIRECT
  - DOMAIN-SUFFIX,udacity.com,DIRECT
  - DOMAIN-SUFFIX,uning.com,DIRECT
  - DOMAIN-SUFFIX,vmware.com,DIRECT
  - DOMAIN-SUFFIX,weather.com,DIRECT
  - DOMAIN-SUFFIX,weico.cc,DIRECT
  - DOMAIN-SUFFIX,weidian.com,DIRECT
  - DOMAIN-SUFFIX,xiachufang.com,DIRECT
  - DOMAIN-SUFFIX,ximalaya.com,DIRECT
  - DOMAIN-SUFFIX,xinhuanet.com,DIRECT
  - DOMAIN-SUFFIX,xmcdn.com,DIRECT
  - DOMAIN-SUFFIX,yangkeduo.com,DIRECT
  - DOMAIN-SUFFIX,zhangzishi.cc,DIRECT
  - DOMAIN-SUFFIX,zhihu.com,DIRECT
  - DOMAIN-SUFFIX,zhimg.com,DIRECT
  - DOMAIN-SUFFIX,zhuihd.com,DIRECT
  - DOMAIN,download.jetbrains.com,DIRECT
  - DOMAIN,images-cn.ssl-images-amazon.com,DIRECT
  - IP-CIDR,119.28.28.28/32,DIRECT,no-resolve
  - GEOIP,CN,DIRECT

  - MATCH,Final

```

## TProxy代理设置

采用clash官方推荐的wiki的防火墙设置，来拦截流量，由于udp不支持redir，所以tcp采用redir，udp采用tproxy(也都可以采用tproxy)，dns拦截端口为上述文件设置的1053，将dns的udp查询均转发至1053端口

为了防止clash代理自身流量，有两种常用方式避免流量回环，cgroup和uid。第一种方式将clash加入特定cgroup足，比如自定义的noproxy组，第二种方式为使用clash用户来启动clash进程。接着使用iptables来匹配对于流量

官方推荐wiki的iptables规则如下

```bash
#tcp
iptables -t nat -N clash
iptables -t nat -A clash -d 0.0.0.0/8 -j RETURN
iptables -t nat -A clash -d 10.0.0.0/8 -j RETURN
iptables -t nat -A clash -d 127.0.0.0/8 -j RETURN
iptables -t nat -A clash -d 169.254.0.0/16 -j RETURN
iptables -t nat -A clash -d 172.16.0.0/12 -j RETURN
iptables -t nat -A clash -d 192.168.0.0/16 -j RETURN
iptables -t nat -A clash -d 224.0.0.0/4 -j RETURN
iptables -t nat -A clash -d 240.0.0.0/4 -j RETURN
iptables -t nat -A clash -p tcp -j REDIRECT --to-port 7892
iptables -t nat -I PREROUTING -p tcp -d 8.8.8.8 -j REDIRECT --to-port 7892
iptables -t nat -I PREROUTING -p tcp -d 8.8.4.4 -j REDIRECT --to-port 7892
iptables -t nat -A PREROUTING -p tcp -j clash
iptables -t nat -A OUTPUT -p tcp -d 198.18.0.0/16 -j REDIRECT --to-port 7892

#udp
ip rule add fwmark 1 table 100
ip route add local default dev lo table 100
iptables -t mangle -N clash
iptables -t mangle -A clash -d 0.0.0.0/8 -j RETURN
iptables -t mangle -A clash -d 10.0.0.0/8 -j RETURN
iptables -t mangle -A clash -d 127.0.0.0/8 -j RETURN
iptables -t mangle -A clash -d 169.254.0.0/16 -j RETURN
iptables -t mangle -A clash -d 172.16.0.0/12 -j RETURN
iptables -t mangle -A clash -d 192.168.0.0/16 -j RETURN
iptables -t mangle -A clash -d 224.0.0.0/4 -j RETURN
iptables -t mangle -A clash -d 240.0.0.0/4 -j RETURN
iptables -t mangle -A clash -p udp -j TPROXY --on-port 7893 --tproxy-mark 1
iptables -t mangle -A OUTPUT -p udp -d 198.18.0.0/16 -j MARK --set-mark 1
iptables -t mangle -A PREROUTING -p udp -j clash
iptables -t nat -N CLASH_DNS
iptables -t nat -F CLASH_DNS
iptables -t nat -A CLASH_DNS -p udp -j REDIRECT --to-port 1053
iptables -t nat -I OUTPUT -p udp --dport 53 -j CLASH_DNS
iptables -t nat -I PREROUTING -p udp --dport 53 -j REDIRECT --to 1053

# 以下二选一
# 使用noproxy的cgroup组来防止流量回环
iptables -t mangle -A OUTPUT -m cgroup --path "noproxy.slice" -j RETURN
# 使用uid来防止流量回环
# iptables -t mangle -A clash-self -m owner --uid-owner clash -j RETURN
```

## iptables规则持久化

iptables 规则在每次系统重启后，都会进行复原，因此如果设置错误导致无法上网时，可以删除防火墙对应规则或直接重启电脑解决

为了让iptables规则持久化，可以设置一个开机自启服务，自动运行对应脚本以设置防火墙。将上述 iptables 规则保存为为 `iptables.sh` ，并创建 `/etc/systemd/system/tproxy.service`，编辑设置内容如下

```conf
[Unit]
Description=Setup ip-rule and ip-route for tproxy local network traffic.
Before=network-pre.target
Wants=network-pre.target

[Service]
Type=oneshot
ExecStart=/bin/bash /path/to/iptables.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

设置服务开机自启

```bash
sudo systemctl enable tproxy.service
```

## 启动服务

clash 需要部分网络代理[能力](https://wiki.archlinux.org/title/Capabilities)，编辑 clash@ 服务文件，添加 clash 的 capability 能力，

```bash
sudo systemctl edit clash@.service
```

添加如下内容

```conf
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
```

1. `CAP_NET_RAW` 让应用可以处理 TProxy 带来的流量

2. `CAP_NET_BIND_SERVICE` 允许应用绑定 1000 以下的端口

如果采用 cgroup 防止流量回环，还需要设置该服务所对应的 [Slice](https://wiki.archlinux.org/title/cgroups#As_service)，采用 UID 识别的话，则需要设置该服务的启动 USER，二选一即可，也可以同时都设置。若设置 UID 时，必须保证系统存在相应用户名

```conf
# cgroup
Slice = noproxy.slice
# UID
User = username
```

同时设置 clash@$USER 服务后台自动，并立即启动

```bash
sudo systemctl enable --now clash@$USER.service
```

## docker流量问题解决

为了解决 cgproxy 代理 bridge 联网的 docker 无法联网问题，需要进行如下而外设置

```bash
sysctl -w net.bridge.bridge-nf-call-iptables=0
sysctl -w net.bridge.bridge-nf-call-ip6tables=0
sysctl -w net.bridge.bridge-nf-call-arptables=0
```

## 参考资料

[Clash github Wik](https://github.com/Dreamacro/clash/wiki)

[Unofficial Clash Wiki](https://lancellc.gitbook.io/clash)

[clash-win-docs-new](https://docs.cfw.lbyczf.com/)

[容器(docker)桥接(bridge)模式时的代理问题](https://github.com/springzfx/cgproxy/issues/10)

[如何使用 Clash 的 TPROXY 功能进行透明代理](https://github.com/escape0707/markdown-practice/blob/master/notes/how-to-use-clash-with-tproxy-and-fakeip-for-tcp-udp.md)
