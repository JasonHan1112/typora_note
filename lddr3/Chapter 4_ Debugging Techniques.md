# Chapter 4: Debugging Techniques
- CONFIG_DEBUG_SLAB
This crucial option turns on several types of checks in the kernel memory allocation functions; with these checks enabled, it is possible to detect a number of memory overrun and missing initialization errors. Each byte of allocated memory This crucial option turns on several types of checks in the kernel memory allocation functions; with these checks enabled, it is possible to detect a number of memory overrun and missing initialization errors. Each byte of allocated memory
- CONFIG_DEBUG_SPINLOCK_SLEEP
This option enables a check for attempts to sleep while holding a spinlock. In fact, it complains if you call a function that could potentially sleep, even if the call in question would not sleep.
- CONFIG_INIT_DEBUG
Items marked with __init (or __initdata) are discarded after system initialization or module load time. This option enables checks for code that attempts to access initialization-time memory after initialization is complete.
- There are eight possible loglevel strings, defined in the header <linux/kernel.h>; we list them in order of decreasing severity:
  - KERN_EMERG
Used for emergency messages, usually those that precede a crash.
  - KERN_ALERT
A situation requiring immediate action.
  - KERN_CRIT
Critical conditions, often related to serious hardware or software failures.
  - KERN_ERR
Used to report error conditions; device drivers often use KERN_ERR to report hard-
ware difficulties.
  - KERN_WARNING
Warnings about problematic situations that do not, in themselves, create serious problems with the system.
  - KERN_NOTICE
Situations that are normal, but still worthy of note. A number of security-related conditions are reported at this level.
  - KERN_INFO
Informational messages. Many drivers print information about the hardware they find at startup time at this level.
  - KERN_DEBUG
Used for debugging messages.
Each string (in the macro expansion) represents an integer in angle brackets. Integers range from 0 to 7, with smaller values representing higher priorities.

- Based on the loglevel, the kernel may print the message to the current console, be it a text-mode terminal, a serial port, or a parallel printer.
- It is also possible to read and modify the console loglevel using the text file /proc/sys/kernel/printk. The file hosts four integer values: the current loglevel, the default level for messages that lack an explicit loglevel, the minimum allowed loglevel, and the boot-time default loglevel. Writing a single value to this file changes the current loglevel to that value; thus, for example, you can cause all kernel messages to appear at the console by simply entering:
```c
echo 7 > /proc/sys/kernel/printk
```

- If you happen to read the kernel messages by hand, after stopping klogd, you’ll find that the /proc file looks like a FIFO, in that the reader blocks, waiting for more data. Obviously, **you can’t read messages this way if klogd or another process is already reading the same data, because you’ll contend for it.**

- If the circular buffer fills up, printk wraps around and starts adding new data to the beginning of the buffer, overwriting the oldest data. Therefore, the logging process loses the oldest data. This problem is negligible compared with the advantages of using such a circular buffer. For example, a circular buffer allows the system to run even without a logging process, while minimizing memory waste by overwriting old data should nobody read it. **Another feature of the Linux approach to messaging is that printk can be invoked from anywhere, even from an interrupt handler, with no limit on how much data can be printed.** The only disadvantage is the possibility of losing some data.

- Turning the Messages On and Off
```c
#undef PDEBUG             /* undef it, just in case */
#ifdef SCULL_DEBUG
#  ifdef __KERNEL__
     /* This one if debugging is on, and kernel space */
#    define PDEBUG(fmt, args...) printk( KERN_DEBUG "scull: " fmt, ## args)
#  else
     /* This one for user space */
#    define PDEBUG(fmt, args...) fprintf(stderr, fmt, ## args)
#  endif
#else
#  define PDEBUG(fmt, args...) /* not debugging: nothing */
#endif
#undef PDEBUGG
#define PDEBUGG(fmt, args...) /* nothing: it's a placeholder */
```

-  strace has many command-line options; the most useful of which are -t to display the time when each call is executed, -T to display the time spent in the call, -e to limit the types of calls traced, and -o to redirect the output to a file. By default, strace prints tracing information on stderr.
-  If a process is looping in kernel space due to a bug in your driver, the schedule calls enable you to kill the process after tracing what is happening.
-  however, not to call schedule any time that your driver is holding a spinlock.


