---
title: windows+django+apache+wsgi项目部署
date: 2016-12-09 22:43:28
categories: 编程
tags:
- Python
- 部署
- Django
- windows
---

## 基本环境

- Windows 7 32位
- Python 2.7 32位
- Django 1.10
- Apache 2.2

## 目录文件位置

| 名称 | 文件目录 |
| ------ | ------ |
| virtualenv | D:/env/dj110/ |
| Django项目目录 |D:/nms |
| wsgi.py | D:/nms/nms/wsgi.py |
| 静态文件目录 | D:/static/nms/ |
| 上传文件目录 | D:/upload/nms/ |
| Apache目录 | C:/Apache22/ |

<!-- more -->

## 步骤

1 下载 Apache 2.2 32位，并解压至 `C:\Apache22\` 。

2 下载mod_wsgi模块文件，将 mod_wsgi 放入 `C:\Apache2.2\modules` 目录。

3 编写 http.conf 文件。

```

WSGIPythonPath D:/nms;D:/env/dj110/Lib/site-packages;

#----------------------------------------------------------------------
#virtual host for nms
<VirtualHost *:8020>
ServerAdmin test@163.com
DocumentRoot D:/nms

Alias /static/ D:/static/nms/
Alias /upload/ D:/upload/nms/

<Directory D:/static/nms>
Order deny,allow
Allow from all
</Directory>

<Directory D:/upload/nms>
Order deny,allow
Allow from all
</Directory>

WSGIScriptAlias / D:/nms/nms/wsgi.py

<Directory D:/nms/nms/>
<Files wsgi.py>
Order deny,allow
Allow from all
</Files>
</Directory>
</VirtualHost>
```

## Q&A

**静态文件404**

使用django 的collectstatic 命令将所有的静态文件收集到一个目录下，可以不在Django项目下，然后使用Alias由Apache接管/static/的访问。

**静态文件403**

Alias /static/ D:/static/nms/中static文件目录最后需要加上目录分隔符 /。
