# JBD2

## Test Code

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
    fsync(fd);

    return 0;
}
```

## open-write JBD2 - nobarrier

```
  test-open-writ-6040  [000] ...1  6883.863812: writeback_mark_inode_dirty: bdi (unknown): ino=59025 state= flags=I_DIRTY_SYNC|I_DIRTY_TIME
  test-open-writ-6040  [000] ...1  6883.863920: writeback_dirty_inode_start: bdi (unknown): ino=59025 state= flags=I_DIRTY_SYNC|I_DIRTY_TIME
  test-open-writ-6040  [000] ...1  6883.863930: writeback_dirty_inode: bdi (unknown): ino=59025 state= flags=I_DIRTY_SYNC|I_DIRTY_TIME
  test-open-writ-6040  [000] ...1  6883.864725: ext4_es_lookup_extent_enter: dev 259,0 ino 2 lblk 0
  test-open-writ-6040  [000] ...1  6883.864749: ext4_es_lookup_extent_exit: dev 259,0 ino 2 found 1 [0/1) 1549 W0x10
  test-open-writ-6040  [000] ...1  6883.865692: ext4_request_inode: dev 259,0 dir 2 mode 0107320
  test-open-writ-6040  [000] ...1  6883.865772: ext4_journal_start: dev 259,0 blocks, 35 rsv_blocks, 0 caller __ext4_new_inode+0x750/0x1338
  test-open-writ-6040  [000] ...1  6883.865962: ext4_mark_inode_dirty: dev 259,0 ino 23 caller ext4_ext_tree_init+0x24/0x30
  test-open-writ-6040  [000] ...1  6883.865967: ext4_mark_inode_dirty: dev 259,0 ino 23 caller __ext4_new_inode+0x1108/0x1338
  test-open-writ-6040  [000] ...1  6883.865972: ext4_allocate_inode: dev 259,0 ino 23 dir 2 mode 0107320
  test-open-writ-6040  [000] ...1  6883.865981: ext4_es_lookup_extent_enter: dev 259,0 ino 2 lblk 0
  test-open-writ-6040  [000] ...1  6883.865986: ext4_es_lookup_extent_exit: dev 259,0 ino 2 found 1 [0/1) 1549 W0x10
  test-open-writ-6040  [000] ...1  6883.866006: ext4_mark_inode_dirty: dev 259,0 ino 2 caller add_dirent_to_buf+0x170/0x1dc
  test-open-writ-6040  [000] ...1  6883.866019: ext4_mark_inode_dirty: dev 259,0 ino 23 caller ext4_add_nondir+0x34/0x7c
  test-open-writ-6040  [000] ...1  6883.866110: ext4_da_write_begin: dev 259,0 ino 23 pos 0 len 12 flags 0
  test-open-writ-6040  [000] ...1  6883.866137: ext4_journal_start: dev 259,0 blocks, 1 rsv_blocks, 0 caller ext4_da_write_begin+0x1d4/0x304
  test-open-writ-6040  [000] ...1  6883.866155: ext4_es_lookup_extent_enter: dev 259,0 ino 23 lblk 0
  test-open-writ-6040  [000] ...1  6883.866156: ext4_es_lookup_extent_exit: dev 259,0 ino 23 found 0 [0/0) 0
  test-open-writ-6040  [000] ...1  6883.866163: ext4_ext_map_blocks_enter: dev 259,0 ino 23 lblk 0 len 1 flags
  test-open-writ-6040  [000] ...1  6883.866178: ext4_es_find_delayed_extent_range_enter: dev 259,0 ino 23 lblk 0
  test-open-writ-6040  [000] ...1  6883.866188: ext4_es_find_delayed_extent_range_exit: dev 259,0 ino 23 es [0/0) mapped 0 status
  test-open-writ-6040  [000] ...1  6883.866194: ext4_es_insert_extent: dev 259,0 ino 23 es [0/4294967295) mapped 576460752303423487 status H
  test-open-writ-6040  [000] ...1  6883.866213: ext4_ext_map_blocks_exit: dev 259,0 ino 23 flags  lblk 0 pblk 18446743800513854160 len 1 mflags  ret 0
  test-open-writ-6040  [000] ...2  6883.866224: ext4_da_reserve_space: dev 259,0 ino 23 mode 0107320 i_blocks 0 reserved_data_blocks 1 reserved_meta_blocks 0
  test-open-writ-6040  [000] ...1  6883.866226: ext4_es_insert_extent: dev 259,0 ino 23 es [0/1) mapped 576460752303423487 status D
  test-open-writ-6040  [000] ...1  6883.866250: ext4_da_write_end: dev 259,0 ino 23 pos 0 len 12 copied 12
  test-open-writ-6040  [000] d..2  6883.866264: writeback_dirty_page: bdi 8:0: ino=23 index=0
  test-open-writ-6040  [000] ...1  6883.866275: writeback_mark_inode_dirty: bdi 8:0: ino=23 state= flags=I_DIRTY_PAGES
  test-open-writ-6040  [000] ...1  6883.866285: writeback_dirty_inode_enqueue: dev 259,0 ino 23 dirtied 4295625682 state I_DIRTY_PAGES mode 0107320

  test-open-writ-6040  [000] ...1  6883.866287: writeback_mark_inode_dirty: bdi 8:0: ino=23 state=I_DIRTY_PAGES flags=I_DIRTY_SYNC|I_DIRTY_DATASYNC|I_DIRTY_PAGES
  test-open-writ-6040  [000] ...1  6883.866289: writeback_dirty_inode_start: bdi 8:0: ino=23 state=I_DIRTY_PAGES flags=I_DIRTY_SYNC|I_DIRTY_DATASYNC|I_DIRTY_PAGES
  test-open-writ-6040  [000] ...1  6883.866290: ext4_journal_start: dev 259,0 blocks, 2 rsv_blocks, 0 caller ext4_dirty_inode+0x30/0x68
  test-open-writ-6040  [000] ...1  6883.866291: ext4_mark_inode_dirty: dev 259,0 ino 23 caller ext4_dirty_inode+0x44/0x68
  test-open-writ-6040  [000] ...1  6883.866296: writeback_dirty_inode: bdi 8:0: ino=23 state=I_DIRTY_PAGES flags=I_DIRTY_SYNC|I_DIRTY_DATASYNC|I_DIRTY_PAGES

   kworker/u16:2-5968  [000] ...2  6888.104338: writeback_start: bdi 8:0: sb_dev 0:0 nr_pages=49320 sync_mode=0 kupdate=1 range_cyclic=1 background=0 reason=periodic cgroup=/
   kworker/u16:2-5968  [000] .n.2  6888.104490: writeback_queue_io: bdi 8:0: older=4295625906 age=2000 enqueue=2 reason=periodic cgroup=/
   kworker/u16:2-5968  [000] ...1  6888.106774: writeback_single_inode_start: bdi 8:0: ino=0 state=I_DIRTY_PAGES|I_SYNC dirtied_when=4295625605 age=5 index=0 to_write=13312 wrote=0 cgroup=/
   kworker/u16:2-5968  [000] ...1  6888.107473: wbc_writepage: bdi 8:0: towrt=13312 skip=0 mode=0 kupd=1 bgrd=0 reclm=0 cyclic=1 start=0x0 end=0x7fffffffffffffff cgroup=/
   kworker/u16:2-5968  [000] ...1  6888.108259: writeback_single_inode: bdi 8:0: ino=0 state=I_SYNC dirtied_when=4295625605 age=5 index=25952258 to_write=13312 wrote=1 cgroup=/
   kworker/u16:2-5968  [000] ...1  6888.108849: writeback_single_inode_start: bdi 8:0: ino=23 state=I_DIRTY_SYNC|I_DIRTY_DATASYNC|I_DIRTY_PAGES|I_SYNC dirtied_when=4295625682 age=4 index=0 to_write=13312 wrote=0 cgroup=/
   kworker/u16:2-5968  [000] ...1  6888.109429: ext4_writepages: dev 259,0 ino 23 nr_to_write 13312 pages_skipped 0 range_start 0 range_end 9223372036854775807 sync_mode 0 for_kupdate 1 range_cyclic 1 writeback_index 0
   kworker/u16:2-5968  [000] ...1  6888.110203: ext4_journal_start: dev 259,0 blocks, 8 rsv_blocks, 0 caller ext4_writepages+0x524/0xd24
   kworker/u16:2-5968  [000] ...1  6888.110800: ext4_da_write_pages: dev 259,0 ino 23 first_page 0 nr_to_write 13312 sync_mode 0
   kworker/u16:2-5968  [000] ...1  6888.111381: ext4_da_write_pages_extent: dev 259,0 ino 23 lblk 0 len 1 flags 0x200
   kworker/u16:2-5968  [000] ...1  6888.111957: ext4_es_lookup_extent_enter: dev 259,0 ino 23 lblk 0
   kworker/u16:2-5968  [000] ...1  6888.112513: ext4_es_lookup_extent_exit: dev 259,0 ino 23 found 1 [0/1) 576460752303423487 D0x10
   kworker/u16:2-5968  [000] ...1  6888.113091: ext4_ext_map_blocks_enter: dev 259,0 ino 23 lblk 0 len 1 flags CREATE|DELALLOC|METADATA_NOFAIL
   kworker/u16:2-5968  [000] ...1  6888.113693: ext4_request_blocks: dev 259,0 ino 23 flags HINT_DATA|DELALLOC_RESV|USE_RESV len 1 lblk 0 goal 0 lleft 0 lright 0 pleft 0 pright 0
   kworker/u16:2-5968  [000] ...1  6888.114506: ext4_mballoc_alloc: dev 259,0 inode 23 orig 0/0/1@0 goal 0/0/1@0 result 0/1563/1@0 blks 1 grps 1 cr 1 flags HINT_DATA|HINT_NOPREALLOC|DELALLOC_RESV|USE_RESV tail 0 broken 0
   kworker/u16:2-5968  [000] ...1  6888.116550: ext4_allocate_blocks: dev 259,0 ino 23 flags HINT_DATA|DELALLOC_RESV|USE_RESV len 1 block 1563 lblk 0 goal 0 lleft 0 lright 0 pleft 0 pright 0
   kworker/u16:2-5968  [000] ...1  6888.117168: ext4_mark_inode_dirty: dev 259,0 ino 23 caller __ext4_ext_dirty+0x6c/0x78
   kworker/u16:2-5968  [000] ...1  6888.117957: ext4_get_reserved_cluster_alloc: dev 259,0 ino 23 lblk 0 len 1
   kworker/u16:2-5968  [000] ...2  6888.118828: ext4_da_update_reserve_space: dev 259,0 ino 23 mode 0107320 i_blocks 0 used_blocks 1 reserved_data_blocks 1 reserved_meta_blocks 0 allocated_meta_blocks 0 quota_claim 1
   kworker/u16:2-5968  [000] ...1  6888.119427: writeback_mark_inode_dirty: bdi 8:0: ino=23 state=I_DIRTY_SYNC|I_DIRTY_DATASYNC|I_DIRTY_PAGES|I_SYNC flags=I_DIRTY_SYNC
   kworker/u16:2-5968  [000] ...1  6888.120003: writeback_dirty_inode_start: bdi 8:0: ino=23 state=I_DIRTY_SYNC|I_DIRTY_DATASYNC|I_DIRTY_PAGES|I_SYNC flags=I_DIRTY_SYNC
   kworker/u16:2-5968  [000] ...1  6888.120599: ext4_journal_start: dev 259,0 blocks, 2 rsv_blocks, 0 caller ext4_dirty_inode+0x30/0x68
   kworker/u16:2-5968  [000] ...1  6888.120686: ext4_mark_inode_dirty: dev 259,0 ino 23 caller ext4_dirty_inode+0x44/0x68
   kworker/u16:2-5968  [000] ...1  6888.120711: writeback_dirty_inode: bdi 8:0: ino=23 state=I_DIRTY_SYNC|I_DIRTY_DATASYNC|I_DIRTY_PAGES|I_SYNC flags=I_DIRTY_SYNC
   kworker/u16:2-5968  [000] ...1  6888.120722: ext4_discard_preallocations: dev 259,0 ino 23
   kworker/u16:2-5968  [000] ...1  6888.120738: ext4_ext_map_blocks_exit: dev 259,0 ino 23 flags CREATE|DELALLOC|METADATA_NOFAIL lblk 0 pblk 1563 len 1 mflags NM ret 1
   kworker/u16:2-5968  [000] ...1  6888.120755: ext4_es_insert_extent: dev 259,0 ino 23 es [0/1) mapped 1563 status W
   kworker/u16:2-5968  [000] ...1  6888.120864: ext4_mark_inode_dirty: dev 259,0 ino 23 caller ext4_writepages+0xc1c/0xd24
   kworker/u16:2-5968  [000] ...1  6888.120967: ext4_journal_start: dev 259,0 blocks, 8 rsv_blocks, 0 caller ext4_writepages+0x524/0xd24
   kworker/u16:2-5968  [000] ...1  6888.120980: ext4_da_write_pages: dev 259,0 ino 23 first_page 1 nr_to_write 13311 sync_mode 0
   kworker/u16:2-5968  [000] ...1  6888.120997: ext4_writepages_result: dev 259,0 ino 23 ret 0 pages_written 1 pages_skipped 0 sync_mode 0 writeback_index 1
   kworker/u16:2-5968  [000] ...1  6888.121008: writeback_write_inode_start: bdi 8:0: ino=23 sync_mode=0 cgroup=/
   kworker/u16:2-5968  [000] ...1  6888.121020: writeback_write_inode: bdi 8:0: ino=23 sync_mode=0 cgroup=/
   kworker/u16:2-5968  [000] ...1  6888.121024: writeback_single_inode: bdi 8:0: ino=23 state=I_SYNC dirtied_when=4295625682 age=4 index=1 to_write=13312 wrote=1 cgroup=/
   kworker/u16:2-5968  [000] ...2  6888.121042: writeback_written: bdi 8:0: sb_dev 0:0 nr_pages=49318 sync_mode=0 kupdate=1 range_cyclic=1 background=0 reason=periodic cgroup=/
   kworker/u16:2-5968  [000] ...2  6888.121051: writeback_start: bdi 8:0: sb_dev 0:0 nr_pages=49318 sync_mode=0 kupdate=1 range_cyclic=1 background=0 reason=periodic cgroup=/
   kworker/u16:2-5968  [000] ...2  6888.121059: writeback_queue_io: bdi 8:0: older=4295625908 age=2000 enqueue=0 reason=periodic cgroup=/
   kworker/u16:2-5968  [000] ...2  6888.121066: writeback_written: bdi 8:0: sb_dev 0:0 nr_pages=49318 sync_mode=0 kupdate=1 range_cyclic=1 background=0 reason=periodic cgroup=/
   kworker/u16:2-5968  [000] .n.1  6888.121226: global_dirty_state: dirty=2 writeback=0 unstable=0 bg_thresh=52194 thresh=209545 limit=352521 dirtied=22058 written=21975
   kworker/u16:2-5968  [000] ...2  6888.122361: global_dirty_state: dirty=2 writeback=0 unstable=0 bg_thresh=52194 thresh=209545 limit=352521 dirtied=22058 written=21975
   kworker/u16:2-5968  [000] ...2  6888.122369: writeback_start: bdi 8:0: sb_dev 0:0 nr_pages=9223372036854775807 sync_mode=0 kupdate=0 range_cyclic=1 background=1 reason=background cgroup=/
   kworker/u16:2-5968  [000] ...2  6888.122376: writeback_queue_io: bdi 8:0: older=4295626108 age=0 enqueue=0 reason=background cgroup=/
   kworker/u16:2-5968  [000] ...2  6888.122382: writeback_written: bdi 8:0: sb_dev 0:0 nr_pages=9223372036854775807 sync_mode=0 kupdate=0 range_cyclic=1 background=1 reason=background cgroup=/
   kworker/u16:2-5968  [000] ...1  6888.122394: writeback_pages_written: 2
    jbd2/sda16-8-587   [000] ...1  6889.063532: ext4_es_lookup_extent_enter: dev 259,0 ino 8 lblk 100
    jbd2/sda16-8-587   [000] ...1  6889.063698: ext4_es_lookup_extent_exit: dev 259,0 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-587   [000] ...1  6889.063719: ext4_ext_map_blocks_enter: dev 259,0 ino 8 lblk 100 len 1 flags
    jbd2/sda16-8-587   [000] ...1  6889.063748: ext4_ext_show_extent: dev 259,0 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-587   [000] ...1  6889.063771: ext4_ext_map_blocks_exit: dev 259,0 ino 8 flags  lblk 100 pblk 623 len 1 mflags M ret 1
    jbd2/sda16-8-587   [000] ...1  6889.063785: ext4_es_insert_extent: dev 259,0 ino 8 es [100/1) mapped 623 status W
    jbd2/sda16-8-587   [000] ...1  6889.063985: ext4_es_lookup_extent_enter: dev 259,0 ino 8 lblk 101
    jbd2/sda16-8-587   [000] ...1  6889.063991: ext4_es_lookup_extent_exit: dev 259,0 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-587   [000] ...1  6889.063995: ext4_ext_map_blocks_enter: dev 259,0 ino 8 lblk 101 len 1 flags
    jbd2/sda16-8-587   [000] ...1  6889.064001: ext4_ext_show_extent: dev 259,0 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-587   [000] ...1  6889.064006: ext4_ext_map_blocks_exit: dev 259,0 ino 8 flags  lblk 101 pblk 624 len 1 mflags M ret 1
    jbd2/sda16-8-587   [000] ...1  6889.064011: ext4_es_insert_extent: dev 259,0 ino 8 es [101/1) mapped 624 status W
    jbd2/sda16-8-587   [000] ...1  6889.064048: ext4_es_lookup_extent_enter: dev 259,0 ino 8 lblk 102
    jbd2/sda16-8-587   [000] ...1  6889.064052: ext4_es_lookup_extent_exit: dev 259,0 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-587   [000] ...1  6889.064057: ext4_ext_map_blocks_enter: dev 259,0 ino 8 lblk 102 len 1 flags
    jbd2/sda16-8-587   [000] ...1  6889.064063: ext4_ext_show_extent: dev 259,0 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-587   [000] ...1  6889.064068: ext4_ext_map_blocks_exit: dev 259,0 ino 8 flags  lblk 102 pblk 625 len 1 mflags M ret 1
    jbd2/sda16-8-587   [000] ...1  6889.064072: ext4_es_insert_extent: dev 259,0 ino 8 es [102/1) mapped 625 status W
    jbd2/sda16-8-587   [000] ...1  6889.064091: ext4_es_lookup_extent_enter: dev 259,0 ino 8 lblk 103
    jbd2/sda16-8-587   [000] ...1  6889.064096: ext4_es_lookup_extent_exit: dev 259,0 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-587   [000] ...1  6889.064100: ext4_ext_map_blocks_enter: dev 259,0 ino 8 lblk 103 len 1 flags
    jbd2/sda16-8-587   [000] ...1  6889.064105: ext4_ext_show_extent: dev 259,0 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-587   [000] ...1  6889.064110: ext4_ext_map_blocks_exit: dev 259,0 ino 8 flags  lblk 103 pblk 626 len 1 mflags M ret 1
    jbd2/sda16-8-587   [000] ...1  6889.064114: ext4_es_insert_extent: dev 259,0 ino 8 es [103/1) mapped 626 status W
    jbd2/sda16-8-587   [000] ...1  6889.064131: ext4_es_lookup_extent_enter: dev 259,0 ino 8 lblk 104
    jbd2/sda16-8-587   [000] ...1  6889.064135: ext4_es_lookup_extent_exit: dev 259,0 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-587   [000] ...1  6889.064140: ext4_ext_map_blocks_enter: dev 259,0 ino 8 lblk 104 len 1 flags
    jbd2/sda16-8-587   [000] ...1  6889.064145: ext4_ext_show_extent: dev 259,0 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-587   [000] ...1  6889.064150: ext4_ext_map_blocks_exit: dev 259,0 ino 8 flags  lblk 104 pblk 627 len 1 mflags M ret 1
    jbd2/sda16-8-587   [000] ...1  6889.064154: ext4_es_insert_extent: dev 259,0 ino 8 es [104/1) mapped 627 status W
    jbd2/sda16-8-587   [000] ...1  6889.064171: ext4_es_lookup_extent_enter: dev 259,0 ino 8 lblk 105
    jbd2/sda16-8-587   [000] ...1  6889.064175: ext4_es_lookup_extent_exit: dev 259,0 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-587   [000] ...1  6889.064179: ext4_ext_map_blocks_enter: dev 259,0 ino 8 lblk 105 len 1 flags
    jbd2/sda16-8-587   [000] ...1  6889.064184: ext4_ext_show_extent: dev 259,0 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-587   [000] ...1  6889.064190: ext4_ext_map_blocks_exit: dev 259,0 ino 8 flags  lblk 105 pblk 628 len 1 mflags M ret 1
    jbd2/sda16-8-587   [000] ...1  6889.064194: ext4_es_insert_extent: dev 259,0 ino 8 es [105/1) mapped 628 status W
    jbd2/sda16-8-587   [000] ...1  6889.064213: ext4_es_lookup_extent_enter: dev 259,0 ino 8 lblk 106
    jbd2/sda16-8-587   [000] ...1  6889.064216: ext4_es_lookup_extent_exit: dev 259,0 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-587   [000] ...1  6889.064221: ext4_ext_map_blocks_enter: dev 259,0 ino 8 lblk 106 len 1 flags
    jbd2/sda16-8-587   [000] ...1  6889.064226: ext4_ext_show_extent: dev 259,0 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-587   [000] ...1  6889.064231: ext4_ext_map_blocks_exit: dev 259,0 ino 8 flags  lblk 106 pblk 629 len 1 mflags M ret 1
    jbd2/sda16-8-587   [000] ...1  6889.064235: ext4_es_insert_extent: dev 259,0 ino 8 es [106/1) mapped 629 status W
    jbd2/sda16-8-587   [000] ...1  6889.078612: ext4_es_lookup_extent_enter: dev 259,0 ino 8 lblk 107
    jbd2/sda16-8-587   [000] ...1  6889.078692: ext4_es_lookup_extent_exit: dev 259,0 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-587   [000] ...1  6889.078701: ext4_ext_map_blocks_enter: dev 259,0 ino 8 lblk 107 len 1 flags
    jbd2/sda16-8-587   [000] ...1  6889.078710: ext4_ext_show_extent: dev 259,0 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-587   [000] ...1  6889.078716: ext4_ext_map_blocks_exit: dev 259,0 ino 8 flags  lblk 107 pblk 630 len 1 mflags M ret 1
    jbd2/sda16-8-587   [000] ...1  6889.078723: ext4_es_insert_extent: dev 259,0 ino 8 es [107/1) mapped 630 status W
    jbd2/sda16-8-587   [000] d..4  6889.082769: writeback_dirty_page: bdi 8:0: ino=0 index=9
    jbd2/sda16-8-587   [000] ...3  6889.082873: writeback_mark_inode_dirty: bdi 8:0: ino=0 state= flags=I_DIRTY_PAGES
    jbd2/sda16-8-587   [000] ...3  6889.082901: writeback_dirty_inode_enqueue: dev 0,3 ino 0 dirtied 4295626204 state I_DIRTY_PAGES mode 060000
    jbd2/sda16-8-587   [000] d..4  6889.082929: writeback_dirty_page: bdi 8:0: ino=0 index=1549
    jbd2/sda16-8-587   [000] ...3  6889.082937: writeback_mark_inode_dirty: bdi 8:0: ino=0 state=I_DIRTY_PAGES flags=I_DIRTY_PAGES
    jbd2/sda16-8-587   [000] d..4  6889.082949: writeback_dirty_page: bdi 8:0: ino=0 index=11
    jbd2/sda16-8-587   [000] ...3  6889.082955: writeback_mark_inode_dirty: bdi 8:0: ino=0 state=I_DIRTY_PAGES flags=I_DIRTY_PAGES
    jbd2/sda16-8-587   [000] d..4  6889.082966: writeback_dirty_page: bdi 8:0: ino=0 index=12
    jbd2/sda16-8-587   [000] ...3  6889.082972: writeback_mark_inode_dirty: bdi 8:0: ino=0 state=I_DIRTY_PAGES flags=I_DIRTY_PAGES
    jbd2/sda16-8-587   [000] d..4  6889.082980: writeback_dirty_page: bdi 8:0: ino=0 index=1
    jbd2/sda16-8-587   [000] ...3  6889.082987: writeback_mark_inode_dirty: bdi 8:0: ino=0 state=I_DIRTY_PAGES flags=I_DIRTY_PAGES
    jbd2/sda16-8-587   [000] d..4  6889.082996: writeback_dirty_page: bdi 8:0: ino=0 index=10
    jbd2/sda16-8-587   [000] ...3  6889.083001: writeback_mark_inode_dirty: bdi 8:0: ino=0 state=I_DIRTY_PAGES flags=I_DIRTY_PAGES
