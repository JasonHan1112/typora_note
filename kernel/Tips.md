# Tips
## 模块编译的Makefile
```c
# If KERNELRELEASE is defined, we've been invoked from the
# kernel build system and can use its language.
ifneq ($(KERNELRELEASE),)
    obj-m := hello.o
# Otherwise we were called directly from the command
# line; invoke the kernel build system.
else
    KERNELDIR ?= /lib/modules/$(shell uname -r)/build
    PWD  := $(shell pwd)
default:
    $(MAKE) -C $(KERNELDIR) M=$(PWD) modules
endif
```
会invoke这个Makefile两次
1. 如果在commandline中，没有进入内核的编译流程，那么KERNELRELEASE= ,然后会去指定的内核目录去找到内核Makefile，执行内核的Makefile
2. 执行内核的Makefile的时候会KERNELRELEASE != ,从而指定obj-m := hello.o进而编译该模块

## 2019/9/24
  - 在linux内核代码中，系统调用都会用sys_作为前缀
  - MMU相关
    - mmu中不存放页表页目录，只是存放页表的地址。系统初始化代码会在内存中生成页表，然后把页表地址设置给MMU对应寄存器，使MMU知道页表在物理内存中的什么位置，以便在需要时进行查找。之后通过专用指令启动MMU，以此为分界，之后程序中所有内存地址都变成虚地址，MMU硬件开始自动完成查表和虚实地址转换。

## 2019/10/1
内核模块的加载用insmod的时候是通过模块代码.c的名字来区分的。通过同一个名字编译出来的模块都要加载的时候会提示
insmod: ERROR: could not insert module xxx.ko: File exists

## 2019/10/2
```c
/**
 * remap_pfn_range - remap kernel memory to userspace
 * @vma: user vma to map to
 * @addr: target user address to start at
 * @pfn: physical address of kernel memory, must physical page frame number
 * @size: size of map area
 * @prot: page protection flags for this mapping
 *
 *  Note: this is only safe if the mm semaphore is held when called.
 */
int remap_pfn_range(struct vm_area_struct *vma, unsigned long addr,
		    unsigned long pfn, unsigned long size, pgprot_t prot)

```
该函数将指定的物理页帧映射到用户空间，使得多个进程能共享一段物理内存。

## 异步非阻塞 I/O（AIO）
同步和异步的概念描述的是用户线程与内核的交互方式：同步是指用户线程发起IO请求后需要等待或者轮询内核IO操作完成后才能继续执行；而异步是指用户线程发起IO请求后仍继续执行，当内核IO操作完成后会通知用户线程，或者调用用户线程注册的回调函数。阻塞和非阻塞的概念描述的是用户线程调用内核IO操作的方式：阻塞是指IO操作需要彻底完成后才返回到用户空间；而非阻塞是指IO操作被调用后立即返回给用户一个状态值，无需等到IO操作彻底完成。
  - 异步非阻塞 I/O 模型是一种处理与 I/O 重叠(并行)进行的模型。
  - steps：
    > a. 读请求会立即返回，说明 read 请求已经成功发起了。
    > b. 在后台完成读操作时，应用程序然后会执行其他处理操作。
    > c. 当 read 的响应到达时，就会产生一个信号或执行一个基于线程的回调函数来完成这次 I/O 处理过程。
![aio.bmp](attachments\be045c9e.bmp)


## kernel timer
- 介绍
  > - linux内核定时器通过在未来的某个时间点（基于jiffies）调度执行某个特定的函数，其实现位于*linux/timer.h和kernel/timer.c*中。
  > - 被调度的函数肯定是异步执行的，它类似于一种“软件中断”，而且是处于非进程的上下文中，所以调度函数必须遵守以下规则：
  > 1. 没有 current 指针、不允许访问用户空间。因为没有进程上下文，相关代码和被中断的进程没有任何联系。
  > 2. 不能执行休眠（或可能引起休眠的函数）和调度。
  > 3. 任何被访问的数据结构都应该针对并发访问进行保护，以防止竞争条件。
  > - 内核定时器的调度函数运行过一次后就不会再被运行了（相当于自动注销），但可以通过在被调度的函数中重新调度自己来周期运行。


- 在内核中的使用
  > - 内核中的API：
  
     > 1. 初始化定时器：
  
```c
void init_timer(struct timer_list * timer);
```

  >> 2. 增加定时器：

```c
//注册定时器，将定时器放到内核定时器链表中
void add_timer(struct timer_list * timer);
```

  >>3. 删除定时器：

```c
int del_timer(struct timer_list * timer);
```

  >>4. 修改定时器的expire：

```c
int mod_timer(struct timer_list *timer, unsigned long expires);
```
  - > 使用流程
>>（1）timer、编写function；

>>（2）为timer的expires、data、function赋值；

>>（3）调用add_timer将timer加入列表；

>>（4）在定时器到期时，function被执行；

>>（5）在程序中涉及timer控制的地方适当地调用del_timer、mod_timer删除timer或修改timer的expires。