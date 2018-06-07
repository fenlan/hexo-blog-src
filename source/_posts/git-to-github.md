---
title: git to gihub
date: 2016-05-26
categories: git
tags:
  - git
  - github
---
**我想想，这个东西好像是上个学期寝室大神告诉我的，然而我现在才知道怎么使用它，主要是之前没接触到这个东西，不知道这东西是干嘛用的，在建blog时接触到了，对于如此难堪的水平的我来说有必要记录一下。想想感觉不应该写在blog上，觉得好好的博客怎么写这些东西了，不过尽管如此，我还是决心写一下**

<!--more-->

### 注册github账号
**这个步骤省略。。。。。**

### 新建repositories
**继续省略。。。。。。**

### 本地配置
**最好新建一个目录，比如Mycode，然后将你要放在github的代码放在这个目录中如果第一次用git，需要表明你的身份，我的理解是这样的，命令行如下：**

``` bash
$ git config --global user.name "yourname"
$ git config --global user.email "youremail"
$ git config --global color.ui auto
```

**然后继续：**

``` bash
$ cd Mycode
$ git init
$ git status
$ git add 要放在github的文件
$ git status
$ git commit -m "c-program"
$ git status
$ git log
$ git remote add origin https://......
$ git push -u origin master
$ git push -u origin
```

**附加一些其他的**

``` bash
$ git clone https://......
$ git pull origin master
$ git log
```

**差不多就这些**

**最近更新：**
**同步远程仓库更新本地仓库命令**
```
$ git pull
```