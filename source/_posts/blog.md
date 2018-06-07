---
title: 搭建博客的大致流程
tags: hexo搭建
date: 2016-05-22 15:18:23
categories: hexo
---
本想着是不是拿英文来写这个流程来锻炼一下自己的英文水平，不过想了想就不装B了，英语就拿磋样还拿出来献丑。言归正传，我以为可以在寝室大神的引导下我能快一点搭好my blog，不过是我想得太简单。经过自己踩了无数坑过后总结大概过程，具体过程网上一搜一大把，不过建议别用百度搜这种东西，能翻墙就翻墙吧。大概过程如下：

<!--more-->

### 安装前环境准备
git+node+hexo

#### 安装git
如果是windows，下个git bash，然后跟着网上的教程自己走一遍配置过程
如果是linux,直接在终端输入代码

``` bash
$ sudo apt-get install git
```

如果不行可能是系统没有这个安装包吧，如果上面命令不能安装，自己想办法吧，比如好像可以从github上clone一个包来安装吧，这些可以自己摸索的，我当时就慢慢摸索下来的，相信你也可以。

#### 安装sublime text
说真的，这就是个编辑器，什么都可以写，可以安很多插件，可以拿它编代码，亲试还不错。不像其他编译器比如VS，eclipse那么复杂,语法高亮和语法提示，自动补全这些功能可以通过安插件实现。忘了说安装命令了

``` bash
$ sudo apt-get install sublime
```

如果安不了问google或baidu,因为我也忘了是不是这个了，反正我安的时候试了类似的命令就ok了。

#### Github
如果你是技术屌丝，这个可以有，而且有很多人在用。讲道理，这不是安装什么，就是去一个网站https://github.com注册一个账号，然后把Github Pages这个东西启用，具体过程google,baidu一大把，累了不想写

然后windows用git bash，linux用终端设置你的用户名密码

``` bash
$ git config --global user.name "your_name"
$ git config --global user.email "your_email"
```

#### 安装node
这个没什么难的，不会自己多问，之间估计会遇到有node install, npm install什么的。

#### 安装hexo
说实话，这一步我走得很艰难，真的很艰难，到现在我都不想再去弄第二次。最开始我在虚拟机linux系统里面弄，没成功，直接导致我转向windows的git bash，具体怎么弄，请放过我，不过我可以回忆一下

安装

``` bash
$ npm install -g hexo
$ hexo --version
```

如果出现版本号，就恭喜你了，安好了。

#### 初始化
新建一个目录，然后cd到那个目录，然后初始化hexo到那个目录。

``` bash
$ mkdir hexo
$ cd hexo
$ hexo init
```

然后干嘛呢，我想想。。。。。。

#### 生成静态页面
cd到你init的目录(这是必须的，请注意)，执行下面的命令，生成静态页面

``` bash
$ hexo g
```

#### 在本地启动
执行下面命令，然后打开浏览器输入http://localhost:4000进行预览

``` bash
$ hexo s
```

如果能行，就恭喜你了，要完了。。。。。。
不过现在很晚了我要睡了，明天再更。。。。。。



还没写完，继续吧，好像最后只有一步了，不知道昨天怎么没写完，最后只需要执行以下代码：

``` bash
$ hexo d
```

然后根据提示输入你的github的username和password就ok了，然后就去你自己的博客看看效果吧
