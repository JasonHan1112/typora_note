# udev机制

在用户空间（systemd-udevd.service）处理内核的uevent请求，当收到一个内核uevent，如果匹配了rules，那么就会采取相应的操作（更改网络接口名称，，执行event handler和创建symlink等）

## udev kernel

在注册kobject的时候调用
```c
int kobject_uevent(struct kobject *kobj, enum kobject_action action);
```
随后调用
```c
int kobject_uevent_env(struct kobject *kobj, enum kobject_action action, char *envp_ext[]);
/*其中会获取kobj的name和kobject path，添加default env（ACTION, DEVPATH, SUBSYSTEM），如果后续有附加env(envp_ext)那么也要添加，调用kset的uevent_ops，创建netlink的skb，添加上之前的env，发送数据广播到用户空间（netlink_broadcast_filtered）*/
```
## udev userspace

- 用户空间有udev守护进程监听netlink的socket。
- 在/lib/udev/rules.d（system rules）、/run/udev/rules.d（volatile runtime directory, 第）、/etc/udev/rules.d（local administration directory, 最高优先级），  
- 配置文件中“#”为注释符

## udev cold boot
当设备在udevd启动之前就已经插入时，需要使用udevadm trigger 命令来让udevd接收kernel event

## udev monitor
udevadm monitor --env 显示完整的事件环境：
