# Chapter 14: The Linux Device Model
## Kobjects, Ksets, and Subsystems
### Reference counting of objects
- One way of tracking the lifecycle of such objects is through reference counting. When no code in the kernel holds a reference to a given object, that object has finished its useful life and can be deleted.
### Sysfs representation
- Every object that shows up in sysfs has, underneath it, a kobject that interacts with the kernel to create its visible representation.
### Data structure glue
- The device model is, in its entirety, a fiendishly complicated data structure made up of multiple hierarchies with numerous links between them. The kobject implements this structure and holds it together.
### Hotplug event handling
- The kobject subsystem handles the generation of events that notify user space about the comings and goings of hardware on the system.
### Kobject Basics
- they are all services performed on behalf of other objects. A kobject, in other words, is of little interest on its own; it exists only to tie a higher-level object into the device model. Thus, it is rare (even unknown) for kernel code to create a standalone kobject; instead, kobjects are used to control access to a larger, domain-specific object.
- Every kobject needs to have an associated kobj_type structure. Confusingly, the pointer to this structure can be found in two different places. The kobject structure itself contains a field (called ktype) that can contain this pointer. 
- Sysfs entries for kobjects are always directories, so a call to kobject_add results in he creation of a directory in sysfs. Usually that directory contains one or more attributes; we see how attributes are specified shortly.
- The name assigned to the kobject (with kobject_set_name) is the name used for the sysfs directory. Thus, kobjects that appear in the same part of the sysfs hierarchy must have unique names. Names assigned to kobjects should also be reasonable file names: they cannot contain the slash character, and the use of white space is strongly discouraged.
- The sysfs entry is located in the directory corresponding to the kobject’s parent pointer. If parent is NULL when kobject_add is called, it is set to the kobject embedded in the new kobject’s kset; thus, the sysfs hierarchy usually matches the internal hierarchy created with ksets. If both parent and kset are NULL, the sysfs directory is created at the top level, which is almost certainly not what you want.
#### Binary attributes
- That need really only comes about when data must be passed, untouched, between user space and the device. for example, uploading firmware to devices requires this feature.
-  they can be called multiple times for a single load with a maximum of one page worth of data in each call.
#### Hotplug Event Generation
- They are generated whenever a kobject is created or destroyed.
- Hotplug events turn into an invocation of /sbin/hotplug, which can respond to each event by loading drivers, creating device nodes, mounting partitions, or taking any other action that is appropriate.

### Buses, Devices, and Drivers
#### Bus methods
- int (*match)(struct device *device, struct device_driver *driver);
This method is called, perhaps multiple times, whenever a new device or driver
is added for this bus. It should return a nonzero value if the given device can be
handled by the given driver. 
```
static int ldd_match(struct device *dev, struct device_driver *driver)
{
    return !strncmp(dev->bus_id, driver->name, strlen(driver->name));
}
```
- int (*hotplug) (struct device *device, char **envp, int num_envp, char *buffer, int buffer_size);
This method allows the bus to add variables to the environment prior to the generation of a hotplug event in user space. 

#### Iterating over devices and drivers
If you are writing bus-level code, you may find yourself having to perform some operation on all devices or drivers that have been registered with your bus.
To operate on every device known to the bus, use: 
```
int bus_for_each_dev(struct bus_type *bus, struct device *start,
  void *data, int (*fn)(struct device *, void *));
  
int bus_for_each_drv(struct bus_type *bus, struct device_driver *start,
  void *data, int (*fn)(struct device_driver *, void *));
```

#### Devices
- struct device *parent
The device’s “parent” device—the device to which it is attached. In most cases, a parent device is some sort of bus or host controller. If parent is NULL, the device is a top-level device.

- Device structure embedding
- Driver structure embedding