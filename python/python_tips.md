# Python Tips

平时使用和看文档时的总结

## 相对导入
Since the name of the main module is always "\_\_main\_\_", modules intended for use as the main module of a Python application must always use absolute imports.

```
from . import xxx # 当前路径导入
from .. import xxx # 上一级路径导入
from ... import xxx # 上上一级路径导入
```
如果一个模块被直接运行，则它自己为顶层模块，不存在层次结构，所以找不到其他的相对路径。
Python 解释器判断一个 py 文件属于哪个 package 时并不完全由该文件所在的文件夹决定。它还取决于这个文件是如何 load 进来的（**直接运行 or import**）。

有两种方式加载一个 py 文件：
- 作为 top-level 脚本
   作为 top-level 脚本指的是直接运行脚本，比如 python myfile.py。有且只能有一个 top-level 脚本，就是最开始执行的那个（比如 python myfile.py 中的 myfile.py）。
- 作为 module
   作为 module 是指，执行 python -m myfile，或者在其它 py 文件中用 import 语句来加载，那么它就会被当作一个 module。

例如，moduleX 被 import 进来，它的名字就是 package.subpackage1.moduleX。如果 import 了 moduleA，它的名字是 package.moduleA。如果直接运行 moduleX 或 moduleA，那么名字就都是`__main__`了。

所以上面的moduleX的`__name__`是`__main__`， 因为他是直接运行的， moduleY的`__name__`是`sub_pkg1.moduleY`，因为他是被import 来使用的。
**总结：不建议用相对导入，如果需要导入上级目录或者平级目录可以通过sys.path.append("..")来导入**
例如：
如在file4.py中想引入import在dir3目录下的file3.py。

这其实是前面两个操作的组合，其思路本质上是将上级目录加到sys.path里，再按照对下级目录模块的方式导入。

同样需要被引文件夹也就是dir3下有空的__init__.py文件。

-- dir
　　| file1.py
　　| file2.py
　　| dir3
　　　| __init__.py
　　　| file3.py
　　| dir4
　　　| file4.py

同时也要将上级目录加到sys.path里：
```
import sys
sys.path.append("..")
from dir3 import file3
```

## pip3安装软件

### 安装pip3

在debian上直接通过**sudo apt-get install python3-pip**安装

### pip3国内源
清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
阿里云 http://mirrors.aliyun.com/pypi/simple/
中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
豆瓣(douban) http://pypi.douban.com/simple/
中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/

### pip3下载
pip3 install xxxx -i https://pypi.tuna.tsinghua.edu.cn/simple/ --timeout=100 #用清华源下载数据 timeout设定100s
### pip3配置文件
~/.pip/pip.conf文件，没有则创建
```
[global]
timeout = 100
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = http://pypi.tuna.tsinghua.edu.cn
```

### pip3下载的文件

linux中通过pip3下载的文件一般都在下边这个目录

~/.local/bin # 可执行程序

~/.local/lib # 库python文件

## python中变量都是地址的引用

```
a = 1 # a指向了内存为1的地址，a->1


a = 1 # a->1

b = a # b->1

a = 2 # a->2

>>>print(b)
1
```
## del命令可以删除变量

val = 0

del val

执行后val再不能当做变量引用

## 不可变变量和可变变量
不可变变量：数字、字符串、元组（在其中的可变变量可以修改）
可变变量：列表、字典

## 字符串”符号和‘符号
在“12345”字符串中加入’不需要转义，同理在‘12345’中加入“不需要转义

## python中没有switch，利用if...elif...elif...else

