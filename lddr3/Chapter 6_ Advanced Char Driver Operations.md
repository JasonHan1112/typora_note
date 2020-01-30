# Chapter 6: Advanced Char Driver Operations
- put_user(datum, ptr)
  - These macros write the datum to user space; they are relatively fast and should be called instead of copy_to_user whenever single values are being transferred.
  - The macros have been written to allow the passing of any type of pointer to put_user, as long as it is a user-space address.
  -  The size of the data transfer depends on the type of the ptr argument and is determined at compile time using the sizeof and typeof compiler builtins. As a result, if ptr is a char pointer, one byte is transferred, and so on for two, four, and possibly eight bytes.
  -  __put_user should only be used if the address has already been verified with access_ok.
 - get_user(local, ptr)
   -  __get_user should only be used if the address has already been verified with access_ok.
   
 - Capabilities and Restricted Operations 
   - For example, not all users of a tape drive should be able to set its default block size, and a user who has been granted read/write access to a disk device should probably still be denied the ability to format it. In cases like these, the driver must perform additional checks to be sure that the user is capable of performing the requested operation.
   - The Linux kernel provides a more flexible system called capabilities. 
   -  When needed, the scull implementation of ioctl checks a user’s privilege level as follows:
```c
if (! capable (CAP_SYS_ADMIN))
  return -EPERM;
```
- Blocking I/O

  - What does it mean for a process to “sleep”? When a process is put to sleep, it is marked as being in a special state and removed from the scheduler’s run queue. Until something comes along to change that state, the process will not be scheduled on any CPU and, therefore, will not run. A sleeping process has been shunted off to the side of the system, waiting for some future event to happen.
  - never sleep when you are running in an atomic context. your driver cannot sleep while holding a spinlock, seqlock, or RCU lock. 
  - You also cannot sleep if you have disabled interrupts.
  - Another thing to remember with sleeping is that, when you wake up, you never know how long your process may have been out of the CPU or what may have changed in the mean time.  The end result is that you can make no assumptions about the state of the system after you wake up, and you must check to
ensure that the condition you were waiting for is, indeed, true.
  - your process cannot sleep unless it is assured that somebody else, somewhere, will wake it up. 

  - It implements a device with simple behavior: any process that attempts to read from the device is put to sleep. Whenever a process writes to the device, all sleeping processes are awakened. This behavior is implemented with the following read and write methods:
```c
static DECLARE_WAIT_QUEUE_HEAD(wq);
static int flag = 0;
ssize_t sleepy_read (struct file *filp, char __user *buf, size_t count, loff_t *pos)
{
    printk(KERN_DEBUG "process %i (%s) going to sleep\n",
            current->pid, current->comm);
    wait_event_interruptible(wq, flag != 0);
    flag = 0;
    printk(KERN_DEBUG "awoken %i (%s)\n", current->pid, current->comm);
    return 0; /* EOF */
}
ssize_t sleepy_write (struct file *filp, const char __user *buf, size_t count,
        loff_t *pos)
{
    printk(KERN_DEBUG "process %i (%s) awakening the readers...\n",
            current->pid, current->comm);
    flag = 1;
    wake_up_interruptible(&wq);
    return count; /* succeed, to avoid retrial */
}
```
The wake_up_interruptible call will cause both sleeping processes to wake up. It is entirely possible that they will both note that flag is nonzero before either has the opportunity to reset it. For this trivial module, this race condition is unimportant. In a real driver, this kind of race can create rare crashes that are difficult to diagnose. 

- The device driver uses a device structure that contains two wait queues and a buffer. The size of the buffer is configurable in the usual ways (at compile time, load time, or runtime).
```c
struct scull_pipe {
        wait_queue_head_t inq, outq;       /* read and write queues */
        char *buffer, *end;                /* begin of buf, end of buf */
        int buffersize;                    /* used in pointer arithmetic */
        char *rp, *wp;                     /* where to read, where to write */
        int nreaders, nwriters;            /* number of openings for r/w */
        struct fasync_struct *async_queue; /* asynchronous readers */
        struct semaphore sem;              /* mutual exclusion semaphore */
        struct cdev cdev;                  /* Char device structure */
};
```
The read implementation manages both blocking and nonblocking input and looks like this:
```c
static ssize_t scull_p_read (struct file *filp, char __user *buf, size_t count,
                loff_t *f_pos)
{
    struct scull_pipe *dev = filp->private_data;
    if (down_interruptible(&dev->sem))
        return -ERESTARTSYS;
    while (dev->rp = = dev->wp) { /* nothing to read */
        up(&dev->sem); /* release the lock */
        if (filp->f_flags & O_NONBLOCK)
            return -EAGAIN;
        PDEBUG("\"%s\" reading: going to sleep\n", current->comm);
        if (wait_event_interruptible(dev->inq, (dev->rp != dev->wp)))
            return -ERESTARTSYS; /* signal: tell the fs layer to handle it */
        /* otherwise loop, but first reacquire the lock */
        if (down_interruptible(&dev->sem))
            return -ERESTARTSYS;
    }
    /* ok, data is there, return something */
    if (dev->wp > dev->rp)
        count = min(count, (size_t)(dev->wp - dev->rp));
    else /* the write pointer has wrapped, return data up to dev->end */
        count = min(count, (size_t)(dev->end - dev->rp));
    if (copy_to_user(buf, dev->rp, count)) {
        up (&dev->sem);
        return -EFAULT;
    }
    dev->rp += count;
    if (dev->rp = = dev->end)
        dev->rp = dev->buffer; /* wrapped */
    up (&dev->sem);
    /* finally, awake any writers and return */
    wake_up_interruptible(&dev->outq);
    PDEBUG("\"%s\" did read %li bytes\n",current->comm, (long)count);
    return count;
}
```
Holding the semaphore in this case is justified since it does not deadlock the system (we know that the kernel will perform the copy to user space and wakes us up without trying to lock the same semaphore in the process), and since it is important that the device memory array not change while the driver sleeps.

