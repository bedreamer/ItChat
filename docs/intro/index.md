# itchat

[![Gitter][gitter-picture]][gitter] ![py27][py27] ![py35][py35] [English version][english-version]

itchat是一个开源的微信个人号接口，使用python调用微信从未如此简单。

使用不到三十行的代码，你就可以完成一个能够处理所有信息的微信机器人。

当然，该api的使用远不止一个机器人，更多的功能等着你来发现，比如[这些][tutorial2]。

如今微信已经成为了个人社交的很大一部分，希望这个项目能够帮助你扩展你的个人的微信号、方便自己的生活。

## 安装

可以通过本命令安装itchat：

```python
pip install itchat
```

## 简单入门实例

有了itchat，如果你想要给文件传输助手发一条信息，只需要这样：

```python
import itchat

itchat.auto_login()

itchat.send('Hello, filehelper', toUserName='filehelper')
```

如果你想要回复发给自己的文本消息，只需要这样：

```python
import itchat

@itchat.msg_register(itchat.content.TEXT)
def text_reply(msg):
    return msg['Text']

itchat.auto_login()
itchat.run()
```

一些进阶应用可以在下面的开源机器人的源码和进阶应用中看到，或者你也可以阅览[文档][document]。

## 试一试

这是一个基于这一项目的[开源小机器人][robot-source-code]，百闻不如一见，有兴趣可以尝试一下。

![QRCode][robot-qr]

## 截屏

![file-autoreply][robot-demo-file] ![login-page][robot-demo-login]

## 进阶应用

### 各类型消息的注册

通过如下代码，微信已经可以就日常的各种信息进行获取与回复。

```python
#coding=utf8
import itchat, time
from itchat.content import *

@itchat.msg_register([TEXT, MAP, CARD, NOTE, SHARING])
def text_reply(msg):
    itchat.send('%s: %s' % (msg['Type'], msg['Text']), msg['FromUserName'])

@itchat.msg_register([PICTURE, RECORDING, ATTACHMENT, VIDEO])
def download_files(msg):
    msg['Text'](msg['FileName'])
    return '@%s@%s' % ({'Picture': 'img', 'Video': 'vid'}.get(msg['Type'], 'fil'), msg['FileName'])

@itchat.msg_register(FRIENDS)
def add_friend(msg):
    itchat.add_friend(**msg['Text']) # 该操作会自动将新好友的消息录入，不需要重载通讯录
    itchat.send_msg('Nice to meet you!', msg['RecommendInfo']['UserName'])

@itchat.msg_register(TEXT, isGroupChat=True)
def text_reply(msg):
    if msg['isAt']:
        itchat.send(u'@%s\u2005I received: %s' % (msg['ActualNickName'], msg['Content']), msg['FromUserName'])

itchat.auto_login(True)
itchat.run()
```

### 命令行二维码

通过以下命令可以在登陆的时候使用命令行显示二维码：

```python
itchat.auto_login(enableCmdQR=True)
```

部分系统可能字幅宽度有出入，可以通过将enableCmdQR赋值为特定的倍数进行调整：

```python
# 如部分的linux系统，块字符的宽度为一个字符（正常应为两字符），故赋值为2
itchat.auto_login(enableCmdQR=2)
```

默认控制台背景色为暗色（黑色），若背景色为浅色（白色），可以将enableCmdQR赋值为负值：

```python
itchat.auto_login(enableCmdQR=-1)
```

### 退出程序后暂存登陆状态

通过如下命令登陆，即使程序关闭，一定时间内重新开启也可以不用重新扫码。

```python
itchat.auto_login(hotReload=True)
```

### 用户搜索

使用`search_friends`方法可以搜索用户，有四种搜索方式：
1. 仅获取自己的用户信息
2. 获取特定`UserName`的用户信息
3. 获取备注、微信号、昵称中的任何一项等于`name`键值的用户
4. 获取备注、微信号、昵称分别等于相应键值的用户

其中三、四项可以一同使用，下面是示例程序：

```python
# 获取自己的用户信息，返回自己的属性字典
itchat.search_friends()
# 获取特定UserName的用户信息
itchat.search_friends(userName='@abcdefg1234567')
# 获取任何一项等于name键值的用户
itchat.search_friends(name='littlecodersh')
# 获取分别对应相应键值的用户
itchat.search_friends(wechatAccount='littlecodersh')
# 三、四项功能可以一同使用
itchat.search_friends(name='LittleCoder机器人', wechatAccount='littlecodersh')
```

