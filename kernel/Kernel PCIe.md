# Kernel PCIe
## kernel启动时设置pci相关启动参数
[pure_initcall] __init pci_setup(char *str)
- 在其中根据不同的kernel启动参数对static变量进行赋值
- pcibios_setup根据不同的启动参数设置pci_probe变量

## 向kobject中注册pci class，并向sys文件系统注册pci bus
[postcore_initcall] pcibus_class_init------>class_register

## 向kobject中注册pci总线数据结构，并向sys文件系统注册pci bus
[postcore_initcall] pci_driver_init------>bus_register(&pci_bus_type)------>bus_register(&pcie_port_bus_type)

## arch初始化（此时pci子系统以及pci class和pci driver都没有初始化）
确定hostbridge类型并且确定最原始的访问方式（IO访问）
1. 初始化函数被编译在__init段中[arch_initcal]

- pci_arch_init

  负责初始化

  - pci_direct_probe------>pci_check_type1------>pci_sanity_check
    负责判断hostbridge的类型
  - pci_mmcfg_early_init------>pci_mmcfg_check_hostbridge
    和已知的hostbridge去比较
    
  - pci_direct_init
    指定最基础的pci读写函数pci_direct_conf1
## acpi对pci设备进行枚举，暂时不分析

## pci子系统初始化
- [subsys_initcall] pci_subsys_init
  - pci_legacy_init------>pcibios_scan_root(0)------>x86_pci_root_bus_resources[获得x86下的root bus以及其资源(resources)]
  - pci_scan_root_bus(NULL, busnum, &pci_root_ops, sd, &resources)
    - pci_create_root_bus
      - pci_alloc_host_bridge
      创建bridge结构体，并给相关成员赋值
      - pci_register_host_bridge
        1. pci_alloc_bus: 创建pcibus成员，以及各个bus下的链表（children devices slots resources）
        2. pci_find_bus: 看是否能通过另一个根找到要创建的root bus，如果能找到说明这个root bus不是真正的root bus
        3. bridge.bus = root_bus (root_bus 和bridge相互包含，互指)
        4. device_register: 注册bridge->dev
        5. 添加root bus到pcibus中，并为root bus分配资源
    - pci_scan_child_bus
    扫描root bus下一级总线，扫描好设备分配好结构体以及bus号
      - pci_scan_child_bus_extend
        - pci_scan_slot
        扫描slot下的设备，并将扫描到的设备添加到pcibus下
          - pci_scan_single_device：看下边有什么设备如果找到将其添加到pci_bus
            - pci_get_slot: 根据bus和devfun获得slot，返回slot下的pci_dev，看是否能找到指定的dev
            - pci_scan_device: 创建pci_dev，并向其赋值          
            - pci_device_add：给pci_dev赋值，将设备添加到pci_bus
          - pcie_aspm_init_link_stat: 如果是pcie设备，初始化链路的aspm
        - pci_iov_bus_range：找到virtual function需要的bus range 
        - 计算有多少个bridge支持hotplug
        - for_each_pci_bridge, pci_scan_bridge_extend扫描已经配置好的bios所有的桥，然后再配置剩下的没有配置的桥（pci_add_new_bus来分bus号）
        - pci_scan_child_bus_extend递归
  - pci_bus_add_devices(bus):将后续扫描到的pci_dev添加到bus上，并创建sysfs
    
- pcibios_init

