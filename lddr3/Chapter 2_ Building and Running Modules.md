# Chapter 2: Building and Running Modules
- hello_world module
  - (MODULE_LICENSE) is used to tell thekernel that this module bears a free license; without such a declaration, the kernel complains when the module is loaded.
  -  The kernel needs its own printing function because it runs by itself, without the help of the C library. The module can call printk because, after insmod has loaded it, the module is linked to the kernel and can access the kernel’s public symbols (functions and variables, as detailed in the next section). 
  -  The message goes to one of the system log files, such as /var/log/messages (the name of the actual file varies between Linux distributions). 
  -   whereas an application that terminates can be lazy in releasing resources or avoids clean up altogether, **the exit function of a module must  carefully undo everything the init function built up, or the pieces remain around until the system is rebooted.**
  -   A module, on the other hand, is linked only to the kernel, and the only functions it can call are the ones **exported by the kernel**; there are no libraries to link to. The printk function used in hello.c earlier, for example, is the version of printf defined within the kernel and exported to modules. It behaves similarly to the original function, with a few minor differences, the main one being **lack of floating-point support.**
  - linking a module to the kernel   
![无标题1.png](attachments\30657281.png)
  -   example: hello world
```c
#include <linux/init.h>
#include <linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");
static int hello_init(void)
{
    printk(KERN_ALERT "Hello, world\n");
    return 0;
}
static void hello_exit(void)
{
    printk(KERN_ALERT "Goodbye, cruel world\n");
}
module_init(hello_init);
module_exit(hello_exit);
```
- Unix transfers execution from user space to kernel space whenever an application issues a system call or is suspended by a hardware interrupt. Kernel code executing a system call is working in the context of a process—it operates on behalf of the calling rocess and is able to access data in the process’s address space. Code that handles interrupts, on the other hand, is asynchronous with respect to rocesses and is not related to any particular process.
- The role of a module is to extend kernel functionality; modularized code runs in kernel space. Usually a driver performs both the tasks outlined previously: some functions in the module are executed as part of system calls, and some are in charge of interrupt handling.
-  Kernel code does not run in such a simple world, and even the simplest kernel modules must be written with the idea that many things can be happening at once.
-  If you do not write your code with concurrency in mind, it will be subject to catastrophic failures that can be exceedingly difficult to debug.
-  Applications are laid out in virtual memory with a very large stack area. The stack, of course, is used to hold the function call history and all automatic variables created by currently active functions. The kernel, instead, has a very small stack; it can be as small as a single, 4096-byte page. Your functions must share that stack with the entire kernel-space call chain. Thus, it is never a good idea to declare large automatic variables; if you need larger structures, you should allocate them dynamically at call time.
-   The kernel build system is a complex beast, and we just look at a tiny piece of it. The files found in the Documentation/kbuild directory in the kernel source are required reading for anybody wanting to understand all that is really going on beneath the surface.
-  Compiling Kernel Mkdule
   - if only one file you need a single line will suffice:
*obj-m := hello.o*
   - If, instead, you have a module called module.ko that is generated from two source files (called, say, file1.c and file2.c), the correct incantation would be:
*obj-m := module.o*
*module-objs := file1.o file2.o*
   - the make command required to build your module (typed in the directory containing the module source and makefile) would be:
*make -C ~/kernel-2.6 M=`pwd` modules*
**This command starts by changing its directory to the one provided with the -C option (that is, your kernel source directory). There it finds the kernel’s top-level makefile. The M= option causes that makefile to move back into your module source
directory before trying to build the modules target. This target, in turn, refers to the list of modules found in the obj-m variable, which we’ve set to  odule.o in our examples.** Typing the previous make command can get tiresome after a while, so the kernel developers have developed a sort of makefile idiom, **which makes life easier for those building modules outside of the kernel tree.** The trick is to write your makefile as follows:
      - When the makefile is invoked from the command line, it notices that the KERNELRELEASE variable has not been set. It locates the kernel source directory by taking advantage of the fact that the symbolic link build in the installed modules directory points back at the kernel build tree. If you are not actually running the kernel that you are building for, you can supply a KERNELDIR= option on the command line, set the KERNELDIR environment variable, or rewrite the line that sets KERNELDIR in the makefile. **Once the kernel source tree has been found, the makefile invokes the default: target, which runs a second make command (parameterized in the makefile as $(MAKE)) to invoke the kernel build system as described previously. On the second reading,** the makefile sets obj-m, and the kernel makefiles take care of actually building the module.
