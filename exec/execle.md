execle
========================================

path: bionic/libc/upstream-openbsd/lib/libc/gen/exec.c
```
int
execle(const char *name, const char *arg, ...)
{
    va_list ap;
    char **argv, **envp;
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
    envp = va_arg(ap, char **);
    va_end(ap);
    return (execve(name, argv, envp));
}
```

execve
----------------------------------------