## ASR PCIe Endpoint Frame
- uboot
配置好PCIe基础的配置（配置空间，中断一类），可以进行正常通信。
- PCIe Endpoint Frame简介
PCIe Endpoint Frame分为两部分，EPC和EPF，使用时通过configfs将EPC和EPF绑定，完成正常的功能。
- PCIe EPC的枚举与初始化
1. ASR将其设置成platform，通过
```c
static const struct of_device_id of_asr_pcie_match[] = {
        {
                .compatible = "asr,dwc-pcie",
                .data = &asr_pcie_rc_of_data,
        },
        {
                .compatible = "asr,dwc-pcie-ep",
                .data = &asr_pcie_ep_of_data,
        },
        {},
};

```
进行match
2. asr_pcie_probe为__init属性，内核启动时进行调用，其中
```c
match = of_match_device(of_match_ptr(of_asr_pcie_match), dev);
if (!match)
                return -EINVAL;
```
dtb中的of_node和内核中的of_asr_pcie_match match成功后，进入资源的分配和赋值。
```c
struct dw_pcie {
        struct device           *dev;
        void __iomem            *dbi_base; /*local access(back-door) to resgister, can change some pci-sig register R/W*/
        void __iomem            *dbi_base2; /*local access(back-door) to some read only shadow iatu and dma regster*/
        void __iomem            *elbi_base;
        void __iomem            *dbi_iatu;
        void __iomem            *dma_base;
        u32                     num_viewport;
        u8                      iatu_unroll_enabled;
        int                     pcie_init_before_kernel;
        struct pcie_port        pp;
        struct dw_pcie_ep       ep;
        const struct dw_pcie_ops *ops;
};

struct asr_pcie {
        struct dw_pcie          *pci;
        void __iomem            *base;          /* DT asr_conf */
        void __iomem            *phy_ahb;               /* DT phy_ahb */
        int                     phy_count;      /* DT phy-names count */
        struct phy              **phy;
        int                     link_gen;
        struct irq_domain       *irq_domain;
        enum dw_pcie_device_mode mode;
        struct  clk *clk_pcie;  /*include master slave slave_lite clk*/
        struct  clk *clk_master;
        struct  clk *clk_slave;
        struct  clk *clk_slave_lite;

        struct  gpio_desc *perst_gpio; /* for PERST# in RC mode*/

};

```
以上资源为从dtb中获取赋值和填充的变量(asr_conf, phy_ahb, pcie-clk, link_gen, dw_pcie_ops, link_gen, dw_pcie_ops.start_link(assert phystart training建立链路), dw_pcie_ops.stop_link(设置ltssm到detect quite), dw_pcie_ops.link_up(判断pl和dl是否已经成功连接), irq(pcie报给arm的)),确定max link speed（link_gen）为gen3。
3. 判断是否pcie在之前已经初始化完成(通过uboot传递的boot args)，现在已经设置在uboot阶段初始化完成（pcie链路也在uboot阶段已经建立）。
4. 通过of_asr_pcie_match->mode来确定pcie是做ep还是rc（目前是做ep, 通过配置PCIECTRL_ASR_CONF_DEVICE_CMD等相关寄存器），添加相关设备（asr_add_pcie_ep asr_add_pcie_port）
5. 分析asr_add_pcie_ep
赋值dw_pcie结构体中其他的成员（dbi_base, dbi_base2, elbi_base, dbi_iatu, dma, addr_space）；
创建dw_pcie_ep结构体
```c
struct dw_pcie_ep {
        struct pci_epc          *epc;/*重要结构体*/
        struct dw_pcie_ep_ops   *ops;
        phys_addr_t             phys_base;
        size_t                  addr_size;
        size_t                  page_size;
        u8                      bar_to_atu[6];
        phys_addr_t             *outbound_addr;
        unsigned long           *ib_window_map;
        unsigned long           *ob_window_map;
        u32                     num_ib_windows;
        u32                     num_ob_windows;
        void __iomem            *msi_mem;
        phys_addr_t             msi_mem_phys;
};
```
赋值ep->phys_base="addr_space" 0x8400000, 执行dw_pcie_ep_init;
6. 分析dw_pcie_ep_init
判断dw_pcie->dbi_base, dw_pcie->dbi_base2是否正确；
读取of_node中num-ib-windows，num_ob_windows(用来做地址转换用的窗口)的数量，并检查其合法性，以及分配地址；
执行ep_init;
7. 分析ep_init(asr_pcie_ep_init)
获取到之前创建的dw_pcie和asr_pcie;
asr_pcie_enable_wrapper_interrupts,通过配置elbi的寄存器使pcie到soc的中断使能，相关dma中断mask打开。执行devm_pci_epc_create
8. 分析devm_pci_epc_create
调用pci-epc-core.c中的__devm_pci_epc_create->__pci_epc_create,初始化设备，填充pci_epc结构体(主要opsa), pci_ep_cfs_add_epc_group配置好epc的configfs的group
9. 配置epc的max_functions = 1，配置epc的addr space（从之前dw_pcie_ep中的值中获取(phys_base 0x8400000)）,从epc的mem中获取ep->msi_mem的空间，设置epc的feature为EPC_FEATURE_NO_LINKUP_NOTIFIER，建立dw_pcie_ep->epc = epc，如果pcie_init_before_kernel=0则执行dw_pcie_setup;
10. 分析dw_pcie_setup
通过设备树获得"num-lanes"，通过dbi给控制器配置链路的速度和
11. devm_request_irq(dev, irq, asr_pcie_irq_handler, IRQF_SHARED, "asr-pcie", asr)，注册中断处理函数asr_pcie_irq_handle
12. 分析asr_pcie_irq_handle
读取PCIE_ELBI_EP_DMA_IRQ_STATUS的中断寄存器并清零irq状态；查看中断寄存器中是哪一位置起来了（reg = dw_pcie_readl_elbi(pci, PCIE_ELBI_EP_DMA_IRQ_STATUS)）; if (reg & PC_TO_STPU_INT)调用bind时注册的函数（asr_pcie_irq_callback(num)后续分析该函数）; if(reg & DMA_READ_INT), dma 读完成，检查dma中断源以及是否发生错误，如果发生错误则处理错误打印log，如果没有错误则执行业务的中断处理函数asr_pcie_irq_callback

