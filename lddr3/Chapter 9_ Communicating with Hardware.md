# Chapter 9: Communicating with Hardware
- I/O Ports and I/O Memory
  > - both of them are accessed by asserting electrical signals on the address bus and control bus (i.e., the read and write signals)* and by reading from or writing to the data bus.
  > - CPU cores access memory much more efficiently, and the compiler has much more freedom in register allocation and addressing-mode selection when accessing memory.

- I/O Registers and Conventional Memory
  > - Despite the strong similarity between hardware registers and memory, a programmer accessing I/O registers must be careful to avoid being tricked by CPU (or compiler) optimizations that can modify the expected I/O behavior.
  > - the only effect of a memory write is storing a value to a location, and a memory read returns the last value written there. Because memory access speed is so critical to CPU performance, the no-side-effects case has been optimized in several ways: values are cached and read/write instructions are reordered.
  > - The compiler can cache data values into CPU registers without writing them to memory, and even if it stores them, both write and read operations can operate on cache memory without ever reaching physical RAM.
  > - Reordering can also happen both at the compiler level and at the hardware level.
  > - Therefore, a driver must ensure that no caching is performed and no read or write reordering takes place when accessing registers.
  > - The solution to compiler optimization and hardware reordering is to place a memory barrier between operations that must be visible to the hardware (or to another processor) in a particular order.
  - void barrier(void);
    > - This function tells the compiler to insert a memory barrier but has no effect on the hardware. Compiled code stores to memory all values that are currently modified and resident in CPU registers, and rereads them later when they are needed. A call to barrier prevents compiler optimizations across the barrier but leaves the hardware free to do its own reordering.
  - void rmb(void); void wmb(void); void mb(void);
    > - An rmb (read memory barrier) guarantees that any reads appearing before the barrier are completed prior to the execution of any subsequent read. wmb guarantees ordering in write operations, and the mb instruction guarantees both. Each of these functions is a superset of barrier.
  - example for memory barriers
    > - A typical usage of memory barriers in a device driver may have this sort of form:
```c
writel(dev->registers.addr, io_destination_address);
writel(dev->registers.size, io_size);
writel(dev->registers.operation, DEV_READ);
wmb( );
writel(dev->registers.control, DEV_GO);
```
   > In this case, it is important to be sure that all of the device registers controlling a particular operation have been properly set prior to telling it to begin. The memory barrier enforces the completion of the writes in the necessary order.
   - I/O Port Allocation
   > - The kernel provides a registration interface that allows your driver to claim the ports it needs. The core function in that interface is request_region:
```c
#include <linux/ioport.h>
struct resource *request_region(unsigend long first, unsigend long n, const char* name); 
```
   > - When you are done with a set of I/O ports (at module unload time, perhaps), they should be returned to the system with:
```c
void release_region(unsigned long start, unsigned long n);
```
  - I/O memory 
    > - the kernel must first arrange for the physical address to be visible from your driver, and this usually means that you must call ioremap before doing any I/O.
    > - struct resource *request_mem_region(unsigned long start, unsigned long len, char *name);
    > - void release_mem_region(unsigned long start, unsigned long len);
    > - int check_mem_region(unsigned long start, unsigned long len);
    > - void *ioremap(unsigned long phys_addr, unsigned long size);
    > - void *ioremap_nocache(unsigned long phys_addr, unsigned long size);
    > - void iounmap(void * addr);
  - accessing I/O memory
    > - unsigned int ioread8(void *addr);
    > - unsigned int ioread16(void *addr);
    > - unsigned int ioread32(void *addr);
    > - void iowrite8(u8 value, void *addr);
    > - void iowrite16(u16 value, void *addr);
    > - void iowrite32(u32 value, void *addr);
  >  addr should be an address obtained from ioremap