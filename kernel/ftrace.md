# ftrace

## 简介

ftrace帮助开发人员了解linux内核的运行时行为，以便进行故障调试或性能分析。

主要tracer功能：

1. **Function tracer** 和 **Function graph tracer**: 跟踪函数调用。

2. **Schedule switch tracer**: 跟踪进程调度情况。

3. **Wakeup tracer**：跟踪进程的调度延迟，即高优先级进程从进入 ready 状态到获得 CPU 的延迟时间。该 tracer 只针对实时进程。

4. **Irqsoff tracer**：当中断被禁止时，系统无法相应外部事件，比如键盘和鼠标，时钟也无法产生  tick 中断。这意味着系统响应延迟，irqsoff 这个 tracer  能够跟踪并记录内核中哪些函数禁止了中断，对于其中中断禁止时间最长的，irqsoff 将在 log  文件的第一行标示出来，从而使开发人员可以迅速定位造成响应延迟的罪魁祸首。

5. **Preemptoff tracer**：和前一个 tracer 类似，preemptoff tracer 跟踪并记录禁止内核抢占的函数，并清晰地显示出禁止抢占时间最长的内核函数。

6. **Preemptirqsoff tracer**: 同上，跟踪和记录禁止中断或者禁止抢占的内核函数，以及禁止时间最长的函数。

7. **Branch tracer**: 跟踪内核程序中的 likely/unlikely 分支预测命中率情况。 Branch tracer 能够记录这些分支语句有多少次预测成功。从而为优化程序提供线索。

8. **Hardware branch tracer**：利用处理器的分支跟踪能力，实现硬件级别的指令跳转记录。在 x86 上，主要利用了 BTS 这个特性。

9. **Initcall tracer**：记录系统在 boot 阶段所调用的 init call 。

10. **Mmiotrace tracer**：记录 memory map IO 的相关信息。

11. **Power tracer**：记录系统电源管理相关的信息。

12. **Sysprof tracer**：缺省情况下，sysprof tracer 每隔 1 msec 对内核进行一次采样，记录函数调用和堆栈信息。

13. **Kernel memory tracer**: 内存 tracer 主要用来跟踪 slab allocator 的分配情况。包括 kfree，kmem_cache_alloc 等 API 的调用情况，用户程序可以根据 tracer 收集到的信息分析内部碎片情况，找出内存分配最频繁的代码片断，等等。

14. **Workqueue statistical tracer**：这是一个 statistic  tracer，统计系统中所有的 workqueue 的工作情况，比如有多少个 work 被插入  workqueue，多少个已经被执行等。开发人员可以以此来决定具体的 workqueue 实现，比如是使用 single threaded  workqueue 还是 per cpu workqueue.

15. **Event tracer**: 跟踪系统事件，比如 timer，系统调用，中断等。

这里还没有列出所有的 tracer，ftrace 是目前非常活跃的开发领域，新的 tracer 将不断被加入内核。

## ftrace的使用

- ftrace在内核态工作，用户通过debugfs接口来控制和使用ftrace。ftrace支持两大类tracer：传统tracer和non-tracer tracer
- 在内核中需要加入kernel_hacking---->trace的编译选项，否则在available_tracer中无法出现相关的支持的选项。

### 传统tracer使用

传统tracer的步骤：

- 挂载 debugfs

- 选择一种tracer

- 使能tracer

- 执行需要trace的应用程序，比如需要跟踪ls，就执行ls

- 关闭ftrace

- 查看trace文件

**1. 挂载debugfs（一般内核都已经挂载上在/sys/kernel/debug/tracing）**

如果内核没有挂载则手动挂载

root# mkdir /debug

root# mount -t debugfs nodev /debug

**2. 选择当前tracer**

root# echo function > /debug/tracing/current_tracer

**3. 使能tracer**

root# echo 1 > /debug/tracing/tracing_on

**4. 关闭tracer**

root# echo 0 > /debug/tracing/tracing_on

**5. 查看trace文件**

trace输出信息主要保存在三个文件中：

