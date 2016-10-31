---
title: ionic安卓构建
date: 2016-10-29 20:48:00
categories:
 - 编程
tags:
 - ionic
---

本文描述了ionic项目构建Android安装包的主要步骤。

<!--more-->

## 1 环境配置

### 1.1 添加Android平台

执行  `ionic platform add android` 即可

### 1.2 Java路径配置

```
PATH=path\to\bin\
```

### 1.3 Android环境配置

```
ANDROID_HOME=你的SDK目录
```
将 `%ANDROID_HOME%\tools\;%ANDROID_HOME%\platform_tools\` 加到PATH变量后面。

## 2 Android自动签名

## 2.1 配置release-signing.properties文件

1. 在platforms\android目录新建名为release-signing.properties的文件，文件内容如下

```
storeFile=path/to/keystore
keyAlias=your key alias
storePassword=your store password
keyPassword=your key password
```

备注：在windows下storeFile文件路径应使用Unix下的目录分隔符 `/`。

## 3 编译

使用 `ionic build --release android` 编译即可，在\platforms\android\build\outputs\apk出现android-release.apk文件即是已签名的安装包。

## 4 注意事项

在升级到ionic2时，使用 `ionic build --release android` 可能会出现 `✗ You cannot run iOS unless you are on Mac OSX.`的错误，可改为 `ionic build android --release` 即可。
