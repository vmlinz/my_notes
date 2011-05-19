# arm linux 系统调用实现 #

In this article we will dive into system call interface implementation in arm linux(with gnu eabi).我们将从bionic中的open函数开始追溯arm linux的系统调用实现（使用gnu eabi）。

Linux的应用程序要想访问内核必须使用系统调用从而实现从usr模式转到svc模式。在arm中，这个过程是通过swi(或者和它等价的指令)来实现模式转换的。

## 从bionic libc中的open函数追溯系统调用实现 ##

相关文件：

- bionic/libc/unistd/open.c

- bionic/libc/arch-arm/syscalls/__open.S

- linux/arch/arm/kernel/entry-common.S

- linux/arch/arm/kernel/entry-armv.S

- linux/arch/arm/kernel/entry-header.S

相关调用过程：

- `int open(const char *pathname, int flags, ...);` bionic/libc/unistd/open.c

- `__open` bionic/libc/arch-arm/syscalls/__open.S

    `__open:`

    `.save   {r4, r7}`

    `stmfd   sp!, {r4, r7}`

    `ldr     r7, =__NR_open`

    `swi     #0`

    `ldmfd   sp!, {r4, r7}`

    `movs    r0, r0`

    `bxpl    lr`

    `b       __set_syscall_errno`

- `vector_swi` linux/arch/arm/kernel/entry-common.S

    `adr	tbl, sys_call_table		@ load syscall table pointer`

    `cmp	scno, #NR_syscalls		@ check upper syscall limit`

    `adr	lr, BSYM(ret_fast_syscall)	@ return address`

    `ldrcc pc, [tbl, scno, lsl #2]		@ call sys_* routine`

- `sys_call_table` 系统调用跳转表，里面保存了各个系统调用实现的地址。

    `.type sys_call_table, #object`

    `ENTRY(sys_call_table)`

    `#include "calls.S"`

## 资源和链接 ##

- [eabi](http://wiki.debian.org/ArmEabiPort)
- [ARM linux系统调用的实现原理](http://blog.csdn.net/hongtao_liu/archive/2009/05/22/4208895.aspx)
- [SWI : SoftWare Interrupt](http://www.heyrick.co.uk/assembler/swi.html)

## TODOS ##

- 找出eabi中寄存器使用和参数传递的规则
- 了解swi跳转的更多详细知识，比如如何跳转到一个固定位置的
- arm系统的中断系统
- 熟悉常用的arm指令和gnu 汇编器指令
