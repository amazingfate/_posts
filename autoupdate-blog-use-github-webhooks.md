---
layout: post
title: 使用github webhooks自动更新博客文章
date: 2016-08-08
tags:
  - blog
  - vps
  - github webhooks
---

  vps迁移到vultr之后，想着荒废已久的blog也该开更了吧。想起来过去的一年还真是没学太多的东西，博客的更新基本处于荒废状态~~叫你装逼用English~~。不过现在想来blog部署在vps上，每次都要顶着高延迟去更新也挺痛苦的，要是能够把这种工作交给机器去做，也许更新会更有积极性吧。在网上一搜就找到了github webhooks这个东西，就是监听git仓库，当有某些事件发生（比如push）会自动发一个POST请求给一个服务器。这样只需要把文章推到github，剩下的都可以让这个请求让服务器去做同步文章，更新博客，比以前ssh到服务器去手动更新不知道高到哪里去了。

  好啦，下面是具体的配置过程

  首先要实现一个可以接收请求的web程序，github的webhooks只是简单的发一个POST请求，所以这个也是相对简单，用python很快就写了一个：

``` python
#!/usr/bin/python3

import tornado.ioloop
import tornado.web
import tornado.options
from tornado.options import define, options
import subprocess
import hmac
from hashlib import sha1
import os

define ("port", default=8765, help="run on the given port", type=int)

class MainHandler(tornado.web.RequestHandler):
    def post(self):
        secret = os.environ['SECRET_TOKEN']
        secret = secret.encode()
        header = self.request.headers.get('X-Hub-Signature')
        sha_name, signature = header.split('=')
        mac = hmac.new(secret, msg=self.request.body, digestmod=sha1)
        if str(mac.hexdigest()) == str(signature):
            self.write('post done')
            args_git_pull = ['git', 'pull']
            args_hexo_generate = ['hexo', 'generate', '2>&1', '>', '/dev/null']
            subprocess.call(args_git_pull, cwd='/home/liujianfeng/blog/source/_posts')
            subprocess.call(args_hexo_generate, cwd='/home/liujianfeng/blog')
        else:
            self.write('token not right, header is %s' % header)
            self.write('mac.hexdigest() is %s' % str(mac.hexdigest()))

application = tornado.web.Application([
    (r"/", MainHandler),
    ])

if __name__ == "__main__":
    application.listen(options.port)
    tornado.ioloop.IOLoop.instance().start()
```

  这里值得注意的是github发的请求本身对内容做了一层加密，详细请看[webhooks的文档](https://developer.github.com/webhooks/securing/)。我们需要在服务端按照同样的方式加密，这样就能得到与github发来一致的X-Hub-Signature。我是把在github设置的SECRET_TOKEN放在了服务器的环境变量里面。最后部署的时候遇到点坑，远程连接vps跑服务没啥问题，但是就算是加了&后台运行，在断开vps的连接后一致是失败，原来是断开连接后程序没有输出的地方了，最后加上nohup运行解决。
