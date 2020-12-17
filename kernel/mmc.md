# emmc简介
- emmc(embedded multi media card)，mmc协会订立，emmc相当于nandflash+controller，对外的接口协议与sd,tf卡相似。协议规定硬件引脚：
clk: 输入时钟，主机提供信号，时钟到来时才可以发送数据，SDR（一个时钟周期发送一位）和DDR（一个时钟周期发送两位）两种方式。 cmd: 双向传输命令线，有开漏和推挽两种模式，分别用来应对初始化和快速命令传输。双向传输，当主机发送命令之后，设备会给主机应答，通过cmd线可以返回给主机。reset: 单项传输信号线，主机发送给device。dat0-dat7: 数据信号线。data storbe: 数据锁存线，用于emmc5.0中hs400模式设备要所存输出信号。
- 内部寄存器
cid: 16 bytes 设备识别寄存器，包含设备的唯一号码
rca: 2 bytes 相关设备地址，初始化中，主机控制器动态分配系统地址。
dsr: 2 bytes 驱动等级寄存器，设置设备的输出驱动。
csd: 16 bytes 设备具体数据寄存器，包含设备操作状态具体信息。
ocr: 4 bytes 操作寄存器，通过广播命令获取寄存器信息，包含设备的供电类型。
- 分区
物理上的分区：
每个硬件分区都是独立编址的，0~partition size，具体的读写数据操作实际访问的是哪个硬件分区，是由emmc的extende csd register的partition_config field中的bit[2:0]:partition_access决定的，用户可以通过配置partition_access来切换硬件分区的访问。
boot area partitions 1, boot area partitions 2:
由厂家在生产工程中配置好，并且不能由AP进行配置，这个两个分区存储的稳定性，可靠性都远比UDA(user data area)要好，不同的公司有不同的存放方式，mtk使用uda来存放boot data，而boot area来存放配置参数，qualcomm使用boot1来存放boot data，boot2来存放配置参数。

rpmb partition:
replay protected memory block，他的存在用来给系统存放一些特殊的，需要进行访问授权的数据。

user data area:
AP及用户可以进行读写存储的区域，通常占emmc大小的93%
boot1, boot2，rpmb和uda区域我们都可以认为她们在物理上是独立的。他们各自的物理其实地址都是0x0。这个在出场的时候就会被设置完成。

general purpose partitions:
emmc的spec上定义每个emmc最多可以通过配置寄存器来定义4个gpap:
gpap配置定义完成之后每一个gpap的起始地址都为0x0，即可以相应的将其认为是独立的一块区域。只是在存放数据的时候会需要从新根据他的其实地址进行计算然后再存储数据。这样必然会增加一定工作量；一般不用。

- mmc framework in linux kernel
mmc、sd、sdio在linux内核中统一使用mmc framework
1. mmc bus
开机后通过mmc_init注册mmc_bus_type到kernel，
mmc_bus_match(match any mmc card),匹配后调用mmc_bus_probe，进而drv->probe

- asr emmc_host的注册
asr通过platform_device的方式来probe，通过sdhci_add_host经过__sdhci_add_host->mmc_add_host->mmc_start_host->_mmc_detect_change->mmc_rescan->mmc_rescan_try_freq->mmc_attach_mmc(完成卡的初始化读写)->mmc_init_card(创建mmc_card以及dev.bus=mmc_bus_type)->mmc_add_card(添加mmc_card->dev到mmc_bus_type上)完成mmc_card的到mmc_bus的注册，

- mmc block
在drivers/mmc/core/block.c中mmc_blk_init完成的register_blkdev，和mmc_register_driver（向mmc_bus中注册了mmcblk driver,从而和上述mmc_card完成匹配），从而调用了mmc_blk_probe，在其中对分区以及各个block需要的参数进行配置。
























