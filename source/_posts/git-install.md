---
title: Git install and setting
date: 2016-05-21 15:18:23
categories: git
tags: git
comments:
---
### git install and setting:

``` bash
$ git config --global user.name "name"
$ git config --global user.email "email"
$ git config --global color.ui auto
```

### and then create a new directory as tutorial

``` bash
$ mkdir tutorial
$ cd tutorial
$ git init
```

### then create a new file as text.txt

``` bash
$ git status
$ git add text.txt
$ git status
$ git commit -m "text"
$ git status
$ git log
```

### next to rename the "https://......." as origin

``` bash
$ git remote add origin https://.........(do this only the first time)
$ git push -u origin master(the first time)
$ git push -u origin(do this the next time)
```

### do clone

``` bash
$ git clone https://....... tutorial2(a new directory)
```

### do pull file in your date-base

``` bash
$ git pull origin master
$ git log
```

finally do another setting to make it better,but I meet a problem.
So I stop it temporarily,continue it if I have time.