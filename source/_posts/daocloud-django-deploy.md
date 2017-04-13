---
title: 使用DaoCloud部署Django项目
date: 2016-07-23 14:56:00
categories:
- 编程
tags:
- Django
- DaoCloud
- 部署
---

本文介绍了在DaoCloud平台部署Django项目的方法。

## 1 DaoCloud云平台

关于DaoCloud云平台。https://www.daocloud.io/

> DaoCloud 为用户提供了 Docker 镜像的自动构建和自动发布功能，当用户完成了 Dockerfile 和 daocloud.yml 文件的编写后，将应用代码推送到第三方代码托管平台上，将其与 DaoCloud 绑定后，在每次修改（commit）后，并将其推送到代码托管平台上，DaoCloud 会检测到代码的变动，并根据 Dockerfile 和 daocloud.yml 进行相应的构建和测试；当触发规定的构建事件（如 tag）时，DaoCloud 会将其进行镜像构建，并推送到相对应的所有生产环节中。

<!-- more -->

## 2 Django项目配置

项目总布局如下：

```
- wcp_platform/
  - admin.py
  - forms.py
  - models.py
  - views.py
- wcp/
  - daocloud_settings.py
  - daocloud_wsgi.py
  - settings.py
  - test_settings.py
  - urls.py
  - wsgi.py
- fixtures/
  - user.json
- static/
- template/
- Dockerfile
- daocloud.yml
- docker-entrypoint.sh
- manage.py
- requirements.txt
```

## 3 基于daocloud的配置

### 3.1 daocloud_settings模块

daocloud_settings模块重写了数据库配置（这里使用了mysql服务）和wsgi配置模块。

```
from __future__ import unicode_literals
from .settings import *
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': os.environ['MYSQL_INSTANCE_NAME'],
        'USER': os.environ['MYSQL_USERNAME'],
        'PASSWORD': os.environ['MYSQL_PASSWORD'],
        'HOST': os.environ['MYSQL_PORT_3306_TCP_ADDR'],
        'PORT': os.environ['MYSQL_PORT_3306_TCP_PORT'],
    }
}

WSGI_APPLICATION = wcp.daocloud_wsgi.application'
```

### 3.2 daocloud_wsgi模块

daocloud_wsgi.py模块设置了环境变量。

```
from __future__ import unicode_literals
import os
from django.core.wsgi import get_wsgi_application
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "wcp.daocloud_settings")
application = get_wsgi_application()
```

### 3.3 Dockerfile

```
FROM daocloud.io/python:2.7
ADD requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt
RUN mkdir /code
WORKDIR /code
COPY . /code
COPY docker-entrypoint.sh docker-entrypoint.sh
RUN chmod +x docker-entrypoint.sh
EXPOSE 8080
CMD /code/docker-entrypoint.sh
```

具体流程

- 安装依赖库
- 拷贝项目代码
- 修改`docker-entrypoint.sh`权限为可执行
- 开放端口
- 执行`docker-entrypoint.sh`

### 3.4 启动脚本

```
ython /code/manage.py migrate --settings=wcp.daocloud_settings --noinput
python /code/manage.py collectstatic -- settings=wcp.daocloud_settings --noinput
/usr/local/bin/gunicorn wcp.daocloud_wsgi:application -w 2 -b :8080 --env DJANGO_SETTINGS_MODULE='wcp.daocloud_settings'
```

启动流程

- 创建数据表
- 收集静态文件
- 使用gunicorn启动Django项目


### 3.5 持续集成：daocloud.yml

```
image: daocloud/ci-python:2.7
script:
  - pip install -r requirements.txt
  - python manage.py test --settings=wcp.test_settings
```
