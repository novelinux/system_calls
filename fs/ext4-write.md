# Ext4 write

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

## write - test-open-write

### vfs_write

```
vfs_write
 |
__vfs_write
 |
new_sync_write
 |
ext4_file_write_iter
 |
 +-> __generic_file_write_iter
 |  |
 |  generic_perform_write
 |  |
 |  +-> ext4_da_write_begin
 |  |
 |  +-> ext4_da_write_end
 |
 +-> generic_write_sync
     |
     vfs_fsync_range -> file->f_op->fsync
```

### ext4_da_write_begin

```
ext4_da_write_begin { event: ext4_da_write_begin }
 |
 +-> ext4_journal_start -> __ext4_journal_start
 |                                  |
 |   __ext4_journal_start_sb { event: ext4_journal_start }
 |   |
 |   +-> jbd2__journal_start { event: jbd2_handle_start }
 |
 ext4_block_write_begin
 |
 ext4_get_block_write
 |
 ext4_da_map_blocks
 |
 +-> ext4_es_lookup_extent { event: ext4_es_lookup_extent_enter/ext4_es_lookup_extent_exit }
 |
 +-> ext4_ext_map_blocks { event: ext4_ext_map_blocks_enter }
 |   |
 |   +-> ext4_ext_put_gap_in_cache
 |   |   |
     |   +-> ext4_es_find_delayed_extent_range { event: ext4_es_find_delayed_extent_range_enter/exit }
 |   |   |
 |   |   +-> ext4_es_insert_extent { event: ext4_es_insert_extent }
 |   |
 |   +-> { event: ext4_ext_map_blocks_exit }
 |
 +-> ext4_da_reserve_space { event: ext4_da_reserve_space }
 |
 +-> ext4_es_insert_extent { event: ext4_es_insert_extent }
```

### ext4_da_write_end

```
ext4_da_write_end { event: ext4_da_write_end }
 |
 +-> generic_write_end
 |   |
 |   +-> **block_write_end**
 |   |
 |   +-> mark_inode_dirty
 |       |
 |   __mark_inode_dirty { event: writeback_mark_inode_dirty }
 |   |
 |   +-> { event: writeback_dirty_inode_start }
 |   |
 |   +-> sb->s_op->dirty_inode -> ext4_dirty_inode
 |   |   |
 |   |   ext4_journal_start
 |   |   |
 |   |   __ext4_journal_start
 |   |   |
 |   |   __ext4_journal_start_sb { event: ext4_journal_start }
 |   |
 |   +-> { event: writeback_dirty_inode }
 |
 +-> ext4_journal_stop
     |
     __ext4_journal_stop -> jbd2_journal_stop { event: jbd2_handle_stats }
```

### block_write_end

```
block_write_end
 |
__block_commit_write
 |
mark_buffer_dirty
 |
 +-> __set_page_dirty
 |   |
 |   account_page_dirtied { event: writeback_dirty_page }
 |
 +-> __mark_inode_dirty { event: writeback_mark_inode_dirty }
     |
     +-> { event: writeback_dirty_inode_enqueue }
```

## ext4_writepages

这个函数在带SYNC的写操作中(O_SYNC或fsync)操作一般由调用write操作的进程本身执行fsync trigger,
正常不带SYNC的写操作kworker来执行对应的writepages.

```
ext4_writepages
 |
 +-> { event: ext4_writepages }
 |
 +-> ext4_journal_start_with_reserve
 |   |
 |   __ext4_journal_start
 |   |
 |   __ext4_journal_start_sb { event: ext4_journal_start }
 |   |
 |   +-> jbd2__journal_start { event: jbd2_handle_start }
 |
 +-> { event: ext4_da_write_pages }
 |
 +-> mpage_prepare_extent_to_map
 |
 +-> mpage_map_and_submit_extent
 |   |
 |   +-> mpage_map_one_extent { event: ext4_da_write_pages_extent }
 |   |   |
 |   |   +-> **ext4_map_blocks**
 |   |
 |   +-> mpage_map_and_submit_buffers
 |   |
 |   +-> ext4_mark_inode_dirty { event: ext4_mark_inode_dirty }
 |
 +-> ext4_journal_stop -> __ext4_journal_stop
 |                                |
 |   jbd2_journal_stop { event: jbd2_handle_stats }
 |
 +-> { event: ext4_writepages_result }
```

### ext4_map_blocks

