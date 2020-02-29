
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











