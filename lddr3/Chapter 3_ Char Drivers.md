# Chapter 3: Char Drivers
- The first step of driver writing is defining the capabilities (the mechanism) the driver will offer to user programs. 
- they are conventionally located in the /dev directory. Special files for char drivers are identified by a “c” in the first column of the output of ls –l. Block devices appear in /dev as well, but they are identified by a “b.”
- the ls –l command, you’ll see two numbers (separated by a comma) in the device file entries before the date of the last modification, where the file length normally appears. These numbers are the major and minor device number for the particular device. The following listing shows a few devices as they appear on a typical system. Their major numbers are 1, 4, 7, and 10, while the minors are 1, 3, 5, 64, 65, and 129.
```c
 crw-rw-rw-    1 root     root       1,   3 Apr 11  2002 null
 crw-------    1 root     root      10,   1 Apr 11  2002 psaux
 crw-------    1 root     root       4,   1 Oct 28 03:04 tty1
 crw-rw-rw-    1 root     tty        4,  64 Apr 11  2002 ttys0
 crw-rw----    1 root     uucp       4,  65 Apr 11  2002 ttyS1
 crw--w----    1 vcsa     tty        7,   1 Apr 11  2002 vcs1
 crw--w----    1 vcsa     tty        7, 129 Apr 11  2002 vcsa1
 crw-rw-rw-    1 root     root       1,   5 Apr 11  2002 zero
```
- The minor number is used by the kernel to determine exactly which device is being referred to. 
- Thus, for new drivers, we strongly suggest that you use dynamic allocation to obtain your major device number, rather than choosing a number randomly from the ones that are currently free. In other words, your drivers should almost certainly be using alloc_chrdev_region rather than register_chrdev_region.
- The following script, scull_load, is part of the scull distribution. The user of a driver that is distributed in the form of a module can invoke such a script from the system’s rc.local file or call it manually whenever the module is needed.
```c
#!/bin/sh
module="scull"
device="scull"
mode="664"
# invoke insmod with all arguments we got
# and use a pathname, as newer modutils don't look in . by default
/sbin/insmod ./$module.ko $* || exit 1
# remove stale nodes
rm -f /dev/${device}[0-3]
# 在/proc/devices中找到相应的major number
major=$(awk "\\$2= =\"$module\" {print \\$1}" /proc/devices)
mknod /dev/${device}0 c $major 0
mknod /dev/${device}1 c $major 1
mknod /dev/${device}2 c $major 2
mknod /dev/${device}3 c $major 3
# give appropriate group/permissions, and change the group.
# Not all distributions have staff, some have "wheel" instead.
group="staff"
grep -q '^staff:' /etc/group || group="wheel"
chgrp $group /dev/${device}[0-3]
chmod $mode  /dev/${device}[0-3]
```
-  Each open file (represented internally by a file structure, which we will examine shortly) is associated with its own set of functions (by including a field called f_op that points to a file_operations structure). The operations are mostly in charge of implementing the system calls and are therefore, named open, read, and so on.
-  We can consider the file to be an “object” and the functions operating on it to be its “methods,” using object-oriented programming terminology to denote actions declared by an object to act on itself.  
- The exact behavior of the kernel when a NULL pointer is specified is different for each function
- We’ve tried to keep the list brief so it can be used as a reference, merely summarizing each operation and the default kernel behavior when a NULL pointer is used
- Note that a file has nothing to do with the FILE pointers of user-space programs. A FILE is defined in the C library and never appears in kernel code. 
- The inode structure is used by the kernel internally to represent files. Therefore, it is different from the file structure that represents an open file descriptor. There can be numerous file structures representing multiple open descriptors on a single file, but they all point to a single inode structure.
- inode structure
  - dev_t i_rdev;
For inodes that represent device files, this field contains the actual device number. 
  - struct cdev *i_cdev;
struct cdev is the kernel’s internal structure that represents char devices; this
field contains a pointer to that structure when the inode refers to a char device
file.
- two macros that can be used to obtain the major and minor number from an inode:
```c
unsigned int iminor(struct inode *inode);
unsigned int imajor(struct inode *inode);
```

