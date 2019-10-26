# Ext4 - open-write

```
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>

#include <fcntl.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    char *filename = argv[1];

    int fd = open(filename, O_CREAT | O_RDWR);
    if (fd < 0) {
        printf("open %s failed: %s\n", filename, strerror(errno));
        return 1;
    }
    if (write(fd, "hello world\n", 12) != 12) {
        printf("write failed: %s\n", strerror(errno));
        return 1;
    }

    return 0;
}
```

* TRACE: https://github.com/novelinux/system_calls/blob/master/fs/ext4-jbd2-open-write-fsync_traces.md

## open | O_CREATE -- test-open-write

```
open
 |
sys_openat
 |
do_sys_open
 |
do_filp_open
 |
path_openat
 |
do_last
 |
lookup_open
 |
 +-> lookup_real
 |
 +-> vfs_create
```

### lookup_real

```
lookup_real
 |
ext4_lookup
 |
ext4_find_entry
 |
ext4_getblk
 |
ext4_map_blocks
 |
ext4_es_lookup_extent { event: ext4_es_lookup_extent_enter/ext4_es_lookup_extent_exit }
```

### vfs_create

```
vfs_create
 |
ext4_create
 |
 +-> ext4_new_inode_start_handle
 |   |
 |   __ext4_new_inode
 |   |
 |   +-> { event: ext4_request_inode }
 |   |
 |   +-> __ext4_journal_start_sb { event: ext4_journal_start }
 |   |   |
 |   |   +-> jbd2__journal_start { event: jbd2_handle_start }
 |   |
 |   +-> ext4_ext_tree_init
 |   |   |
 |   |   +-> ext4_mark_inode_dirty { event: ext4_mark_inode_dirty }
 |   |
 |   +-> ext4_mark_inode_dirty { event: ext4_mark_inode_dirty }
 |   |
 |   +-> { event: ext4_allocate_inode }
 |
 +-> ext4_journal_current_handle
 |
 +-> ext4_add_nondir
 |   |
 |   +-> ext4_add_entry
 |   |   |
 |   |   +-> ext4_read_dirblock -> __ext4_read_dirblock
 |   |   |   |
 |   |   |   +-> ext4_bread -> ext4_getblk
 |   |   |                           |
 |   |   |   ext4_map_blocks  <------+
 |   |   |   |
 |   |   |   ext4_es_lookup_extent
 |   |   |   |
 |   |   |   +-> { event: ext4_es_lookup_extent_enter/ext4_es_lookup_extent_exit }
 |   |   |
 |   |   +-> add_dirent_to_buf
 |   |   |
 |   |   +-> ext4_mark_inode_dirty { event: ext4_mark_inode_dirty }
 |   |
 |   +-> ext4_mark_inode_dirty { event: ext4_mark_inode_dirty }
 |
 +-> ext4_journal_stop
     |
     __ext4_journal_stop
     |
     jbd2_journal_stop { event: jbd2_handle_stats }
```
