---
title: logrotate
tags: logrotate
date: 2017-06-14 15:18:23
categories: linux
---

## logrotate介绍
**对于Linux系统安全来说，日志文件是及其重要的工具。日志文件包含了关于系统中发生的事件的有用信息，在排障过程中或者系统性能分析时经常被用到。当日志文件不断增长的时候，就需要定时切割，否则，写日志的速度和性能也会下降，更不便于我们归档和查询。**
<!--more-->
**所以便有了使用logrotate的时候，logrotate是十分有用的工具，它可以自动对日志进行截断、压缩以及删除旧的日志文件。例如，你可以设置logrotate，让/var/log里的日志文件没30天轮循，并删除超过6个月的日志。配置完后，logrotate的运作完全自动化，不必进行任何一步的人为干预。**

## logrotate配置文件位置
**Linux系统默认安装logrotate工具，它默认的配置文件在:**
`/etc/logrotate.conf`
`/etc/logrotate.d/`

## 定时轮循机制
- **`/etc/cron.daily/logrotate`中定义了每天定时执行的任务**
- **`/etc/cron.weekly/logrotate`中定义了每个星期定时执行的任务**
- **`/etc/cron.hourly/logrotate`中定义了每小时定时执行的任务**
- **`/etc/cron.monthly/logrotate`中定义了每个月定时执行的任务**

**`/etc/crontab`规定了轮循的时间**
![crontab](/images/crontab.png)

- **`/etc/cron.daily/`下面的任务都是每天6:25执行**
- **`/etc/cron.weekly/`下面的任务都是每周日6:47执行**
- **`/etc/cron.monthly/`下面的任务都是每月1号6:52执行**