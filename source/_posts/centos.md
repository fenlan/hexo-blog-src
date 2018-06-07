---
title: CentOS
date: 2017-06-16 15:18:23
categories: linux
tags:
  - centos
  - firewall
  - lnmp
  - python
  - laravel
---

## 第一关：设置windows启动项
**在安装双系统时，选择centos会默认没有windows的启动项，因此要添加windows的引导选项，过程如下：**

- **开机进入CentOS系统**
- **进入目录`/boot/grub2`，打开`grub.cfg`进行编辑**
- **找到其中第70行，在指定位置添加内容 **
<!--more-->
**添加内容如下：**
``` bash
### END /etc/grub.d/00 header ###
menuentry 'Windows 10' {
	set root=(hd0,1)
    chainloader +1
}
### BEGIN /etc/grub.d/10_linux ###
```
- **重启系统，引导菜单中新增了windows 10选项**

## 第二关：防火墙
**下载防火墙服务：**
``` bash
[root@localhost]$ yum install iptables-services
```
**查看防火墙状态：**
``` bash
[root@localhost]$ nmap localhost -p 0-10000
```
**启用ssh服务：**
``` bash
[root@localhost]$ yum install openssh-server
[root@localhost]$ chkconfig sshd on
[root@localhost]$ service sshd restart
[root@localhost]$ /etc/init.d/sshd start
```
**在CentOS 7之前，系统默认的防火墙为`iptables`，因此上述为`iptables`版防火墙设置，下列为`firewall`版防火墙设置，`firewall`是centOS 7系统默认的防火墙，想必也是之后的趋势。**
**查看`firewall`状态：**
``` bash
[root@localhost]$ systemctl status firewalld.service
```
**开启`firewall`服务（两者均可）**
``` bash
[root@localhost]$ systemctl start firewalld.service
[root@localhost]$ service firewalld start
```
**开启`ssh`服务：**
``` bash
[root@localhost]$ yum install openssh-server
```
**更改默认`ssh`端口：**
``` bash
[root@localhost]$ vim /etc/ssh/sshd_config
```
**修改里面的端口为自己想要设置的端口，保存退出，由于系统默认不支持`ssh`使用22端口之外的端口，因此需要做修改：**
``` bash
[root@localhost]$ semanage port -a -t ssh_port_t -p tcp ××××
[root@localhost]$ semanage port -l | grep ssh
[root@localhost]$ systemctl restart sshd.service
[root@localhost]$ firewall-cmd --permanent --zone=public --add-port=××××/tcp
[root@localhost]$ firewall-cmd --reload
[root@localhost]$ ss -tnlp | grep ssh
```
## 第三关：lnmp配置
**安装`nginx`:**
``` bash
[root@localhost]$ yum install epel-release
[root@localhost]$ yum -y install nginx
[root@localhost]$ service nginx start
```
或者`/etc/yum.repo.d/nginx.repo`
``` bash
[nginx]
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1

```
**安装`php7.0`(CentOS 7.x):**
``` bash
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
yum install php70w php70w-opcache
yum install php70w-fpm php70w-opcache
```
**安装php7.0不太容易，建议多在网上查**

**设置`nginx`，更改root目录问题：**
**之前将`nginx`的root目录改了过后出现了403问题，搞了好久还是没有一个很完整的解决方案，不过找到一个方案，但还是不太理解。将`nginx`中的配置文件的`user nginx` 改为自己的用户名，即更改过后root目录的所有者。**
![](/images/nginxUser.png)
**困惑点在于`nginx`默认的目录`/usr/share/nginx/html`的所有者为`root`但没有出现403**

**安装`mysql`:**
``` bash
# yum install mysql
# yum install mysql-devel
# wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
# rpm -ivh mysql-community-release-el7-5.noarch.rpm
# yum install mysql-community-server
# service mysqld restart
```

**配置问题**
**1.配置`php.ini`: 只修改一行代码，通过`vim`查找`cgi.fix_pathinfo`，并将其修改为0,去注释.**
**2.配置`www.conf`: 修改`listen`选项设置为`127.0.0.1:9000`; 设置`user`为`/home`下的一个用户,`group`也同样设置**
**3.配置`nginx.conf`,将`user`改为`www.conf`里面的`user`相同，网站根目录必须在`nginx`的`user`下,具体配置如下**
``` bash
location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
```

