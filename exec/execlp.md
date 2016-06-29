execlp
========================================

path: bionic/libc/upstream-openbsd/lib/libc/gen/exec.c
```
int
execlp(const char *name, const char *arg, ...)
{
    va_list ap;
    char **argv;
    int n;

    va_start(ap, arg);
    n = 1;
    while (va_arg(ap, char *) != NULL)
        n++;
    va_end(ap);
    argv = alloca((n + 1) * sizeof(*argv));
    if (argv == NULL) {
        errno = ENOMEM;
        return (-1);
    }
    va_start(ap, arg);
    n = 1;
    argv[0] = (char *)arg;
    while ((argv[n] = va_arg(ap, char *)) != NULL)
        n++;
    va_end(ap);
    return (execvp(name, argv));
}
```

execvp
----------------------------------------

https://github.com/novelinux/system_calls/blob/master/exec/execvp.md