```
if x==1:
    print("xxx")
elif x==2:
    print("xx")
elif x==3:
    print("x")
else:
    print("")
```
## dict.items()返回key和value
```
dict = {"1": a, "2": b}
for i, d in dict.iterms():
    print(i, d)
```
## for循环通过enumerate来处理索引，一般list或者tuple通过其来生成索引，enumerate(dict)
```
list = [1, 2, 3, 4]
for index, data in enumerate(list):
    print(index, data)

0 1
1 2
2 3
3 4
----------------------------------------------
dict = {"fst": 1, "sec": 2}
for i, val in enumerate(a):
    print("%d: %s" %(i, val))

0: fst
1: sec
```
## for循环的else
如果在循环中没有经过break那么就会执行else的语句，如果经过了break 那么就不会执行else的语句
```
for x in range(10):
    if x % 2 == 0:
        print("it can be divided by 2")
        break
else:
    print("no one can be divided by 2")
```
## pass的使用
pass多用来做占位，如：
```
class a():
    pass
def b():
    pass
```
## 函数注释的添加docstring，要养成该习惯
```
def test_docstring():
    """
    this is a test docstring
    """
    print("this is a test docstring")

help (test_docstring)
会打印注释
```
## 全局变量，非局部变量
- 通过global声明全局变量：
  1. 如果在函数中有同样名称的局部变量（没有用global声明）则在函数内部使用的是局部变量。
  2. 如果在函数中没有与全局变量同名的局部变量（没有global声明）的话，如果只读那么读到的是全局变量，直接写的话会报错（g += 1）
- nolocal来声明外围变量（非全局变量）
只在闭包里面生效，作用域就是闭包里面的，外函数和内函数都影响，但是闭包外面不影响。
```
b = 0
def test_global():
    global b; #在局部使用全局变量时需要在局部声明该变量
    print("local print b = %d" % b)
print("global print b = %d" % b)

def test_global():
    a = 10
    def test_nonlocal():
        nonlocal a
        a += 10
        print("local print nonlocal a = %d" % a)
    print("print local a = %d", %a)

```
下边例子说明了global 和nonlocal和local的区别
```
def scope_test():
    def do_local():
        spam = "local spam" #新建了一个local变量和字符串绑定
    def do_nonlocal():
        nonlocal spam #将上一层spam变量和一个字符串绑定
        spam = "nonlocal spam"
    def do_global():
        global spam #声明一个全局变量
        spam = "global spam" #将这个全局变量和字符串绑定
    spam = "test spam"
    do_local() #给函数内local字符串赋值
    print("After local assignment:", spam) #打印当前local字符串
    do_nonlocal() #给函数外nonlocal变量赋值（当前的local字符串赋值）
    print("After nonlocal assignment:", spam) #打印当前local字符串
    do_global() #给global变量赋值
    print("After global assignment:", spam) #打印当前local字符串

scope_test()
print("In global scope:", spam) #打印global字符串
-----------------------------------------------------------------------
After local assignment: test spam
After nonlocal assignment: nonlocal spam
After global assignment: nonlocal spam
In global scope: global spam
```
## in的使用
可以通过**in**关键词来测试是否包含一个特定的值。
```
# test = ("a", "b", "c"); it's ok
test = ["a", "b", "c"]
if a in test:
    print("a in test")

```
## is的使用
在**if**语句中可以通过**is**命令来判断，注意**is**和**==**有区别
```
a=1
if a is 1:
    print("a is 1")
```
## 函数参数*arguments(positional arguments) **keyarguments(keyword arguments)
- \*arguments作为参数时会将参数表示成tuple，\*\*keyartuments会将参数表示成dictionary。\*arguments必须在 \*\*keyarguments之前