```

## open-write-O_SYNC - nobarrier

```
open:
  test-open-writ-4922  [000] ...1  1425.147209: ext4_request_inode: dev 259,13 dir 2 mode 0100000
  test-open-writ-4922  [000] ...1  1425.147433: ext4_journal_start: dev 259,13 blocks, 35 rsv_blocks, 0 caller __ext4_new_inode+0x750/0x1338
  test-open-writ-4922  [000] ...1  1425.147470: jbd2_handle_start: dev 259,13 tid 68 type 4 line_no 2460 requested_blocks 35
  test-open-writ-4922  [000] ...1  1425.147634: ext4_mark_inode_dirty: dev 259,13 ino 14 caller ext4_ext_tree_init+0x24/0x30
  test-open-writ-4922  [000] ...1  1425.147638: ext4_mark_inode_dirty: dev 259,13 ino 14 caller __ext4_new_inode+0x1108/0x1338
  test-open-writ-4922  [000] ...1  1425.147644: ext4_allocate_inode: dev 259,13 ino 14 dir 2 mode 0100000
  test-open-writ-4922  [000] ...1  1425.147662: ext4_es_lookup_extent_enter: dev 259,13 ino 2 lblk 0
  test-open-writ-4922  [000] ...1  1425.147670: ext4_es_lookup_extent_exit: dev 259,13 ino 2 found 1 [0/1) 1549 W0x10
  test-open-writ-4922  [000] ...1  1425.147699: ext4_mark_inode_dirty: dev 259,13 ino 2 caller add_dirent_to_buf+0x170/0x1dc
  test-open-writ-4922  [000] ...1  1425.147709: ext4_mark_inode_dirty: dev 259,13 ino 14 caller ext4_add_nondir+0x34/0x7c
  test-open-writ-4922  [000] ...1  1425.147730: jbd2_handle_stats: dev 259,13 tid 68 type 4 line_no 2460 interval 0 sync 0 requested_blocks 35 dirtied_blocks 4

