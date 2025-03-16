# Hey! I'm Filing Here

In this lab, I successfully implemented the ext2 filesystem (1MiB).

I will detail some notable features of the implementation.

*1. in `write_superblock`:*
    - I defined two additional macros to better fit the ext2 conventions I read about
    - I used the given macros, but left some values hardcoded
    - I defined the `EXT2_ERRORS_CONTINUE` macro as well, I figured that would add some clarity

*2. In `write_block_group_descriptor`*
    - I defined all of the necessary fields using the given macros
    - I defined my own macro `NUM_DIRS` to be 2, since we have both the root dir itself and lost + found

*3. In `write_block_bitmap`:*
    - I defined `INODE_TABLE_BLOCK_SIZE ((NUM_INODES * 128) / 1024)` and `INODE_TABLE_SPAN (INODE_TABLE_BLOCKNO + INODE_TABLE_BLOCK_SIZE - 1)` to serve as parameters to measure how many blocks the inode table occupies. It was given that it starts at 5, and for this lab the block size is 16--so this will mark blocks 5-20 for the inode table
    - I offset each of the blockno macros by one for indexing, and then used bit shifting and used bitwise OR to assign the bits for each block--dividing by 8 gives the index in map_value
    - I then used `memset` to set remaining entries in the filesystem to `0xFF` so that the system does not read blocks 1024-8192 as free, but rather reserved

*4. In `write_inode_bitmap`:*
    - I used incredibly similar logic to the block bitmap, see that section for details
    - One notable thing it that the first 11 inodes must be reserved (according to nongnu), so then I used a loop from `EXT2_ROOT_INO + 1` to the final inode at 11 `LOST_AND_FOUND_INO` to ensure that the first 11 inodes are resreved
    - I then used `memset` in the same way as the previous bitmap

*5. In `write_inode_table`:*
    - Pretty straightforward, I just followed the lead given by the lost and found inode while setting appropriate macros and conditions
    - I was sure to adjust the link count for each (root inode has three since it contains lost and found inode as well as itself twice)
    - Hello world inode has a size of 12 since that is exactly how many characters are in the message
    - From nongnu, since the symlink hello is shorter than 60 bytes, we store it within the inode itself using `memcpy`, (hello-world message is 11 in length)
*6,7. In `write_root_dir_block` and `write_lost_and_found_dir`:*
    - Also pretty straightforward, followed given example
    - To adhere to convention, included empty entry with the remaining bytes using `bytes_remaining` in both directory blocks
*8. In `write_hello_world_file_block`:*
    - Used a write call with error handling

## Building

```shell
>make #to compile
```

## Running

```shell
>./ext2-create                      #execute
>dumpe2fs cs111-base.img            #check correctness
>fsck.ext2 cs111-base.img           #check correctness
>mkdir mnt                          #create dir to mount fs on
>sudo -o loop cs111-base.img mnt    #mount the fs
>ls -ain mnt                        #look at contents! try other dir commands if u like
```


## Cleaning up

TODO
```shell
>sudo umount mnt       #unmount it
>rmdir mnt              #delete dir 
>make clean             #clean up binaries
```