- PCIe EPF初始化(pnet)
1. pcie_ipc_ep_init编译到__init区域，通过pci_epf_register_driver注册epf driver bus和configfs
2. 在configfs中pci_epf_make(config_group_init_type_name pci_epf_create)会在mkdir的时候执行从而创建epf_device
3. 匹配之后会调用bus_type中的pci_epf_device_probe, 从dtb中找到"reserved-memory"节点，在其中找到"pcie-bar"节点中的数值作为bar空间地址，调用相关pci_epf_driver的probe(pci_epf_asr_probe)
4. 分析pci_epf_asr_probe，通过match匹配到epf device后，创建pci_epf_asr(pnet)，填充pci_epf_asr->header和pci_epf_asr->epf
- PCIe EPF和EPC的绑定
1. pci_ep_cfs_init创建pci_ep/functions，pci_ep/controllers
2. 挂载好configfs后会出现两个文件夹，pci_ep_cfs_add_epf_group创建pci_epf_asr, pci_ep_cfs_add_epc_group创建
3. mkdir functions/pci_epf_asr/func1; 在pci_epf_asr文件夹中创建func1，回调pci_epf_make，其中文件夹中有pci_epf_attrs(vendorid, deviceid, revid...)通过store和show写入和读取, 创建epf_deivce(通过pci_epf_create),
4. ln -s functions/pci_epf_asr/func1 controller/8000000.pcie/;会去调用pci_epc_epf_link，在其中会将epc和epf结构连接，并调用epf中的bindi(pci_epf_asr_bind)进行功能的初始化相关function。
5. 分析pci_epf_asr_bind，通过pci_epc_write_header配置config空间的寄存器（通过dbi的方式去配），pci_epf_alloc_space，给epf分配bar空间(dma_alloc_coherent),并存下来bar空间的物理地址和size和bar_number，设置bar空间的inbound_atu的地址映射，pci_epc_set_msi设置msi中断,设置DMA收发数据的缓存区作为pnet收发数据的缓存区，初始化pnet_init
6. 分析pnet_init




















