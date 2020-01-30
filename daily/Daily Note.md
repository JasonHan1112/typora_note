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