- you will want to embed the cdev structure within a device-specific structure of your own; that is what scull does. In that case, you should initialize the structure that you have already allocated with:
```c
void cdev_init(struct cdev *cdev, struct file_operations *fops);

/**
 * cdev_init() - initialize a cdev structure
 * @cdev: the structure to initialize
 * @fops: the file_operations for this device
 *
 * Initializes @cdev, remembering @fops, making it ready to add to the
 * system with cdev_add().
 */
void cdev_init(struct cdev *cdev, const struct file_operations *fops)
{
	memset(cdev, 0, sizeof *cdev);
	INIT_LIST_HEAD(&cdev->list);
	kobject_init(&cdev->kobj, &ktype_cdev_default);
	cdev->ops = fops;
}

```
- Once the cdev structure is set up, the final step is to tell the kernel about it with a call to:
```c
int cdev_add(struct cdev *dev, dev_t num, unsigned int count);

/**
 * cdev_add() - add a char device to the system
 * @p: the cdev structure for the device
 * @dev: the first device number for which this device is responsible
 * @count: the number of consecutive minor numbers corresponding to this
 *         device
 *
 * cdev_add() adds the device represented by @p to the system, making it
 * live immediately.  A negative error code is returned on failure.
 */
int cdev_add(struct cdev *p, dev_t dev, unsigned count)
{
	int error;

	p->dev = dev;
	p->count = count;

	error = kobj_map(cdev_map, dev, count, NULL,
			 exact_match, exact_lock, p);
	if (error)
		return error;

	kobject_get(p->kobj.parent);

	return 0;
}
```
- There are a couple of important things to keep in mind when using cdev_add. The first is that this call can fail. If it returns a negative error code, your device has not been added to the system. It almost always succeeds, however, and that brings up the other point: as soon as cdev_add returns, your device is “live” and its operations can be called by the kernel. You should not call cdev_add until your driver is completely ready to handle operations on the device.
- the dup and fork system calls create copies of open files without calling open; 
- Open Method
  - Check for device-specific errors (such as device-not-ready or similar hardware problems)
  - Initialize the device if it is being opened for the first time
  - Update the f_op pointer, if necessary
  - Allocate and fill any data structure to be put in filp->private_data
  - Neither fork nor dup creates a new file structure (only open does that); 
  - example
```c
int scull_open(struct inode *inode, struct file *filp)
{
    struct scull_dev *dev; /* device information */
    dev = container_of(inode->i_cdev, struct scull_dev, cdev);
    filp->private_data = dev; /* for other methods */
    /* now trim to 0 the length of the device if open was write-only */
    if ( (filp->f_flags & O_ACCMODE) = = O_WRONLY) {
        scull_trim(dev); /* ignore errors */
    }
    return 0;          /* success */
}
```
- Release Method
  - Deallocate anything that open allocated in filp->private_data
  - Shut down the device on last close
  - The close system call executes the release method only when the counter for the file structure drops to 0, which happens when the structure is destroyed. 

- Scull's memory usage

![tmp.png](attachments\40a6329c.png)


```c
struct scull_qset {
    void **data;
    struct scull_qset *next;
};
```
  - The function scull_trim is in charge of freeing the whole data area and is invoked by scull_open when the file is opened for writing. It simply walks through the list and frees any quantum and quantum set it finds.
```c
int scull_trim(struct scull_dev *dev)
{
    struct scull_qset *next, *dptr;
    int qset = dev->qset;   /* "dev" is not-null */
    int i;
    for (dptr = dev->data; dptr; dptr = next) { /* all the list items */
      if (dptr->data) {
        for (i = 0; i < qset; i++)
          kfree(dptr->data[i]);
          kfree(dptr->data);
          dptr->data = NULL;
        }
        next = dptr->next;
        kfree(dptr);
    }
    dev->size = 0;
    dev->quantum = scull_quantum;
    dev->qset = scull_qset;
    dev->data = NULL;
    return 0;
}
```
- Read and Write Method
```c
ssize_t read(struct file *filp, char __user *buff, size_t count, loff_t *offp);
ssize_t write(struct file *filp, const char __user *buff, size_t count, loff_t *offp);
```


   - Let us repeat that the buff argument to the read and write methods is a user-space pointer. Therefore, it cannot be directly dereferenced by kernel code. There are a few reasons for this restriction:
     - Depending on which architecture your driver is running on, and how the kernel was configured, the user-space pointer may not be valid while running in kernel mode at all. There may be no mapping for that address, or it could point to some other, random data.
     - Even if the pointer does mean the same thing in kernel space, user-space memory is paged, and the memory in question might not be resident in RAM when the system call is made. Attempting to reference the user-space memory directly could generate a page fault, which is something that kernel code is not allowed to do. The result would be an “oops,” which would result in the death of the process that made the system call.
     - The pointer in question has been supplied by a user program, which could be buggy or malicious. If your driver ever blindly dereferences a user-supplied pointer, it provides an open doorway allowing a user-space program to access or overwrite memory anywhere in the system. If you do not wish to be responsible for compromising the security of your users’ systems, you cannot ever dereference a user-space pointer directly.
   -  We introduce some of those functions (which are defined in <asm/uaccess.h>) here
   -  copy a whole segment of data to or from the user address space. 
