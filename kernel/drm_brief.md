# Linux DRM

## introduction

support the needs of complex graphics devices. gracphic drivers in the kernel may make use of DRM functions to make task like memory management, interrupt handling and DMA easier, and provide a uniform interface to applicatinos.
- The DRM layer provides several services to graphics drivers, many of them driven by the application interfaces it provides through libdrm, the library that wraps most of the DRM ioctls. These include vblank event handling, memory management, output management, framebuffer management, command submission & fencing, suspend/resume support, and DMA services.

## device instance and driver handling

- a device instance for a drm driver is represented by *struct drm_device* this is allocated and initialized with *devm_drm_dev_alloc()*, usually from bus-specific-probe() callbacks implemented by the driver. the driver then needs to initialize all the various subsystems for the drm device like memory management, vblank handling, modesetting support and initial output configuiration plus obvious initialize all th corresponding hardware bites. when everything is up and running and ready for userspace the device instance can be published using *drm_dev_register()*.
- The driver must protect regions that is accessing device resources to prevent use after they’re released. This is done using drm_dev_enter() and drm_dev_exit().

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















