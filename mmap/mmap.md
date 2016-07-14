mmap
========================================

C标准库提供了mmap函数建立映射。在内核一端，提供了两个系统调用mmap和mmap2。
某些体系结构实现了两个版本。两个函数的参数相同。这两个调用都会在用户虚拟
地址空间中的pos位置，建立一个长度为len的映射，其访问权限通过prot定义。
flags是一个标志集，用于设置一些参数。相关的文件通过其文件描述符fd标识。
mmap和mmap2之间的差别在于偏移量的语义（off）。在这两个调用中，它都表示
映射在文件中开始的位置。对于mmap，位置的单位是字节，而mmap2使用的单位
则是页（PAGE_SIZE）。因此即使文件比可用地址空间大，也可以映射文件的一部分。
通常C标准库只提供一个函数，由应用程序用来创建内存映射。接下来该函数调用在
内部转换为适合于体系结构的系统调用。

mmap
----------------------------------------

path: bionic/libc/bionic/mmap.cpp
```
// mmap2(2) is like mmap(2), but the offset is in 4096-byte blocks, not bytes.
extern "C" void*  __mmap2(void*, size_t, int, int, int, size_t);

#define MMAP2_SHIFT 12 // 2**12 == 4096

static bool kernel_has_MADV_MERGEABLE = true;

void* mmap64(void* addr, size_t size, int prot, int flags, int fd, off64_t offset) {
  if (offset < 0 || (offset & ((1UL << MMAP2_SHIFT)-1)) != 0) {
    errno = EINVAL;
    return MAP_FAILED;
  }

  bool is_private_anonymous = (flags & (MAP_PRIVATE | MAP_ANONYMOUS)) != 0;
  void* result = __mmap2(addr, size, prot, flags, fd, offset >> MMAP2_SHIFT);

  if (result != MAP_FAILED && kernel_has_MADV_MERGEABLE && is_private_anonymous) {
    ErrnoRestorer errno_restorer;
    int rc = madvise(result, size, MADV_MERGEABLE);
    if (rc == -1 && errno == EINVAL) {
      kernel_has_MADV_MERGEABLE = false;
    }
  }

  return result;
}

void* mmap(void* addr, size_t size, int prot, int flags, int fd, off_t offset) {
  return mmap64(addr, size, prot, flags, fd, static_cast<off64_t>(offset));
}
```

flags
----------------------------------------

* MAP_FIXED: 指定了除了给定地址以外，不能将其它地址用于映射. 如果没有设置
该标志，内核可以在受阻的情况下随时改变目标地址.
* MAP_SHARED: 如果一个对象(通常是文件)在几个进程之间共享时，必须使用MAP_SHARED.
* MAP_PRIVATE: 创建一个与数据源分离的私有映射，对映射区域的写入操作不影响文件中的数据.
* MAP_ANONYMOUS: 创建与任何数据源都不相关的匿名映射，fd和off参数被忽略. 此类映射可用于为应用程序分配类似malloc所用的内存.

prot
----------------------------------------

prot可指定PROT_EXEC、PROT_READ、PROT_WRITE、PROT_NONE值的组合，
来定义访问权限。并非所有处理器都实现了所有组合，因而区域实际授予
的权限可能比指定的要多。尽管内核尽力设置指定的权限，但它只能保证
实际设置的访问权限不会比指定的权限有更多的限制。

__mmap2
----------------------------------------

https://github.com/novelinux/system_calls/blob/master/mmap/__mmap2.md
