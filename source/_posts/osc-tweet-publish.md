---
title: 自动发送动弹到开源中国
date: 2016-05-17 20:53:09
categories:
 - 编程
tags:
 - Python
 - 开源中国
---

自动发送动弹到开源中国。

1. 登录系统，密码使用sha1算法加密。
2. 获取用户信息，包括user_code和user_id。
3. 发送动弹。

代码仅在Python3有效。

<!-- more -->

```python
# coding=utf8

from __future__ import unicode_literals

import hashlib
from datetime import datetime
import re

import requests


class OSCRobot:
    HEADERS = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1500.72 Safari/537.36'
    }

    def __init__(self, *, username, password, user_code=None, user_id=None):
        self.username = username
        self.password = password
        self.user_code = user_code
        self.user_id = user_id
        self.client = requests.session()
        self.client.headers = self.HEADERS

    def login(self, **kwargs):
        if not (self.user_id and self.user_code):
            self._login(**kwargs)

    def _login(self, **kwargs):
        data = {
            'email': self.username,
            'pwd': hashlib.sha1(self.password.encode('utf8')).hexdigest()
        }
        has_login = False
        login_rsp = self.client.post('https://www.oschina.net/action/user/hash_login', data)
        if login_rsp.status_code == 200:
            home_rsp = self.client.get('https://www.oschina.net/')
            if home_rsp.status_code == 200:
                text = home_rsp.text
                m_regx = re.search(r":bind=\"user_code\"\svalue='(?P<user_code>[a-zA-Z0-9]+)'", text)
                if m_regx:
                    self.user_code = m_regx.group('user_code')
                    m_regx = re.search(r":bind=\"user\"\svalue='(?P<user_id>\d+)'", text)
                    if m_regx:
                        self.user_id = m_regx.group('user_id')
                        has_login = True
                    else:
                        print('[Fail] No user id found!')
                else:
                    print('[Fail] No user code found!')
            else:
                print('[Fail] Request home page fail with status code {}'.format(home_rsp.status_code))
        else:
            print('[Fail] Login fail with status_code {}'.format(login_rsp.status_code))
        if has_login:
            print('[Success] Login success! User info')
        else:
            print('[Fail] fail to login!')
        return has_login

    def publish_tweet(self, msg, **kwargs):
        post_data = {
            'user_code': self.user_code,
            'user': self.user_id,
            'msg': msg
        }
        rsp = self.client.post('https://www.oschina.net/action/tweet/pub', post_data)
        if 200 <= rsp.status_code < 300:
            print('[Success] Push tweet success!')
        else:
            print('[Fail] Fail to push tweet!')


def main():
    robot = OSCRobot(
        username='YOUR USERNAME',
        password='YOUR PASSWORD'
    )
    robot.login()
    robot.publish_tweet('Hello everyone! The current time is {}'.format(datetime.now().strftime('%Y-%m-%d %H:%M:%S')))


if __name__ == '__main__':
    main()

```
