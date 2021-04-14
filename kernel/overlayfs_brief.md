# overlayfs简介
    Overlayfs是一种堆叠文件系统，它依赖并建立在其它的文件系统之上（例如ext4fs和xfs等等），并不直接参与磁盘空间结构的划分，仅仅将原来底层文件系统中不同的目录进行“合并”，然后向用户呈现。因此对于用户来说，它所见到的overlay文件系统根目录下的内容就来自挂载时所指定的不同目录的“合集”。
如下图：
merge dir: file a, file b, file c, file d
    |
upper dir: file a, file b
    |
lower dir1: file c
    |
lower dir2: file d
上一层的相同文件会将下一层的文件覆盖

## overlayfs简单的使用
    - 手动挂载 
    mount -t overlay overlay -o lowerdir=/overlayfs/low1:/overlayfs/low2,upperdir=/overlayfs/rootfs,workdir=/overlayfs/work /overlayfs/merge
upperdir中的文件是可读写，增量文件。/overlayfs/merge为最终给用户使用的目录。后续操作也应该操作该目录
    - 开机自动挂载
    在/etc/fstab中添加
    overlay /home/root/mergdir overlay auto,lowerdir=/home/root/lowdir1:/home/root/lowdir2:/home/root/lowdir3,upperdir=/home/root/updir1,workdir=/home/root/workdir 0 0

