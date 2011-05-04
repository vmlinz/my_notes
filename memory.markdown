# Linux 内存基础 #

## 地址类型 ##
linux内核中有许多种不同的地址类型

* 用户虚拟地址
用户空间看到的常规地址，通过页表可以将虚拟地址和物理地址映射起来
* 物理地址
用在cpu和内存之间的地址叫做物理地址
* 总线地址
外围总线和内存之间的地址叫做总线地址。通常他们和物理地址相同
* 内核逻辑地址
内核的常规地址空间，必定有对应的物理内存与之映射。kmalloc返回的就是内核逻辑地址
* 内核虚拟地址
内核虚拟地址和内核逻辑地址的相同之处在于，他们都将内核空间的地址映射到物理内存上。但是内核虚拟地址不一定是线性的和一对一的。vmalloc返回的是虚拟地址。

## 虚拟内存 ##

虚拟内存是用来描述一种不直接映射计算机物理内存的方法。分页是在虚拟内存与物理内存转换时用到的。请参阅intel手册了解更多分页系统的知识。

低于896MB的每页内存都被直接映射到内核空间。高于896的内存，又称高端内存，不会一直映射到内存空间，而是使用kmap和kmap_atomic来临时映射。剩余的126MB内存的一部分用于映射高端内存。

内核内存从PAGE\_OFFSET开始，在x86架构中它的值是0xc0000000(3G)，高于PAGE\_OFFSET的虚拟内存用于内核空间，低于的用于用户空间。

### x86系统内存子系统初始化 ###

* 首先设置页表(可能有多级)
* 完成内核内存映射(内核中的物理内存和逻辑地址只有一个固定的OFFSET，PAGE\_OFFSET)

### 用户空间的内存管理 ###

* struct mm\_struct 进程内存空间的最高级别管理结构
* struct vm\_area\_struct 内存区域，组成进程内存
* pgd_t *pgd 进程页表指针

阅读ULK3学习更多内存管理的细节

## 物理地址和页 ##

物理地址被分成离散的单元，成为页。目前大多数系统的页面大小都为4k。实际使用的时候应该使用指定体系架构下的页面大小PAGE\_SIZE。PAGE\_SHIFT可以将地址转换为页帧。

## 高端和低端内存 ##

系统中逻辑地址和虚拟地址不一致的情况产生了高端内存和低端内存的说法。

通常linux x86内核将4GB的虚拟地址分割为用户空间和内核空间；在二者的上下文中使用相同的映射。一个典型的分配是将低地址3GB分给用户空间，将剩下的高地址1GB分给内核空间。这样由于内核只能直接操作已经映射了物理内存的虚拟地址，所以内核在大内存系统中就不能直接访问所有的物理内存。这样就产生了高端内存和低端内存的说法。

### 高端内存 ###

高端内存是没有直接映射到物理内存的内核逻辑地址