```
def cheeseshop(kind, *arguments, **keywords):
    print("-- Do you have any", kind, "?")
    print("-- I'm sorry, we're all out of", kind)
    for arg in arguments:
        print(arg)
    print("-" * 40)
    for kw in keywords:
        print(kw, ":", keywords[kw])


-- Do you have any Limburger ?
-- I'm sorry, we're all out of Limburger
It's very runny, sir.
It's really very, VERY runny, sir.
----------------------------------------
shopkeeper : Michael Palin
client : John Cleese
sketch : Cheese Shop Sketch
```
- 函数调用时做实参传入
```
def test_args(first, second, third, fourth, fifth):
    print 'First argument: ', first
    print 'Second argument: ', second
    print 'Third argument: ', third
    print 'Fourth argument: ', fourth
    print 'Fifth argument: ', fifth

# Use *args
args = [1, 2, 3, 4, 5]
test_args(*args)
# results:
# First argument:  1
# Second argument:  2
# Third argument:  3
# Fourth argument:  4
# Fifth argument:  5

# Use **kwargs
kwargs = {
    'first': 1,
    'second': 2,
    'third': 3,
    'fourth': 4,
    'fifth': 5
}

test_args(**kwargs)
# results:
# First argument:  1
# Second argument:  2
# Third argument:  3
# Fourth argument:  4
# Fifth argument:  5
```
- 函数参数的特殊字符
```
def f(pos1, pos2, /, pos_or_kwd, *, kwd1, kwd2):
      -----------    ----------     ----------
           |              |              |
           |              |              |
           |              |              |
           |              |         Keyword only
      Positional only     |
                          |
               Positional or keyword
```
## 字典数据结构（dictionary）
- dict = {key: value} #key可以是string也可以是数字，value也是
- 通过del可以删除dict 某一项
del dict[key]
- list(dict)列出所有的key
- sorted(dict)，按照key来排序
- in dict，来判断是否是字典的key
## set数据结构
- 没有顺序
- set()函数来创建
- 两个set可以进行and or not等来运算
## 判断是否是通过直接执行方式运行脚本
通过python3 test.py来执行时 \_\_name\_\_ == "__main__"
```
...
if __name__ == "__main__":
    ...
```

## dir()函数
该函数可以查看模块中的名称（定义的函数等等），但是不会列出built in的名称，可以通过dir(builtins)来查看built in的名称

## 一个.py文件是一个module，多个.py文件组成package（该文件夹需要有__init__.py）
- 在package里边的\_\_init\_\_.py中有\_\_all\_\_，通过from xxx import \*会将\_\_all\_\_中的module导入.
- Since the name of the main module is always "\_\_main\_\_", modules intended for use as the main module of a Python application must always use absolute imports.
```
__all__ = ["echo", "surround", "reverse"]


from xxx import xxx #会导入echo.py surround.py reverse.py
```
## with的方式打开文件，系统会自动关闭
```
with open("myfile.txt") as f:
    for line in f:
        print(line, end="")
```
## 简单类的定义
```
class Complex:
    def __init__(self, realpart, imagpart):
        self.r = realpart
        self.i = imagpart
    def f(self):
        return "hello world"
```

## 类成员变量可以在使用时定义
```
class Complex:
    def __init__(self, realpart, imagpart):
        self.r = realpart
        self.i = imagpart
    def f(self):
        return "hello world"
        
x = Complex()
x.a = 10 #定义了一个Complex的成员变量a
```
## 类方法和对象方法，类变量和对象变量
```
class Complex:
    class_v = "all the same in class"
    def __init__(self, realpart, imagpart):
        self.r = realpart
        self.i = imagpart
    def f(self):
        return "hello world"
        
x = Complex()
x.f() #等价于Complex.f(x)
```

- 类变量不用self来修饰如，class_v，类变量被所有对象共有，**当该变量是可变变量时，使用时要注意。修改任意一个对象的类变量也会修改其他的，当变量是不可变变量时，不会有影响（可能是因为当是可变变量时，所有对象的类变量都指向同一个区域，改变时所有对象都会跟着变，当是不可变变量修改时，不能在变量原地修改，那个类变量会指向不同区域，因此不会有影响）**
- 对象变量用self来修饰如，self.r，每个对象有自己的变量，不共享。
## 类的继承
```
class subclass(baseclass):
    ...
```
如果父类在其他module中可以通过以下方式继承
```
class subclass(module.baseclass):
    ...
```

