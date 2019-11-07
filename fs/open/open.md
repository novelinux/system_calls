open
========================================

* open

```
open
 |
__openat
 |
sys_openat
 |
do_sys_open
 |
 +-> build_open_flags
 |
 +-> getname
 |
 +-> getunused_fd_flags
 |
 +-> do_filp_open
 |   |
 |   +-> set_nameidata
 |   |
 |   +-> path_openat
 |   |   |
 |   |   +-> get_empty_filep (struct file)
 |   |   |
 |   |   +-> path_init
 |   |   |
 |   |   +-> link_path_walk
 |   |   |
 |   |   +-> do_last
 |   |
 |   +-> restore_nameidata
 |
 +-> fd_install +-> __fd_install
```

* do_last

```
do_last
 |
 +-> handle_dots (nd->last_type != LAST_NORM)
 |
 +-> lookup_fast (!(open_flag & O_CREAT))
 |
 +-> mnt_want_write (op->open_flag & (O_CREAT | O_TRUNC | O_WRONLY | O_RDWR))
 |
 +-> lookup_open
 |   |
 |   +-> lookup_dcache
 |   |
 |   +-> lookup_real
 |   |
 |   +-> vfs_create
 |       |
 |       +-> may_create
 |       |
 |       +-> dir->i_op->create (ext4_dir_inode_operations)
 |
 +-> may_open
 |
 +-> vfs_open
```

* ext4_create

open
----------------------------------------

在读和写文件之前，我们必须先打开文件。从应用程序的角度来看，
这是通过标准库的open函数完成的，该函数返回一个文件描述符。
该函数使用了sys_open系统调用.

path: bionic/libc/bionic/open.cpp
```
int open(const char* pathname, int flags, ...) {
  mode_t mode = 0;

  if ((flags & O_CREAT) != 0) {
    va_list args;
    va_start(args, flags);
    mode = (mode_t) va_arg(args, int);
    va_end(args);
  }

  return __openat(AT_FDCWD, pathname, force_O_LARGEFILE(flags), mode);
}
__strong_alias(open64, open);
```

__openat
----------------------------------------

path: bionic/libc/arch-arm/syscalls/__openat.S
```
/* Generated by gensyscalls.py. Do not edit. */

#include <private/bionic_asm.h>

ENTRY(__openat)
    mov     ip, r7
    ldr     r7, =__NR_openat
    swi     #0
    mov     r7, ip
    cmn     r0, #(MAX_ERRNO + 1)
    bxls    lr
    neg     r0, r0
    b       __set_errno_internal
END(__openat)
```

sys_openat
----------------------------------------

最终通过系统调用会切换到内核态调用sys_openat函数.

https://github.com/novelinux/linux-4.x.y/tree/master/fs/open.c/sys_openat.md

samples
----------------------------------------

https://github.com/novelinux/linux-4.x.y/tree/master/samples/fs/test_open/README.md