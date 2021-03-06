# Linux DRM

## introduction

support the needs of complex graphics devices. gracphic drivers in the kernel may make use of DRM functions to make task like memory management, interrupt handling and DMA easier, and provide a uniform interface to applicatinos.
- The DRM layer provides several services to graphics drivers, many of them driven by the application interfaces it provides through libdrm, the library that wraps most of the DRM ioctls. These include vblank event handling, memory management, output management, framebuffer management, command submission & fencing, suspend/resume support, and DMA services.

## device instance and driver handling

- a device instance for a drm driver is represented by *struct drm_device* this is allocated and initialized with *devm_drm_dev_alloc()*, usually from bus-specific-probe() callbacks implemented by the driver. the driver then needs to initialize all the various subsystems for the drm device like memory management, vblank handling, modesetting support and initial output configuiration plus obvious initialize all th corresponding hardware bites. when everything is up and running and ready for userspace the device instance can be published using *drm_dev_register()*.
- The driver must protect regions that is accessing device resources to prevent use after they’re released. This is done using drm_dev_enter() and drm_dev_exit().

```c
struct drm_device {
  struct list_head legacy_dev_list; //legacy, list of devices per driver
  int if_version;   //highest interface verion set
  struct kref ref;
  struct device *dev;
  struct drm_driver *driver;    //drm driver managing the device
  void *dev_private;
  struct drm_minor *primary;    //primary node
  struct drm_minor *render; //render node
  bool registered; //internally used by drm_dev_register() and drm_connector_register()
  struct drm_master *master;
  u32 driver_features;  //per-device driver features
  bool unplugged;
  struct inode *anon_inode;
  char *unique; //unique name of the device
  struct mutex struct_mutex;
  struct mutex master_mutex;
  atomic_t open_count;
  struct mutex filelist_mutex;
  struct list_head filelist;
  struct list_head filelist_internal;
  struct mutex clientlist_mutex;
  struct list_head clientlist;
  bool irq_enabled;
  int irq;
  bool vblank_disable_immediate;
  struct drm_vblank_crtc *vblank;
  spinlock_t vblank_time_lock;
  spinlock_t vbl_lock;
  u32 max_vblank_count;
  struct list_head vblank_event_list;
  spinlock_t event_lock;
  struct drm_agp_head *agp;
  struct pci_dev *pdev; //pci device structure
#ifdef __alpha__;
  struct pci_controller *hose;
#endif;
  unsigned int num_crtcs;   //number of crtc on this device
  struct drm_mode_config mode_config;
  struct mutex object_name_lock;
  struct idr object_name_idr;
  struct drm_vma_offset_manager *vma_offset_manager;
  struct drm_vram_mm *vram_mm; //VRAM MM memory manager
  enum switch_power_state switch_power_state;
  struct drm_fb_helper *fb_helper;  //pointer to the fbdev emulation structure. Set by drm_fb_helper_init()
};

struct drm_driver {
  int (*load) (struct drm_device *, unsigned long flags);   //backward-compatible driver callback to complete initialization
  int (*open) (struct drm_device *, struct drm_file *); //driver callback when a new struct drm_file is opened
  void (*postclose) (struct drm_device *, struct drm_file *);   //the display/modeset side of DRM can only be owned by exactly one struct drm_file
  void (*lastclose) (struct drm_device *);
  void (*unload) (struct drm_device *);
  void (*release) (struct drm_device *);
  irqreturn_t(*irq_handler) (int irq, void *arg);   //Interrupt handler called when using drm_irq_install().
  void (*irq_preinstall) (struct drm_device *dev);  //Optional callback used by drm_irq_install() which is called before the interrupt handler is registered. 
  int (*irq_postinstall) (struct drm_device *dev);  //Optional callback used by drm_irq_install() which is called after the interrupt handler is registered. 
  void (*irq_uninstall) (struct drm_device *dev);
  int (*master_set)(struct drm_device *dev, struct drm_file *file_priv, bool from_open);
  void (*master_drop)(struct drm_device *dev, struct drm_file *file_priv);
  int (*debugfs_init)(struct drm_minor *minor);
  void (*gem_free_object) (struct drm_gem_object *obj);
  void (*gem_free_object_unlocked) (struct drm_gem_object *obj);
  int (*gem_open_object) (struct drm_gem_object *, struct drm_file *);  //This callback is deprecated in favour of drm_gem_object_funcs.open 通过drm的ioctl,或者drm_framebuffer_funcs
  void (*gem_close_object) (struct drm_gem_object *, struct drm_file *);    //This callback is deprecated in favour of drm_gem_object_funcs.close.
  void (*gem_print_info)(struct drm_printer *p, unsigned int indent, const struct drm_gem_object *obj);
  struct drm_gem_object *(*gem_create_object)(struct drm_device *dev, size_t size);
  int (*prime_handle_to_fd)(struct drm_device *dev, struct drm_file *file_priv, uint32_t handle, uint32_t flags, int *prime_fd);
  int (*prime_fd_to_handle)(struct drm_device *dev, struct drm_file *file_priv, int prime_fd, uint32_t *handle);
  struct dma_buf * (*gem_prime_export)(struct drm_gem_object *obj, int flags);
  struct drm_gem_object * (*gem_prime_import)(struct drm_device *dev, struct dma_buf *dma_buf);
  int (*gem_prime_pin)(struct drm_gem_object *obj);
  void (*gem_prime_unpin)(struct drm_gem_object *obj);
  struct sg_table *(*gem_prime_get_sg_table)(struct drm_gem_object *obj);
  struct drm_gem_object *(*gem_prime_import_sg_table)(struct drm_device *dev,struct dma_buf_attachment *attach, struct sg_table *sgt);
  void *(*gem_prime_vmap)(struct drm_gem_object *obj);
  void (*gem_prime_vunmap)(struct drm_gem_object *obj, void *vaddr);
  int (*gem_prime_mmap)(struct drm_gem_object *obj, struct vm_area_struct *vma);
  int (*dumb_create)(struct drm_file *file_priv,struct drm_device *dev, struct drm_mode_create_dumb *args); //This creates a new dumb buffer in the driver's backing storage manager This handle can then be wrapped up into a framebuffer modeset object
  int (*dumb_map_offset)(struct drm_file *file_priv,struct drm_device *dev, uint32_t handle, uint64_t *offset);
  int (*dumb_destroy)(struct drm_file *file_priv,struct drm_device *dev, uint32_t handle);
  const struct vm_operations_struct *gem_vm_ops;
  int major;
  int minor;
  int patchlevel;
  char *name;
  char *desc;
  char *date;
  u32 driver_features;
  const struct drm_ioctl_desc *ioctls;
  int num_ioctls;
  const struct file_operations *fops;   //drm device file_operations
};
```