- 继承后可以在子类的\_\_init\_\_()函数中执行super().\_\_init\_\_(...)，但不是必须
- 在子类中调用父类的方法
父类的名字.method就可以（module.baseclass.method()），或者用super().method()
- isinstance()可以检查实例的类型
- issubclass()检查继承
- 私有变量和方法
1. 通过用"\_\_"来修饰变量和方法为私有变量和方法，也可以强制调用（不建议）
2. 私有属性或者方法不能在类外或者继承的对象中使用，只允许在本类中使用。调用时需要注意添加self.\_\_xxxx

## 注意常用的库
import os
import sys
import re
import glob
import argparse
import logging
import threading

import xxx后，可以通过dir(xxx)查看库中的字段，然后通过help(xxx_xxx)查看帮助，xxx_xxx可以是具体的对象的方法例如：
```
file_test = open("test", "r+")
help(file_test.readline)
```

## python的执行流程
.py被翻译成字节码生成.pyc，然后将其加载到python虚拟机（PVM）中执行。
以上的过程都是在执行中执行。

## try ... except ... finally ...
try语句中有异常发生那么会执行except中的语句，finally中的语句无论一场是否发生，都会执行。
```
a = 0
b = 1
try:
    c = b/a
except Exception as e:
    print("出现异常--->" + str(e))
finally:
    print("always exe")
```



## python重新加载module
通过import xxxx，只能加载一次模块，下次再执行import的时候不会再重新加载，通过以下方式可以再次加载module
```
import importlib
importlib.reload(xxx)

```
## 通过import xxx as A，简写
例如：
```
import os as O
O.chdir("xxx")
```

## logging模块的简单使用
```
import logging

# create logger
logger = logging.getLogger('simple_example')
logger.setLevel(logging.DEBUG)

# create console handler and set level to debug
ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)

# create formatter
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

# add formatter to ch
ch.setFormatter(formatter)

# add ch to logger
logger.addHandler(ch)

# 'application' code
logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.error('error message')
logger.critical('critical message')

# 输出是：
$ python simple_logging_module.py
2005-03-19 15:10:26,618 - simple_example - DEBUG - debug message
2005-03-19 15:10:26,620 - simple_example - INFO - info message
2005-03-19 15:10:26,695 - simple_example - WARNING - warn message
2005-03-19 15:10:26,697 - simple_example - ERROR - error message
2005-03-19 15:10:26,773 - simple_example - CRITICAL - critical message
```
## http协议请求
- GET：
GET请求会将请求通过URL网址传递信息，将信息转话到URL中向服务器提交。
- POST：
比较安全的提交方式，如用户登录时
- PUT：
请求服务器存储一个资源，通常要指定存储位置
- DELETE：
请求服务器删除一个资源
- HEAD：
请求获取服务器包头信息
- OPTIONS：
可以获取当前URL所支持的请求类型

