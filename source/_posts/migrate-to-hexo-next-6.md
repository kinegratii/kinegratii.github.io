---
title: Hexo NexT 6升级笔记
date: 2018-07-23 15:18:52
categories: 技术研究
tags:
- 软件迁移
---

 博客V5.0已发布，参见 [博客V5.0升级笔记](/2019/09/30/blog_upgrade_note/) 。

## A 概述

本文叙述了将 NexT 5.1.0 迁移到 6.3.0 的过程中的一些笔记。使用 NexT 6 的考虑在于：

- 优化配置项
- 支持 Valine 评论系统
- 代码快支持复制功能
- 其他从 v5.1.0 起内置的功能

<!-- more -->

## B 准备工作

如何安装 NexT 6 有多种选择，这里仅介绍的是一种简单机械的方法。

1. **备份** : 将 *themes\next* 文件夹复制备份到另外一个目录下，然后删除原有的 next 文件夹。

2. **下载**：打开 [代码仓库](https://github.com/theme-next/hexo-theme-next) ，切换到标签 v6.3.0 ，点击 “Download zip” 下载压缩包。

3. **替换**: 解压并重名为 next ，放入 *themes* 目录。

> 在 github 访问缓慢时，与 克隆代码 和 浏览器下载 相比，使用迅雷工具下载会提高不少的速度。


## C 迁移步骤

### 文件配置

通常来说，需要改动一下几个文件：

- 博客配置文件 *_config.yml*
- 主题配置文件 *themes\next\_config.yml*
- 语言翻译文件 *themes\next\languages\zh-CN.yml*

### 语言&翻译

在 NexT 6 中，简体中文的名称变为 zh-CN ，因此在 **博客配置文件** 里需要将原有的：

```yaml
language: zh-Hans
```

改为

```yaml
language: zh-CN
```

另外现在语言文件名称为 *zh-CN.yaml* ，格式没有变化，可以使用原有的 *zh-Hans.yaml* 覆盖。

### 菜单链接

NexT 6 将分开的 链接 和 图标 配置合并在了一个配置项中，并使用 || 分割。受这一特性影响，以下配置项目有所变化，需手动修改：

- 菜单 theme.menu
- 社交 theme.social
- 代码Fork theme.github_banner

Hexo 5 风格：

```yaml
menu:
  home: /
  archives: /archives
  topics: /topics
  library: /library
  about: /about
menu_icons:
  enable: true
  home: home
  archives: archive
  topics: file
  library: book
  about: user
```

Hexo 6 风格：

```yaml
menu:
  home: / || home
  archives: /archives/ || archive
  topics: /topic/ || file
  library: /library/ || book
  about: /about/ || user
```

### 其他配置

这些配置无任何变化，将配置内容复制到对应选项即可。

- 右下角的“回到顶部”按钮
- 打赏文字、图片 theme.reward_comment
- 网站 icon 图片 theme.favicon
- 本地搜索 theme.local_search

## D 插件

这些功能之前是使用修改源代码方式，现在可以使用 “包/插件引入 + 选项配置” 的方式激活该功能。

> 插件路径定义在 *themes\next\source\lib* 目录下。

| 功能 | 插件 | 引入方式 | 配置项 |
| ------ | ------ | ------ | ------ |
| 字数统计 | [hexo-symbols-count-time](https://github.com/theme-next/hexo-symbols-count-time) | 包 | theme.symbols_count_time |
| 图片浏览 | [theme-next-fancybox3](https://github.com/theme-next/theme-next-fancybox3) | 插件 | theme.fancybox |
| 顶部进度条 | [theme-next-pace](https://github.com/theme-next/theme-next-pace) | 插件 | theme.pace |
| leancloud访问计数 | [leancloud-visitors](https://github.com/theme-next/hexo-leancloud-counter-security) | 插件 | theme.leancloud_visitors |

**引入方式**

以包方式引入比较简单，使用 命令 `npm install <package-name> -save` 即可。

以插件方式引入，在 *theme\next* 目录使用代码克隆命令。

```
git clone <github-url> source\lib\<plugin-name>
```

**进度条**

进度条使用 [pace.js](http://github.hubspot.com/pace/) 插件，[点此](http://github.hubspot.com/pace/docs/welcome/) 查看每个配置的效果图。

## E 评论系统

NexT 6 已经集成这个功能了，可以使用和访问量同一个应用。

1 在云端的 leancloud 应用中创建一个名为 `Comment` 的类，使用默认的 ACL 权限设置。

2 在主题配置文件中设置 app_id 和 app_key 即可。

```yaml
valine:
  enable: true
  appid:   # your leancloud application appid
  appkey:  # your leancloud application appkey
  notify: false # mail notifier , https://github.com/xCss/Valine/wiki
  verify: false # Verification code
  placeholder: Just go go # comment box placeholder
  avatar: mm # gravatar style
  guest_info: nick,mail # custom comment header
  pageSize: 10 # pagination size
```

## F 部署

修改 *theme\next\.gitignore* 文件，将 *theme\next\source\lib* 下的文件也提交到版本库。

具体做法是删除以下内容：

```
# Ignore optional external libraries
source/lib/*

# Track internal libraries & Ignore unused verdors files
source/lib/font-awesome/less/
source/lib/font-awesome/scss/
!source/lib/font-awesome/*

!source/lib/jquery/

source/lib/ua-parser-js/*
!source/lib/ua-parser-js/dist/

!source/lib/velocity/
```

## G 参考资料

- [hexo-theme-next](https://github.com/theme-next/hexo-theme-next)
