2019/9/13
# linux系统调用定义
定义：通过SYSCALL_DEFINEx定义，其中x为参数的个数
例子：
net/socket.c
```c
SYSCALL_DEFINE4(send, int, fd, void __user *, buff, size_t, len,
		unsigned int, flags)
{
	return __sys_sendto(fd, buff, len, flags, NULL, 0);
}
```