## DRM Memory Management

- Modern Linux systems require large amount of graphics memory to store frame buffers, textures, vertices and other graphics-related data. Given the very dynamic nature of many of that data, managing graphics memory efficiently is thus crucial for the graphics stack and plays a central role in the DRM infrastructure.

- Modern Linux systems require large amount of graphics memory to store frame buffers, textures, vertices and other graphics-related data. Given the very dynamic nature of many of that data, managing graphics memory efficiently is thus crucial for the graphics stack and plays a central role in the DRM infrastructure.

- TTM was the first DRM memory manager to be developed and tried to be a one-size-fits-them all solution. It provides a single userspace API to accommodate the need of all hardware, supporting both Unified Memory Architecture (UMA) devices and devices with dedicated video RAM (i.e. most discrete video cards).

- This resulted in a large, complex piece of code that turned out to be hard to use for driver development.

- instead of providing a solution to every graphics memory-related problems, GEM identified common code between drivers and created a support library to share it. GEM has simpler initialization and execution requirements than TTM, but has no video RAM management capabilities and is thus limited to UMA devices.

## TTM initialization

- Drivers wishing to support TTM must pass a filled ttm_bo_driver structure to ttm_bo_device_init, together with an initialized global reference to the memory manager. The ttm_bo_driver structure contains several fields with function pointers for initializing the TTM, allocating and freeing memory, waiting for command completion and fence synchronization, and memory migration.

