# Daily Note
- 2019/9/9
  - 修改read_pcie_cfg write_pcie_cfg interface, 使其发出1DW TLP
  - 并修改mmap不能写访问的bug
  - 参考i40e驱动
- 2019/9/12
  - linux的问题多从dmesg里找答案
    - dmesg | grep bug 
  - **竟然忘记了**  free查看内存使用情况

- 2019/9/26
  - 在x86环境下配置使用MP0 CCP的记录
    1. 需要修改MP0 CCP初始化代码，使其空间能被public进行访问
    2. 修改cmd queue的存放地址，（给硬件必须是物理地址，在DDR上）
    3. 需要关闭IOMMU，由于IOMMU的表没有更新，告诉CCP的物理地址不能被正确的路由到正确的ddr地址
    4. 注意查看ERROR STATUS，【MP0_CCP_CMD_STATUS_Q0， PSPCCP_DMA_READ_STATUS_Q0， PSPCCP_DMA_WRITE_STATUS_Q0】当有ERROR_DESCRIPTOR错误时说明cmd有问题，如果是ERROR_DATA说明是返回的数据有问题。
    5. 遇到的问题：DDR-*MMIO（没有ERROR STATUS），MMIO-*DDR（有ERROR DATA）由于RC发出write request是post，而read是non-post，bar空间有些不允许访问，会回CA，导致有ERROR STATUS
- 2019/9/28
  - Makefile vpath
Makefile文件中的特殊变量“VPATH”就是完成这个功能的，如果没有指明这个变量，make只会在当前的目录中去找寻依赖文件和目标文件。如果定义了这个变量，那么，make就会在当当前目录找不到的情况下，到所指定的目录中去找寻文件了。
    VPATH = src:../headers
上面的的定义指定两个目录，“src”和“../headers”，make会按照这个顺序进行搜索。**目录由“冒号”分隔**。（当然，当前目录永远是最高优先搜索的地方）
另一个设置文件搜索路径的方法是使用make的“vpath”关键字（注意，它是全小写的）， 这不是变量，这是一个make的关键字，这和上面提到的那个VPATH变量很类似，但是它更为灵活。它可以指定不同的文件在不同的搜索目录中。这是一个很 灵活的功能。它的使用方法有三种：
    1、vpath *pattern* *directories*
    为符合模式*pattern*的文件指定搜索目录*directories*。
    2、vpath *pattern*
    清除符合模式*pattern*的文件的搜索目录。
    3、vpath
    清除所有已被设置好了的文件搜索目录。
vapth使用方法中的*pattern*需要包含“%”字符。“%”的意思是匹 配零或若干字符，例如，“%.h”表示所有以“.h”结尾的文件。*pattern*指定了要搜索的文件集， 而*directories*则指定了*pattern*的文件集的搜索的目录。例如：
    vpath %.h ../headers
该语句表示，要求make在“../headers”目录下搜索所有以“.h”结尾的文件。（如果某文件在当前目录没有找到的话）
我们可以连续地使用vpath语句，以指定不同搜索策略。如果连续的vpath语句中出现了相同的*pattern*，或是被重复了的*pattern*，那么，make会按照vpath语句的先后顺序来执行搜索。如：
    vpath %.c foo
    vpath %   blish
    vpath %.c bar
其表示“.c”结尾的文件，先在“foo”目录，然后是“blish”，最后是“bar”目录。
    vpath %.c foo:bar
    vpath %   blish
而上面的语句则表示“.c”结尾的文件，先在“foo”目录，然后是“bar”目录，最后才是"blish"目录

- 2019/10/8
  - VCO 指输出频率与输入控制电压有对应关系的振荡电路(VCO)，频率是输入信号电压的函数的振荡器VCO，振荡器的工作状态或振荡回路的元件参数受输入控制电压的控制，就可构成一个压控振荡器。
![新建位图图像.bmp](attachments\3b423a74.bmp)
图 2是克拉泼型LC压控振荡器的原理电路。图中，T为晶体管，L为回路电感，C1、C2、Cv为回路电容,Cv为变容二极管反向偏置时呈现出的容量;C1、C2通常比Cv大得多。当输入控制电压uc改变时，Cv随之变化，因而改变振荡频率。