write:
  test-open-writ-4922  [000] ...1  1425.147815: ext4_da_write_begin: dev 259,13 ino 14 pos 0 len 12 flags 0
  test-open-writ-4922  [000] ...1  1425.147838: ext4_journal_start: dev 259,13 blocks, 1 rsv_blocks, 0 caller ext4_da_write_begin+0x1d4/0x304
  test-open-writ-4922  [000] ...1  1425.147840: jbd2_handle_start: dev 259,13 tid 68 type 2 line_no 2765 requested_blocks 1
  test-open-writ-4922  [000] ...1  1425.147854: ext4_es_lookup_extent_enter: dev 259,13 ino 14 lblk 0
  test-open-writ-4922  [000] ...1  1425.147854: ext4_es_lookup_extent_exit: dev 259,13 ino 14 found 0 [0/0) 0
  test-open-writ-4922  [000] ...1  1425.147862: ext4_ext_map_blocks_enter: dev 259,13 ino 14 lblk 0 len 1 flags
  test-open-writ-4922  [000] ...1  1425.147874: ext4_es_find_delayed_extent_range_enter: dev 259,13 ino 14 lblk 0
  test-open-writ-4922  [000] ...1  1425.147881: ext4_es_find_delayed_extent_range_exit: dev 259,13 ino 14 es [0/0) mapped 0 status
  test-open-writ-4922  [000] ...1  1425.147887: ext4_es_insert_extent: dev 259,13 ino 14 es [0/4294967295) mapped 576460752303423487 status H
  test-open-writ-4922  [000] ...1  1425.147909: ext4_ext_map_blocks_exit: dev 259,13 ino 14 flags  lblk 0 pblk 18446743800474598096 len 1 mflags  ret 0
  test-open-writ-4922  [000] ...2  1425.147919: ext4_da_reserve_space: dev 259,13 ino 14 mode 0100000 i_blocks 0 reserved_data_blocks 1 reserved_meta_blocks 0
  test-open-writ-4922  [000] ...1  1425.147922: ext4_es_insert_extent: dev 259,13 ino 14 es [0/1) mapped 576460752303423487 status D
  test-open-writ-4922  [000] ...1  1425.147951: ext4_da_write_end: dev 259,13 ino 14 pos 0 len 12 copied 12
  test-open-writ-4922  [000] ...1  1425.147981: ext4_journal_start: dev 259,13 blocks, 2 rsv_blocks, 0 caller ext4_dirty_inode+0x30/0x68
  test-open-writ-4922  [000] ...1  1425.147982: ext4_mark_inode_dirty: dev 259,13 ino 14 caller ext4_dirty_inode+0x44/0x68
  test-open-writ-4922  [000] ...1  1425.147984: jbd2_handle_stats: dev 259,13 tid 68 type 2 line_no 2765 interval 0 sync 0 requested_blocks 1 dirtied_blocks 0