- There should be one global reference structure for your memory manager as a whole, and there will be others for each object created by the memory manager at runtime. Your global TTM should have a type of TTM_GLOBAL_TTM_MEM.

- and the init and release hooks should point at your driver-specific init and release routines, which probably eventually call ttm_mem_global_init and ttm_mem_global_release, respectively.

- Once your global TTM accounting structure is set up and initialized by calling ttm_global_item_ref() on it, you need to create a buffer object TTM to provide a pool for buffer object allocation by clients and the kernel itself.

- driver-specific init and release functions may be provided, likely eventually calling ttm_bo_global_ref_init() and ttm_bo_global_ref_release(), respectively. Also, like the previous object, ttm_global_item_ref() is used to create an initial reference count for the TTM, which will call your initialization function.

## PRIME
- 是管理共享dma_buf的一种机制，最初是为了OPTIMUS multi-gpu平台使用，对于用户空间来说，PRIMEbuffers是基于文件描述符的.
- 和GEM global name相似，PRIME文件描述符也可以用来在进程之间共享buffer，他提供了额外的安全机制：文件描述符要在进程间共享，必须明确通过UNIX socket。
- 支持PRIME的驱动必须设置struct drm_driver中的DRIVER_PRIME bit，而且必须实现prime_handle_to_fd和prime_fd_to_handle。
- gem_prime_export是将该drm_gem_object中的dma_buf导出，gem_prime_import是将dma_buf导入到drm_gem_object中。

## The Graphics Execution Manager (GEM)

- GEM exposes a set of standard memory-related operations to userspace and a set of helper functions to drivers, and let drivers implement hardware-specific operations with their own private API.

- GEM is data-agnostic. It manages abstract buffer objects without knowing what individual buffers contain. APIs that require knowledge of buffer contents or purpose, such as buffer allocation or synchronization primitives, are thus outside of the scope of GEM and must be implemented using driver-specific ioctls.

### GEM Initialization

- Drivers that use GEM must set the DRIVER_GEM bit in the struct struct drm_driver driver_features field. The DRM core will then automatically initialize the GEM core before calling the load operation. Behind the scene, this will create a DRM Memory Manager object which provides an address space pool for object allocation.

- In a KMS configuration, drivers need to allocate and initialize a command ring buffer following core GEM initialization if required by the hardware. UMA devices usually have what is called a “stolen” memory region, which provides space for the initial framebuffer and large, contiguous memory regions required by the device. This space is typically not managed by GEM, and must be initialized separately into its own DRM MM object.

### GEM Objects Creation

- GEM objects are represented by an instance of struct struct drm_gem_object. Drivers usually need to extend GEM objects with private information and thus create a driver-specific GEM object structure type that embeds an instance of struct struct drm_gem_object.

- To create a GEM object, a driver allocates memory for an instance of its specific GEM object type and initializes the embedded struct struct drm_gem_object with a call to drm_gem_object_init(). 

- GEM uses shmem to allocate anonymous pageable memory. drm_gem_object_init() will create an shmfs file of the requested size and store it into the struct struct drm_gem_object filp field. The memory is used as either main storage for the object when the graphics hardware uses system memory directly or as a backing store otherwise.

### GEM Objects Lifetime

- All GEM objects are reference-counted by the GEM core. References can be acquired and release by calling drm_gem_object_get() and drm_gem_object_put() respectively.

### GEM Objects Naming

- Communication between userspace and the kernel refers to GEM objects using local handles, global names or, more recently, file descriptors. All of those are 32-bit integer values; the usual Linux kernel limits apply to the file descriptors.

- GEM handles are local to a DRM file. Applications get a handle to a GEM object through a driver-specific ioctl, and can use that handle to refer to the GEM object in other standard or driver-specific ioctls. Closing a DRM file handle frees all its GEM handles and dereferences the associated GEM objects.