- 2019/10/25
宏定义调试打印
```
#ifdef DEBUG
    #define dg_print(...) printf(__VA_ARGS__)
#else
    #define dg_print(...)
#endif

#define nd_print(...) printf(__VA_ARGS__)
``````


- 2019/11/4
  - vim替换

    替换标志
上文中命令结尾的g即是替换标志之一，表示全局global替换（即替换目标的所有出现）。 还有很多其他有用的替换标志：
空替换标志表示只替换从光标位置开始，目标的第一次出现：

    :%s/foo/bar
i表示大小写不敏感查找，I表示大小写敏感：

    :%s/foo/bar/i
等效于模式中的\c（不敏感）或\C（敏感）

    :%s/foo\c/bar
c表示需要确认，例如全局查找"foo"替换为"bar"并且需要确认：

    :%s/foo/bar/gc
回车后Vim会将光标移动到每一次"foo"出现的位置，并提示
replace with bar (y/n/a/q/l/^E/^Y)?
按下y表示替换，n表示不替换，a表示替换所有，q表示退出查找模式， l表示替换当前位置并退出。^E与^Y是光标移动快捷键，参考： Vim中如何快速进行光标移

- 2019/11/6
在vim中命令模式下  !  表示要执行命令如ls grep等，%表示
在vim下看二进制通过%!xxd来将文字转换成16进制
vim替换^M,在命令行模式下输入:%s/cntl+v cntl+m//gc
- 2019/11/10
export 这个是用来提供该子目录makefile（sub make）中访问的,同一级的另外一个makefile中，是无法访问/得到的。
(可以通过makefile中内置变量MAKELEVEL查看得知当前makefile的levlel)

- 2019/11/17
linux 中cut工具，cut -d: -f1-3[按照:划分取1到3列]



- 2019/12/4
```
#if 1
#elif 2
#elif 3
#endif

```
```
#if defined TEST
#else defined TEST
#endif

