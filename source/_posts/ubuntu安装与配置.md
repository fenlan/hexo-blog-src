---
title: ubuntu安装配置
date: 2016-08-10 15:18:23
categories: linux
tags:
  - vim
  - ubuntu
---

**想了一下，最终还是将电脑安成windows10 + ubuntu16双系统，以后多用用ubuntu试试。我选择的安装方式使用u盘安装，在安装ubuntu时一直没有选对启动盘制作工具，之前安装windows10的时候使用的是UltraIso，这个安windows没什么问题，但是安装ubuntu时，安不上，后面上网查了很多次，也用了其他的软件，比如linuxLive USB Creator,Universal-USB-Installer等等，其中Universal-usb-installer还是ubuntu推荐的软件，但是仍然不行，就这我折腾了两天还是没成功，后来在qq上的一个业余群里面，有人叫我用USBwriter，结果就成功安装上了。**
<!--more-->

## ubuntu有线联网

**感觉每一项都挺折腾的，百度google都查了很多，这一步我是在终端完成的**
```
$ sudo pppoeconf
```

**然后根据提示输入用户名和密码就行了**

## Chrome安装
**很抱歉，我有google强迫症，没它不行，所以直接在官网下载软件包，然后再终端安装**
```
$ sudo dpkg -i 软件包名
```
**其实有很多安装方式，百度google多问一下就行了**

## 安装软件出现未满足依赖
**方法就是软件更新器更新，也有这个命令**
```
$ sudo apt-get install -f
$ sudo apt-get update
```
**我上面的命令都不管用了，因为我下载了android studio，然后就出问题了，出现没有满足依赖，后来把android studio卸载了才能更新软件包的**
```
$ sudo apt-get remove android-studio
```
**系统软件包更新后仍然没有安上android studio**

## 安装jdk
**跟安装chrome一样，不过要配置环境**
**这里就不浪费时间，直接引用**
[Ubuntu 11.04,14.04 下安装配置 JDK 7,JDK 8](http://blog.csdn.net/sunlovefly2012/article/details/42747929)
**具体根据自己的情况去设置**

## chrome翻墙
**这里有写得很好的博客关于翻墙的**
[linux-ubuntu使用shadowsocks客户端配置](https://aitanlu.com/ubuntu-shadowsocks-ke-hu-duan-pei-zhi.html#comment-3874)

**里面全部都有，很详细。**

## git安装
**这个我的博客里有，还有hexo和博客关联都有**

## 安装android studio
**还在努力中。。。。。**
**已经完成了，不过并不想写。哎，太懒了，其实也没有多难，只是要配好java环境,然后我的android studio是用命令打开，打开一个后缀是.sh的文件就行**

## vim配置
**主要是两个文件夹，一个是/etc/vim,一个是~/.vim,具体的还是要多google，我到现在也没弄清楚**

**经过我不懈的努力，终于比较完善了，有代码高亮，自动补全，错误检测，还有nerdtree功能，这些都可以用bundle安装，先安装bundle，然后再在vimrc文件中配置Plugin + 插件，完成后重新打开vim，输入命令PluginInstall就ok。总结一下我用的vim的插件：syntastic(实现代码检测功能)、YouCompleteMe(自动补全最好的插件，没有之一，我这么觉得的)、nerdtree(可以在vim的左侧显示文件目录)，其他的我有点忘了，不过弄好这三个我已经满意了，以后试着有vim,不过讲真，vim还是不太适合写较大的程序，我这么觉得的**

## gcc编译器问题
**最近我自己写了一些代码，有自己定义的头文件，在ubuntu中用gcc编译器编译时总是出现许多对‘XXX’未定义的引用，我疑惑了很久，后来意外的解决了，一个原因是要编译多个源文件的程序时，所有的源文件都要放在一起编译，还有就是用math.h头文件时要在命令最后加上-lm,还有用ncurses.h头文件时也要用-lcurses,什么原因要是要google搜搜，最后吐槽一下自己，发现自己已经在c这门语言的路上越走越远，我并不想这样，感觉写什么程序都要被迫用c，我想学用python,java。然而到目前为止，我仍然对他们很陌生，好苦逼**

## atom编辑器
**[atom](http://www.jianshu.com/p/b4c8479cfaa5)是一个类似sublime text的编译器，但是个人观点atom好看到无解，好用好看，是真的好。这是我强烈推荐的一款软件，不用真的会后悔，这还是寝室大神告诉我的，我偶然看见他在用，就问是什么编译器。我推荐给另一个人的时候，他居然居然说什么编译器都一样，只要能用就行。我愣了！他还在用Dev，有visual studio这样强大的东西，又有atom那么好看的东西，他居然还在用Dev。反正我是无法理解。**

## 主题推荐
**ubuntu上主题好看的不少，个人钟情与flatabulous，这个主题满足了我的个人审美，具体操作看这篇博文： http://www.jcodecraeer.com/a/chengxusheji/chengxuyuan/2015/0923/3502.html**

## ubuntu配置QQ
**之前试过好多次没成功，意外之间找到了一个网站便用上了，能满足一般的QQ要求，跟windows的QQ很像，网站如下： http://ttop5.net/?p=1316**

## ubuntu写markdown工具
**推荐haroopad，很好用，可以实时显示效果,推荐博文： http://www.jianshu.com/p/064aec7a5164**