O_SYNC:
  test-open-writ-4922  [000] ...1  1425.148001: ext4_sync_file_enter: dev 259,13 ino 14 parent 2 datasync 0
  test-open-writ-4922  [000] ...1  1425.148010: ext4_writepages: dev 259,13 ino 14 nr_to_write 9223372036854775807 pages_skipped 0 range_start 0 range_end 11 sync_mode 1 for_kupdate 0 range_cyclic 0 writeback_index 0
  test-open-writ-4922  [000] ...1  1425.148029: ext4_journal_start: dev 259,13 blocks, 8 rsv_blocks, 0 caller ext4_writepages+0x524/0xd24
  test-open-writ-4922  [000] ...1  1425.148031: jbd2_handle_start: dev 259,13 tid 68 type 2 line_no 2573 requested_blocks 8
  test-open-writ-4922  [000] ...1  1425.148036: ext4_da_write_pages: dev 259,13 ino 14 first_page 0 nr_to_write 9223372036854775807 sync_mode 1
  test-open-writ-4922  [000] ...1  1425.148055: ext4_da_write_pages_extent: dev 259,13 ino 14 lblk 0 len 1 flags 0x200
  test-open-writ-4922  [000] ...1  1425.148056: ext4_es_lookup_extent_enter: dev 259,13 ino 14 lblk 0
  test-open-writ-4922  [000] ...1  1425.148056: ext4_es_lookup_extent_exit: dev 259,13 ino 14 found 1 [0/1) 576460752303423487 D0x10
  test-open-writ-4922  [000] ...1  1425.148058: ext4_ext_map_blocks_enter: dev 259,13 ino 14 lblk 0 len 1 flags CREATE|DELALLOC|METADATA_NOFAIL

  test-open-writ-4922  [000] ...1  1425.148076: ext4_request_blocks: dev 259,13 ino 14 flags HINT_DATA|DELALLOC_RESV|USE_RESV len 1 lblk 0 goal 0 lleft 0 lright 0 pleft 0 pright 0
  test-open-writ-4922  [000] ...1  1425.148135: ext4_mb_new_group_pa: dev 259,13 ino 14 pstart 2560 len 512 lstart 2560
  test-open-writ-4922  [000] ...1  1425.148170: ext4_mballoc_alloc: dev 259,13 inode 14 orig 0/0/1@0 goal 0/0/512@0 result 0/2560/512@0 blks 1 grps 1 cr 0 flags HINT_DATA|HINT_GRP_ALLOC|DELALLOC_RESV|USE_RESV tail 0 broken 0
  test-open-writ-4922  [000] ...1  1425.148176: ext4_allocate_blocks: dev 259,13 ino 14 flags HINT_DATA|DELALLOC_RESV|USE_RESV len 1 block 2560 lblk 0 goal 0 lleft 0 lright 0 pleft 0 pright 0

  test-open-writ-4922  [000] ...1  1425.148192: ext4_mark_inode_dirty: dev 259,13 ino 14 caller __ext4_ext_dirty+0x6c/0x78
  test-open-writ-4922  [000] ...1  1425.148205: ext4_get_reserved_cluster_alloc: dev 259,13 ino 14 lblk 0 len 1
  test-open-writ-4922  [000] ...2  1425.148210: ext4_da_update_reserve_space: dev 259,13 ino 14 mode 0100000 i_blocks 0 used_blocks 1 reserved_data_blocks 1 reserved_meta_blocks 0 allocated_meta_blocks 0 quota_claim 1
  test-open-writ-4922  [000] ...1  1425.148216: ext4_journal_start: dev 259,13 blocks, 2 rsv_blocks, 0 caller ext4_dirty_inode+0x30/0x68
  test-open-writ-4922  [000] ...1  1425.148216: ext4_mark_inode_dirty: dev 259,13 ino 14 caller ext4_dirty_inode+0x44/0x68
  test-open-writ-4922  [000] ...1  1425.148219: ext4_ext_map_blocks_exit: dev 259,13 ino 14 flags CREATE|DELALLOC|METADATA_NOFAIL lblk 0 pblk 2560 len 1 mflags NM ret 1
  test-open-writ-4922  [000] ...1  1425.148222: ext4_es_insert_extent: dev 259,13 ino 14 es [0/1) mapped 2560 status W
  test-open-writ-4922  [000] ...1  1425.148271: ext4_mark_inode_dirty: dev 259,13 ino 14 caller ext4_writepages+0xc1c/0xd24
  test-open-writ-4922  [000] ...1  1425.148274: jbd2_handle_stats: dev 259,13 tid 68 type 2 line_no 2573 interval 0 sync 0 requested_blocks 8 dirtied_blocks 1
  test-open-writ-4922  [000] .n.1  1425.148651: ext4_writepages_result: dev 259,13 ino 14 ret 0 pages_written 1 pages_skipped 0 sync_mode 1 writeback_index 0

