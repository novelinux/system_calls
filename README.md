system calls
========================================

Linux的系统调用是通过软中断swi来实现的从用户态到内核态.

fork
----------------------------------------

https://github.com/novelinux/system_calls/blob/master/fork.md

user --> kernel
----------------------------------------

在系统调用通过软中断swi从用户态切换到内核态的压栈过程如下所示:

```
| kernel stack |  struct pt_regs
|--------------|-----------------------
|   r0 ~ r12   |
|    sp_usr    |    sp
|    lr_usr    |    lr
|    lr_svc    |    pc
|     cpsr     |   cpsr
|      r0      |
|              |------------------------ <- sp_svc
```

**注意**:

1.lr_svc中是指向swi中断指令的下一条指令.
2.这里r0会存两个位置: 一个代表第一个参数，另一个代表返回值。

### swi

软中断调用过程:

https://github.com/novelinux/arch-arm-common/tree/master/swi/README.md

### vector_swi

软中断处理函数:

https://github.com/novelinux/linux-4.x.y/tree/master/arch/arm/kernel/entry-common.S/vector_swi.md

### struct pt_regs

https://github.com/novelinux/linux-4.x.y/tree/master/arch/arm/include/asm/ptrace.h/struct_pt_regs.md

kernel --> user
----------------------------------------

在系统调用从内核态返回到用户态的调用过程如下所示:

```
| kernel stack |  struct pt_regs
|--------------|----------------------- <- sp_svc
|   r0 ~ r12   |         --> r1 ~ r12
|    sp_usr    |    sp   --> sp_usr
|    lr_usr    |    lr   --> lr_usr
|    lr_svc    |    pc   --> lr_svc    --> pc
|     cpsr     |   cpsr  --> spsr_cxsf
|      r0      |
|              |------------------------
```

**注意**: ret_fast_syscall不会将r0寄存器值恢复,r0要用来保存返回值.

在从内核态返回到用户态的过程是由ret_fast_syscall来完成的:

https://github.com/novelinux/linux-4.x.y/tree/master/arch/arm/kernel/entry-common.S/ret_fast_syscall.md
