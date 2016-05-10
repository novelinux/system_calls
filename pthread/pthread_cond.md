pthread cond and mutex
========================================

一.  互斥锁和条件变量是同步的基本组成部分
----------------------------------------

　　互斥锁和条件变量出自Posix.1线程标准，多用来同步一个进程中各个线程。
但如果将二者存放在多个进程间共享的内存区中，它们也可以用来进行进程间的同步。

1. 互斥锁用于保护临界区 - 以保护任何时刻只有一个线程在执行其中的代码.

其大体轮廓大体如下：

```
　　lock_the_mutex(...);

　　临界区

　　unlock_the_mutex(...);
```

下列三个函数给一个互斥锁上锁和解锁：

```
    #include <pthread.h>

　  // 若不能立刻获得锁，将阻塞在此处
    int pthread_mutex_lock(pthread_mutex_t *mptr);
　  // 若不能立刻获得锁，将返回EBUSY，用户可以根据此返回值做其他操作，非阻塞模式
    int pthread_mutex_trylock(pthread_mutex_t *mptr);
　　// 释放锁
　　int pthread_mutex_unlock(pthread_mutex_t *mptr);
```

互斥锁通常用于保护由多个线程或多个进程分享的共享数据(Share Data)

2.条件变量 - 它是发送信号与等待信号。

互斥锁用于上锁，条件变量则用于等待。一般来说，在一个线程中调用pthread_cond_wait(..)
等待某个条件的成立，此时该进程阻塞在这里，另外一个进程/线程进行某种操作，当某种条件成立时，
调用pthread_cond_signal(...)来发送信号，从而使pthread_cond_wait(...)返回。
此处要注意的是，这里所谈到的信号，不是系统级别的SIGXXXX信号，只是用信号这个词语更容易理解。
条件变量与信号量更接近或者就可以认为是信号量。

下列两个函数用来对条件变量进行控制：

```
　　#include <pthread.h>

　　int pthread_cond_wait(pthread_cond_t *cptr, pthread_mutex_t *mptr);
　　int pthread_cond_signal(pthread_cond_t *cptr);
```

由代码我们可以看出，条件变量的使用是需要结合锁机制的，即上面所提到的互斥锁。
也就是说，一个进程/线程要等到临界区的共享数据达到某种状态时再进行某种操作，
而这个状态的成立，则是由另外一个进程/线程来完成后发送信号来通知的。
其实想一想，pthread_cond_wait函数也可以用一个while死循环来等待条件的成立，但要注意的是，
使用while死循环会严重消耗CPU，而pthread_cond_wait则是采用线程睡眠的方式，
它是一种等待模式，而不是一直的检查模式。

总的来说，给条件变量发送信号的代码大体如下：

```
　　pthread_mutex_lock(&mutex);

　　设置条件为真

　　pthread_mutex_unlock(&mutex);　　

    pthread_cond_signal(&cond);　　//发送信号
```

等待条件并进入睡眠以等待条件变为真的代码大体如下：

```
　　pthread_mutex_lock(&mutex);　

　　while(条件为假)
　　　　pthread_cond_wait(&cond,&mutex);　　

　　执行某种操作

　　pthread_mutex_unlock(&mutex);
```

　　在这里需要注意的是，pthread_cond_wait(&cond,&mutex)是一个原子操作，当它执行时，
首先对mutex解锁，这样另外的线程才能得到锁来修改条件，pthread_cond_wait解锁后，
再将本身的线程投入睡眠，另外，当该函数返回时，会再对mutex进行加锁，
这样才能“执行某种操作”后unlock锁。