关于公众号、群聊的获取与搜索在文档中有更加详细的介绍。

### 附件的下载与发送

itchat的附件下载方法存储在msg的Text键中。

发送的文件的文件名（图片给出的默认文件名）都存储在msg的FileName键中。

下载方法接受一个可用的位置参数（包括文件名），并将文件相应的存储。

```python
@itchat.msg_register(['Picture', 'Recording', 'Attachment', 'Video'])
def download_files(msg):
    msg['Text'](msg['FileName'])
    itchat.send('@%s@%s'%('img' if msg['Type'] == 'Picture' else 'fil', msg['FileName']), msg['FromUserName'])
    return '%s received'%msg['Type']
```

如果你不需要下载到本地，仅想要读取二进制串进行进一步处理可以不传入参数，方法将会返回图片的二进制串。

```python
@itchat.msg_register(['Picture', 'Recording', 'Attachment', 'Video'])
def download_files(msg):
    with open(msg['FileName'], 'wb') as f:
        f.write(msg['Text']())
```

### 用户多开

使用如下命令可以完成多开的操作：

```python
import itchat

newInstance = itchat.new_instance()
newInstance.auto_login(hotReload=True, statusStorageDir='newInstance.pkl')

@newInstance.msg_register(TEXT)
def reply(msg):
    return msg['Text']

newInstance.run()
```

### 退出及登陆完成后调用特定方法

登陆完成后的方法需要赋值在`loginCallback`中。

而退出后的方法需要赋值在`exitCallback`中。

```python
import time

import itchat

def lc():
    print('finish login')
def ec():
    print('exit')

itchat.auto_login(loginCallback=lc, exitCallback=ec)
time.sleep(3)
itchat.logout()
```

若不设置loginCallback的值，则将会自动删除二维码图片并清空命令行显示。

## 问题和建议

如果有什么问题或者建议都可以在这个[Issue][issue#1]和我讨论

或者也可以在gitter上交流：[![Gitter][gitter-picture]][gitter]

当然也可以加入我们新建的QQ群讨论：549762872

[gitter-picture]: https://badges.gitter.im/littlecodersh/ItChat.svg
[gitter]: https://gitter.im/littlecodersh/ItChat?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge
[py27]: https://img.shields.io/badge/python-2.7-ff69b4.svg
[py35]: https://img.shields.io/badge/python-3.5-red.svg
[english-version]: https://github.com/littlecodersh/ItChat/blob/master/README_EN.md
[document]: https://itchat.readthedocs.org/zh/latest/
[tutorial2]: http://python.jobbole.com/86532/
[robot-source-code]: https://gist.github.com/littlecodersh/ec8ddab12364323c97d4e36459174f0d
[robot-qr]: http://7xrip4.com1.z0.glb.clouddn.com/ItChat%2FQRCode2.jpg?imageView/2/w/400/
[robot-demo-file]: http://7xrip4.com1.z0.glb.clouddn.com/ItChat%2FScreenshots%2F%E5%BE%AE%E4%BF%A1%E8%8E%B7%E5%8F%96%E6%96%87%E4%BB%B6%E5%9B%BE%E7%89%87.png?imageView/2/w/300/
[robot-demo-login]: http://7xrip4.com1.z0.glb.clouddn.com/ItChat%2FScreenshots%2F%E7%99%BB%E5%BD%95%E7%95%8C%E9%9D%A2%E6%88%AA%E5%9B%BE.jpg?imageView/2/w/450/
[fields.py-2]: https://gist.github.com/littlecodersh/9a0c5466f442d67d910f877744011705
[fields.py-3]: https://gist.github.com/littlecodersh/e93532d5e7ddf0ec56c336499165c4dc
[littlecodersh]: https://github.com/littlecodersh
[tempdban]: https://github.com/tempdban
[Chyroc]: https://github.com/Chyroc
[liuwons-wxBot]: https://github.com/liuwons/wxBot
[zixia-wechaty]: https://github.com/zixia/wechaty
[Mojo-Weixin]: https://github.com/sjdy521/Mojo-Weixin
[issue#1]: https://github.com/littlecodersh/ItChat/issues/1