- To create a handle for a GEM object drivers call drm_gem_handle_create(). The function takes a pointer to the DRM file and the GEM object and returns a locally unique handle. When the handle is no longer needed drivers delete it with a call to drm_gem_handle_delete(). Finally the GEM object associated with a handle can be retrieved by a call to drm_gem_object_lookup().

- Handles don’t take ownership of GEM objects, they only take a reference to the object that will be dropped when the handle is destroyed. 

- GEM names are similar in purpose to handles but are not local to DRM files. They can be passed between processes to reference a GEM object globally. Names can’t be used directly to refer to objects in the DRM API, applications must convert handles to names and names to handles using the DRM_IOCTL_GEM_FLINK and DRM_IOCTL_GEM_OPEN ioctls respectively. The conversion is handled by the DRM core without any driver-specific support.

### GEM Obkects Mapping

- Because mapping operations are fairly heavyweight GEM favours read/write-like access to buffers, implemented through driver-specific ioctls, over mapping buffers to userspace. However, when random access to the buffer is needed (to perform software rendering for instance), direct access to the object can be more efficient.

- The mmap system call can’t be used directly to map GEM objects, as they don’t have their own file handle. Two alternative methods currently co-exist to map GEM objects to userspace. The first method uses a driver-specific ioctl to perform the mapping operation, calling do_mmap() under the hood. This is often considered dubious, seems to be discouraged for new GEM-enabled drivers, and will thus not be described here. The second method uses the mmap system call on the DRM file handle. void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset); DRM identifies the GEM object to be mapped by a fake offset passed through the mmap offset argument. Prior to being mapped, a GEM object must thus be associated with a fake offset. To do so, drivers must call drm_gem_create_mmap_offset() on the object.

- The GEM core provides a helper method drm_gem_mmap() to handle object mapping. The method can be set directly as the mmap file operation handler. It will look up the GEM object based on the offset value and set the VMA operations to the struct drm_driver gem_vm_ops field. Note that drm_gem_mmap() doesn’t map memory to userspace, but relies on the driver-provided fault handler to map pages individually, To use drm_gem_mmap(), drivers must fill the struct struct drm_driver gem_vm_ops field with a pointer to VM operations.

### Memory Coherency

- if the CPU accesses an object after the GPU has finished rendering to the object, then the object must be made coherent with the CPU’s view of memory, usually involving GPU cache flushing of various kinds. This core CPU<->GPU coherency management is provided by a device-specific ioctl, which evaluates an object’s current domain and performs any necessary flushing or synchronization to put the object into the desired coherency domain (note that the object may be busy, i.e. an active render target; in that case, setting the domain blocks the client and waits for rendering to complete before performing any necessary flushing operations).

### Command Execution

- Client programs construct command buffers containing references to previously allocated memory objects, and then submit them to GEM. At that point, GEM takes care to bind all the objects into the GTT, execute the buffer, and provide necessary synchronization between clients accessing the same buffers. This often involves evicting some objects from the GTT and re-binding others (a fairly expensive operation), and providing relocation support which hides fixed GTT offsets from clients. Clients must take care not to submit command buffers that reference more objects than can fit in the GTT; otherwise, GEM will reject them and no rendering will occur. Similarly, if several objects in the buffer require fence registers to be allocated for correct rendering (e.g. 2D blits on pre-965 chips), care must be taken not to require more fence registers than are available to the client. Such resource management should be abstracted from the client in libdrm.

## GEM TTM PRIME

GEM提供了一个用户空间的通用接口，TTM可以认为是GEM的底层，用来表示VRAM的底层，PRIME也是一个drm_gem_object之间共享DMA BUFFER的机制。

## GEM frame 数据结构 

