execvp
========================================

path: bionic/libc/upstream-openbsd/lib/libc/gen/exec.c
```
int
execvp(const char *name, char *const *argv)
{
    return execvpe(name, argv, environ);
}
```

execvpe
----------------------------------------

https://github.com/novelinux/system_calls/blob/master/exec/execvpe.md