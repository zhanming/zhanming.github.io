---

layout: post
title: Rocky 9 安装后的系统初始化 
categories: [Linux]
tags: [rocky, linux]
summary: 在 Red Hat Enterprise Linux 9 (RHEL 9) 于 2022-05-18 发布后，Rocky Linux 9 (Rocky 9) 于 2022-07-14 发布，支持周期到 2032-05-31, 也是十年左右的生命周期。本文概述一下 Rocky 9 安装之后，初始化的配置过程，主要包括新建用户，分配 `sudo` 命令，配置远程访问等。

---

## 前言

在 Red Hat Enterprise Linux 9 (RHEL 9) 于 2022-05-18 发布后，Rocky Linux 9 (Rocky 9) 于 2022-07-14 发布，支持周期到 2032-05-31, 也是十年左右的生命周期。本文概述一下 Rocky 9 安装之后，初始化的配置过程，主要包括新建用户，分配 `sudo` 命令，配置远程访问等。

服务器安装完 Rocky （Minimal Install）操作系统后，需要对其进行初始化的配置，以增强服务器的安全性和可用性，本文介绍几个基本的步骤。跟以前的 [Rocky 8 安装后的初始化配置][1] 基本相同。

Rocky Linux 的[官方网站][3] 

### 环境说明

Rocky 9（Minimal Install）

```bash
# cat /etc/system-release
Rocky Linux release 9.1 (Blue Onyx)
```

## 配置

### Root 登录

root 账号是 Rocky 的根用户，刚装完的 Rocky 9(Minial Install) 操作系统，一般只有 root 账号，并且 ssh 服务对外开放。

本地登录

```terminal
local login: root
password:
```

或是使用 ssh 从远程登录

```terminal
local$ ssh root@SERVER_IP_ADDRESS
```

### 查看防火墙

```terminal
# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

可以看到默认开启了 `dhcpv6-client` 和 `ssh` 服务，还开启了 `cockpit` ，`cockpit` 服务主要用于使用浏览器远程管理服务器，虽然默认开启了端口，但是默认的服务是没有开启的。

如果想开启 cockpit 服务

```terminal
# systemctl enable --now cockpit.socket
```

### 创建一个新用户

当你使用 root 登录到服务器，我们一般都是新建一个普通用户。本文以用户 `admin` 为例，您可以将 `admin` 替换为您喜欢的名字。

```terminal
# adduser admin
```

下一步，设置这个用户的密码。

```terminal
# passwd admin
```

输入密码，并再次输入进行确认。

### Root 权限

现在，我们有了普通用户，进行普通的操作，但是有时，我们需要更大的权限进行操作，如 `dnf update`，进行这样的操作，我们一般不会使用 root 登进登录，一般使用 `sudo` （Super User do）命令。

为了将 `sudo` 权限给普通用户，我们需要将新用户加入 `wheel` 组中，Rocky Linux 默认的 wheel 组有运行 `sudo` 的权限。

我们使用 root 用户，将 admin 用户加入到 wheel 组中。

```terminal
# gpasswd -a admin wheel
```

现在，admin 用户可以使用超级用户的权限运行各种命令了。

### 添加服务器的公钥授权（推荐）

如果客户端想访问服务端，可以使用 ssh 命令，SSH 支持用户名和密码方式，也支持公钥授权。

一般是在本机（客户端）新建一对 SSH key（包含公钥和私钥）,将公钥放入服务器（服务端）的 autohrized_keys 中，这样安全性更高些。

#### 生成一对密钥

如果你没有这一对密钥，可以使用一下命令生成,本例中本机用户名为 `admin`。

> **`注意`**
> 
> 输入密码的部分：
> 
> 1. 如果不输入密码，每次访问服务器不会提示输入密码；
> 2. 如果输入密码，每次访问服务器都需要输入这个密码；

```terminal
local$ ssh-keygen -t rsa -C youremail@yourdomain.com
Generating public/private rsa key pair.
Enter file in which to save the key (/home/admin/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/admin/.ssh/id_rsa.
Your public key has been saved in /home/admin/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Ky1iXd6................... youremail@yourdomain.com
The key's randomart image is:
+---[RSA 2048]----+
|        .....    |
|         * =     |
|        o B      |
|         B       |
|      . S E      |
|     .......     |
|    .........    |
|   .........     |
|    .......      |
+----[SHA256]-----+
```

这里使用命令参数 `-t` 指定了密钥的算法，`-C` 指定了密钥的注释

执行命令之后，会在 `$HOME` 下的 .ssh 目录下生成 2 个文件，`id_rsa` 为私钥，`id_rsa.pub` 为公钥

```terminal
local$ ls ~/.ssh/
id_rsa  id_rsa.pub
```

### 拷贝公钥到服务器

生成 SSH 密钥对之后，您需要将本地（客户端）的公钥，拷贝到服务器（服务端）

```terminal
local$ ssh-copy-id admin@SERVER_IP_ADDRESS
```

这会将本地用户的公钥拷贝到服务端的 admin 用户的 `.ssh/authorized_keys` 文件中

之后远程登录到服务器

```terminal
local$ ssh admin@SERVER_IP_ADDRESS
```

### 配置 SSH 服务

我们添加了普通用户，也可以执行 sudo 命令，需要配置 ssh 服务，去掉 root 用户的远程登录，这样更加安全。

先登录到服务器

```terminal
local$ ssh admin@SERVER_IP_ADDRESS
```

进行配置

```terminal
$ sudo vi /etc/ssh/sshd_config
```

将

```terminal
#PermitRootLogin prohibit-password
```

改为

```terminal
PermitRootLogin no
```

`:wq` 之后，保存退出。

重启 ssh 服务

```terminal
$ sudo systemctl reload sshd
```

这样，服务器的 root 用户就被禁止了远程登录。

## 参考资料

[Rocky 8 安装后的初始化配置][1]  
[How To Configure SSH Key-Based Authentication on a Linux Server][2]  

[1]: {% post_url 2022-01-17-initial-server-setup-with-rocky-8 %}  
[2]: https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server  
[3]:https://rockylinux.org/  