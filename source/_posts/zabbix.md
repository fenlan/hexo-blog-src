---
title: zabbix安装配置
tag: zabbix
date: 2017-08-22 15:18:23
categories: linux
---

## 下载软件包
**从[zabbix](https://www.zabbix.com/download)上下载所需要的软件包，包括`zabbix-server-mysql`和`zabbix-frontend-php`，如果还要安装客户端，还要下载`zabbix-agent`。(推荐直接添加`zabbix`源)**

## 安装软件
``` bash
sudo dpkg -i zabbix-server-mysql_3.4.0-1+xenial_amd64.deb
sudo dpkg -i zabbix-frontend-php_3.4.0-1+xenial_all.deb
```

## 配置Zabbix server
``` bash
sudo vim /etc/zabbix/zabbix_server.conf
```
修改其中的`DBhost`, `DBUser`, `DBPassword`， 如果有必要，需要设置`socket`与`mysql`的`socket`相同。
<!-- more -->
## 配置MySQL
``` sql
create database zabbix character set utf8 collate utf8_bin;
grant all privileges on zabbix.* to 'zabbix'@'localhost';
flush privileges;
exit;
```
找到`zabbix-server-mysql`文件夹，文件夹中是zabbix数据库数据填充文件用来填充数据库
``` bash
zcat *.sql.gz | mysql -uzabbix -p zabbix
```
如果是三个sql文件，填充顺序是`schema.sql.gz`->`images.sql.gz`->`data.sql`

## 配置PHP
按照需要修改`php.ini`配置项

## Zabbix web
**将Zabbix的web页面文件拷贝到LNMP的指定目录，启动`Zabbix-server`进入`http://ip/zabbix`按步骤填写相关信息，最后成功安上，最后再配合客户端进行监控**

## Zabbix 检测数据库
**首先在Zabbix网页上为host添加一个数据库模块，然后需要修改客户端数据，在客户端的`/etc/zabbix/zabbix-agentd.d/userparameter_mysql.conf`文件中根据文件指示将文件中所有HOME变量换为含有数据库连接文件`.my.cnf`的目录，我暂时将`.my.cnf`放在`/etc/zabbix`下面，然后编写`.my.cnf`**
``` bash
[mysql]
user=zabbix
password=zabbix
host=localhost

[mysqladmin]
user=zabbix
password=zabbix
host=localhost
```
**最后也是最重要的一步，重启数据库，上面步骤都做了，但网页上数据没有更新，原因就是数据库没有重启**

## 附言
**写得很草率，还在学习中，会努力更新....**
