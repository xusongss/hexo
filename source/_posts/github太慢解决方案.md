---
title: github访问太慢
date: 2017-02-27 23:02:11
tags:
---
# github访问太慢解决方案

  为了提高速度，可以使用HOSTS加速对Github的域名解析

### 修改hosts主机映射文件, 添加github一系列网址的IP地址、域名映射。但是github域名所对应IP好像是随时间变化的，挨个查询太麻烦，来个脚本先建立一个域名列表haha.txt，下面列表中的gist.github.com是代码片功能，被墙得死死地。无论如何打不开。

```shell
linux /private/etc/hosts
window C:\Windows\System32\drivers\etc\hosts

github.com
assets-cdn.github.com
avatars0.githubusercontent.com
avatars1.githubusercontent.com
documentcloud.github.com
gist.github.com
help.github.com
nodeload.github.com
raw.github.com
status.github.com
training.github.com
github.io
```

<!--more-->

### 用python语言使用requests+beautifulsoup制作一个小爬虫

``` python
import requests
from bs4 import BeautifulSoup

for i in open("haha.txt"):
    url = "http://ip.chinaz.com/" + i.strip()
    resp = requests.get(url)
    soup=BeautifulSoup(resp.text)
    x=soup.find(class_="IcpMain02")
    x=x.find_all("span",class_="Whwtdhalf")
    print(x[5].string.strip(),i.strip())

```

```shell
192.30.253.113 github.com
151.101.100.133 assets-cdn.github.com
151.101.100.133 avatars0.githubusercontent.com
151.101.100.133 avatars1.githubusercontent.com
151.101.100.133 documentcloud.github.com
8.7.198.45 gist.github.com
151.101.100.133 help.github.com
192.30.253.121 nodeload.github.com
151.101.100.133 raw.github.com
174.129.214.132 status.github.com
151.101.100.133 training.github.com
23.235.33.133 github.io
```