1. trace：改文件保存输出信息，可以直接阅读。
2. latency_trace: 保存于trace相同的信息，组织方式略有不同。主要为方便用户分析系统有关延迟的信息
3. trace_pipe是一个管道文件，主要方便应用程序读取trace内容。
**6. 查看可用tracer列表**
root# cat available_tracers
root# blk    mmiotrace    function_graph    wakeup_rt    wakeup    function    nop
- function ：无需参数的函数跟踪程序
- function_graph：函数调用跟踪程序
- blk：与块设备相关的调用和时间跟踪程序
- mmiotrace：一个内存映射I/O操作跟踪程序
- nop：最简单的跟踪程序
**7. ftrace 使用脚本**
```
#!/bin/bash
dir=/sys/kernel/debug/tracing
echo function > ${dir}/current_tracer
echo 1 > ${dir}/tracing_on
sleep 1
echo 0 > ${dir}/tracing_on

```
**8. 跟踪特定的function**
root# echo schedule > set_ftrace_filter 只跟踪schedule
**9. 不跟踪特定的funcion**
root# echo '!schedule' > set_ftrace_filter
或者
root# echo schedule > set_ftrace_notrace
其中支持正则表达式
root# echo '\*lock\*' < set_ftrace_notrace
**10. 耗时符号表示**
```
'$' - greater than 1 second
'@' - greater than 100 milisecond
'*' - greater than 10 milisecond
'#' - greater than 1000 microsecond
'!' - greater than 100 microsecond
'+' - greater than 10 microsecond
' ' - less than or equal to 10 microsecond.
```

**function tracer的输出**

```
# tracer: function 
 # 
 #  TASK-PID   CPU#    TIMESTAMP        FUNCTION 
 #   |  |       |          |                | 
  bash-4251  [01]  10152.583854:    path_put <-path_walk 
  bash-4251  [01] 10152.583855: dput <-path_put 
  bash-4251  [01] 10152.583855: _atomic_dec_and_lock <-dput
```

第一行显示当前 tracer 的类型。第三行是 header。

第一列是进程信息，包括进程名和 PID ；第二列是 CPU，在 SMP 体系下，该列显示内核函数具体在哪一个 CPU 上执行；第三列是时间戳；第四列是函数信息，缺省情况下，这里将显示内核函数名以及它的上一层调用函数。

如上例所示，path_walk() 调用了 path_put 。此后 path_put 又调用了 dput，进而 dput 再调用 _atomic_dec_and_lock 。

**schedule switch tracer的输出**

Schedule switch tracer 记录系统中的进程切换信息。在其输出文件 trace 中 , 输出行的格式有两种：

第一种表示进程切换信息：

```
Context switches: 
       Previous task              Next Task 
  <pid>:<prio>:<state>  ==>  <pid>:<prio>:<state>
```

第二种表示进程 wakeup 的信息：

```
	Wake ups: 
       Current task               Task waking up 
  <pid>:<prio>:<state>    +  <pid>:<prio>:<state>
```

这里举一个实例：

```
# tracer: sched_switch 
 # 
 #  TASK_PID   CPU#     TIMESTAMP             FUNCTION 
 #     |         |            |                  | 
   fon-6263  [000] 4154504638.932214:  6263:120:R +   2717:120:S 
   fon-6263  [000] 4154504638.932214:  6263:120:? ==> 2717:120:R 
   bash-2717 [000] 4154504638.932214:  2717:120:S +   2714:120:S
```

第一行表示进程 fon 进程 wakeup 了 bash 进程。其中 fon 进程的 pid 为 6263，优先级为 120，进程状态为 Ready 。她将进程 ID 为 2717 的 bash 进程唤醒。

第二行表示进程切换发生，从 fon 切换到 bash 。

**irqsoff tracer 输出**

有四个 tracer 记录内核在某种状态下最长的时延，irqsoff 记录系统在哪里关中断的时间最长；  preemptoff/preemptirqsoff 以及 wakeup 分别记录禁止抢占时间最长的函数，或者系统在哪里调度延迟最长  (wakeup) 。这些 tracer 信息对于实时应用具有很高的参考价值。

为了更好的表示延迟，ftrace 提供了和 trace 类似的 latency_trace 文件。以 irqsoff 为例演示如何解读该文件的内容。

```
# tracer: irqsoff 
 irqsoff latency trace v1.1.5 on 2.6.26 
 -------------------------------------------------------------------- 
 latency: 12 us, #3/3, CPU#1 | (M:preempt VP:0, KP:0, SP:0 HP:0 #P:2) 
    ----------------- 
    | task: bash-3730 (uid:0 nice:0 policy:0 rt_prio:0) 
    ----------------- 
 => started at: sys_setpgid 
 => ended at:   sys_setpgid 
 #                _------=> CPU# 
 #               / _-----=> irqs-off 
 #              | / _----=> need-resched 
 #              || / _---=> hardirq/softirq 
 #              ||| / _--=> preempt-depth 
 #              |||| / 
 #              |||||     delay 
 #  cmd     pid ||||| time  |   caller 
 #     \   /    |||||   \   |   / 
    bash-3730  1d...    0us : _write_lock_irq (sys_setpgid) 
    bash-3730  1d..1    1us+: _write_unlock_irq (sys_setpgid) 
    bash-3730  1d..2   14us : trace_hardirqs_on (sys_setpgid)
```