- [HighMemory](http://linux-mm.org/HighMemory)


#### 应对高端内存 ####

* 高端物理内存在需要使用时会被临时映射到内核虚拟内存上
* 内核经常访问的数据被放在低端内存上
* 内核偶尔访问的数据最好放在高端内存上
* 不同内存区域的内存分配和换页应该有一个平衡

#### 临时映射 ####

* kmap和kunmap，产生一个永久的内存映射，但他们有一个全局锁，不适合SMP系统
* kmap\_atomic和kunmap\_atomic，常用于SMP系统，产生的映射地址是每个CPU私有的

在访问特定的高端内存之前，内核必须建立明确的虚拟映射，使该页可以在内核地址空间被访问。

总的来说高端内存就是没有逻辑地址的内存，反之就是低端内存。

## 内存映射和结构 ##

由于高端内存中无法使用逻辑地址，所以内核中处理内存的函数趋向于使用指向page结构的指针。该结构保存了内核需要知道的所有物理内存的信息。系统中的每个物理页都和一个page结构对应。

page结构和虚拟地址之间转换的函数和宏：

* struct page \*virt\_to\_page(void \*kaddr);
* struct page \*pfn\_to\_page(int pfn);
* void \*page_addr(struct page \*page);

## 分页 ##

In a virtual memory system all of these addresses are virtual addresses and not physical addresses. These virtual addresses are converted into physical addresses by the processor based on information held in a set of tables maintained by the operating system.在虚拟内存系统中，所有的地址都是虚拟地址而不是物理地址。这些虚拟地址可以通过操作系统维护的一系列的表转换为物理地址。

To make this translation easier, virtual and physical memory are divided into handy sized chunks called pages. These pages are all the same size, they need not be but if they were not, the system would be very hard to administer. Linux on Alpha AXP systems uses 8 Kbyte pages and on Intel x86 systems it uses 4 Kbyte pages. Each of these pages is given a unique number; the page frame number (PFN).为了使这个转换更加简单，虚拟地址和物理地址都被分成叫做内存页面小的内存块。所有的页面都是同样大小。每页内存都有一个唯一的编号，这种编号叫做页帧号。

In this paged model, a virtual address is composed of two parts; an offset and a virtual page frame number. If the page size is 4 Kbytes, bits 11:0 of the virtual address contain the offset and bits 12 and above are the virtual page frame number. Each time the processor encounters a virtual address it must extract the offset and the virtual page frame number. The processor must translate the virtual page frame number into a physical one and then access the location at the correct offset into that physical page. To do this the processor uses page tables.在这种分页模式下，虚拟地址由两部分组成；页帧内的偏移和虚拟页帧号。如果页面大小是4KB，11:0这些位就是页帧内偏移，12位以上的叫做页帧号。每当处理器遇到虚拟内存地址，它就会把地址中的页内偏移和页帧号解出来。处理器通过页表把虚拟帧号转换成物理帧号，然后在加上页内偏移就可以找到对应的物理地址了。

### 页表 ###

现代系统中，处理器需要使用某种机制将虚拟地址转换成物理地址。这种机制被成为页表；它基本上是一个多层树形结构，结构化的数组中包含了虚拟地址到物理地址的映射和相关的标志位。

### demand paging ###

Linux uses demand paging to load executable images into a processes virtual memory. Whenever a command is executed, the file containing it is opened and its contents are mapped into the processes virtual memory. This is done by modifying the data structures describing this processes memory map and is known as memory mapping. However, only the first part of the image is actually brought into physical memory. The rest of the image is left on disk. As the image executes, it generates page faults and Linux uses the processes memory map in order to determine which parts of the image to bring into memory for execution.Linux使用按需分页来将可执行镜像载入到进程的虚拟内存空间。每当命令执行时，命令的文件被打开，内容被映射到进程的虚拟内存上。这里是通过修改进程的内存映射相关结构体来实现的，这个过程也叫做内存映射。不过，只有镜像的开头部分被真正的放进了物理内存。余下部分还在磁盘上。镜像执行的时候，它将持续的产生页面异常，linux通过进程的内存映射表来确定镜像的哪个部分需要被载入物理内存执行。

### Shared virtual memory ###

Virtual memory makes it easy for several processes to share memory. All memory access are made via page tables and each process has its own separate page table. For two processes sharing a physical page of memory, its physical page frame number must appear in a page table entry in both of their page tables. 虚拟内存使得多个进程共享内存更加简单。所有的内存访问都要通过页表来实现。对于共享一个物理页的两个进程来说，这个物理页面必须同时在两个进程的页表中都有相应的页表项。

## 虚拟内存区 ##

VMA是用于管理进程地址空间中不同区域的内核数据结构。

进程的内存映射至少包含下面这些区域：

* 程序可执行代码区域（text）
* 数据区（bss，stack，data）
* 与每个活动的内存映射区对应的区域

可以`cat /proc/<pid/maps>`来查看具体进程的内存映射。

当用户空间进程调用mmap时，系统会创建一个新的VMA来相应它。

注意vm\_area\_struct这个重要的数据结构（定义在<linux/mm.h>中）。

## 内存映射处理 ##

系统中每个进程（除了内核空间的辅助线程）都有一个struct mm_struct结构（定义在<linux/sched.h>中），其中包含了大量的内存管理信息。多个进程可以共享内存管理结构，linux就是使用这种方法实现线程的。

## mmap设备操作 ##

mmap可以将用户空间的内存和设备内存映射起来，这样在访问分配地址范围内的内存时就相当于访问设备内存了。

并非所有的设备都能进行mmap抽象：

* 像串口这样面向流的设备就不能
* mmap的另一个限制：必须以`PAGE_SIZE`为单位进行映射，因为内核只能在页表一级上对虚拟地址进行管理。

为了执行mmap，驱动程序只需要为该地址范围建立合适的页表，并将`vma->vm_ops`替换为一系列的新操作就可以了。有两种建立页表的方法：使用`remap_pfn_range`函数一次全部建立；通过VMA的fault方法一次建立一个新页表。

## 内存映射的方法 ##

* 重新映射特定的I/O区域
* 重新映射ram
* 重新映射内核虚拟地址
* 执行直接I/O访问

## 分配内存 ##

这里我们来看看内核为设备驱动程序提供的内存管理接口。

### Kmalloc函数 ###

kmalloc内存分配工具和malloc的使用方法很接近。
它的原型是：

`
\#include <linux/slab.h>
void *kmalloc(size_t size, int flags);
`

* flags参数 以多种方式控制kmalloc的行为

最常用的标志是GFP\_KERNEL(GFP的来源是因为kmalloc最终会调用get\_free\_pages函数)，这个标志允许kmalloc在页面不足的情况下休眠。

如果在进程上下文之外使用kmalloc，比如中断处理例程中就需要使用GFP_ATOMIC标志，不会休眠

其他标志都定义在<linux/gfp.h>文件中，请阅读该文件后使用他们

* size参数

内核中使用基于页面的方式管理内存，因此和用户空间的基于堆的简单内存管理有很大的差别

由于slab分配器（即kmalloc的底层实现）最大分配的内存单元是128KB，所以如果分配的内存过大，最好不要使用kmalloc方法

### 高速缓存 ###

内核实现了一些内存池，内核驱动程序通过使用它们可以减少内存分配的次数。

它的api在<linux/slab.h>中，类型为kmem\_cache\_t。

### 内存池 ###

内存池其实是某种形式的高速缓冲，它试图始终保持空闲的状态，方便那些要求内存分配不能失败的代码使用。

它的api在<linux/mempool.h>中，类型为mempool\_t。

### get\_free\_pages和相关函数 ###

如果驱动使用较大块的内存，则适合使用面向页的分配技术。

* get\_zeroed\_page(unsigned int flags);
返回指向新页面的指针并清零

* __get\_free\_page(unsigned int flags);
返回指针但不清零

* __get\_free\_pages(unsigned int flags, unsigned int order);
分配2^order个连续页面，不清零

### 页分配核心 alloc\_pages ###

alloc_pages用来分配描述用struct page描述的页面内存，使用这种结构描述的内核内存在某些地方使用起来非常方便。

struct page *alloc\_pages\_node(int nid, unsigned int flags, unsigned int order);

### vmalloc以及相关函数 ###

vmalloc分配虚拟地址空间的连续内存。尽管可能这段内存在物理上可能不是连续的。

通过vmalloc获得的内存使用起来效率不高，如果可能，应该直接和单个的页面打交道，也就是使用前面的函数来处理而不是使用vmalloc。vmalloc分配的虚拟地址上可能没有物理内存对应。

kmalloc和__get\_free\_pages返回的虚拟地址内存范围与物理内存的范围是一一对应的。但vmalloc和ioremap使用的地址范围则是完全虚拟的，每次分配都需要适当的设置页表来建立内存区域。

ioremap也和vmalloc一样建立新页表，但它不会分配内存。它更多用于映射设备缓冲到虚拟内核空间。值得注意的是不能把ioremap返回的指针直接当作内存使用，应该使用I/O函数来访问。

vmalloc的一个小缺点是它不能在原子上下文中使用。

相关的函数定义在<linux/vmalloc.h>中。

### per-CPU变量 ###

当建立一个per-CPU变量时，系统的每个处理器都会拥有该变量的副本。对于per-CPU变量的访问几乎不需要锁定，因为每个处理器有自己的副本。

注意当处理器在修改某个per-CPU变量的临界区中间时，它可能被抢占，需要避免这种情况发生。所以我们应该显式地调用get\_cpu\_var访问某给定变量的当前处理器副本，结束后调用put\_cpu\_var。

使用方法：

* DEFINE\_PER\_CPU(type, name);
* 动态分配：
void *alloc_percpu(type);
void *\_\_alloc\_percpu(size\_t size, size\_t align);
* per\_cpu\_ptr(void *per\_cpu\_var, int cpu\_id);返回给定cpu\_id的per\_cpu\_var指针

## DMA ##

DMA(Direct Memory Access)是一种高级的硬件机制，它允许外设直接和主内存之间进行I/O传输而不用CPU的干预。

### DMA数据传输概览 ###

有两种方式可以引发DMA数据传输：软件对数据的请求；硬件异步地把数据传给系统。

* 第一种情况：
 - 进程调用read，驱动程序分配一个DMA缓冲，然后让硬件把数据传输到这里，此时进程睡眠
 - 硬件把数据写入到DMA缓冲，写完之后产生一个中断
 - 中断处理程序获取输入的数据，应答中断并唤醒进程，进程即可读取DMA缓冲里面的数据

* 第二种情况：
 - 硬件产生中断，宣告有数据到来
 - 中断处理程序分配缓冲区，告诉硬件向哪里传输数据
 - 外设将数据写入缓冲，完成后产生另一个中断
 - 处理程序分发新数据，唤醒相关进程，然后执行清理工作

可以看出，高效的DMA传输依赖于中断报告。

### 分配DMA缓冲区 ###

DMA缓冲区的主要问题是：当大于一页时，它必须占用连续的物理页，这是因为多数外设总线都使用物理地址。

* 驱动作者必须谨慎的为DMA分配正确的内存类型，并不是所有的内存区间都适合DMA操作。外设不能使用高端内存。
* 对于有限制的设备，应使用GFP_DMA标志调用内存分配函数。

#### DIY分配 ####

使用get\_free\_pages分配大于128KB内存的时候很容易失败返回-ENOMEM。此时的办法是在引导时分配内存或者为缓冲区保留顶部物理内存。

如果要为DMA分配一大块内存，最好考虑分散聚集I/O。

### 总线地址 ###

硬件和程序代码使用不同的地址，所以需要有一个地址转换。

### 通用DMA层 ###

由于多种系统对缓存和DMA的处理不同，内核提供了一个通用DMA层，建议在用到DMA时使用该层。struct device隐藏了描述设备的总线细节，在使用通用DMA层时需要使用到该结构的指针。

接下来的DMA函数都需要包含文件<linux/dma-mapping.h>

#### 确定设备的DMA能力 ####

int dma\_set\_mask(struct device *dev, u64 mask); 可以用来确定设备是否支持DMA。

## 内存分配实现 ##

下面将介绍linux内核中针对不同的应用场景实现的不同内存分配算法。

- 页框分配，zoned buddy算法
- 内存区分配，slab 分配器
- 非连续内存区管理，虚拟内存映射

在支持NUMA的linux内核当中，系统的物理内存被分为多个节点，在单独节点内，任一给定cpu访问页面所需要的时间都是相同的。每个节点的物理内存又分成多个内存区（zone）。x86下内存区有`ZONE_DMA`、`ZONE_NORMAL`和`ZONE_HIGHMEM`。x86_32系统上，`ZONE_HIGHMEM`中的内存没有直接映射到内核线性地址上，在每次使用之前都需要先设置页表映射内存。每个zone下面的内存都是以页框为单位来管理的。

每个zone的内存页面是通过buddy算法来管理的。

### zoned buddy ###

页框分配算法需要解决external fragmentation的内存管理问题。linux内核使用buddy算法来解决这个问题。把所有的空闲页分组为2^(order-1)大小的块链表。order的最大值为11，所以一共有11个这样的链表。链表的元素最小的为4k(1一个页面大小)，最大的为4M(2^10个页面大小)。请求内存时，内核首先从最接近请求大小的链表中查询，如果有这样的空闲单元，则直接使用。如果没有则一次递增到更大块的内存链表中查询，如果有则将内存分出最接近请求大小的块，在把余下的内存拆分添加到较小的内存链表中。

#### 核心函数 ####

核心接口

- `unsigned long __get_free_pages(gfp_t gfp_mask, unsigned int order);`
- `struct page * alloc_pages(gfp_t gfp_mask, unsigned int order);`
- `void __free_pages(struct page *page, unsigned int order);`

核心实现

- `struct page * __alloc_pages_nodemask(gfp_t gfp_mask, unsigned int order, struct zonelist *zonelist, nodemask_t *nodemask);`

### Slab Allocator(对象缓存分配) ###

The fundamental idea behind slab allocation technique is based on the observation that some kernel data objects are frequently created and destroyed after they are not needed anymore. This implies that for each allocation of memory for these data objects, some time is spent to find the best fit for that data object. Moreover, deallocation of the memory after destruction of the object contributes to fragmentation of the memory, which burdens the kernel some more to rearrange the memory.Slab 算法的发现是基于内核中内存使用的一些特点：一些需要频繁使用的同样大小数据经常在使用后不久又再次被用到；找到合适大小的内存所消耗的时间远远大于释放内存所需要的时间。所以Slab算法的发明人认为内存对象在使用之后不是立即释放给系统而是将它们用链表之类的数据结构管理起来以备将来使用，频繁分配和释放的内存对象应该用缓存管理起来。Linux的slab分配器就是基于这样的想法实现的，这个算法在空间和时间上都有很高的效率。

#### Slab implementation ####

下面是Slab分配器中的主要数据结构之间的关系图。最上层是`cache_chain`，它是slab缓存的一个链表。这个链表用来查找需要分配内存大小的最佳匹配。链表中的元素是用来管理给定大小内存的一个缓存池指针`kmem_cache`。

![Slab分配器的主要结构](http://www.ibm.com/developerworks/linux/library/l-linux-slab-allocator/figure1.gif)

每个缓存对象包含了slab对象的链表。一个slab对象是一块连续内存（页面）。其中有三个slab对象链表：

- `slabs_full`,完全分配好的slab对象链表
- `slabs_partial`,部分分配好的slab对象链表
- `slabs_empty`,空对象链表，slab对象都没有分配好

其中空对象链表是内存回收的主要候选来源。slab链表上的对象是一块连续内存，这块内存被分成内存数据对象。这些数据对象是slab缓存上分配和释放的最小单位。

由于数据对象是从slab上分配的，所以单个的slab可以从一个slab链表移动到另一个slab链表。比如当`slabs_partial`上的一个slab的内存对象全部被分配了之后，这个slab就从`slabs_partial`上移动到`slabs_full`。当`slabs_full`上的一个slab上的部分内存对象被释放之后，这个slab就从`slabs_full`链表移动到`slabs_partial`上，当这个slab上的所有内存对象都被释放之后，它就会再次移动到`slabs_empty`上。

#### slab分配器带来的好处 ####

- 通过缓存类似对象数据，内核中频繁的小数据对象的分配不会再消耗过多的时间，同时减少了系统的内存碎片
- slab分配支持常用对象数据的初始化，减少了同类对象数据重复的初始化过程
- slab分配支持硬件缓存对齐和着色，这样不同缓存下的对象数据可以使用同样的硬件缓存行，可以提高系统的性能

#### 接口 ####

- `struct kmem_cache *;`缓存指针，用于分配，释放缓存中的数据对象
- `struct kmem_cache *kmem_cache_create( const char *name, size_t size,`
  `size_t align, unsigned long flags,`
  `void (*ctor)(void*, struct kmem_cache *, unsigned long),`
  `void (*dtor)(void*, struct kmem_cache *, unsigned long));`创建缓存，`ctor`和`dtor`两个回调函数是提供给用户初始化和释放对象数据用的
- `void kmem_cache_destroy( struct kmem_cache *cachep );`销毁缓存
- `void* kmem_cache_alloc( struct kmem_cache *cachep, gfp_t flags );`从缓存中分配对象数据，其中的`flags`和`kmalloc`函数使用的标记一样，用于控制缓存内部从buddy system中分配内存页面的行为
- `void kmem_cache_free( struct kmem_cache *cachep, void *objp );`回收数据到缓存中的slab对象

- `void *kmalloc( size_t size, int flags );`kmalloc和前面的从缓存中分配对象数据一样，只不过它不需要提供一个特定的缓存，它通过遍历系统中可用的缓存来分配对象数据。这样我们终于知道kmalloc的实现了，它是slab分配器的一个接口
- `void kfree( const void *objp );`

#### further reading ####

- [SLOB Allocator](http://en.wikipedia.org/wiki/SLOB)
- [SLAB](http://en.wikipedia.org/wiki/Slab_allocation)
- [SLUB](http://lwn.net/Articles/229984/)
- [Anatomy of the Linux slab allocator](http://www.ibm.com/developerworks/linux/library/l-linux-slab-allocator/)

## TODOS ##

- segmentation
- swaping
- demand paging
