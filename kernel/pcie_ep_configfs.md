# PCI endpoint frame

## 内核路径：

- /drivers/pci/endpoint
- Documents/PCI/endpoint
- 使用时，需要配置configfs将pcie endpoint controller与pcie endpoint function进行一个绑定。

## pci endpoint frame有3部分组成：

pci endpoint controller library，pci endpoint function library，configfs layer

## pci endpoint controller library

- 通过devm_pci_epc_create()创建

- 提供controller能操作的pci的功能：

  pci_epc_linkup() pci_epc_mem_init() pci_epc_write_header() pci_epc_set_bar() pci_epc_clear_bar() pci_epc_raise_irq() pci_epc_start() pci_epc_stop()等

  
## pci endpoint function(EPF) library
- pci endpoint function 是一个总线，内核提供了驱动，通过pci_epf_register_driver()注册

## 重要的数据结构
```c
/*代表一个dw的pcie设备其中pcie_port表示是rc，ep表示是endpoint
后边重点分析ep*/
struct dw_pcie {
    struct device       *dev;                               
    void __iomem        *dbi_base;                           
    void __iomem        *dbi_base2;                         
    /* Used when iatu_unroll_enabled is true */                 void __iomem        *atu_base;             
    u32         num_viewport;                               
    u8          iatu_unroll_enabled;                         
    struct pcie_port    pp;                                 
    struct dw_pcie_ep   ep;                                 
    const struct dw_pcie_ops *ops;      
};
/*该结构体代表一个pcie的ep，其中epc表示是一个
endpoint controller*/
struct dw_pcie_ep {                                         
    struct pci_epc      *epc;                               
    struct dw_pcie_ep_ops   *ops;                           
    phys_addr_t     phys_base;                               
    size_t          addr_size;                               
    size_t          page_size;                               
    u8          bar_to_atu[6];                               
    phys_addr_t     *outbound_addr;                         
    unsigned long       *ib_window_map;                     
    unsigned long       *ob_window_map;                         u32         num_ib_windows;
    u32         num_ob_windows;                             
    void __iomem        *msi_mem;                           
    phys_addr_t     msi_mem_phys;                           
    u8          msi_cap;    /* MSI capability offset */     
    u8          msix_cap;   /* MSI-X capability offset */ 
}; 
/*ops表示endpoint controller能做什么*/
struct pci_epc {                                             
    struct device           dev;                             
    struct list_head        pci_epf;                         
    const struct pci_epc_ops    *ops;                       
    struct pci_epc_mem      *mem;                           
    u8              max_functions;                           
    struct config_group     *group;                         
    /* spinlock to protect against concurrent access of EP controller */  
    spinlock_t          lock;
};       

/*表示一个endpoint function，其中epc表示与该function绑定的controller*/
struct pci_epf {                                             
    struct device       dev;                                 
    const char      *name;                                   
    struct pci_epf_header   *header;                         
    struct pci_epf_bar  bar[6];                             
    u8          msi_interrupts;                             
    u16         msix_interrupts;                             
    u8          func_no;                                     
    struct pci_epc      *epc;                               
    struct pci_epf_driver   *driver;                         
    struct list_head    list;                               
};          
```
## PCI endpoint frame初始化流程

- 在pcie-designware-plat.c中进行框架的初始化。资源的获取是以platform设备的probe从设备树中来获取，并创建dw_pcie结构体
- 通过dw_plat_add_pcie_ep()创建了dw_pcie_ep设备，资源也是通过platform设备从设备树中拿取，从而通过devm_pci_epc_create()创建pci_epc，在其中会通过pci_ep_cfs_add_epc_group()将epc的configfs注册上。
## pci endpoint function初始化流程

在系统中会注册上pci_epf的driver（pcie_ipc_ep_init()，其中注册完pci_epf的driver之后，会创建一个与pci_epf的driver同名的configfs_group，后续会在configfs中用到该名称创建pci_epf的device），随后通过configfs来创建pci_epf的device

# PCI endpoint configfs

## configfs

可以通过mkdir ，ln -s xxx xxx以及rmdir等操作和内核交互，也可对一些attributs进行read 和write操作和内核进行交互。

## pci-ep-configfs的初始化

- 在插入pci-ep-cfs.ko模块的时候，会注册pci_ep_cfs_subsys，该subsys文件名是”pcie_ep“，还会在其中新建functions_group和controllers_group (functions文件夹和controllers文件夹)，但是其中的configfs_group_operations没有make_group和drop_item
- 在pci_epf_register_driver的时候会向pci_ep_cfs中添加一个与pci_epf_driver名字相同的config_group (通过pci_ep_cfs_add_epf_group())
- 上述添加的group的名字与pci_epf_driver的名字一致（pcie_asr_epf），为后续pci_epf_device probe的时候进行match，并且将pci_epf_group_type与config_group相绑定，在pci_epf_group_type中有make_group和drop_item方法，用于在configfs中创建和删除时回调。
- 设置完config_group之后，会在configfs中以functions_group为父节点建立相应的目录和文件
- 用户在configfs的pcie_asr_epf/functions文件夹下mkdir func1时会调用之前注册在pci_epf_group_type下的make_group，从而通过pci_epf_create创建了pci_epf_device，以及相关属性attrs(vendorid, deviceid, revid, msi_interrupts, subclass_code, baseclass_code等)随后进行bus（pci-epf bus）的枚举，执行pci_epf_asr bus 的 driver中的probe。
- ln -s xxx/functions/func1 controllers/xxx (在controller中建立一个func1的软链接)该操作会调用allow_link（pci_epc_epf_link）向epc添加epf，并通过pci_epf_bind()去调用自己的epf中需要实现的回调。
- 删除symbole link 会调用.drop_link，从而去调用pci_epf_unbind()（自己的epf中要实现的）
- 给controllers/start中写1，让pcie链路training