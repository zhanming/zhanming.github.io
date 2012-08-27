---
layout: post
title: Go语言介绍 - Part 2：源码安装
categories: [Go]
tags: [go, centos]
---

#### 简介
今天Go语言更新了，版本为1.0.1，与第一个版本发布，时隔了一个月左右的时间，更新速度不可谓不快。此次更新主要是bug fix，用户接口没有更改。  
如此快的更新速度，从binary安装感觉不是很舒服了，从源码安装变得更加顺手，写几行命令就轻松完成。

原文如下：Most users don't need to do this, and will instead install from precompiled binary packages as described in [Getting Started][2], a much simpler process. If you want to help develop what goes into those precompiled packages, though, read on.
#### 引文
本文以CentOS 6.2为例介绍，从源码进行Go语言的安装和更新。

#### 安装Mercurial
从源码安装需要使用hg命令，另一种SCM(source code management)，更确切的说是DSCM（分布式版本控制系统）。
    # yum install mercurial
查看一下版本
    # hg --version
    Mercurial Distributed SCM (version 1.4)
    
    Copyright (C) 2005-2009 Matt Mackall <mpm@selenic.com> and others
    This is free software; see the source for copying conditions. There is NO
    warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
如上，下载的是1.4版本。    
#### 下载Go源码
Go的源码在google code上存放，国内用户由于某些原因可能无法访问，最简单的办法是修改hosts文件。
    # vi /etc/hosts
添加如下IP到hosts文件中，这样使用https访问google code，下载Go语言源码应该没有问题。
    203.208.45.200 code.google.com

接下来开始下载Go源码。请先确保当前目录没有go文件夹，以免混淆。
    $ cd /usr/local
    $ hg clone -u release https://code.google.com/p/go
    destination directory: go
    requesting all changes
    adding changesets
    adding manifests
    adding file changes
    added 13122 changesets with 49619 changes to 7193 files (+5 heads)
    updating to branch release-branch.go1
    3206 files updated, 0 files merged, 0 files removed, 0 files unresolved
默认将源码下载到当前目录的go文件夹中，本例在`/usr/local/go`中。
#### 安装依赖
在CentOS6.2 Desktop Edition上，默认只需要安装gcc。
    # yum install gcc
#### 安装Go
构建二进制分发版
    # cd /usr/local/go/src
    # ./all.bash
如果一切顺利，最后会出现
    # Checking API compatibility.
    
    ALL TESTS PASSED
    
    ---
    Installed Go for linux/amd64 in /usr/local/go
    Installed commands in /usr/local/go/bin
此时Go安装完毕。
#### 验证
设置环境变量，设置方法与[Go语言介绍 - Part 1：安装][0]相同。
    # go version
    go version go1.0.1
安装成功，版本go1.0.1。
#### 保持更新
以后每当要更新版本，就方便好多。
    # cd /usr/local/go/src
    # hg pull
    # hg update release
    # ./all.bash
即可完成更新。
#### 参考文档
[Installing Go from source][1]

[0]: http://qizhanming.com/2012/04/21/go-intro-1-install-on-centos
[1]: http://golang.org/doc/install/source
[2]: http://golang.org/doc/install