```c
struct drm_gem_object_funcs {
  void (*free)(struct drm_gem_object *obj); //Deconstructor for drm_gem_objects.
  int (*open)(struct drm_gem_object *obj, struct drm_file *file);   //Called upon GEM handle creation.
  void (*close)(struct drm_gem_object *obj, struct drm_file *file); //Called upon GEM handle release.
  void (*print_info)(struct drm_printer *p, unsigned int indent, const struct drm_gem_object *obj);
  struct dma_buf *(*export)(struct drm_gem_object *obj, int flags); //Export backing buffer as a dma_buf. Optional
  int (*pin)(struct drm_gem_object *obj);   //Pin backing buffer in memory. Used by the drm_gem_map_attach() helper.
  void (*unpin)(struct drm_gem_object *obj);    //Unpin backing buffer. Used by the drm_gem_map_detach() helper.
  struct sg_table *(*get_sg_table)(struct drm_gem_object *obj);
  void *(*vmap)(struct drm_gem_object *obj);    //Returns a virtual address for the buffer. Used by the drm_gem_dmabuf_vmap() helper.
  void (*vunmap)(struct drm_gem_object *obj, void *vaddr);
  int (*mmap)(struct drm_gem_object *obj, struct vm_area_struct *vma);  //Virtual memory operations used with mmap.
  const struct vm_operations_struct *vm_ops;    //Virtual memory operations used with mmap.
};

//This structure defines the generic parts for GEM buffer objects, which are mostly around handling mmap and userspace handles.
struct drm_gem_object {
  struct kref refcount; //Reference count of this object
  unsigned handle_count;    //This is the GEM file_priv handle count of this object.
  struct drm_device *dev;
  struct file *filp;
  struct drm_vma_offset_node vma_node;
  size_t size;
  int name;
  struct dma_buf *dma_buf;
  struct dma_buf_attachment *import_attach;
  struct dma_resv *resv;
  struct dma_resv _resv;
  const struct drm_gem_object_funcs *funcs; //Optional GEM object functions. If this is set, it will be used instead of the corresponding drm_driver GEM callbacks. New drivers should use this.
};
/*drm_gem_object负责drm的内存管理，可以和PRIME框架结合*/
```
- 
## DRM by ASR initialize

### 在ASR的数据结构中将drm_device注册为miscdevice的子设备
    首先注册miscdevice，并设置DMA_BIT_MASK(64)
    ```c
    static const struct file_operations asr_drm_miscdev_fops = {
        .owner = THIS_MODULE,
    };

    static struct miscdevice asr_drm_miscdev = {
        .minor = MISC_DYNAMIC_MINOR,
        .name = "asrdrm",
        .fops = &asr_drm_miscdev_fops,
    };

    ```
### 使用asr_drm_driver创建drm设备

    drm_dev_alloc(&asr_drm_driver, asr_drm_miscdev.this_device);
    ```c
    static struct drm_driver asr_drm_driver = {
    .driver_features    = DRIVER_GEM | DRIVER_PRIME, //设置了PRIME需要实现prime_handle_to_fd prime_fd_to_handle
    .gem_vm_ops     = &asr_drm_vm_ops, //open gem object ref++, close gem object ref--, 一般用为了gem mmap做准备
    .gem_free_object    = asr_gem_free_object, //free掉gem object
    
    /*GEM drivers must use the drm_gem_prime_handle_to_fd() and drm_gem_prime_fd_to_handle() helper functions. Those helpers rely on the driver gem_prime_export and gem_prime_import operations to create a dma-buf instance from a GEM object (dma-buf exporter role) and to create a GEM object from a dma-buf instance (dma-buf importer role).*/

    .prime_handle_to_fd = drm_gem_prime_handle_to_fd, //从drm_device中导出dma_buf，可以给另外的dev用fb，prime export function for gem drivers
    .prime_fd_to_handle = drm_gem_prime_fd_to_handle, //从fd找到dma_buf，然后将其导入到drm_device中，并建立handle，prime import function for gem drivers
    .gem_prime_import   = drm_gem_prime_import, //向drm_device中导入dma_buf
    .gem_prime_export   = drm_gem_prime_export, //从drm_device中导出dma_buf
    .gem_prime_get_sg_table = asr_gem_prime_get_sg_table, //(copy)新建一个sg_table. provide a scatter/gather table of pinned pages
    .gem_prime_vmap     = asr_gem_prime_vmap,
    .gem_prime_vunmap   = asr_gem_prime_vunmap,
    .gem_prime_mmap     = asr_gem_prime_mmap,
    .open           = asr_drm_open,
    .ioctls         = ioctls, //新添加的ioctls
#if defined(CONFIG_DEBUG_FS)
    .debugfs_init       = asr_drm_debugfs_init,
#endif
    .num_ioctls     = ASR_DRM_NUM_IOCTLS,
    .fops           = &asr_drm_driver_fops,
    .name           = DRIVER_NAME,
    .desc           = DRIVER_DESC,
    .date           = DRIVER_DATE,
    .major          = DRIVER_MAJOR,
    .minor          = DRIVER_MINOR,
};

    ```
