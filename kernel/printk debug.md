2019/9/6
# printk debug
- 启动参数添加 **console=ttyS0,115200n8**
  - 进入系统/boot

- 进系统后，**echo 7 > /proc/sys/kernel/printk**