jbd2:
    jbd2/sda16-8-582   [001] ...1  1425.156069: jbd2_start_commit: dev 259,13 transaction 68 sync 0
    jbd2/sda16-8-582   [001] ...2  1425.156093: jbd2_commit_locking: dev 259,13 transaction 68 sync 0
    jbd2/sda16-8-582   [001] ...3  1425.156101: jbd2_checkpoint_stats: dev 259,13 tid 66 chp_time 0 forced_to_close 0 written 0 dropped 5
    jbd2/sda16-8-582   [001] ...3  1425.156107: jbd2_drop_transaction: dev 259,13 transaction 66 sync 0
    jbd2/sda16-8-582   [001] ...2  1425.156116: jbd2_commit_flushing: dev 259,13 transaction 68 sync 0
    jbd2/sda16-8-582   [001] ...1  1425.156121: jbd2_commit_logging: dev 259,13 transaction 68 sync 0
    jbd2/sda16-8-582   [001] ...1  1425.156128: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 61
    jbd2/sda16-8-582   [001] ...1  1425.156131: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [001] ...1  1425.156135: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 61 len 1 flags
    jbd2/sda16-8-582   [001] ...1  1425.156138: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [001] ...1  1425.156140: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 61 pblk 584 len 1 mflags M ret 1
    jbd2/sda16-8-582   [001] ...1  1425.156141: ext4_es_insert_extent: dev 259,13 ino 8 es [61/1) mapped 584 status W
    jbd2/sda16-8-582   [001] ...1  1425.156170: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 62
    jbd2/sda16-8-582   [001] ...1  1425.156171: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [001] ...1  1425.156172: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 62 len 1 flags
    jbd2/sda16-8-582   [001] ...1  1425.156174: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [001] ...1  1425.156175: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 62 pblk 585 len 1 mflags M ret 1
    jbd2/sda16-8-582   [001] ...1  1425.156175: ext4_es_insert_extent: dev 259,13 ino 8 es [62/1) mapped 585 status W
    jbd2/sda16-8-582   [001] ...1  1425.156184: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 63
    jbd2/sda16-8-582   [001] ...1  1425.156184: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [001] ...1  1425.156185: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 63 len 1 flags
    jbd2/sda16-8-582   [001] ...1  1425.156186: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [001] ...1  1425.156187: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 63 pblk 586 len 1 mflags M ret 1
    jbd2/sda16-8-582   [001] ...1  1425.156189: ext4_es_insert_extent: dev 259,13 ino 8 es [63/1) mapped 586 status W
    jbd2/sda16-8-582   [001] ...1  1425.156192: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 64
    jbd2/sda16-8-582   [001] ...1  1425.156193: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [001] ...1  1425.156194: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 64 len 1 flags
    jbd2/sda16-8-582   [001] ...1  1425.156195: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [001] ...1  1425.156195: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 64 pblk 587 len 1 mflags M ret 1
    jbd2/sda16-8-582   [001] ...1  1425.156196: ext4_es_insert_extent: dev 259,13 ino 8 es [64/1) mapped 587 status W
    jbd2/sda16-8-582   [001] ...1  1425.156199: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 65
    jbd2/sda16-8-582   [001] ...1  1425.156200: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [001] ...1  1425.156201: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 65 len 1 flags
    jbd2/sda16-8-582   [001] ...1  1425.156201: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [001] ...1  1425.156202: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 65 pblk 588 len 1 mflags M ret 1
    jbd2/sda16-8-582   [001] ...1  1425.156203: ext4_es_insert_extent: dev 259,13 ino 8 es [65/1) mapped 588 status W
    jbd2/sda16-8-582   [001] ...1  1425.156206: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 66
    jbd2/sda16-8-582   [001] ...1  1425.156207: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [001] ...1  1425.156207: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 66 len 1 flags
    jbd2/sda16-8-582   [001] ...1  1425.156208: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [001] ...1  1425.156209: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 66 pblk 589 len 1 mflags M ret 1
    jbd2/sda16-8-582   [001] ...1  1425.156209: ext4_es_insert_extent: dev 259,13 ino 8 es [66/1) mapped 589 status W
    jbd2/sda16-8-582   [001] ...1  1425.156495: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 67
    jbd2/sda16-8-582   [001] ...1  1425.156497: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [001] ...1  1425.156498: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 67 len 1 flags
    jbd2/sda16-8-582   [001] ...1  1425.156500: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [001] ...1  1425.156501: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 67 pblk 590 len 1 mflags M ret 1
    jbd2/sda16-8-582   [001] ...1  1425.156502: ext4_es_insert_extent: dev 259,13 ino 8 es [67/1) mapped 590 status W
    jbd2/sda16-8-582   [001] ...3  1425.157280: jbd2_checkpoint_stats: dev 259,13 tid 67 chp_time 0 forced_to_close 0 written 0 dropped 5
    jbd2/sda16-8-582   [001] ...3  1425.157297: jbd2_drop_transaction: dev 259,13 transaction 67 sync 0
    jbd2/sda16-8-582   [001] ...2  1425.157304: jbd2_run_stats: dev 259,13 tid 68 wait 0 request_delay 0 running 10 locked 0 flushing 0 logging 0 handle_count 3 blocks 5 blocks_logged 6
    jbd2/sda16-8-582   [001] ...1  1425.157307: jbd2_end_commit: dev 259,13 transaction 68 sync 0 head 60
  test-open-writ-4922  [000] ...1  1425.157331: ext4_sync_file_exit: dev 259,13 ino 14 ret 0
  test-open-writ-4922  [000] ...1  1425.157692: ext4_discard_preallocations: dev 259,13 ino 14
   kworker/u16:4-352   [003] ...1  1427.254685: ext4_writepages: dev 259,13 ino 14 nr_to_write 13312 pages_skipped 0 range_start 0 range_end 9223372036854775807 sync_mode 0 for_kupdate 1 range_cyclic 1 writeback_index 0
   kworker/u16:4-352   [003] ...1  1427.254866: ext4_writepages_result: dev 259,13 ino 14 ret 0 pages_written 0 pages_skipped 0 sync_mode 0 writeback_index 0