```
ext4_map_blocks
 |
 +-> ext4_es_lookup_extent { event: ext4_es_lookup_extent_enter/exit }
 |
 +-> ext4_ext_map_blocks
 |   |
 |   +-> { event: ext4_ext_map_blocks_enter }
 |   |
 |   +-> ext4_mb_new_blocks { event: ext4_request_blocks }
 |   |   |
 |   |   +-> ext4_mb_release_context - ext4_mb_collect_stats
 |   |   |                                       |
 |   |   |                     { event: ext4_mballoc_alloc }
 |   |   |
 |   |   +-> { event: ext4_allocate_blocks }
 |   |
 |   +-> ext4_ext_insert_extent
 |   |   |
 |   |   +-> ext4_ext_dirty -> __ext4_ext_dirty
 |   |                               |
 |   |   ext4_mark_inode_dirty { event: ext4_mark_inode_dirty }
 |   |
 |   +-> get_reserved_cluster_alloc { event: ext4_get_reserved_cluster_alloc }
 |   |
 |   +-> ext4_da_update_reserve_space { event: ext4_da_update_reserve_space }
 |   |   |
 |   |   +-> dquot_claim_block -> mark_inode_dirty_sync
 |   |   |                               |
 |   |   |   ext4_dirty_inode <- __mark_inode_dirty
 |   |   |   |
 |   |   |   +-> ext4_journal_start { event: ext4_journal_start }
 |   |   |   |
 |   |   |   +-> ext4_mark_inode_dirty { event: ext4_mark_inode_dirty }
 |   |   |
 |   |   +-> ext4_discard_preallocations { event: ext4_discard_preallocations }
 |   |
 |   +-> { event: ext4_ext_map_blocks_exit }
 |
 +-> ext4_es_insert_extent { event: ext4_es_insert_extent }
```

## kworker - process wb_workfn

https://github.com/novelinux/linux-4.x.y/tree/master/kernel/workqueue.c/README.md

```
ret_from_fork
 |
kthread
 |
worker_thread
 |
process_one_work
 |
wb_workfn
 |
wb_do_writeback
 |
wb_check_old_data_flush
 |
wb_writeback
 |
__writeback_inodes_wb
 |
writeback_sb_inodes
 |
__writeback_single_inode
 |
do_writepages
 |
ext4_writepages
```

## kworker - insert wb_workfn

```
ret_from_fork
 |
kthread
 |
worker_thread
 |
process_one_work
 |
async_run_entry_fn
 |
entry->func = ufshcd_async_scan
 |
ufshcd_probe_hba
 |
ufshcd_scsi_add_wlus
 |
__scsi_add_device
 |
scsi_probe_and_add_lun
 |
scsi_alloc_sdev
 |
scsi_alloc_queue
 |
__scsi_alloc_queue
 |
blk_init_queue
 |
blk_init_queue_node
 |
blk_alloc_queue_node
 |
bdi_init
 |
cgwb_bdi_init
 |
wb_init
 |
INIT_DELAYED_WORK(&wb->dwork, wb_workfn)
```

## kworker - insert async_run_entry_fn

```
ret_from_fork
 |
kernel_init
 |
kernel_init_freeable
 |
do_basic_setup
 |
do_initcalls
 |
do_initcall_level
 |
do_one_initcall
 |
module_platform_dirver(ufs_qcom_pltform)
 |
module_driver(ufs_qcom_pltform) = ufs_qcom_pltform_init
 |
platform_driver_register
 |
__platform_driver_register
 |
dirver_register
 |
bus_add_driver
 |
driver_attach
 |
bus_for_each_dev
 |
__driver_attach
 |
driver_probe_device
 |
really_probe
 |
dev->bus->probe = platform_drv_probe
 |
drv->probe = ufs_qcom_probe
 |
ufshcd_pltfrm_init
 |
ufshcd_init
 |
async_schedule(ufshcd_async_scan)
 |
__async_schedule
 |
 +-> INIT_WORK(entry->work, async_run_entry_fn)
 |
 +-> async_entry->func = ufshcd_async_scan
 |
 +-> queue_work(system_unbound_wq, &entry->work)
```

## kjournald2 - jbd2/sda16-8-578

```
ext4_fill_super
 |
ext4_load_journal
 |
jbd2_journal_load
 |
 +-> load_superblock
 |
 +-> jbd2_journal_create_slab
 |
 +-> journal_reset
     |
 jbd2_journal_start_thread
 |
 kthread_run( kjournald2 )
 |
 loop: jbd2_journal_commit_transaction
  |
  +-> { event: jbd2_start_commit }
  |
  +-> { event: jbd2_commit_locking }
  |
  +-> { event: jbd2_commit_flushing }
  |
  +-> { event: jbd2_commit_logging }
  |
  +-> journal_submit_commit_record
  |    |
  |   jbd2_journal_get_descriptor_buffer
  |    |
  |   jbd2_journal_next_log_block
  |    |
  |   jbd2_journal_bmap
  |    |
  |   bmap -> inode->i_mapping->a_ops->bmap
  |           |
  |   ext4_bmap
  |    |
  |   generic_block_bmap
  |    |
  |   ext4_get_block -> _ext4_get_block
  |                     |
  |   **ext4_map_blocks** [ loop 7 times ]
  |
  +->  __jbd2_journal_remove_checkpoint { event: jbd2_checkpoint_stats }
  |    |
  |    __jbd2_journal_drop_transaction { event: jbd2_drop_transaction }
  |
  +-> { event: jbd2_run_stats }
  |
  +-> { event: jbd2_end_commit }
```