在文件的头部，irqsoff tracer 记录了中断禁止时间最长的函数。在本例中，函数 trace_hardirqs_on 将中断禁止了 12us 。

文件中的每一行代表一次函数调用。 Cmd 代表进程名，pid 是进程 ID 。中间有 5 个字符，分别代表了 CPU#，irqs-off 等信息，具体含义如下：

CPU# 表示 CPU ID ；

irqs-off 这个字符的含义如下：’ d ’表示中断被 disabled 。’ . ’表示中断没有关闭；

need-resched 字符的含义：’ N ’表示 need_resched 被设置，’ . ’表示 need-reched 没有被设置，中断返回不会进行进程切换；

hardirq/softirq 字符的含义 : 'H' 在 softirq 中发生了硬件中断， 'h' – 硬件中断，’ s ’表示 softirq，’ . ’不在中断上下文中，普通状态。

preempt-depth: 当抢占中断使能后，该域代表 preempt_disabled 的级别。

在每一行的中间，还有两个域：time 和 delay 。 time: 表示从 trace 开始到当前的相对时间。 Delay 突出显示那些有高延迟的地方以便引起用户注意。当其显示为 ! 时，表示需要引起注意。

**function graph tracer 输出**

Function graph tracer 和 function tracer 类似，但输出为函数调用图，更加容易阅读：

```
# tracer: function_graph 
 # 
 # CPU  OVERHEAD/DURATION      FUNCTION CALLS 
 # |     |   |                 |   |   |   | 
 0)               |  sys_open() { 
 0)               |    do_sys_open() { 
 0)               |      getname() { 
 0)               |        kmem_cache_alloc() { 
 0)   1.382 us    |          __might_sleep(); 
 0)   2.478 us    |        } 
 0)               |        strncpy_from_user() { 
 0)               |          might_fault() { 
 0)   1.389 us    |            __might_sleep(); 
 0)   2.553 us    |          } 
 0)   3.807 us    |        } 
 0)   7.876 us    |      } 
 0)                |      alloc_fd() { 
 0)   0.668 us    |        _spin_lock(); 
 0)   0.570 us    |        expand_files(); 
 0)   0.586 us    |        _spin_unlock();
```

OVERHEAD 为 ! 时提醒用户注意，该函数的性能比较差。上面的例子中可以看到 sys_open 调用了 do_sys_open，依次又调用了 getname()，依此类推。

**Sysprof tracer 的输出**

Sysprof tracer 定时对内核进行采样，她的输出文件中记录了每次采样时内核正在执行哪些内核函数，以及当时的内核堆栈情况。

每一行前半部分的格式和 3.1.1 中介绍的 function tracer 一样，只是，最后一部分 FUNCTION 有所不同。

Sysprof tracer 中，FUNCTION 列格式如下：

```
Identifier  address frame_pointer/pid
```

当 identifier 为 0 时，代表一次采样的开始，此时第三个数字代表当前进程的 PID ；

Identifier 为 1 代表内核态的堆栈信息；当 identifier 为 2  时，代表用户态堆栈信息；显示堆栈信息时，第三列显示的是 frame_pointer，用户可能需要打开 system map  文件查找具体的符号，这是 ftrace 有待改进的一个地方吧。

当 identifier 为 3 时，代表一次采样结束。

### non-tracer tracer使用

主要包括以下几种：

- Max Stack Tracer
- Profiling (branches / unlikely / likely / Functions)
- Event tracing
**max stack tracer的使用**
这个 tracer 记录内核函数的堆栈使用情况，用户可以使用如下命令打开该 tracer：

root# echo 1 > /proc/sys/kernel/stack_tracer_enabled

从此，ftrace 便留心记录内核函数的堆栈使用。 Max Stack Tracer 的输出在 stack_trace 文件中：

root# cat /debug/tracing/stack_trace 
 Depth Size Location (44 entries) 
----- ---- --------
 0) 3088 64 update_curr+0x64/0x136 
 1) 3024 64 enqueue_task_fair+0x59/0x2a1 
 2) 2960 32 enqueue_task+0x60/0x6b 
 3) 2928 32 activate_task+0x27/0x30 
 4) 2896 80 try_to_wake_up+0x186/0x27f 
…
 42)  80 80 sysenter_do_call+0x12/0x32

从上例中可以看到内核堆栈最满的情况如下，有 43 层函数调用，堆栈使用大小为 3088 字节。此外还可以在 Location 这列中看到整个的 calling stack 情况。这在某些情况下，可以提供额外的 debug 信息，帮助开发人员定位问题。