```c
# If KERNELRELEASE is defined, we've been invoked from the
# kernel build system and can use its language.
ifneq ($(KERNELRELEASE),)
    obj-m := hello.o
# Otherwise we were called directly from the command
# line; invoke the kernel build system.
else
    KERNELDIR ?= /lib/modules/$(shell uname -r)/build
    PWD  := $(shell pwd)
default:
    $(MAKE) -C $(KERNELDIR) M=$(PWD) modules
endif
```

- If you actually look in the kernel source, you’ll find that the names of the system calls are prefixed with sys_. This is true for all system calls and no other functions; it’s useful to keep this in mind when grepping for the system calls in the sources.
- When a module is loaded, the kernel checks the processor specific configuration options for the module and makes sure they match the running kernel. If the module was compiled with different options, it is not loaded.
-  As a general rule, distributing things in source form is an easier way to make your way in the world.
-  This variable is stored in a special part of the module executible (an “ELF section”) that is used by the kernel at load time to find the variables exported by the module.
```c
EXPORT_SYMBOL(name);
EXPORT_SYMBOL_GPL(name);
```
- We’ll examine these files as we come to them, but there are a few that are specific to modules, and must appear in every loadable module. Thus, just about all module code has the following:

```c
#include <linux/module.h>//contains a great many definitions of symbols and functions needed by loadable modules
#include <linux/init.h>//specify initialization and cleanup functions
#include <linux/moduleparam.h>//enable the passing of parameters to the module at load time
```
- It is not strictly necessary, but your module really should specify which license applies to its code. Doing so is just a matter of including a MODULE_LICENSE line:

```c
MODULE_LICENSE("GPL");

```
- The **__init** token in the definition may look a little strange; it is a hint to the kernel that the given function is used only at initialization time. The module loader drops the initialization function after the module is loaded, making its memory available for other uses. 
```c
static int __init initialization_function(void)
{
    /* Initialization code here */
}
module_init(initialization_function);
```
- The __exit modifier marks the code as being for module unload only (by causing the compiler to place it in a special ELF section). 
```c
static void __exit cleanup_function(void)
{
    /* Cleanup code here */
}
module_exit(cleanup_function);
```
- If it turns out that your module simply cannot load after a particular type of failure, you must undo any registration activities performed before the failure.
- Error recovery is sometimes best handled with the goto statement. We normally hate to use goto, but in our opinion, this is one situation where it is useful. Careful use of goto in error situations can eliminate a great deal of complicated, highly-indented, “structured” logic. Thus, in the kernel, goto is often used as shown here to deal with errors.
```c
/*
注意顺序
goto err1
goto err2
goto err3
...
err1:
err2:
err3:
*/
int __init my_init_function(void)
{
    int err;
    /* registration takes a pointer and a name */
    err = register_this(ptr1, "skull");
    if (err) goto fail_this;//err1
    err = register_that(ptr2, "skull");
    if (err) goto fail_that;//err2
    err = register_those(ptr3, "skull");
    if (err) goto fail_those;//err3
    return 0; /* success */
  fail_those: unregister_that(ptr2, "skull");//err3
  fail_that: unregister_this(ptr1, "skull");//err2
  fail_this: return err; //err1 /* propagate the error */
 }
```
- The return value of my_init_function, err, is an error code. In the Linux kernel, error codes are negative numbers belonging to the set defined in <linux/errno.h>.
- You should always remember that some other part of the kernel can make use of any facility you register immediately after that registration has completed. It is entirely possible, in other words, that the kernel will make calls into your module while your initialization function is still running. Do not register anyfacility until all of your internal initialization needed to support that facility has been completed.

- Module Parameters
```c
insmod hellop howmany=10 whom="Mom"
```
in the module
```c
#include <moduleparam.h>

static char *whom = "world";
static int howmany = 1;
module_param(howmany, int, S_IRUGO);
module_param(whom, charp, S_IRUGO);
```
- You should probably not make module parameters writable, unless you are prepared to detect the change and react accordingly.