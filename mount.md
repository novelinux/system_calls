mount
========================================

mount
----------------------------------------

path: bionic/libc/arch-arm/syscalls/mount.S
```
/* Generated by gensyscalls.py. Do not edit. */

#include <private/bionic_asm.h>

ENTRY(mount)
    mov     ip, sp
    stmfd   sp!, {r4, r5, r6, r7}
    .cfi_def_cfa_offset 16
    .cfi_rel_offset r4, 0
    .cfi_rel_offset r5, 4
    .cfi_rel_offset r6, 8
    .cfi_rel_offset r7, 12
    ldmfd   ip, {r4, r5, r6}
    ldr     r7, =__NR_mount
    swi     #0
    ldmfd   sp!, {r4, r5, r6, r7}
    .cfi_def_cfa_offset 0
    cmn     r0, #(MAX_ERRNO + 1)
    bxls    lr
    neg     r0, r0
    b       __set_errno_internal
END(mount)
```

sys_mount
----------------------------------------

最终通过系统调用会切换到内核态调用sys_mount函数.

https://github.com/novelinux/linux-4.x.y/tree/master/fs/namespace.c/sys_mount.md