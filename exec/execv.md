execv
========================================

path: bionic/libc/upstream-openbsd/lib/libc/gen/exec.c
```
int
execv(const char *name, char *const *argv)
{
    (void)execve(name, argv, environ);
    return (-1);
}
```

execve
----------------------------------------
