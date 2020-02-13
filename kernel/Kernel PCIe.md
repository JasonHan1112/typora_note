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

