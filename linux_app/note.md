
# dpkg安装软件报缺少依赖的错误
- 安装软件
  dpkg -i xxxx.deb

- 修复缺少依赖
  sudo apt --fix-broken install

# 查看linux发行版版本
- lsb_release -a
  查看全称
  
- lsb_release -cs
  查看简称
  
# 添加额外的源
- 修改/etc/apt/sources.list
- 添加key: sudo apt-key add xxxx.gpg
- sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
- sudo apt-get update
- sudo apt-get install xxxxxxxxxxx
# 开机启动
- 在/etc/init.d/中添加执行脚本

- 在/etc/rcx.d/中添加软链接

  0：关机
  1：单用户模式
  2：无网络支持的多用户模式
  3：有网络支持的多用户模式
  4：保留，未使用
  5：有网络支持有X-Window支持的多用户模式
  6：重新引导系统，即重启

# 开机mount

- 查看uuid: 
root# blkid
- 修改权限
root # chmod 777 /etc/fstab
- 添加
root# echo "UUID=e943fbb7-020a-4c64-a48a-2597eb2496df /vdb1 ext4 defaults 0 0" >> /etc/fstab
- 恢复权限
root# chmod 644 /etc/fstab
# systemctl 开机启动
1. 在/etc/systemd/system/中创建xxxx.service。
2. 参考sshd.service中的内容，编写xxxx.service。
3. systemctl enable xxxx.services。
4. systemctl start xxxx.service
5. systemctl status xxxx.service.
# cflow的使用

## 安装

apt-get install cflow

## 使用

cflow --help
重点关注以下选项
```
Usage: cflow [OPTION...] [FILE]...
generate a program flowgraph

 General options:
  -d, --depth=NUMBER         Set the depth at which the flowgraph is cut off
  -o, --output=FILE          Set output file name (default -, meaning stdout)
  -r, --reverse              * Print reverse call tree
  -m, --main=NAME            Assume main function to be called NAME
  -n, --number               * Print line numbers                       
  -T, --tree                 * Draw ASCII art tree
  -V, --version              print program version
  --cpp[=command]            * Run the specified preprocessor command.

```
## example
cflow -T -m main -n timer.c

## tree2dotx转换成dot文件
cflow -T -m main -n timer.c > main.txt
cat main.txt | tree2dotx > main.dot
## 通过graphviz将dot文件生成图片
dot -Tgif main.dot -o main.gif

# tmux的使用
Tmux 就是会话与窗口的"解绑"工具，将它们彻底分离。
（1）它允许在单个窗口中，同时访问多个会话。这对于同时运行多个命令行程序很有用。
（2） 它可以让新窗口"接入"已经存在的会话。
（3）它允许每个会话有多个连接窗口，因此可以多人实时共享会话。
（4）它还支持窗口任意的垂直和水平拆分。

## 安装
sudo apt-get install tmux
## 使用
- tmux list-session：显示当前server中已经打开的的tmux session
- tmux new -s xxx：打开新的session
- tmux attach -t xxx：当前窗口与session xxx进行attach
- tmux kill-session -t xxx：杀掉session xxx
- tmux rename -t s1 s2　　重命名会话s1为s2
- 在~/.tmux.conf中可以对tmux进行配置，包括颜色的配置，也可以配置$PS1
### 快捷键
- ctrl-b %: 纵向拆分窗口
- ctrl-b "：横向拆分窗口
- 在开启的tmux中选择打开的会话
    1. 按下组合键 Ctrl-b (Tmux 快捷键前缀)
    2. 放开组合键 Ctrl-b
    3. 按下 s 键 
- ctrl-b 然后按o（小写O），可以切到下一个pane。
- ctrl-b 然后按；就可以切换到上一个pane。
- ctrl-b o调整当前光标所在pane的窗口的内容到下一窗口内容。
- ctrl-b 然后按q，弹出数字选中想要跳转到的那个编号即可0 1 2...
- ctrl-b arrow_up arrow_down arrow_up arrow_left arrow_right：调整pane形状和大小
- ctrl-b 然后d，deattach这个current client，退出当前的窗口
- ctrl-b 然后 $，重命名会话
- ctrl-b d，分离当前会话
- ctrl-b D，分离指定会话
- ctrl-b c　　创建一个新window（window上有多个pane）
- ctrl-b ,　　重命名当前window
- ctrl-b w　　列出所有window，可进行切换
- ctrl-b n　　进入下一个window
- ctrl-b p　　进入上一个window
- ctrl-b l　　进入之前操作的window
- ctrl-b 0~9　　选择编号0~9对应的window
- ctrl-b .　　修改当前window索引编号
- ctrl-b '　　切换至指定编号（可大于9）的window
- ctrl-b f　　根据显示的内容搜索pane
- ctrl-b &　　关闭当前window
- ctrl-b [   进入翻屏模式，q退出
- ctrl-b x   删除当前pane
- ctrl-b 'space'   调整pane水平垂直分布
- ctrl-b &   删除窗口


## nfs挂载

在配置好nfs的机器上执行

 sudo mount -t nfs 192.168.1.1:/root/workspace/hanxueqing/tmp /mnt

将远程目录挂载到本地

## gtags使用
- 参考链接
  https://www.gnu.org/software/global/
### 简单介绍
#### 下载
地址：https://ftp.gnu.org/pub/gnu/global/
- 解压
- 编译、安装

```
% ./configure
% make
```
#### 准备工作
- 生成gtags
```
$ cd ${project_path}
$ gtags
$ ls G*
- GTAGS: definition database
- GRTAGS: reference database
- GPATH: path name database
```
#### 基本使用
- 查找函数
```
$ global func1
DIR1/fileB.c            # func1() is defined in fileB.c
$ cd DIR1
$ global func1
fileB.c                 # relative path from DIR1
$ cd ../DIR2
$ global func1
../DIR1/fileB.c         # relative path from DIR2
```
- 查找引用
```
$ global -r func2
../DIR1/fileA.c         # func2() is referred from fileA.c
```
- POSIX 正则表达式
```
$ cd /home/user/ROOT
$ global 'func[1-3]'
DIR1/fileB.c            # func1, func2 and func3 are matched
DIR2/fileC.c
```
- 查找后显示详情
```
$ global func2
DIR2/fileC.c
$ global -x func2
func2              2 DIR2/fileC.c       func2(){ i++; }
func2              4 DIR2/fileC.c       func2(){ i--; }
```
- 查找没有在GTAGS中的符号
```
$ global -xs X
X                  1 DIR2/fileC.c #ifdef X
```
#### vim使用global
- 准备
```
$ cp /usr/local/share/gtags/gtags.vim $HOME/.vim/plugin
```
- 使用
```
:Gtags main
:cn
go to the next entry.

:cp
go to the previous entry.

:ccN
go to the N’th entry.

:cl
list all entries.

To go to the referenced point of func1, add the -r option.
    :Gtags -r func1
To locate symbols which are not defined in GTAGS, try this:
    :Gtags -s lbolt
To locate strings, try this:
    :Gtags -g int argc

    :Gtags -g "root"

    :Gtags -ge -C               <- locate ’-C’
To get a list of tags in specified files, use the -f command.
    :Gtags -f main.c            <- locate tags in main.c
If you are editing main.c itself, you can use ‘%’ instead.

    :Gtags -f %                 <- locate tags in main.c
```






