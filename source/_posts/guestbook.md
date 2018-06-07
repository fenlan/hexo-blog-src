---
title: 通过留言板的添加所得到的想法
tags: hexo
date: 2016-05-24
categories: hexo
---

在添加留言板时意外收获到了我想要的外部链接，但不知道适不适用与除主页上的menu以外的地方。假设一个场景，就是你要在主页上的某个地方添加一个超链接，使点击事件发生后跳到另外一个界面，设置如下：

先找到themes目录下的_config.yml这个文件打开，这个文件是themes设置文件，你可以在里面设置themes属性

![1](https://github.com/fenlan/fenlan.github.io/blob/master/css/images/1.png?raw=true =100*20)

这里面有layout布局，比如menu,content,sidebar等设置，我的留言板是在menu里面的，所以我在menu里面添加了这行代码：
<!--more-->
``` bash
Guestbook: /guestbook
```

解释一下，Guestbook是在Header的menu中显示，效果如下

![](https://github.com/fenlan/fenlan.github.io/blob/master/css/images/2.png?raw=true =100*20)

后面的guestbook是在你博客主目录下的source中的一个界面名字。ok，就这样就ok了。