## XPath

    XPath，全称 XML Path Language，即 XML 路径语言，它是一门在 XML 文档中查找信息的语言。初是用来搜寻 XML 文档的，但同样适用于 HTML 文档的搜索。所以在做爬虫时完全可以使用XPath 做相应的信息抽取。python中使用lxml库

  - 常用规则

    | 表达式   | 描述                     |
    | -------- | ------------------------ |
    | nodename | 选取此节点的所有子节点   |
    | /        | 从当前节点选区直接子节点 |
    | //       | 从当前节点选取子孙节点   |
    | .        | 选取当前节点             |
    | ..       | 选取当前节点的父节点     |
    | @        | 选取属性                 |

  - 用以 // 开头的 XPath 规则来选取所有符合要求的节点

    ```python
    from lxml import etree
    html = etree.parse('./test.html', etree.HTMLParser())
    result = html.xpath('//li') #选取所有的li的节点
    print(result)
    ```

  - 通过 / 或 // 即可查找元素的子节点或子孙节点。选择 li 节点的所有直接 a 子节点

    ```python
    from lxml import etree
    html = etree.parse('.test.html', etree.HTMLParser())
    result = html.xpath('//li/a') #选取所有的li节点下的直接a子节点
    print(result)
    ```

  - 知道子节点，查询父节点可以用 .. 来实现

    ```python
    from lxml import etree
    html = etree.parse('./test.html', etree.HTMLParser())
    #a节点href属性为link4.html的节点且属性为class的节点的父节点
    result = html.xpath('//a[@href="link4.html"]/../@class')
    print(result)
    ```

  - 属性匹配

    ```python
    from lxml import etree
    html = etree.parse('./test.html', etree.HTMLParser())
    #li节点，并且class属性为item-inactive
    result = html.xpath('//li[@class="item-inactive"]')
    print(result)
    ```

  - 文本获取

    ```python
    from lxml import etree
    html = etree.parse('./test.html', etree.HTMLParser())
    #获取li节点，class属性为item=0子节点的文本
    result = html.xpath('//li[@class="item-0"]/a/text()')
    print(result)
    ```

  - 属性获取

    ```python
    from lxml import etree
    html = etree.parse('./test.html', etree.HTMLParser())
    #获取li节点字节点为a，属性为href的值
    result = html.xpath('//li/a/@href')
    print(result)
    ```

  - 属性多值匹配

    ```python
    from lxml import etree
    text = '''
    <li class="li li-first"><a href="link.html">first item</a></li>
    '''
    html = etree.HTML(text)
    #li节点class属性包含li字节点为a的文字
    result = html.xpath('//li[contains(@class, "li")]/a/text()')
    print(result)
    ```

  - 扩展运算符

    | 运算符 | 描述           | 实例              | 返回值                               |
    | :----- | :------------- | :---------------- | :----------------------------------- |
    | or     | 或             | age=18 or age=20  | age=18：True；age=21：False          |
    | and    | 与             | age>18 and age<21 | age=20：True；age=21：False          |
    | mod    | 计算除法的余数 | 5 mod 2           | 1                                    |
    | \|     | 计算两个节点集 | //book \| //cd    | 返回所有拥有 book 和 cd 元素的节点集 |
    | +      | 加法           | 5 + 3             | 8                                    |
    | -      | 减法           | 5 - 3             | 2                                    |
    | *      | 乘法           | 5 * 3             | 15                                   |
    | div    | 除法           | 8 div 4           | 2                                    |
    | =      | 等于           | age=19            | 判断简单，不再赘述                   |
    | !=     | 不等于         | age!=19           | 判断简单，不再赘述                   |
    | <      | 小于           | age<19            | 判断简单，不再赘述                   |
    | <=     | 小于等于       | age<=19           | 判断简单，不再赘述                   |
    | >      | 大于           | age>19            | 判断简单，不再赘述                   |
    | >=     | 大于等于       | age>=19           | 判断简单，不再赘述                   |

  - 按照顺序选择

    匹配结果有多个节点，需要选中第二个或最后一个，可以按照中括号内加索引或其他相应语法获得：

      ```python
      from lxml import etree
      text = '''
      <div>
      <ul>
      <li class="item-0"><a href="link1.html">first item</a></li>
      <li class="item-1"><a href="link2.html">second item</a></li>
      <li class="item-inactive"><a href="link3.html">third item</a></li>
      <li class="item-1"><a href="link4.html">fourth item</a></li>
      <li class="item-0"><a href="link5.html">fifth item</a>
      </ul>
      </div>
      '''
      html = etree.HTML(text)
      # 获取第一个
      result = html.xpath('//li[1]/a/text()')
      print(result)
      # 获取最后一个
      result = html.xpath('//li[last()]/a/text()')
      print(result)
      # 获取前两个
      result = html.xpath('//li[position()<3]/a/text()')
      print(result)
      # 获取倒数第三个
      result = html.xpath('//li[last()-2]/a/text()')
      print(result)
      
      """
      运行结果：
      ['first item']
      ['fifth item']
      ['first item', 'second item']
      ['third item']
      """
      ```

  - 节点轴选择
    XPath 提供了很多节点轴选择方法，包括子元素、兄弟元素、父元素、祖先元素等：

  ```python
  from lxml import etree
  
  text = '''
  <div>
  <ul>
  <li class="item-0"><a href="link1.html"><span>first item</span></a>    </li>
  <li class="item-1"><a href="link2.html">second item</a></li>
  <li class="item-inactive"><a href="link3.html">third item</a></li>
  <li class="item-1"><a href="link4.html">fourth item</a></li>
  <li class="item-0"><a href="link5.html">fifth item</a>
  </ul>
  </div>
  '''
  
  html = etree.HTML(text)
  # 获取所有祖先节点
  result = html.xpath('//li[1]/ancestor::*')
  print(result)
  # 获取 div 祖先节点
  result = html.xpath('//li[1]/ancestor::div')
  print(result)
  # 获取当前节点所有属性值
  result = html.xpath('//li[1]/attribute::*')
  print(result)
  # 获取 href 属性值为 link1.html 的直接子节点
  result = html.xpath('//li[1]/child::a[@href="link1.html"]')
  print(result)
  # 获取所有的的子孙节点中包含 span 节点但不包含 a 节点
  result = html.xpath('//li[1]/descendant::span')
  print(result)
  # 获取当前所有节点之后的第二个节点
  result = html.xpath('//li[1]/following::*[2]')
  print(result)
  # 获取当前节点之后的所有同级节点
  result = html.xpath('//li[1]/following-sibling::*')
  print(result)
  
  """
  [<Element html at 0x231a8965388>, <Element body at 0x231a8965308>,     <Element div at 0x231a89652c8>, <Element ul at 0x231a89653c8>]
  [<Element div at 0x231a89652c8>]
  ['item-0']
  [<Element a at 0x231a89653c8>]
  [<Element span at 0x231a89652c8>]
  [<Element a at 0x231a89653c8>]
  [<Element li at 0x231a8965308>, <Element li at 0x231a8965408>, <Element   li at 0x231a8965448>, <Element li at 0x231a8965488>]
  """
  ```

