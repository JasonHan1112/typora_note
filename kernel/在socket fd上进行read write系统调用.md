2019/9/14
# 在socket fd上进行read write系统调用
- 概述
1. 用户调用read(fd, buf, count)
2. ksys_read(fd, buf, count);--->
3. file_pos_read(f.file);
4. vfs_read(f.file, buf, count, &pos);--->
5. __vfs_read(file, buf, count, pos);--->
6. 
```c
if (file->f_op->read)
		return file->f_op->read(file, buf, count, pos);
	else if (file->f_op->read_iter)
    return new_sync_read(file, buf, count, pos);
```
7. new_sync_read(file, buf, count, pos);--->
8. call_read_iter(filp, &kiocb, &iter);--->
9. return file->f_op->read_iter(kio, iter);
**最终调用到了net/socket.c中注册的sock_read_iter(struct kiocb \*iocb, struct iov_iter \*to)**
- 源码
```c
ssize_t ksys_read(unsigned int fd, char __user *buf, size_t count)
{
	struct fd f = fdget_pos(fd);
	ssize_t ret = -EBADF;

	if (f.file) {
		loff_t pos = file_pos_read(f.file);
		ret = vfs_read(f.file, buf, count, &pos);
		if (ret >= 0)
			file_pos_write(f.file, pos);
		fdput_pos(f);
	}
	return ret;
}

```
```c
SYSCALL_DEFINE3(read, unsigned int, fd, char __user *, buf, size_t, count)
{
	return ksys_read(fd, buf, count);
}

```

```c
ssize_t ksys_write(unsigned int fd, const char __user *buf, size_t count)
{
	struct fd f = fdget_pos(fd);
	ssize_t ret = -EBADF;

	if (f.file) {
		loff_t pos = file_pos_read(f.file);
		ret = vfs_write(f.file, buf, count, &pos);
		if (ret >= 0)
			file_pos_write(f.file, pos);
		fdput_pos(f);
	}

	return ret;
}
```
```c

SYSCALL_DEFINE3(write, unsigned int, fd, const char __user *, buf,
		size_t, count)
{
	return ksys_write(fd, buf, count);
}
```