```

## open-write-fsync - nobarrier

```
open:
  test-open-writ-4930  [001] ...1  1495.732428: ext4_request_inode: dev 259,13 dir 2 mode 0105120
  test-open-writ-4930  [001] ...1  1495.732608: ext4_journal_start: dev 259,13 blocks, 35 rsv_blocks, 0 caller __ext4_new_inode+0x750/0x1338
  test-open-writ-4930  [001] ...1  1495.732644: jbd2_handle_start: dev 259,13 tid 69 type 4 line_no 2460 requested_blocks 35
  test-open-writ-4930  [001] ...1  1495.732810: ext4_mark_inode_dirty: dev 259,13 ino 16 caller ext4_ext_tree_init+0x24/0x30
  test-open-writ-4930  [001] ...1  1495.732815: ext4_mark_inode_dirty: dev 259,13 ino 16 caller __ext4_new_inode+0x1108/0x1338
  test-open-writ-4930  [001] ...1  1495.732821: ext4_allocate_inode: dev 259,13 ino 16 dir 2 mode 0105120
  test-open-writ-4930  [001] ...1  1495.732842: ext4_es_lookup_extent_enter: dev 259,13 ino 2 lblk 0
  test-open-writ-4930  [001] ...1  1495.732849: ext4_es_lookup_extent_exit: dev 259,13 ino 2 found 1 [0/1) 1549 W0x10
  test-open-writ-4930  [001] ...1  1495.732881: ext4_mark_inode_dirty: dev 259,13 ino 2 caller add_dirent_to_buf+0x170/0x1dc
  test-open-writ-4930  [001] ...1  1495.732890: ext4_mark_inode_dirty: dev 259,13 ino 16 caller ext4_add_nondir+0x34/0x7c
  test-open-writ-4930  [001] ...1  1495.732915: jbd2_handle_stats: dev 259,13 tid 69 type 4 line_no 2460 interval 0 sync 0 requested_blocks 35 dirtied_blocks 4

