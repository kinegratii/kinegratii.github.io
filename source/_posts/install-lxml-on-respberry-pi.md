---
title: 在树莓派安装lxml
date: 2017-05-14 21:15:46
categories: 编程
tags:
- 树莓派
- lxml
- 安装
---

安装和配置是最令人费劲的：

- 经常在解决一个问题过程又出现另外一个不得不先解决问题，有点像栈(stack)，忘记了最先要解决的问题
- 网上的解决方法只是一个参考，时间和环境不一定一样，需要自己去阐释
- Google搜索/Bing搜索/stackoverflow是搜索的利器，错误信息一复制粘贴基本上可以找到一些结果，当然和搜索技巧也是有很大关系的

本文记录在树莓派安装lxml的过出现的一些问题和解决方案，本来lxml安装过程比较简单，安装依赖和pip安装两条命令即可。但是由于各种各样的状况和环境导致这过程花费的时间有点长。

```
$ sudo apt-get install libxml2-dev libxslt-dev python-dev
$ sudo pip3 install lxml
```
*lxml标准的安装过程*


<!-- more -->

## 安装python3.6

树莓派3默认安装的是python3.4，版本有点旧，决定自己安装python3.6，从官网下载源代码并自己编译，步骤如下：

```
wget https://www.python.org/ftp/python/3.6.1/Python-3.6.1.tgz
tar xvf Python-3.6.1.tgz
cd Python-3.6.1
./configure --enable-optimizations
make
sudo make altinstall
python3.6
```

安装完后python3.6路径为 `/usr/local/bin/python3.6`，另外使用软连接将python3指向3.6，

```
sudo ln -s -f /usr/local/bin/python3.6 /usr/local/bin/python3
sudo ln -s -f /usr/local/bin/python3.6 /usr/bin/python3
```

## 问题: py3clean: permission denied

由于安装python3.6没有完全配置好，导致使用apt-get安装任何包都会出现这个错误信息。

从字面上是权限的问题，使用chmod命令或者root用户也无效，后来发现是linux shabang符号的问题，py3clean可执行文件是一个python脚本，使用编辑器（nano）发现其为 `#! /usr/bin/python3` 改成 `#! /usr/local/bin/python3`，同时修改的还有py3compile脚本，这两个脚本都在 `/usr/bin` 目录下。

## 问题：找不到libxslt-dev

libxml2-dev包很快就安装成功了，但是libxslt-dev包却却提示找不到。

```
pi@raspberrypi:~ $ sudo apt-get install libxslt-devel
Reading package lists... Done
Building dependency tree
Reading state information... Done
E: Unable to locate package libxslt-devel
```

在Debian源找到了 libxslt-devel2 包页面，地址是 https://packages.debian.org/jessie/libxml2-dev ，按照页面提示添加apt源并安装。

```
W: GPG error: http://ftp.cn.debian.org jessie Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 8B48AD6246925553 NO_PUBKEY 7638D0442B90D010 NO_PUBKEY CBF8D6FD518E17E1
```

该消息显示这个源未经过验证，所以需要添加plublic key到系统中，如果显示 imported 1 字样就是导入成功了.

```
pi@raspberrypi:~ $ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 8B48AD6246925553
Executing: gpg --ignore-time-conflict --no-options --no-default-keyring --homedir /tmp/tmp.5TjGyQBYus --no-auto-check-trustdb --trust-model always --keyring /etc/apt/trusted.gpg --primary-keyring /etc/apt/trusted.gpg --keyserver keyserver.ubuntu.com --recv-keys 8B48AD6246925553
gpg: requesting key 46925553 from hkp server keyserver.ubuntu.com
gpg: key 46925553: public key "Debian Archive Automatic Signing Key (7.0/wheezy) <ftpmaster@debian.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
```

使用apt安装libxslt-dev。

```
pi@raspberrypi:~ $ sudo apt install libxslt-dev
Reading package lists... Done
Building dependency tree
Reading state information... Done
Note, selecting 'libxslt1-dev' instead of 'libxslt-dev'
...
```

另外要说的是apt源如果连接不上，树莓派直接死机，使用局域网扫描工具没找到树莓派设备，这个问题也困扰了好几天，没找到直接的解决方法（比如跳过或者设置超时），最后直接注释。

## 成功安装

最后使用pip3安装，大功告成！

```

pi@raspberrypi:~ $ sudo pip3 install lxml
Collecting lxml
  Using cached lxml-3.7.3.tar.gz
Building wheels for collected packages: lxml
  Running setup.py bdist_wheel for lxml ... done
  Stored in directory: /root/.cache/pip/wheels/df/32/5f/0acd510ac7d66ebe5f35155508972fa732ec45acd5f79146d2
Successfully built lxml
Installing collected packages: lxml
Successfully installed lxml-3.7.3
```
