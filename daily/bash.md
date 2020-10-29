### 在bash中 . 表示在当前shell中执行命令
例如切换目录指令如果在脚本中写了cd那么表示会另起一个shell在那个shell中cd而不表示在当前shell中切换目录。如果需要在当前目录下按照脚本的路径进行切换目录需要加 . shellscript

- 输入参数时只有一个 \ 表示换行，如果\后边跟着其他的则表示literal value of the next character that follows

# bash 
- 数值测试
  -eq
  等于则为真
  -ne
  不等于则为真
  -gt
  大于则为真
  -ge
  大于等于则为真
  -lt
  小于则为真
  -le
  小于等于则为真

- 字符串
  =
  等于则为真
  !=
  不相等则为真
  -z 字符串
  字符串的长度为零则为真
  -n 字符串
  字符串的长度不为零则为真

- 文件
  -e 文件名
  如果文件存在则为真
  -r 文件名
  如果文件存在且可读则为真
  -w 文件名
  如果文件存在且可写则为真
  -x 文件名
  如果文件存在且可执行则为真
  -s 文件名
  如果文件存在且至少有一个字符则为真
  -d 文件名
  如果文件存在且为目录则为真
  -f 文件名
  如果文件存在且为普通文件则为真
  -c 文件名
  如果文件存在且为字符型特殊文件则为真
  -b 文件名
  如果文件存在且为块特殊文件则为真

- Shell还提供了与( -a )、或( -o )、非( ! )三个逻辑操作符用于将测试条件连接起来，其优先级为："!"最高，"-a"次之，"-o"最低。

- () []
在shell中使用()表示需要重新起一个新的sub shell进行命令的执行
在shell中使用[]表示在当前shell中执行命令

- ```cpp
  if [ "$a" == "$b" ],与=等价 
  ```

- if-then-elseif-else-fi
```c
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```

- for 循环

```c
/*************************/
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
/*************************/
/*************************/
//类似于C语言
for (( ; ; ))
do

done
/*************************/

```
- while 循环
```
while condition
do
    command
done
```
```
echo '按下 <CTRL-D> 退出'
echo -n '输入你最喜欢的网站名: '
while read FILM
do
    echo "是的！$FILM 是一个好网站"
done

```
- case
```
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2）
    command1
    command2
    ...
    commandN
    ;;
esac

```
```
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac

```
- 函数
```
$#
传递到脚本的参数个数
$*
以一个单字符串显示所有向脚本传递的参数
$$
脚本运行的当前进程ID号
$!
后台运行的最后一个进程的ID号
$@
与$*相同，但是使用时加引号，并在引号中返回每个参数。
$-
显示Shell使用的当前选项，与set命令功能相同。
$?
显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。
```
```c
foo () {
a=$1
echo $a
}
$1为foo的参数1
result=$(foo)
函数返回值尽量用echo, return会不好用
```
- 赋值
通过(())来赋值
$((case_size=$1/1024+1))

$((spi_flash_mmio=0xff000000))

$((skip_offset=$spi_flash_mmio/10240))

$((($base_addr+$i)+($die_num<<20)+($port_offset<<12)))

- bash 算数运算
1. let 表达式
```c
x=3
y=5
let z1=3+5;         echo $z1    # output: 8, notice no whitespace is allowed.
let z1=x+y;         echo $z1    # output: 8
let z1=$x+$y;       echo $z1    # output: 8
```
3. expr 表达式
```c
let z1=$x+$y;       echo $z1    # output: 8
```
5. 双括号表达式
```c
((z=3+5));          echo $z # output: 8
((z = 3 + 5));      echo $z # output: 8
```
- 进制转换
  - 八进制转十进制：
[chengmo@centos5 ~]$ ((num=0123));
[chengmo@centos5 ~]$ echo $num;
83
[chengmo@centos5 ~]$ ((num=8#123));
[chengmo@centos5 ~]$ echo $num;    
83
((表达式))，（（））里面可以是任意数据表达式。如果前面加入：”$”可以读取计算结果。
  - 十六进制转十进制：
[chengmo@centos5 ~]$ ((num=0xff)); 
[chengmo@centos5 ~]$ echo $num;    
255
[chengmo@centos5 ~]$ ((num=16#ff));
[chengmo@centos5 ~]$ echo $num;    
255
  - 十进制转八进制
这里使用到：bc外部命令完成。bc命令格式转换为：echo "obase=进制;值"|bc
[chengmo@centos5 ~]$ echo "obase=8;01234567"|bc
4553207

- 变量作用域
默认变量是全局的，但是在函数中添加了local来修饰则表示该变量为局部变量。如果全局变量与local相同则优先使用local变量。

- 在bash中一定要注意16进制和10进制进行区分，尤其是setpci的时候

- #!/bin/bash

  set -e 

  -e的参数的作用是：每条指令之后后，都可以用#?去判断

  他的返回值，零就是正确执行，非零就是执行有误，加了-e

  之后，就不用自己写代码去判断返回值，返回非零，脚本就会
  
  退出.
  
  set -x
  
  显示每条执行的指令
  
- pushd popd和dirs
  - dirs显示目录栈的内容
  
    | 选项 | 含义                                          |
    | ---: | :-------------------------------------------- |
    |   -p | 每行显示一条记录                              |
    |   -v | 每行显示一条记录，同时展示该记录在栈中的index |
    |   -c | 清空目录栈                                    |
  
  - pushd
    - 每次pushd命令执行完成之后，默认都会执行一个dirs命令来显示目录栈的内容。
    - `pushd +n`切换到目录栈中的第n个目录(这里的n就是`dirs -v`命令展示的index)，并将该目录以栈循环的方式推到栈顶。
    - +n的含义是从栈顶往栈底方向进行计数，从0开始；-n的含义刚好相反，从栈底向栈顶方向计数，从0开始。
    
  - popd
  
    - popd不带任何参数执行的效果，就是将目录栈中的栈顶元素出栈。
    - 将目录栈中的第n个元素删除(这里的n就是命令`dirs -v`显示的目录index)。
    - +n的含义是从栈顶往栈底方向进行计数，从0开始；-n的含义刚好相反，从栈底向栈顶方向计数，从0开始。
- 通过bash对串口进行操作
    ```c
    stty -F /dev/ttyS1 raw ispeed 115200 ospeed 115200 cs8 -parenb -cstopb -echo -ixoff
    exec 99</dev/ttyS1
    cat <&99 > /tmp/ttyDump.dat &
    PID=$!
    echo -ne "\x5c\x8d\x00\x4b\x00\xb5\x5c\xd8" > /dev/ttyS1
    sleep 0.2s
    kill $PID
    wait $PID 2>/dev/null
    exec 99<&-
    hexdump -e '16/1 "%02x" "\n"' /tmp/ttyDump.dat
    ```
