---
title: 获取局域网连接设备mac地址
date: 2017-03-22 15:18:23
categories: linux
tags:
  - nmap
  - python
---

# 使用工具
**nmap: 也就是Network Mapper,最早是linux下的网络扫描和嗅探工具包**
**nmap是一个网络连接端扫描软件，用来扫描网上电脑开放的网络端口。确定哪些服务运行在哪些连接端，并且推断计算机运行哪个操作系统。他是网络管理员比用的软件之一，以及用以评估网络系统安全。 **
<!--more-->
# 步骤
### 1.通过nmap扫描出连接同一局域网的设备的ip 地址以及 mac 地址，扫描完成再以xml 文件形式储存起来。具体命令如下：
``` bash
# nmap -sP -oX myscan.xml 192.168.1.0/24
```
**需要注意的问题： 如果要获取mac 地址，需要操作系统的管理者权限，对于Linux来说就是root 权限；另外，由于是通过发包探测，如果遇上有防火墙的路由器，会比较麻烦。**
**缺点： 在扫描设备多的情况下，时间会偏长，亲试最长时间21秒，这根被扫描设备的状态有关。**

### 2.使用python 获取xml 文件中的mac 地址存入指定文件：
``` python
# readxml.py
from xml.dom import minidom

f = open("maclist.txt", 'wb')   # 存入mac 地址的目标文件
xmldoc = minidom.parse('myscan.xml')    # 获取 nmap 导出的 xml 文件

addrlist = xmldoc.getElementsByTagName('address')
len = (len(addrlist)-1) / 2  # 计算连接设备数量
# 在addrlist 中有 IP 地址 和 mac 地址，因此要减半

print "len :", len
f.write(str(len))
f.write("\n")
for s in addrlist :
	if s.attributes['addrtype'].value == "mac" :
    	f.write(s.attributes['addr'].value)
        f.write("\n")
        
f.close()
```

### 3.将两个命令写在一个脚本里面：
``` bash
#!/bin/bash

nmap -sP -oX myscan.xml 192.168.1.0/24
python readxml.py
```

### 4.运行脚本搞定