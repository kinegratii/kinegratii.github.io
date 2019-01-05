---
title: WordScapes(纵横填字母)游戏工具
date: 2018-10-29 18:54:38
categories: 编程
---

WordScapes 是一个有关英文单词的简单游戏，游戏要求从打乱的字母选择若干个组成一系列有效的单词，填入图中的纵横框。

游戏链接： https://play.google.com/store/apps/details?id=com.peoplefun.wordcross

# 1 概述

在玩游戏时，写了一个简单的脚本， 列出从给定的字母中排序组成和合法英文单词。

<!-- more -->

![WordScapes](https://user-images.githubusercontent.com/9875406/49424677-48213d00-f7d6-11e8-8bc0-acf41dad27b9.jpg)

对于该局游戏，依次使用下面命令：

```shell
$ python word_scapes.py dvmeo 3
med
emo
doe
mod
ode
$ python word_scapes.py dvmeo 4
dome
move
mode
dove
demo
$ python word_scapes.py dvmeo 5
moved
```

需要注意的是，脚本的输出包括了缩略词，但在 WordScapes 中，缩略词不被认为是一个有效的单词。

# 2 安装和部署

脚本仅依赖 拼音检查库 [PyEnchant](https://sourceforge.net/projects/pyenchant/)

需要先安装Enchant，才能再安装 PyEnchant，可以使用下面的命令安装：

```shell
apt-get install enchant
pip install pyenchant
```

在 PyEnchant 中最主要的就是 `Dict` 对象，我们可以使用它来检查单词的拼写是否正确，使用的方式也很简单：

```shell
>>> import enchant
>>> d = enchant.Dict("en_US")
>>> d.check("Hello")
True
>>> d.check("Helo")
False
```

# 3 API文档

该脚本共有两种使用方式。

## 方式 1: 生成特定长度的英文单词  

**命令**

```shell
$ python word_scapes.py LETTERS [LENGTH]
```

**注意**

- `LENGTH` 默认为字母个数
- `LENGTH` 的有效值为 1 ~ 字母个数

**例子**

```shell
$ python word_scapes.py dvmeo
moved
$ python word_scapes.py dvmeo 4
move
dome
demo
mode
dove
```

## 方式 2: 生成符合特定模式的单词

**命令**

```shell
$ python word_scapes.py <LETTERS> <PATTERN>
```

**注意**

- `PATTERN` 由字母和 `*` 组成，一个 `*` 字符代表 `LETTERS` 中的任意一个字符
- `PATTERN` 参数需要使用 `''` 包裹起来

**例子**

```shell
$ python word_scapes.py dvmeo 'm**'
mod
med
$ python word_scapes.py dvmeo '*e*'
med
```

# 4 源代码

```python
# coding=utf8

"""
Usage
scapes.py arest
scapes.py arest 2
scapes.py arest '**e**'
"""
import sys
from itertools import permutations

import enchant


def list_words_with_count(letters, count):
    for word in permutations(letters, count):
        yield ''.join(word)


def list_words_with_pattern(letters, pattern):
    random_chars = list(letters)
    random_count = 0
    for c in pattern:
        if c == '*':
            random_count += 1
        else:
            try:
                random_chars.remove(c)
            except ValueError:
                raise ValueError('Invalid letter in pattern: {}'.format(c))

    for chars in permutations(random_chars, random_count):
        _p = list(pattern)
        i = 0
        for _i, _c in enumerate(_p):
            if _c == '*':
                _p[_i] = chars[i]
                i += 1
        yield ''.join(_p)


def list_words(letters, param):
    if param is None or isinstance(param, int):
        g = list_words_with_count(letters, param)
    else:
        g = list_words_with_pattern(letters, param)

    d = enchant.Dict('en_US')
    word_set = {w for w in g if d.check(w)}
    print('\n'.join(word_set))


if __name__ == '__main__':
    args = sys.argv[1:]
    if len(args) == 2:
        letters, param = args
        try:
            param = int(param)
        except ValueError:
            pass
    else:
        letters = args[0]
        param = None
    list_words(letters, param)
```