## 第四关：ssh登录邮件通知（后面有彩蛋哟）
**游手好闲地折腾了几天，突发奇想，前几天租了一个vps，想随时检测有没有人通过ssh登录我的vps，如果有人登录，就发邮件通知，于是有了以下的折腾：**
- **首先安装必备的软件`sendmail` 和`mailx`**
- **通过`last`命令和`awk`结合获取ssh登录情况**
- **最后写个python程序，让它一直跑着随时准备发邮件**
- **将写好的程序运行命令写入/etc/rc.d/rc.local，让它开机运行**

**那么命令如下：**
``` bash
[root@localhost]$ yum install sendmail sendmail-cf mailx
[root@localhost]$ service sendmail start
[root@localhost]$ last | awk '$1=="root"'
```
**接下来需要修改一个配置文件，让`sendmail`可以给整个英特网发送邮件，配置文件为`/etc/mail/sendmail.mc`，其中有一行**
``` bash
DAEMON_OPTIONS('port=smtp,Addr=127.0.0.1, Name=MTA-v6, Family=net6')dnl
```
**将127.0.0.1改为0.0.0.0，这样就可以给整个英特网发送邮件了**
**python脚本如下**

``` python
import os

inst = "last | awk \'$1!=\"reboot\"\'"
str = os.popen(inst).read()
ssh_login = str.split("\n")
length = len(ssh_login)

while True:
    inst1 = "last | awk \'$1!=\"reboot\"\' > ssh_login.txt"
    os.system(inst1)
    str = os.popen(inst).read()
    ssh_login = str.split("\n  length1 = len(ssh_login)

    if length1 > length:
        login_name = ssh_login[0]
        mail = "echo %s | mail -s \'$1!=\"reboot\"\' ××××@qq.com" % login_name
        os.system(mail)
        length = length1
```

**最重要的一点：qq邮箱发送成功一次过后vps的域名就会被拉黑，被认为是垃圾邮件，因此需要在邮箱管理界面设置域名白名单`localhost.localdomain`，然后大功告成，哈哈**

**你以为这样就了事了？如果你将这个程序通过开机启动让它一直跑着，那么很开心的告诉你，你的机器cpu用不了多久就会被占满，所以，这个方法还是不行滴，这只是一种理想思路，想要处理的话可以选择指定时间运行一次或者每隔一段时间运行一次。接下来我去找个比较好的方式来实现这个功能，并实现每天发送当天的日志文件。**

**我的vps跑了几个小时后的cpu受不了了，跑了不到两个小时，就被厂家限制了。**
![](/images/cpu.png)

## 第五关：python实现邮件发送
**没时间了，直接贴代码**
``` python
from email import encoders
from email.header import Header
from email.mime.text import MIMEText
from email.utils import parseaddr, formataddr

import smtplib

def _format_addr(s):
    name, addr = parseaddr(s)
    return formataddr((Header(name, 'utf-8').encode(), addr))

from_addr = "××××××××××@qq.com"
password = "××××××××××"　　　# SMTP独立密码
to_addr = "××××××××××@qq.com"
smtp_server = "smtp.qq.com"

msg = MIMEText('hello, send by Python...', 'plain', 'utf-8')
msg['From'] = _format_addr('fenlan <%s>' % from_addr)
msg['To'] = _format_addr('管理员 <%s>' % to_addr)
msg['Subject'] = Header('vps远程登录日志', 'utf-8').encode()

server = smtplib.SMTP_SSL(smtp_server, 465)
server.login(from_addr, password)
server.sendmail(from_addr, [to_addr], msg.as_string())
server.quit()

```

## 第六关：搭建laravel框架
**首先安装`composer`(一个php工具，具体查看官网)**
``` php
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```
**然后全局使用`composer`**
``` bash
mv composer.phar /usr/local/bin/composer
```

**接着安装`laravel`**
``` bash
composer create-project --prefer-dist laravel/laravel your_project_name
```
**进入项目目录更新`composer`**
``` bash
composer update
```
**在这里我遇到了一个问题，我搭建三次只遇见过一次**
![](/images/laravel_problem.png)
**根据提示进入里面说的网站上，跟着网站描述解决问题就ok**