## python logging简单使用

    ```python
    import os
    import logging  # 引入logging模块
    logging.basicConfig(level=logging.DEBUG,
                        format='%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s')  # logging.basicConfig函数对日志的输出格式及方式做相关配置
    # 由于日志基本配置中级别设置为DEBUG，所以一下打印信息将会全部显示在控制台上
    #fh = logging.FileHandler(logfile, mode='w')
    #fh.setLevel(logging.DEBUG)  # 输出到file的log等级的开关
    def test_log():
        logging.info('this is a loggging info message')
        logging.debug('this is a loggging debug message')
        logging.warning('this is loggging a warning message')
        logging.error('this is an loggging error message')
        logging.critical('this is a loggging critical message')
    
    def test_log_file():
        #set path/file
        log_path = os.path.dirname(os.getcwd())
        log_name = log_path + '/log'
        logfile = log_name
        logging.info(log_name)
        #set logger
        logger = logging.getLogger()
        fh = logging.FileHandler(logfile, mode='w')
        fh.setLevel(logging.DEBUG)  # 输出到file的log等级的开关
        formatter = logging.Formatter("%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s")
        fh.setFormatter(formatter)
        logger.addHandler(fh)
        #logging
        logging.info('this is a loggging_file info message')
        logging.debug('this is a loggging_file debug message')
        logging.warning('this is loggging_file a warning message')
        logging.error('this is an loggging_file error message')
        logging.critical('this is a loggging_file critical message')
    ```