- Advanced Sleeping ( the lower level to get an understanding of what is really going on when a process sleeps.)
  - The first step in putting a process to sleep is usually the allocation and initialization of a wait_queue_t structure, followed by its addition to the proper wait queue. 
  - The next step is to set the state of the process to mark it as being asleep. changing the current state of a process does not, by itself, put it to sleep. By changing the current state, you have changed the way the scheduler treats a process, but you have not yet yielded the processor.
  - Giving up the processor is the final step, but there is one thing to do first: you must
check the condition you are sleeping for first. 
- Exclusive waits
  -  If the number of processes in the wait queue is large, this “thundering herd” behavior can seriously degrade the performance of the system.
  - When a wait queue entry has the WQ_FLAG_EXCLUSIVE flag set, it is added to the end of the wait queue. Entries without that flag are, instead, added to the beginning. When wake_up is called on a wait queue, it stops after waking the first process that has the WQ_FLAG_EXCLUSIVE flag set.

- poll and select
  - they are often used in applications that must use multiple input or output streams without getting stuck on any one of them.
  - Whenever a user application calls poll, select, or epoll_ctl,* the kernel invokes the poll method of all files referenced by the system call, passing the *same poll_table* to each of them. The poll_table structure is just a wrapper around a function that builds the actual data structure. That structure, for poll and select, is a linked list of memory pages containing poll_table_entry structures. Each poll_table_entry holds the struct file and wait_queue_head_t pointers passed to poll_wait, along with an associated wait queue entry. The call to poll_wait sometimes also adds the process to the given wait queue. The whole structure must be maintained by the kernel so that
the process can be removed from all of those queues before poll or select returns.

- Asynchronous Notiﬁcation
  - By enabling asynchronous notification, this application can receive a signal whenever data becomes available and need not concern itself with polling.
  - User programs have to execute two steps to enable asynchronous notification from an input file.
    - > they specify a process as the “owner” of the file. When a process invokes the F_SETOWN command using the fcntl system call, the process ID of the owner process is saved in filp->f_owner for later use. This step is necessary for the kernel to know just whom to notify. 
    - > the user programs must set the FASYNC flag in the device by means of the F_SETFL fcntl command.
  - The Driver’s Point of View
    - > When F_SETOWN is invoked, nothing happens, except that a value is assigned to filp->f_owner.
    - > When F_SETFL is executed to turn on FASYNC, the driver’s fasync method is called. This method is called whenever the value of FASYNC is changed in filp->f_flags to notify the driver of the change, so it can respond properly. The flag is cleared by default when the file is opened. We’ll look at the standard implementation of the driver method later in this section.
    - > When data arrives, all the processes registered for asynchronous notification must be sent a SIGIO signal.
    - > struct fasync_struct; 
    - > fasync_helper; fasync_helper is invoked to add or remove entries from the list of interested processes when the FASYNC flag changes for an open file. 
    - > kill_fasync; kill_fasync is used to signal the interested processes when data arrives. Its arguments are the signal to send (usually SIGIO) and the band, which is almost always POLL_IN* (but that may be used to send “urgent” or out-of-band data in the networking code).
    - > We must invoke our fasync method when the file is closed to remove the file from the list of active asynchronous readers.





- The llseek Implementation
```c
loff_t scull_llseek(struct file *filp, loff_t off, int whence)
{
    struct scull_dev *dev = filp->private_data;
    loff_t newpos;
    switch(whence) {
      case 0: /* SEEK_SET */
        newpos = off;
        break;
      case 1: /* SEEK_CUR */
        newpos = filp->f_pos + off;
        break;
      case 2: /* SEEK_END */
        newpos = dev->size + off;
        break;
      default: /* can't happen */
        return -EINVAL;
    }
    if (newpos < 0) return -EINVAL;
    filp->f_pos = newpos;
    return newpos;
}
```