```c
unsigned long copy_to_user(void __user *to, const void *from, unsigned long count);
unsigned long copy_from_user(void *to, const void __user *from, unsigned long count);
```

   - The kernel then propagates the file position change back into the file structure when appropriate. The pread and pwrite system calls have different semantics, however; they operate from a given file offset and do not change the file position as seen by any other system calls. These calls pass in a pointer to the user-supplied position, and discard the changes that your driver makes.
```c
ssize_t scull_read(struct file *filp, char __user *buf, size_t count,
                loff_t *f_pos)
{
    struct scull_dev *dev = filp->private_data;
    struct scull_qset *dptr;    /* the first listitem */
    int quantum = dev->quantum, qset = dev->qset;
    int itemsize = quantum * qset; /* how many bytes in the listitem */
    int item, s_pos, q_pos, rest;
    ssize_t retval = 0;
    if (down_interruptible(&dev->sem))
        return -ERESTARTSYS;
    if (*f_pos >= dev->size)
        goto out;
    if (*f_pos + count > dev->size)
        count = dev->size - *f_pos;
    /* find listitem, qset index, and offset in the quantum */
    item = (long)*f_pos / itemsize;
    rest = (long)*f_pos % itemsize;
    s_pos = rest / quantum; q_pos = rest % quantum;
    /* follow the list up to the right position (defined elsewhere) */
    dptr = scull_follow(dev, item);
    if (dptr = = NULL || !dptr->data || ! dptr->data[s_pos])
        goto out; /* don't fill holes */
    /* read only up to the end of this quantum */
    if (count > quantum - q_pos)
        count = quantum - q_pos;
    if (copy_to_user(buf, dptr->data[s_pos] + q_pos, count)) {
        retval = -EFAULT;
        goto out;
    }
    *f_pos += count;
    retval = count;
  out:
    up(&dev->sem);
    return retval;
}
```
- Write Method

```c
ssize_t scull_write(struct file *filp, const char __user *buf, size_t count,
                loff_t *f_pos)
{
    struct scull_dev *dev = filp->private_data;
    struct scull_qset *dptr;
    int quantum = dev->quantum, qset = dev->qset;
    int itemsize = quantum * qset;
    int item, s_pos, q_pos, rest;
    ssize_t retval = -ENOMEM; /* value used in "goto out" statements */
    if (down_interruptible(&dev->sem))
        return -ERESTARTSYS;
    /* find listitem, qset index and offset in the quantum */
    item = (long)*f_pos / itemsize;
    rest = (long)*f_pos % itemsize;
    s_pos = rest / quantum; q_pos = rest % quantum;
    /* follow the list up to the right position */
    dptr = scull_follow(dev, item);
    if (dptr = = NULL)
        goto out;
    if (!dptr->data) {
        dptr->data = kmalloc(qset * sizeof(char *), GFP_KERNEL);
        if (!dptr->data)
         goto out;
        memset(dptr->data, 0, qset * sizeof(char *));
    }
    if (!dptr->data[s_pos]) {
        dptr->data[s_pos] = kmalloc(quantum, GFP_KERNEL);
        if (!dptr->data[s_pos])
            goto out;
    }
    /* write only up to the end of this quantum */
    if (count > quantum - q_pos)
        count = quantum - q_pos;
    if (copy_from_user(dptr->data[s_pos]+q_pos, buf, count)) {
        retval = -EFAULT;
        goto out;
    }
    *f_pos += count;
    retval = count;
        /* update the size */
    if (dev->size < *f_pos)
        dev->size = *f_pos;
  out:
    up(&dev->sem);
    return retval;
}
```
- readv and writev method
  -  A readv call would then be expected to read the indicated amount into each buffer in turn. writev, instead, would gather together the contents of each buffer and put them out as a single write operation.
  -  Each iovec describes one chunk of data to be transferred; it starts at iov_base (in user space) and is iov_len bytes long. The count parameter tells the method how many iovec structures there are. These structures are created by the application, but the kernel copies them into kernel space before calling the driver.
```c
ssize_t (*readv) (struct file *filp, const struct iovec *iov, unsigned long count, loff_t *ppos);
ssize_t (*writev) (struct file *filp, const struct iovec *iov, unsigned long count, loff_t *ppos);

struct iovec
{
  void __user *iov_base;
  __kernel_size_t iov_len;
};
```




