---
title: 博客V5.0升级笔记
date: 2019-09-30 10:48:54
categories:
 - 技术研究
comments: false
---

2019年9月30日，本博客升级了相关库，具体信息如下：

| 库              | 现有版本 | 原有版本 |
| --------------- | -------- | -------- |
| hexo-cli        | 2.0.0    | 1.1.0    |
| hexo            | 3.9.0    | 3.2.2    |
| hexo-theme-next | 7.4.0    | 6.3.0    |

<!-- more -->

## 更新内容

- Hexo 更新至 v3.9.0
- NextT 主题更新至 v7.4.0
- 新增 pjax 特性
- 新增 github Banner
- 新增相关文章模块
- 更新正文字体大小

## 升级步骤

### 依赖库更新

和正常 Node 项目一致，使用 npm 更新即可。

更新全局 hexo-cli

```shell
npm update -g hexo-cli
```

进入博客目录

查看可升级的包：

```
> npm outupdated
Package                  Current  Wanted  Latest  Location
hexo                       3.2.2   3.9.0   3.9.0  Kinegratii-blog
hexo-deployer-git          0.2.0   0.2.0   2.0.0  Kinegratii-blog
hexo-generator-archive     0.1.4   0.1.5   1.0.0  Kinegratii-blog
hexo-generator-category    0.1.3   0.1.3   1.0.0  Kinegratii-blog
hexo-generator-feed        1.2.0   1.2.2   2.0.0  Kinegratii-blog
hexo-generator-index       0.2.0   0.2.1   1.0.0  Kinegratii-blog
hexo-generator-searchdb    1.0.3   1.0.8   1.0.8  Kinegratii-blog
hexo-generator-sitemap     1.1.2   1.2.0   1.2.0  Kinegratii-blog
hexo-generator-tag         0.2.0   0.2.0   1.0.0  Kinegratii-blog
hexo-renderer-ejs          0.2.0   0.2.0   1.0.0  Kinegratii-blog
hexo-renderer-marked      0.2.11  0.2.11   2.0.0  Kinegratii-blog
hexo-renderer-stylus       0.3.1   0.3.3   1.1.0  Kinegratii-blog
hexo-server                0.2.0   0.2.2   1.0.0  Kinegratii-blog
hexo-symbols-count-time    0.4.4   0.4.4   0.6.1  Kinegratii-blog
hexo-wordcount             2.0.1   2.0.1   6.0.1  Kinegratii-blog
```

全局安装 npm-check工具

```shell
npm install -g npm-check
```

更新所有包

```shell
npm-check -u --registry https://registry.npm.taobao.org
```

### Next主题更新

几个文档采用手动合并更新：

- next主题配置文件：_config.yaml  （必须以现有配置文件为基础）
- 中文翻译文件： zh-CN.yaml
- 插件库：lib
- 插件库版本忽略文件：.gitignore