```
- 2019/12/4
  - 通过nc可以创建listen，和发送消息
nc -l -t 9988
nc localhost 9988

- 2019/12/6
  - bash 算数运算
  1. let 表达式
```
x=3
y=5
let z1=3+5;         echo $z1    # output: 8, notice no whitespace is allowed.
let z1=x+y;         echo $z1    # output: 8
let z1=$x+$y;       echo $z1    # output: 8
```
  2. expr 表达式
```
let z1=$x+$y;       echo $z1    # output: 8
```
  3. 双括号表达式
```
((z=3+5));          echo $z # output: 8
((z = 3 + 5));      echo $z # output: 8
```
- 2019/12/7
  
- vim 向外界复制
vim --version | grep clipboard
看是否有+clipboard，如果有那么说明可以向剪切板复制
（1）选中文字
（2）"+y进行复制
  （3）"+p进行粘贴
  
- 2019/12/11
  通过ip addr add xxx.xxx.xxx.xxx/24 dev enos1
  设置ip后需要ip link set enos1 up来启动网卡 否则设置不成功。

- 2019/12/30
  TLB可以理解为页表cache。快表，直译为旁路快表缓冲，也可以理解为页表缓冲，地址变换高速缓存。TLB是一种高速缓存，内存管理硬件使用它来改善虚拟地址到物理地址的转换速度。当前所有的个人桌面，笔记本和服务器处理器都使用TLB来进行虚拟地址到物理地址的映射。使用TLB内核可以快速的找到虚拟地址指向物理地址，而不需要请求RAM内存获取虚拟地址到物理地址的映射关系。这与data cache和instruction caches有很大的相似之处。
  想像一下x86_32架构下没有TLB的存在时的情况，对线性地址的访问，首先从PGD中获取PTE（第一次内存访问），在PTE中获取页框地址（第二次内存访问），最后访问物理地址，总共需要3次RAM的访问。如果有TLB存在，并且TLB hit，那么只需要一次RAM访问即可。

- https://hygon.webex.com/hygon/j.php?MTID=m07b1bafdfc0a0fd4a5bf3dfefabc5605

- 2020/2/4

  1. grep -v "xxx": 对选择的内容取反查找

  2. 将多行替换成一行

     - 实现方法

       將文件內連續的空白行 , 刪除它們成為一行

       sed -e '/^$/{N;/\n$/D};'

       參照[section4.16])表示 , 將空白行的下一行資料添加至 pattern space 內。函數參數 /^$/D  表示 , 當添加的是空白行時 , 刪除第一行空白行 , 而且剩下的空白行則再重新執行指令一次。指令重新執行一次 , 刪除一行空白行 ,  如此反覆直至空白行後添加的為非空白行為止 , 故連續的空白行最後只剩一空白行被輸出。 

     - 详细分析

        函数参数 D 表示删除 pattern space 内的第一行资料。其指令格式如下: 

       [address1,address2]D 
       
       对上述格式有下面几点说明 : 

       函数参数 D 最多配合两个地址参数。 
       函数参数 D 与 d 的比较如下 : 
       当 pattern space 内只有一数据行时 , D 与 d 作用相同。 
       当 pattern space 内有多行资料行时 
       D 表示只删除 pattern space 内第一行资料 ; d 则全删除。 
       D 表示执行删除后 , pattern space 内不添加下一笔数据 , 而将剩下的数据重新执行 sed script ; d 则读入下一行后执行 sed script。

       例如：

       1. This line is followed by 1 blank line. 
       2.  
       3. This line is followed by 2 blank lines. 
       4.  
       5.  
       6. This line is followed by 3 blank lines. 
       7.  
       8.  
       9.  
       10. This line is followed by 4 blank lines. 
       11.  
       12.  
       13.  
       14.  
       15. This is the end. 
       
        我想得到的效果是

       1. This line is followed by 1 blank line. 
       2.  
       3. This line is followed by 2 blank lines. 
       4.  
       5. This line is followed by 3 blank lines. 
       6.  
       7. This line is followed by 4 blank lines. 
       8.  
       9. This is the end. 

       
        代码如下

       1. /^$/{ 
       2. N 
       3. /^\n$/d 
       4. } 

       执行后效果如下

       1. This line is followed by 1 blank line. 
       2.  
       3. This line is followed by 2 blank lines. 
       4. This line is followed by 3 blank lines. 
       5.  
       6. This line is followed by 4 blank lines. 
       7. This is the end. 

       这个过程是这样的  sed是一行一行读入数据的，

       首先读入第一行，因为并不匹配，所以直接打印出来，如下：

       1. This line is followed by 1 blank line. 

       然后读入第二行，匹配，所以N继续读入第三行，然后再与/^\n$/进行匹配，很明显不匹配，所以内容也直接打印出来，如下：

       1. This line is followed by 1 blank line. 
       2.  
       3. This line is followed by 2 blank lines. 

       接着读入第四行，匹配，所以N继续读入第5行，然后与/^\n$/进行匹配，因为此时读入的为2个空行，显然是匹配的这时候此2行被删除，所以此时打印的结果依旧如上。

       接着再读入第5行，不匹配，直接打印出来，如下：


       1. This line is followed by 1 blank line. 
       2.  
       3. This line is followed by 2 blank lines. 
       4. This line is followed by 3 blank lines. 
    
       接着读入第6行，匹配，N继续读入第7行，显然继续匹配，所以次2行被删除
    
       接着读入8行，匹配，N继续读入第9行，不匹配，所以把第8 9行打印出来，如下
    
       1. This line is followed by 1 blank line. 
       2.  
       3. This line is followed by 2 blank lines. 
       4. This line is followed by 3 blank lines. 
       5.  
       6. This line is followed by 4 blank lines. 
    
       接着读入第10行，匹配，N继续读入第11行，显然继续匹配，所以次2行被删除
    
       接着读入第12行，匹配，N继续读入第13行，显然继续匹配，所以次2行被删除
    
       最后读入最后14行，不匹配，直接打印，如下：


       1. This line is followed by 1 blank line. 
       2.  
       3. This line is followed by 2 blank lines. 
       4. This line is followed by 3 blank lines. 
       5.  
       6. This line is followed by 4 blank lines. 
       7. This is the end. 
    
       而代码使用D
    
       1. /^$/{ 
       2. N 
       3. /^\n$/D 
       4. } 
    
       他的流程就不同了


       首先读入第一行，因为并不匹配，所以直接打印出来，如下：
    
       1. This line is followed by 1 blank line. 


​       
​        然后读入第二行，匹配，所以N继续读入第三行，然后再与/^\n$/进行匹配，很明显不匹配，所以内容也直接打印出来，如下：


       1. This line is followed by 1 blank line. 
       2.  
       3. This line is followed by 2 blank lines. 


​       
​        接着读入第四行，匹配，所以N继续读入第5行，然后与/^\n$/进行匹配，因为此时读入的为2个空行，显然是匹配的，所以D把第4行删除，剩下第5行，但是此时第5行并不会打印出来，而是作为读入，继续运行这个脚本，也就是说第5行又先匹配/^$/,然后执行N，读入第6行，而此时内容已经不匹配/^\n$/，这样第5行和第6行就直接打印出来了，如下：


       1. This line is followed by 1 blank line. 
       2.  
       3. This line is followed by 2 blank lines. 
       4.  
       5. This line is followed by 3 blank lines. 
    
       接着读入第7行，此时匹配，然后N继续读入第8行，依旧匹配，所以执行D，删除第7行，而第8行继续匹配，重新执行脚本，N继续读入第9行，此时依旧匹配，所以删除第8行，第9行继续匹配，N读入第10行，而此时不再匹配，所以打印出第9和10行，如下：


       1. This line is followed by 1 blank line. 
       2.  
       3. This line is followed by 2 blank lines. 
       4.  
       5. This line is followed by 3 blank lines. 
       6.  
       7. This line is followed by 4 blank line. 
    
       然后读入11直到最后一行，最后得到所需要的结果


       1. This line is followed by 1 blank line. 
       2.  
       3. This line is followed by 2 blank lines. 
       4.  
       5. This line is followed by 3 blank lines. 
       6.  
       7. This line is followed by 4 blank line. 
       8.  
       9. This is the end. 
- 2020/2/4

1. du查看文件夹使用情况
2. debian在改变字体时，需要修改/etc/locale.gen，并且需要执行locale-gen命令

- 2020/5/12

  git submodule init 

  git submodule update

  git fetch origin remote_branch: local_branch

- 2020/5/16

  ```c
  //xxx.h 有可能在cpp中被引用，也可能在c中被引用
  #ifdef __cplusplus//如果是cpp代码（c++）
    extern "C" {//在cpp的代码中，告诉编译器下边的代码是用C写成的，需要用C的处理方式来链接该内容
  #endif
  
    ...
    
  #ifdef __cplusplus
    }
  #endif
  
  ```

  - C++为了支持重载机制，在编译生成的汇编码中需要对函数的名字、参数类型、返回值类型等进行处理，而在C中只是简单的函数名字而已，不会加入其他信息，也就是说C++和C对产生的函数名字处理是不一样的地方。

  - 在C的代码中不能出现extern “C”，否则会编译出错。

  - 一般声明放在头文件中，当我们函数有肯能被C或C++使用，但是我们无法确定实在.c还是.cpp中定义，也就无法确定是否该需要extern "C"，因此需要以上方法来进行处理

- 2020/5/16

  ```c
  __attribute__(visibility("default"))//被导出到库的符号表
  __attribute__(visibility("hidden"))//被hidden，但是还保留在符号表中用于静态链接
  ```
  程序调用某函数A，A函数在两个动态库中（liba.so, libb.so）执行需要链接两个库，此时程序调用的A函数来自于哪个库呢？
  以上问题取决于链接的先后顺序，如果先链接liba.so，那么在到处符号表就可以看到函数A的定义，并加入到符号表中，链接libb.so的时候，符号表中已经存在A，就不会在更新符号表，那后续调用的始终是liba.so的A函数。为了避免后续的混乱，通过attribute来确定最终执行的是哪个函数。

- 2020/5/28

  gcc中({...})会被视为一条语句，该语句的结果是最后一条语句

  ```c
  #define typecheck(type, x)\
  ({ type __dumy;\
      typeof(x) __dummy2;\
      (void)(&__dummy == &dummy2);\
      1;\
  })
  ```

  上述代码返回1

- 2020/7/6

  - git lfs

    1. git lfs的安装：

       sudo apt install git-lfs

       git lfs install (install git lfs configuration)

    2. git lfs的使用(每一次有更改要提交的时候都需要执行下边的步骤)：

       step 1: git lfs track "*.bin" (追踪仓库中的所有.bin文件，生成.gitattributes)

       step 2: git add . (注意一定要在step 1之后，并且一定要添加好.gitattributes)

       step 3: git commit ...

       step 4: git push origin master
       
   3. git config --global credential.helper store (保存lfs devcenter的密码)
  
   - git 打补丁
  
     1. path 和 diff两种方案，git diff(只记录文件的改动)和git format-patch(记录文件改动并带有commit记录)
     2. git format-patch [commit sha1 id] -n (n指的是从sha1 id对应的commit开始算起n个提交，如 -1 -2)
     3. git format-patch [commit1 sha1 id]..[commit2 sha1 id]某两次提交的所有patch，会生成好多patch文件0001... 0002... 0003... 0004...
     4. 打入patch: git apply --check [xxxx.patch]
     5. 版本回退，patch撤销：
        - git reset: 默认的方式，回退到某个版本，只保留回退前的源码，回退commit和index(暂存区)
        - git reset --soft: 回退到某个版本，只回退了commit。index和源码都还保留，如果还需要提交只需要commit即可
        - git reset --hard: 彻底回退到某个版本，本地的源码也会变成为上一个版本。
     
   - git submodule
  
  项目的版本库在某些情况下需要引用其他版本库中的文件，例如有一套公用的代码库，可以被多个项目调用，这个公用代码库能直接放在某个项目的代码中，而是要独立为一个代码库，那么其他要调用公用的代码库该如何处理？分别把公用的代码库拷贝到各自的项目中会造成冗余，丢弃了公共代码库的维护历史，这些显示不是好的办法，现在要了解的git子模组(**git submodule**)就解决了这个问题。
      
      - 制作submodules的仓库
      
        1. 将两个仓库分别建好。（repo1, repo2）
        2. 将两个仓库分别拉到本地。
        3. 在repo2中执行git submodule add ...../path/repo1.git ./lib，该操作进行submodule的关联
  
  
  ​      
  ​    
      - 使用submodules的仓库
      
        git submodule init
      
        git submodule update
  
  
  ​    
  

- 2020/7/7

  ### gdb交互命令

  启动gdb后，进入到交互模式，通过以下命令完成对程序的调试；注意高频使用的命令一般都会有缩写，熟练使用这些缩写命令能提高调试的效率；

  #### 运行

  - run：简记为 r ，其作用是运行程序，当遇到断点后，程序会在断点处停止运行，等待用户输入下一步的命令。
  - continue （简写c ）：继续执行，到下一个断点处（或运行结束）
  - next：（简写 n），单步跟踪程序，当遇到函数调用时，也不进入此函数体；此命令同 step 的主要区别是，step 遇到用户自定义的函数，将步进到函数中去运行，而 next 则直接调用函数，不会进入到函数体内。
  - step （简写s）：单步调试如果有函数调用，则进入函数；与命令n不同，n是不进入调用的函数的
  - until：当你厌倦了在一个循环体内单步跟踪时，这个命令可以运行程序直到退出循环体。
  - until+行号： 运行至某行，不仅仅用来跳出循环
  - finish： 运行程序，直到当前函数完成返回，并打印函数返回时的堆栈地址和返回值及参数值等信息。
  - call 函数(参数)：调用程序中可见的函数，并传递“参数”，如：call  gdb_test(55)
  - quit：简记为 q ，退出gdb

  #### 设置断点

  - - break n （简写b n）:在第n行处设置断点

      （可以带上代码路径和代码名称： b OAGUPDATE.cpp:578）

  - b fn1 if a＞b：条件断点设置

  - break func（break缩写为b）：在函数func()的入口处设置断点，如：break cb_button

  - delete 断点号n：删除第n个断点

  - disable 断点号n：暂停第n个断点

  - enable 断点号n：开启第n个断点

  - clear 行号n：清除第n行的断点

  - info b （info breakpoints） ：显示当前程序的断点设置情况

  - delete breakpoints：清除所有断点：

  #### 查看源代码

  - list ：简记为 l ，其作用就是列出程序的源代码，默认每次显示10行。
  - list 行号：将显示当前文件以“行号”为中心的前后10行代码，如：list 12
  - list 函数名：将显示“函数名”所在函数的源代码，如：list main
  - list ：不带参数，将接着上一次 list 命令的，输出下边的内容。

  #### 打印表达式

  - print 表达式：简记为 p ，其中“表达式”可以是任何当前正在被测试程序的有效表达式，比如当前正在调试C语言的程序，那么“表达式”可以是任何C语言的有效表达式，包括数字，变量甚至是函数调用。
  - print a：将显示整数 a 的值
  - print ++a：将把 a 中的值加1,并显示出来
  - print name：将显示字符串 name 的值
  - print gdb_test(22)：将以整数22作为参数调用 gdb_test() 函数
  - print gdb_test(a)：将以变量 a 作为参数调用 gdb_test() 函数
  - display 表达式：在单步运行时将非常有用，使用display命令设置一个表达式后，它将在每次单步进行指令后，紧接着输出被设置的表达式及值。如： display a
  - watch 表达式：设置一个监视点，一旦被监视的“表达式”的值改变，gdb将强行终止正在被调试的程序。如： watch a
  - info function： 查询函数
  - 扩展info locals： 显示当前堆栈页的所有变量

  #### 查询运行信息

  - where/bt ：当前运行的堆栈列表；
  - bt backtrace 显示当前调用堆栈
  - up/down 改变堆栈显示的深度
  - set args 参数:指定运行时的参数
  - show args：查看设置好的参数
  - info program： 来查看程序的是否在运行，进程号，被暂停的原因。

  #### 分割窗口

  - layout：用于分割窗口，可以一边查看代码，一边测试：
  - layout src：显示源代码窗口
  - layout asm：显示反汇编窗口
  - layout regs：显示源代码/反汇编和CPU寄存器窗口
  - layout split：显示源代码和反汇编窗口
  - Ctrl + L：刷新窗口ll