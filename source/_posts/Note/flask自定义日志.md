---
title: flask自定义日志
date: 2020-08-06
tags: [Note,python,flask]
categories:
- [Note,Python]
comments: false
---
# flask自定义日志

## logger

首先我们要了解python实现记录程序日志功能可以使用logging模块，关于logging的用法，继承类自定义日志，日志等级，等等。

日志输出到文件，用FileHandler，RotatingFileHandler

<!-- more -->

## app.logger

看看app.logger源码

[![image-20200806155213786](https://blogjpg.yanmy.top/blog/20200806155213.png)](https://blogjpg.yanmy.top/blog/20200806155213.png)

[![image-20200806155326597](https://blogjpg.yanmy.top/blog/20200806155326.png)](https://blogjpg.yanmy.top/blog/20200806155326.png)

得到几点信息

- 在初始化app时,便创建了一个app.name的日志记录器,其是继承于python的logging模块的
- 该记录器已经添加一个default_handler

写一个小功能:flask实现类似nginx的access.log

```
from flask import Flask, request
from flask.logging import default_handler
from logging import Formatter, INFO
from logging.handlers import RotatingFileHandler

app = Flask(__name__)
app.debug = True


# 主路由index page
@app.route('/', methods=['Get'])
def index():
    return "hello"


# 在每一次请求前执行
@app.before_request
def log_each_request():
    params = request.args if request.args is None else ''
    app.logger.info('{}-{}-{}'.format(request.method, request.path, params))


if __name__ == '__main__':
    # 日志格式化类            记录时间      等级               代码行        日志信息
    formatter = Formatter('%(asctime)s  - %(levelname)s - %(lineno)d - %(message)s')
    info_log_path = 'logs/flask.log'
    # 自动分割日志功能的Handler
    file_handler = RotatingFileHandler(info_log_path, maxBytes=10 * 1024 * 1024, backupCount=10)
    file_handler.setFormatter(formatter)
    file_handler.setLevel(INFO)
    if not app.debug:
        app.logger.removeHandler(default_handler)
        app.logger.addHandler(file_handler)
    app.run()
```

[![image-20200806160238707](https://blogjpg.yanmy.top/blog/20200806160238.png)](https://blogjpg.yanmy.top/blog/20200806160238.png)

如果我们将这行代码注释掉

- app.debug=True的情况下，日志可以输出，但是会将控制台所有的信息都输出到日志文件，这是我们不想的，我们只想要app.logger.info写的内容。
- app.debug=False的情况下，日志不工作了，之前还能全部输出只是有些是我们不想要的，现在都没了，就好像女朋友说分手就分手。

### Why？

跟进default_handler

[![image-20200806161324214](https://blogjpg.yanmy.top/blog/20200806161324.png)](https://blogjpg.yanmy.top/blog/20200806161324.png)

default_handler是一个StreamHandler，将日志记录输出发送到流，例如*sys.stdout*，*sys.stderr*或任何类似文件的对象（或更准确地说，是支持`write()` 和`flush()`方法的任何对象）

所以控制台的输出也会一起输出至你的文件了

app.debug问题：应用程序控制台中看到的输出来自底层的Werkzeug记录器，没有设置logleve，日志就没有了。

## 然后

然后，然后干嘛？

然后就去玩呀😝