### 设置unique
    drm_dev_set_unique(drm, "asrdrm");


### 创建drm private data
    asr_drm_priv_init(struct drm_device *drm)
        |
        asr_drm_mempool_init(priv)
    - 查找reserved-memory的子节点gem-pool的地址及大小
    - dev_private中保存了gem-pool的大小和起始地址

### 注册drm_dev
    drm_dev_register(drm, 0);
        |
        drm_minor_register(dev, DRM_MINOR_CONTROL);
        |
        drm_minor_register(dev, DRM_MINOR_RENDER);
        |
        drm_minor_register(dev, DRM_MINOR_PRIMARY);
        |
        dev->driver->load
        如果有DRIVER_MODESET还要注册MODESET(ASR只是drm以及gem)





## KMS
可以理解成KMS是FBDEV的替代，KMS中会虚拟出fbdev
### SMI DRM Frame 初始化
SMI用的是pcie下drm kms的框架来做的显示
```c
static struct drm_driver driver = {
#ifdef PRIME
    .driver_features = DRIVER_HAVE_IRQ | DRIVER_IRQ_SHARED |DRIVER_GEM |DRIVER_PRIME | DRIVER_MODESET,
#else
    .driver_features = DRIVER_HAVE_IRQ | DRIVER_IRQ_SHARED |DRIVER_GEM | DRIVER_MODESET,
#endif
    .load = smi_driver_load,
    .unload = smi_driver_unload,
#if (((LINUX_VERSION_CODE >= KERNEL_VERSION(3,17,0)) && \
    (LINUX_VERSION_CODE < KERNEL_VERSION(4,14,0)))&& !defined(RHEL_RELEASE_VERSION) )|| \
    (defined(RHEL_RELEASE_VERSION) && RHEL_VERSION_LOWER_THAN(7,5))
    .set_busid = drm_pci_set_busid,
#endif
    .fops = &smi_driver_fops,
    .name = DRIVER_NAME,
    .desc = DRIVER_DESC,
    .date = DRIVER_DATE,
    .major = DRIVER_MAJOR,
    .minor = DRIVER_MINOR,
    .patchlevel = DRIVER_PATCHLEVEL,
#if ((LINUX_VERSION_CODE <= KERNEL_VERSION(3,12,0))&& !defined(RHEL_RELEASE_VERSION)  )|| \
    (defined(RHEL_RELEASE_VERSION) && RHEL_VERSION_LOWER_THAN(7,3))
    .gem_init_object = smi_gem_init_object,
#endif
#if ((LINUX_VERSION_CODE < KERNEL_VERSION(4,7,0))&& !defined(RHEL_RELEASE_VERSION)  )|| \
    (defined(RHEL_RELEASE_VERSION) && RHEL_VERSION_LOWER_THAN(7,4))
    .gem_free_object = smi_gem_free_object,
#else
    .gem_free_object_unlocked = smi_gem_free_object,
#endif
    .dumb_create = smi_dumb_create,
    .dumb_map_offset = smi_dumb_mmap_offset,
#if ((LINUX_VERSION_CODE < KERNEL_VERSION(3,12,0))&& !defined(RHEL_RELEASE_VERSION)  )|| \
    (defined(RHEL_RELEASE_VERSION) && RHEL_VERSION_LOWER_THAN(7,4))
    .dumb_destroy = smi_dumb_destroy,
#else
    .dumb_destroy = drm_gem_dumb_destroy,
#endif
#if (((LINUX_VERSION_CODE >= KERNEL_VERSION(4,4,0)) && \
    (LINUX_VERSION_CODE < KERNEL_VERSION(4,12,0))&& !defined(RHEL_RELEASE_VERSION)) )|| \
    (defined(RHEL_RELEASE_VERSION) && \
    (RHEL_VERSION_HIGHER_THAN(7,4) && RHEL_VERSION_LOWER_THAN(7,5)))
    .get_vblank_counter = drm_vblank_no_hw_counter,
#endif
    .enable_vblank      = smi_enable_vblank,
    .disable_vblank     = smi_disable_vblank,
    .irq_preinstall = smi_irq_preinstall,
    .irq_postinstall = smi_irq_postinstall,
    .irq_uninstall = smi_irq_uninstall,
    .irq_handler        = smi_drm_interrupt,
#ifdef PRIME
#if ((LINUX_VERSION_CODE > KERNEL_VERSION(3,14,0)   )&& !defined(RHEL_RELEASE_VERSION) )|| \
    (defined(RHEL_RELEASE_VERSION) && RHEL_VERSION_HIGHER_THAN(7,3))
    .prime_handle_to_fd = drm_gem_prime_handle_to_fd,
    .prime_fd_to_handle = drm_gem_prime_fd_to_handle,

    .gem_prime_import   = drm_gem_prime_import,
    .gem_prime_export   = drm_gem_prime_export,

    .gem_prime_get_sg_table = smi_gem_prime_get_sg_table,
    .gem_prime_import_sg_table = smi_gem_prime_import_sg_table,
    .gem_prime_vmap     = smi_gem_prime_vmap,
    .gem_prime_vunmap   = smi_gem_prime_vunmap,
    .gem_prime_pin      = smi_gem_prime_pin,
    .gem_prime_unpin    = smi_gem_prime_unpin,
#endif
#if ((LINUX_VERSION_CODE >= KERNEL_VERSION(3,17,0)  )&& !defined(RHEL_RELEASE_VERSION) )|| \
    (defined(RHEL_RELEASE_VERSION) && RHEL_VERSION_HIGHER_THAN(7,3))
    .gem_prime_res_obj = smi_gem_prime_res_obj,
#endif
#endif
};



struct ttm_bo_driver smi_bo_driver = {
    .ttm_tt_create = smi_ttm_tt_create,
    .ttm_tt_populate = smi_ttm_tt_populate,
    .ttm_tt_unpopulate = smi_ttm_tt_unpopulate,
    .init_mem_type = smi_bo_init_mem_type,
    .evict_flags = smi_bo_evict_flags,
#if ((LINUX_VERSION_CODE >= KERNEL_VERSION(4,10,0))  && !defined(RHEL_RELEASE_VERSION) ) || \
    (defined(RHEL_RELEASE_VERSION) && RHEL_VERSION_HIGHER_THAN(7,4))
    .eviction_valuable = ttm_bo_eviction_valuable,
#endif
#if ((LINUX_VERSION_CODE < KERNEL_VERSION(4,8,0) )  && !defined(RHEL_RELEASE_VERSION) ) \
    || (defined(RHEL_RELEASE_VERSION) && RHEL_VERSION_LOWER_THAN(7,4))
    .move = smi_bo_move,
#else
    .move = NULL,
#endif
#if ( (LINUX_VERSION_CODE >= KERNEL_VERSION(4,7,0) && \
    LINUX_VERSION_CODE < KERNEL_VERSION(4,11,0)) && !defined(RHEL_RELEASE_VERSION) )  || \
    (defined(RHEL_RELEASE_VERSION) && (RHEL_VERSION_HIGHER_THAN(7,4) && RHEL_VERSION_LOWER_THAN(7,5)))
    .lru_tail = &ttm_bo_default_lru_tail,
    .swap_lru_tail = &ttm_bo_default_swap_lru_tail,
#endif
    .verify_access = smi_bo_verify_access,
    .io_mem_reserve = &smi_ttm_io_mem_reserve,
    .io_mem_free = &smi_ttm_io_mem_free,
#if ((LINUX_VERSION_CODE >= KERNEL_VERSION(4,11,0) && \
    LINUX_VERSION_CODE < KERNEL_VERSION(4,15,0))  && !defined(RHEL_RELEASE_VERSION) ) || \
    (defined(RHEL_RELEASE_VERSION) && (RHEL_VERSION_HIGHER_THAN(7,5) && RHEL_VERSION_LOWER_THAN(7,6)))
    .io_mem_pfn = ttm_bo_default_io_mem_pfn,
#endif
};


```
smi_init
  |
  pci_register_driver
    |
    smi_pci_probe
      |
      drm_get_pci_dev
        |
        drm_dev_register
          |
          drm_driver->load