write:
  test-open-writ-4930  [001] ...1  1495.733006: ext4_da_write_begin: dev 259,13 ino 16 pos 0 len 12 flags 0
  test-open-writ-4930  [001] ...1  1495.733036: ext4_journal_start: dev 259,13 blocks, 1 rsv_blocks, 0 caller ext4_da_write_begin+0x1d4/0x304
  test-open-writ-4930  [001] ...1  1495.733038: jbd2_handle_start: dev 259,13 tid 69 type 2 line_no 2765 requested_blocks 1
  test-open-writ-4930  [001] ...1  1495.733053: ext4_es_lookup_extent_enter: dev 259,13 ino 16 lblk 0
  test-open-writ-4930  [001] ...1  1495.733054: ext4_es_lookup_extent_exit: dev 259,13 ino 16 found 0 [0/0) 0
  test-open-writ-4930  [001] ...1  1495.733063: ext4_ext_map_blocks_enter: dev 259,13 ino 16 lblk 0 len 1 flags
  test-open-writ-4930  [001] ...1  1495.733076: ext4_es_find_delayed_extent_range_enter: dev 259,13 ino 16 lblk 0
  test-open-writ-4930  [001] ...1  1495.733083: ext4_es_find_delayed_extent_range_exit: dev 259,13 ino 16 es [0/0) mapped 0 status
  test-open-writ-4930  [001] ...1  1495.733090: ext4_es_insert_extent: dev 259,13 ino 16 es [0/4294967295) mapped 576460752303423487 status H
  test-open-writ-4930  [001] ...1  1495.733109: ext4_ext_map_blocks_exit: dev 259,13 ino 16 flags  lblk 0 pblk 18446743805028793040 len 1 mflags  ret 0
  test-open-writ-4930  [001] ...2  1495.733121: ext4_da_reserve_space: dev 259,13 ino 16 mode 0105120 i_blocks 0 reserved_data_blocks 1 reserved_meta_blocks 0
  test-open-writ-4930  [001] ...1  1495.733123: ext4_es_insert_extent: dev 259,13 ino 16 es [0/1) mapped 576460752303423487 status D
  test-open-writ-4930  [001] ...1  1495.733148: ext4_da_write_end: dev 259,13 ino 16 pos 0 len 12 copied 12
  test-open-writ-4930  [001] ...1  1495.733183: ext4_journal_start: dev 259,13 blocks, 2 rsv_blocks, 0 caller ext4_dirty_inode+0x30/0x68
  test-open-writ-4930  [001] ...1  1495.733186: ext4_mark_inode_dirty: dev 259,13 ino 16 caller ext4_dirty_inode+0x44/0x68
  test-open-writ-4930  [001] ...1  1495.733190: jbd2_handle_stats: dev 259,13 tid 69 type 2 line_no 2765 interval 0 sync 0 requested_blocks 1 dirtied_blocks 0

fsync:
  test-open-writ-4930  [001] ...1  1495.733213: ext4_sync_file_enter: dev 259,13 ino 16 parent 2 datasync 0
  test-open-writ-4930  [001] ...1  1495.733222: ext4_writepages: dev 259,13 ino 16 nr_to_write 9223372036854775807 pages_skipped 0 range_start 0 range_end 9223372036854775807 sync_mode 1 for_kupdate 0 range_cyclic 0 writeback_index 0
  test-open-writ-4930  [001] ...1  1495.733240: ext4_journal_start: dev 259,13 blocks, 8 rsv_blocks, 0 caller ext4_writepages+0x524/0xd24
  test-open-writ-4930  [001] ...1  1495.733242: jbd2_handle_start: dev 259,13 tid 69 type 2 line_no 2573 requested_blocks 8
  test-open-writ-4930  [001] ...1  1495.733248: ext4_da_write_pages: dev 259,13 ino 16 first_page 0 nr_to_write 9223372036854775807 sync_mode 1
  test-open-writ-4930  [001] ...1  1495.733267: ext4_da_write_pages_extent: dev 259,13 ino 16 lblk 0 len 1 flags 0x200
  test-open-writ-4930  [001] ...1  1495.733269: ext4_es_lookup_extent_enter: dev 259,13 ino 16 lblk 0
  test-open-writ-4930  [001] ...1  1495.733271: ext4_es_lookup_extent_exit: dev 259,13 ino 16 found 1 [0/1) 576460752303423487 D0x10
  test-open-writ-4930  [001] ...1  1495.733274: ext4_ext_map_blocks_enter: dev 259,13 ino 16 lblk 0 len 1 flags CREATE|DELALLOC|METADATA_NOFAIL

  test-open-writ-4930  [001] ...1  1495.733294: ext4_request_blocks: dev 259,13 ino 16 flags HINT_DATA|DELALLOC_RESV|USE_RESV len 1 lblk 0 goal 0 lleft 0 lright 0 pleft 0 pright 0

  test-open-writ-4930  [001] ...1  1495.733356: ext4_mb_new_group_pa: dev 259,13 ino 16 pstart 3072 len 512 lstart 3072
  test-open-writ-4930  [001] ...1  1495.733397: ext4_mballoc_alloc: dev 259,13 inode 16 orig 0/0/1@0 goal 0/0/512@0 result 0/3072/512@0 blks 1 grps 1 cr 0 flags HINT_DATA|HINT_GRP_ALLOC|DELALLOC_RESV|USE_RESV tail 512 broken 1024

  test-open-writ-4930  [001] ...1  1495.733404: ext4_allocate_blocks: dev 259,13 ino 16 flags HINT_DATA|DELALLOC_RESV|USE_RESV len 1 block 3072 lblk 0 goal 0 lleft 0 lright 0 pleft 0 pright 0

  test-open-writ-4930  [001] ...1  1495.733421: ext4_mark_inode_dirty: dev 259,13 ino 16 caller __ext4_ext_dirty+0x6c/0x78
  test-open-writ-4930  [001] ...1  1495.733433: ext4_get_reserved_cluster_alloc: dev 259,13 ino 16 lblk 0 len 1
  test-open-writ-4930  [001] ...2  1495.733440: ext4_da_update_reserve_space: dev 259,13 ino 16 mode 0105120 i_blocks 0 used_blocks 1 reserved_data_blocks 1 reserved_meta_blocks 0 allocated_meta_blocks 0 quota_claim 1
  test-open-writ-4930  [001] ...1  1495.733445: ext4_journal_start: dev 259,13 blocks, 2 rsv_blocks, 0 caller ext4_dirty_inode+0x30/0x68
  test-open-writ-4930  [001] ...1  1495.733446: ext4_mark_inode_dirty: dev 259,13 ino 16 caller ext4_dirty_inode+0x44/0x68
  test-open-writ-4930  [001] ...1  1495.733450: ext4_ext_map_blocks_exit: dev 259,13 ino 16 flags CREATE|DELALLOC|METADATA_NOFAIL lblk 0 pblk 3072 len 1 mflags NM ret 1
  test-open-writ-4930  [001] ...1  1495.733454: ext4_es_insert_extent: dev 259,13 ino 16 es [0/1) mapped 3072 status W
  test-open-writ-4930  [001] ...1  1495.733508: ext4_mark_inode_dirty: dev 259,13 ino 16 caller ext4_writepages+0xc1c/0xd24
  test-open-writ-4930  [001] ...1  1495.733512: jbd2_handle_stats: dev 259,13 tid 69 type 2 line_no 2573 interval 0 sync 0 requested_blocks 8 dirtied_blocks 1
  test-open-writ-4930  [001] ...1  1495.733603: ext4_journal_start: dev 259,13 blocks, 8 rsv_blocks, 0 caller ext4_writepages+0x524/0xd24
  test-open-writ-4930  [001] ...1  1495.733606: jbd2_handle_start: dev 259,13 tid 69 type 2 line_no 2573 requested_blocks 8
  test-open-writ-4930  [001] ...1  1495.733607: ext4_da_write_pages: dev 259,13 ino 16 first_page 1 nr_to_write 9223372036854775806 sync_mode 1
  test-open-writ-4930  [001] ...1  1495.733614: jbd2_handle_stats: dev 259,13 tid 69 type 2 line_no 2573 interval 0 sync 0 requested_blocks 8 dirtied_blocks 0
  test-open-writ-4930  [001] .n.1  1495.733841: ext4_writepages_result: dev 259,13 ino 16 ret 0 pages_written 1 pages_skipped 0 sync_mode 1 writeback_index 1

