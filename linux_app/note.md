
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








