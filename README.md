system calls
========================================

Linux的系统调用是通过软中断swi来实现的从用户态到内核态.

fork
----------------------------------------

https://github.com/novelinux/system_calls/blob/master/fork.md

user --> kernel
----------------------------------------

```
| kernel stack |
|--------------|-----------|------------
|   r0 ~ r12   |           |
|     sp^      |           |
|     lr^      |           | {struct  pt_regs}
|     lr       |           | { S_FRAME_SIZE  }
|    cpsr      |           |
|     r0       | <- sp     |
|              |------------------------
```