drm_driver.smi_driver_load
  |
  pci_enable_device
  |
  smi_device_init//设置crtc的数量，设置pci dma mask，获取bar1空间(寄存器空间)，初始化vram
    |
    pci_set_dma_mask
    |
    pci_resource_start
    |
    pci_resource_len
    |
    smi_vram_init//获取bar0的空间
  |
  ddk768_initChip
  |
  ddk768_deInit
  |
  hw768_init_hdmi
  |
  smi_mm_init//创建ttm相关数据结构，用于内核管理vram
    |
    smi_ttm_global_init
      |
      drm_global_item_ref
    |
    ttm_bo_device_init//创建ttm_bo_device，注册ttm_bo_driver
    |
    ttm_bo_init_mm//调用ttm_bo_driver->init_mem_type初始化ttm_mem_type_manager
  |
  drm_vblank_init//initialize vblank support
  |
  drm_irq_install//注册irq
  |
  smi_modeset_init
    |
    drm_mode_config_init//initialize DRM mode_config structure
    //根据实际情况填充mode_config
    smi_crtc_init
      |
      smi_plane_init//初始化两次，DRM_PLANE_TYPE_PRIMARY，DRM_PLANE_TYPE_CURSOR，
        |
        drm_universal_plane_init //初始化universal plane object注册相关plane的formats以及functions
        |
        drm_plane_helper_add //向plane中添加drm_plane_help_funcs
      drm_crtc_init_with_planes//初始化crtc并向其注册指定的plane
      |
      drm_crtc_helper_add//向crtc中添加helper_functions
    |
    smi_encoder_init//初始化encoder
      |
      drm_encoder_init//初始化encoder
      |
      drm_encoder_helper_add//添加helper函数
    smi_connector_init//connector 初始化
      |
      drm_connector_init//注册connector函数，type并且初始化connector
    |
    drm_mode_connector_attach_encoder//connector和encoder进行attach
    |
    smi_fbdev_init//初始化虚拟framebuffer
      |
        drm_fb_helper_prepare//建立一个drm_fb_helper(emulate fbdev on top of KMS)
      |
      drm_fb_helper_init//初始化一个drm_fb_helper，向其中填充上connector info，crtc info结构体
      |
      drm_fb_helper_single_add_all_connectors//add all connectors to fbdev emulation helper
      |
      drm_helper_disable_unused_functions//disable all the possible outputs/crtcs before entering KMS mode
      |
      drm_fb_helper_initial_config//Scans the CRTCs and connectors and tries to put together an initial setup.
        |
        __drm_fb_helper_initial_config_and_unlock
          |
          drm_setup_crtcs//使能connector并且设置drm_fb_helper_crtc
          |
          drm_fb_helper_single_fb_probe//分配存储，创建fbdev info structure通过->fb_probe
          |
          drm_setup_crtcs_fb//设置fb_info和connector匹配
          |
          register_framebuffer //registers the fbdev and so allows userspace to call into the driver through the fbdev interfaces.
  drm_kms_helper_poll_init
    |
    drm_kms_helper_poll_enable//re-enable output polling
