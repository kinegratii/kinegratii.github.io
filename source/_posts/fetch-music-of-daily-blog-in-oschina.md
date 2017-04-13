---
title: 爬取OSC每日乱弹的音乐
date: 2017-02-13 08:24:58
categories: 编程
tags:
- 开源中国
- 异步IO
- 爬虫
---

使用aiohttp + lxml/爬取OSC乱弹的歌曲


## 一 原理分析

**爬取路径与结果**

爬取路径：博客列表 =》 提取博客链接 =》 获取博客内容 =》 提取歌曲信息，最后获取的结果包含了博客链接、博客标题、歌曲链接，歌曲标题的一个列表。
```python
[
    {'blog_url': 'xxx', 'blog_title': 'xxx', 'music_url': 'xxx', 'music_title': 'xxx'},
    {'blog_url': 'xxx', 'blog_title': 'xxx'},# 该博客歌曲提取失败
]
```

**起点url**

爬取的起点url为博客列表的页面链接。
```
https://my.oschina.net/xxiaobian/blog?catalog=547834&sort=time&p={page}
```
其中page表示页码，范围取值为1-24,25页之后的乱弹就没有网易云音乐的歌曲了。

**歌曲链接**

歌曲链接的格式为：`http://music.163.com/#/song?id=5038302`或者`http://music.163.com/#/song/187747/`，可归纳为以`http://music.163.com/#/song`开头的均符合要求。

**歌曲标题**

首先在博客中查找。歌曲标题通常为 `- ` 分割的一整个段落，表示歌曲和歌手。比如最新博客使用“单独段落+书名号”（如`<p>《Victory》- Two Steps From Hell<p>`）这种就比较好匹配，没有书名号的就很少能够提取到。

在博客文档中无法找到（通常是由于匹配规则无法覆盖所有情况），向云音乐网站获取数据，注意两点：
- 需要将 `http://music.163.com/#/song?id=28285557` 转换成 `http://music.163.com/song?id=28285557` 格式
- 网页标题去掉末尾的字符串“- 网易云音乐”即是所需要的歌手歌曲信息。

<!-- more -->

## 二 实现

**协程并发执行**

程序的核心利用协程实现并发执行，和多线程不同，所有代码都在一个线程执行，减少上下文切换，无需要线程之间的锁机制，效率可大大提高。

> 协程看上去也是子程序，但执行过程中，在子程序内部可中断，然后转而执行别的子程序，在适当的时候再返回来接着执行。  ——  《[协程 - 廖雪峰的官方网站](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432090171191d05dae6e129940518d1d6cf6eeaaa969000)》

程序主要使用了以下几个库：

- asyncio：Python3标准库，异步IO，
- aiohttp:异步HTT库，类似于requests的异步实现
- lxml：html解析，使用xpath语法
- re：正则表达式

按照[aiohttp](http://aiohttp.readthedocs.io/en/stable/#getting-started)的文档将每一个HTTP请求和内容解析都包装成一个协程函数：

- `fetch_music_title(song_url)`获取歌手歌曲信息
- `fetch_music(blog_url)` 获取歌曲信息
- `fetch_blog_list(page)` 获取博客列表

根据逻辑将这三个协程包裹新的协程`fetch_page_music(page)`，用于获取每一页博客中的歌曲信息列表。程序运行时，为每一页创建一个`fetch_page_music`协程并放进事件循环，待所有协程执行完整后处理相关数据并显示。

**代码注释**

- `loop.run_until_complete()` 返回值为各个协程函数的返回值组成的列表
- `//a[@class="blog-title"]`表示获取class为blog-title所有a元素
- `//a[starts-with(@href, "http://music.163.com/")]`表示获取href值以http://music.163.com/ 开头的所有a元素。
- `itertools.chain.from_iterable(iterable)`将数组“压平”，如 `[[1,2], [3,4],[5,6]]` 返回 `[1,2,3,4,5,6]`

## 三 运行原理结果

![执行结果](https://static.oschina.net/uploads/img/201702/09163540_RXFl.png "执行结果")

## 四 其他

**源代码**

链接：[爬取OSC乱弹的歌曲](https://git.oschina.net/kinegratii/codes/hjug0mlicztve9yd18xn553)

（要求Python3.5+）

**爬取结果**

一共爬取了185首歌曲，码云代码片段没有md功能，存自己博客上了，[点击访问链接](https://kinegratii.github.io/2017/02/13/osc-daily-blog-music/)。
