mmap
========================================

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

__mmap2
----------------------------------------

https://github.com/novelinux/system_calls/blob/master/mmap/__mmap2.md
