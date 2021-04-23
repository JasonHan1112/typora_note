# kernel mount rootfs brief
kernel在启动的时候会有rootfs挂载的流程，如果系统支持ramddisk，那在一开始会先挂载ramdisk作为rootfs进行相关的初始化，然后再挂载真正的存储设备上的根分区。

## kernel ramdisk处理流程
1. 注册rootfs
start_kernel->vfs_caches_init->mnt_init->init_rootfs:register_filesystem(&rootfs_fs_type)->init_mount_tree
2. 处理initrd
主要在populate_rootfs中进行解压处理
kernel_init->kernel_init_freeable->do_basic_setup->do_initcalls->populate_rootfs
