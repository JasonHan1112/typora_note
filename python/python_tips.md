# Python Tips

## 相对导入

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

