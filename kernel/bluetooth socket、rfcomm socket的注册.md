2019/9/7
# bluetooth socket、rfcomm socket的注册
- 开机后bluetooth向socket层注册bt_sock_family_ops
  - __init bt_init(void)
    - ===>sock_register(&bt_sock_family_ops);


```c
static const struct net_proto_family bt_sock_family_ops = {
	.owner	= THIS_MODULE,
	.family	= PF_BLUETOOTH,
	.create	= bt_sock_create,
};

//用户创建socket时会调到该函数
static int bt_sock_create(struct net *net, struct socket *sock, int proto,
			  int kern)
{
	int err;

	if (net != &init_net)
		return -EAFNOSUPPORT;

	if (proto < 0 || proto >= BT_MAX_PROTO)
		return -EINVAL;

	if (!bt_proto[proto])
		request_module("bt-proto-%d", proto);

	err = -EPROTONOSUPPORT;

	read_lock(&bt_proto_lock);
  //调用注册在bluetooth family 下的 protocol 【rfcomm】的create
	if (bt_proto[proto] && try_module_get(bt_proto[proto]->owner)) {
		err = bt_proto[proto]->create(net, sock, proto, kern);
		if (!err)
			bt_sock_reclassify_lock(sock->sk, proto);
		module_put(bt_proto[proto]->owner);
	}

	read_unlock(&bt_proto_lock);

	return err;
}

//向net_families中注册bluetooth family
int sock_register(const struct net_proto_family *ops)
{
	int err;

	if (ops->family >= NPROTO) {
		pr_crit("protocol %d >= NPROTO(%d)\n", ops->family, NPROTO);
		return -ENOBUFS;
	}

	spin_lock(&net_family_lock);
	if (rcu_dereference_protected(net_families[ops->family],
				      lockdep_is_held(&net_family_lock)))
		err = -EEXIST;
	else {
		rcu_assign_pointer(net_families[ops->family], ops);//向
		err = 0;
	}
	spin_unlock(&net_family_lock);

	pr_info("NET: Registered protocol family %d\n", ops->family);
	return err;
}
EXPORT_SYMBOL(sock_register);


```



 
- bluetooth在socket注册完成后，rfcomm向bt_proto中注册rfcomm_sock_family_ops
  - rfcomm_init_sockets(void)
    - bt_sock_register(BTPROTO_RFCOMM, &rfcomm_sock_family_ops); 
      - bt_proto[proto] = ops;



```c
static const struct net_proto_family rfcomm_sock_family_ops = {
	.family		= PF_BLUETOOTH,
	.owner		= THIS_MODULE,
	.create		= rfcomm_sock_create
};

//用户创建socket时最终会调到该函数
static int rfcomm_sock_create(struct net *net, struct socket *sock,
			      int protocol, int kern)
{
	struct sock *sk;

	BT_DBG("sock %p", sock);

	sock->state = SS_UNCONNECTED;

	if (sock->type != SOCK_STREAM && sock->type != SOCK_RAW)
		return -ESOCKTNOSUPPORT;

	sock->ops = &rfcomm_sock_ops;

	sk = rfcomm_sock_alloc(net, sock, protocol, GFP_ATOMIC, kern);
	if (!sk)
		return -ENOMEM;

	rfcomm_sock_init(sk, NULL);
	return 0;
}
```
