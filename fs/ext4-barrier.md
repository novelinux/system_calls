## Barrier

barrier是保证日志文件系统的WAL(write ahead logging)一种手段--现代日志文件系统如ext4有个journal区,类似数据库领域的redo log,用于意外崩溃后的快速恢复.数据写入磁盘时,理应先写入journal区,再写入数据在磁盘的实际对应位置;磁盘厂商为了加快磁盘写入速度,磁盘都内置cache,数据一般都先写入磁盘的cache.

cache能加快写入速度,当然是极好的东西,但磁盘一般会对cache内缓存数据排序使之最优刷新到磁盘,这样就可能导致要刷新的实际数据和journal顺序错乱;一旦系统崩溃,下次开机时磁盘要参考journal区来恢复,但此时journal记录顺序与数据实际刷新顺序不同就会导致数据反而"恢复"到不一致了.而barrier如其名--'栅栏',先加一个'栅栏',保证journal总是先写入记录,然后对应数据才刷新到磁盘,这种方式保证了系统崩溃后磁盘恢复的正确性,但对写入性能有蛮大影响.

前面说道barrier不开启情况下journal记录会和数据刷新顺序不一致,但journal只有恢复时才会用,所以只要disk无需恢复就行:

无cache或禁用cache这样子写就是同步的了,自然barrier也就没什么意义;Percona的《Tuning For Speed: Percona Server and Fusion-io》上说利用ioMemory的无cache特性,自然无需write barrier.
