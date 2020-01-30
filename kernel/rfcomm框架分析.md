# rfcomm框架分析
user rfcomm socket
  |
rfcomm
  |
l2cap socket

## rfcomm框架初始化
### 整体流程
- rfcomm/core.c--->rfcomm_init
  - 创建内核线程，在其中会创建l2cap的socket
    - kthread_run(rfcomm_run, NULL, "krfcommd");--->
  - 初始化rfcomm的tty设备，向tty层注册设备和驱动
    - rfcomm_init_ttys();
  - 初始化rfcomm的socket
    - rfcomm_init_sockets();
### 上述各个接口的说明
- rfcomm_run
主要完成对底层l2cap的操作【数据的发送接收以及socket的管理】
DEFINE_WAIT_FUNC(wait)--->rfcomm_add_listener--->add_wait_queue--->rfcomm_process_sessions

  - DEFINE_WAIT_FUNC(wait)
    - 创建等待队列
  - rfcomm_add_listener
    - 创建一个l2cap socket【BTPROTO_L2CAP【之前已经注册到系统中】】 rfcomm_l2sock_create--->sock_create_ker--->__sock_create
    - bind socket
    - 创建rfcomm_session并将创建的l2cap的socket与其绑定
  - add_wait_queue
    - 将wait添加到wait_queue(rfcomm_wq)
  - rfcomm_process_sessions
    - 根据rfcomm_session的状态处理l2cap socket的行为和数据 
- rfcomm_init_ttys
  - 主要完成rfcomm在tty层driver和device的注册
- rfcomm_init_sockets
  - 主要完成对上一层的socket【BTPROTO_RFCOMM】的注册
## rfcomm数据的发送
- 用户调用系统调用
  - send(sockfd, buf, len, flags)
  
```c
  SYSCALL_DEFINE4(send, int, fd, void __user *, buff, size_t, len,unsigned int, flags)
{
	return __sys_sendto(fd, buff, len, flags, NULL, 0);
}
```
  
- 内核中的处理
  - __sys_sendto--->
  - sock_sendmsg--->
  - sock_sendmsg_nosec--->
  - sock->ops->sendmsg(sock, msg, msg_data_left(msg));[rfcomm_sock_sendmsg]--->
  - rfcomm_dlc_send--->rfcomm_make_uih--->skb_queue_tail[放进发送队列]--->rfcomm_schedule[如果没有设置门限，唤醒等待的进程]
  - kernel thread一直在处理dlcs[rfcomm_process_dlcs]--->
  - rfcomm_process_tx--->rfcomm_send_frame--->[调用之前创建的l2cap的socket的发送函数，将数据发送到下一层(l2cap)]
```c
int __sys_sendto(int fd, void __user *buff, size_t len, unsigned int flags,
		 struct sockaddr __user *addr,  int addr_len)
{
	struct socket *sock;
	struct sockaddr_storage address;
	int err;
	struct msghdr msg;
	struct iovec iov;
	int fput_needed;

	err = import_single_range(WRITE, buff, len, &iov, &msg.msg_iter);
	if (unlikely(err))
		return err;
	sock = sockfd_lookup_light(fd, &err, &fput_needed);
	if (!sock)
		goto out;

	msg.msg_name = NULL;
	msg.msg_control = NULL;
	msg.msg_controllen = 0;
	msg.msg_namelen = 0;
	if (addr) {
		err = move_addr_to_kernel(addr, addr_len, &address);
		if (err < 0)
			goto out_put;
		msg.msg_name = (struct sockaddr *)&address;
		msg.msg_namelen = addr_len;
	}
	if (sock->file->f_flags & O_NONBLOCK)
		flags |= MSG_DONTWAIT;
	msg.msg_flags = flags;
	err = sock_sendmsg(sock, &msg);//关键步骤

out_put:
	fput_light(sock->file, fput_needed);
out:
	return err;
}

```
```c
int sock_sendmsg(struct socket *sock, struct msghdr *msg)
{
	int err = security_socket_sendmsg(sock, msg,
					  msg_data_left(msg));//总是返回0

	return err ?: sock_sendmsg_nosec(sock, msg);
}

```
```c
static inline int sock_sendmsg_nosec(struct socket *sock, struct msghdr *msg)
{
	int ret = sock->ops->sendmsg(sock, msg, msg_data_left(msg));//执行socket各自自己注册的sendmsg函数
	BUG_ON(ret == -EIOCBQUEUED);
	return ret;
}
```
  
## rfcomm数据的接收