jbd2:
    jbd2/sda16-8-582   [000] ...1  1495.743853: jbd2_start_commit: dev 259,13 transaction 69 sync 0
    jbd2/sda16-8-582   [000] ...2  1495.743890: jbd2_commit_locking: dev 259,13 transaction 69 sync 0
    jbd2/sda16-8-582   [000] ...2  1495.743904: jbd2_commit_flushing: dev 259,13 transaction 69 sync 0
    jbd2/sda16-8-582   [000] ...1  1495.743918: jbd2_commit_logging: dev 259,13 transaction 69 sync 0

    jbd2/sda16-8-582   [000] ...1  1495.743927: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 68
    jbd2/sda16-8-582   [000] ...1  1495.743930: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [000] ...1  1495.743934: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 68 len 1 flags
    jbd2/sda16-8-582   [000] ...1  1495.743939: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [000] ...1  1495.743943: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 68 pblk 591 len 1 mflags M ret 1
    jbd2/sda16-8-582   [000] ...1  1495.743947: ext4_es_insert_extent: dev 259,13 ino 8 es [68/1) mapped 591 status W

    jbd2/sda16-8-582   [000] ...1  1495.743996: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 69
    jbd2/sda16-8-582   [000] ...1  1495.743997: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [000] ...1  1495.743998: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 69 len 1 flags
    jbd2/sda16-8-582   [000] ...1  1495.744000: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [000] ...1  1495.744001: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 69 pblk 592 len 1 mflags M ret 1
    jbd2/sda16-8-582   [000] ...1  1495.744003: ext4_es_insert_extent: dev 259,13 ino 8 es [69/1) mapped 592 status W

    jbd2/sda16-8-582   [000] ...1  1495.744015: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 70
    jbd2/sda16-8-582   [000] ...1  1495.744017: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [000] ...1  1495.744018: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 70 len 1 flags
    jbd2/sda16-8-582   [000] ...1  1495.744019: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [000] ...1  1495.744021: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 70 pblk 593 len 1 mflags M ret 1
    jbd2/sda16-8-582   [000] ...1  1495.744021: ext4_es_insert_extent: dev 259,13 ino 8 es [70/1) mapped 593 status W

    jbd2/sda16-8-582   [000] ...1  1495.744027: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 71
    jbd2/sda16-8-582   [000] ...1  1495.744028: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [000] ...1  1495.744030: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 71 len 1 flags
    jbd2/sda16-8-582   [000] ...1  1495.744031: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [000] ...1  1495.744032: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 71 pblk 594 len 1 mflags M ret 1
    jbd2/sda16-8-582   [000] ...1  1495.744033: ext4_es_insert_extent: dev 259,13 ino 8 es [71/1) mapped 594 status W

    jbd2/sda16-8-582   [000] ...1  1495.744038: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 72
    jbd2/sda16-8-582   [000] ...1  1495.744040: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [000] ...1  1495.744041: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 72 len 1 flags
    jbd2/sda16-8-582   [000] ...1  1495.744042: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [000] ...1  1495.744043: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 72 pblk 595 len 1 mflags M ret 1
    jbd2/sda16-8-582   [000] ...1  1495.744044: ext4_es_insert_extent: dev 259,13 ino 8 es [72/1) mapped 595 status W

    jbd2/sda16-8-582   [000] ...1  1495.744049: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 73
    jbd2/sda16-8-582   [000] ...1  1495.744050: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [000] ...1  1495.744051: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 73 len 1 flags
    jbd2/sda16-8-582   [000] ...1  1495.744053: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [000] ...1  1495.744054: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 73 pblk 596 len 1 mflags M ret 1
    jbd2/sda16-8-582   [000] ...1  1495.744055: ext4_es_insert_extent: dev 259,13 ino 8 es [73/1) mapped 596 status W

    jbd2/sda16-8-582   [000] ...1  1495.744465: ext4_es_lookup_extent_enter: dev 259,13 ino 8 lblk 74
    jbd2/sda16-8-582   [000] ...1  1495.744466: ext4_es_lookup_extent_exit: dev 259,13 ino 8 found 0 [0/0) 0
    jbd2/sda16-8-582   [000] ...1  1495.744468: ext4_ext_map_blocks_enter: dev 259,13 ino 8 lblk 74 len 1 flags
    jbd2/sda16-8-582   [000] ...1  1495.744469: ext4_ext_show_extent: dev 259,13 ino 8 lblk 0 pblk 523 len 1024
    jbd2/sda16-8-582   [000] ...1  1495.744471: ext4_ext_map_blocks_exit: dev 259,13 ino 8 flags  lblk 74 pblk 597 len 1 mflags M ret 1
    jbd2/sda16-8-582   [000] ...1  1495.744472: ext4_es_insert_extent: dev 259,13 ino 8 es [74/1) mapped 597 status W

    jbd2/sda16-8-582   [000] ...3  1495.745339: jbd2_checkpoint_stats: dev 259,13 tid 68 chp_time 0 forced_to_close 0 written 0 dropped 5
    jbd2/sda16-8-582   [000] ...3  1495.745345: jbd2_drop_transaction: dev 259,13 transaction 68 sync 0
    jbd2/sda16-8-582   [000] ...2  1495.745352: jbd2_run_stats: dev 259,13 tid 69 wait 0 request_delay 0 running 10 locked 0 flushing 0 logging 0 handle_count 4 blocks 5 blocks_logged 6
    jbd2/sda16-8-582   [000] ...1  1495.745357: jbd2_end_commit: dev 259,13 transaction 69 sync 0 head 60

  test-open-writ-4930  [001] ...1  1495.745396: ext4_sync_file_exit: dev 259,13 ino 16 ret 0
  test-open-writ-4930  [001] ...1  1495.746460: ext4_discard_preallocations: dev 259,13 ino 16

   kworker/u16:4-352   [001] ...1  1500.744633: ext4_writepages: dev 259,13 ino 16 nr_to_write 13312 pages_skipped 0 range_start 0 range_end 9223372036854775807 sync_mode 0 for_kupdate 1 range_cyclic 1 writeback_index 1
   kworker/u16:4-352   [001] ...1  1500.744799: ext4_writepages_result: dev 259,13 ino 16 ret 0 pages_written 0 pages_skipped 0 sync_mode 0 writeback_index 1
```