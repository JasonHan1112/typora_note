# Chapter 8: Allocating Memory
- kmalloc
> - The allocated region is also contiguous in physical memory.
> - Using GFP_KERNEL means that kmalloc can put the current process to sleep waiting for a page when called in low-memory situations. 
> - GFP_KERNEL isn’t always the right allocation flag to use; sometimes kmalloc is called from outside a process’s context. This type of call can happen, for instance, in interrupt handlers, tasklets, and kernel timers. In this case, the current process should not be put to sleep, and the driver should use a flag of GFP_ATOMIC instead. The kernel normally tries to keep some free pages around in order to fulfill atomic allocation. When GFP_ATOMIC is used, kmalloc can use even the last free page. If that last page does not exist, however, the allocation fails.
>  - There is an upper limit to the size of memory chunks that can be allocated by kmalloc. 
- The size argument
> - Linux handles memory allocation by creating a set of pools of memory objects of fixed sizes. The one thing driver developers should keep in mind, though, is that the kernel can allocate only certain predefined, fixed-size byte arrays. 
- vmalloc
> - allocates a contiguous memory region in the virtual address space.
> - The right time to call vmalloc is when you are allocating memory for a large sequential buffer that exists only in software.
> - It’s important to note that vmalloc has more overhead than __get_free_pages, because it must both retrieve the memory and build the page tables. Therefore, it doesn’t make sense to call vmalloc to allocate just one page.
> vmalloc returns virtual addresses just above the mapping used for physical memory.
- ioremap
> - ioremap builds new page tables; unlike vmalloc, however, it doesn’t actually allocate any memory. The return value of ioremap is a special virtual address that can be used to access the specified physical address range; the virtual address obtained is eventually released by calling iounmap.