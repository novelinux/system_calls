## read

用户进程读取文件数据,有两种情形:
* (1) 所需要的数据不在内存中,也即不在页面Cache中。这时就需要直接从磁盘上读取。
* (2) 所需的数据已经在内存中,此时只需从页面Cache中找到具体位置,然后将数据拷贝到用户缓冲区。而不需要进行磁盘I/O操作。

### 数据不在页面Cache中

### 数据在页面Cache中