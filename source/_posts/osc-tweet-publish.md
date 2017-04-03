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

<!-- more -->

```
# coding=utf8
from __future__ import unicode_literals
import requests
import hashlib
import re
from datetime import datetime
class User(object):
    def __init__(self, username, password, user_code=None, user_id=None):
        self.username = username
        self.password = password
        self.user_code = user_code
        self.user_id = user_id
    @property
    def password_hash(self):
        return hashlib.sha1(self.password).hexdigest()
class OSCTweetRobot(object):
    LOGIN_URL = 'https://www.oschina.net/action/user/hash_login'
    HOME_URL = 'https://www.oschina.net/'
    PUBLISH_URL = 'https://www.oschina.net/action/tweet/pub'
    HEADERS = {'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1500.72 Safari/537.36'}
    def __init__(self, user):
        self.user = user
        self.client = requests.session()
        self.client.headers = self.HEADERS
    def login(self, **kwargs):
        data = {
            'email':self.user.username,
            'pwd':self.user.password_hash
        }
        has_login = False
        login_rsp = self.client.post(self.LOGIN_URL, data)
        if login_rsp.status_code == 200:
            home_rsp = self.client.get(self.HOME_URL)
            if home_rsp.status_code == 200:
                content = home_rsp.content
                m_regx = re.search(r"name='user_code' value='(?P<user_code>.*?)'/>", content)
                if m_regx:
                    self.user.user_code = m_regx.group('user_code')
                    m_regx = re.search(r"name='user' value='(?P<user_id>.*?)'/>", content)
                    if m_regx:
                        self.user.user_id =  m_regx.group('user_id')
                        has_login = True
        if has_login:
            print('[Success] Login success! User info')
            print(self.user.__dict__)
        else:
            print('[Fail] fail to login!')
        return has_login
    def publish_tweet(self, msg, **kwargs):
        post_data = {
            'user_code':self.user.user_code,
            'user':self.user.user_id,
            'msg':msg
        }
        rsp = self.client.post(self.PUBLISH_URL,post_data)
        if 200 <= rsp.status_code < 300:
            print('[Success] Push tweet success!')
        else:
            print('[Fail] Fail to push tweet!')
def main():
    u = User('YOUR USERNAME', 'YOUR PASSWORD')
    robot = OSCTweetRobot(u)
    robot.login()
    s = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    # robot.publish_tweet(msg=s)
```
