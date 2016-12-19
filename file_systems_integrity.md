# File Systems Integrity 

## FS Comparison 

|                             | FAT             | FFS                          | NTFS                     |
|-----------------------------|-----------------|------------------------------|--------------------------|
| Index Structure             | Linked List     | Tree (inodes)                | Tree (extents)           |
| Index Structure Granularity | Block           | Block                        | Extent                   |
| Free Space Management       | FAT array       | Bitmap                       | Bitmap                   |
| Locality Heuristics         | Defragmentation | Block groups, reserved space | Best Fit Defragmentation |

### FAT 

- file allocation table 
  - Late 70s
  - blocks allocated in a linked index structure 
- directories map file name to first block of file 
- stores both linked list of blocks belonging to files and the free blocks 
- Limitations:  
  + poor random access
  + poor locality 
  + limited file metadata and access control 
  
### FFS: Consitency Issues - Overview 

- Inodes: fixed size structure in cylinder groups 
- metadata updates must be synchronous operations 
- file system operations affect multiple metadata blocks 
  + write newly allocated inode to disk before its name is entered in a directory 
  + remove a directory name before the inode is deallocated 
  + deallocate an inode (mark as free in bitmap) before that file's data blocks are placed into the cylinder
  group free list 
  
### FFS Observation 1: Crash recovery 

- if the server crashes in between any of these synchronous operation, then the file system is in an inconsitent state
- Solutions: 
  + fsck - post-crash recovery process to scan file system structure and restore consitency 
    * All data blocks pointed to by inodes and indirect blocks must be marked allocated in the data bitmap
    * inode link count must match 
  + log updates to enable roll-back or roll-forward 

### Crash Consistency 

- What if only one write succeeds before a crash?
 1. If the second one succeeds 
  + no inode, no bitmap => as if the write did not occur 
  + FS not inconsitent, but data is lost 
2. Just second inode write succeeds 
  + no data block => will read garbage data from disk 
  + no bitmap entry, but inode has a pointer to second write => FS inconsistency 
3. Just second bitmap write succeeds 
  + bitmap says write is allocated, inode has no pointer to it 
    * => again, FS inconsitent + the write can never be used again 
    
### Otehr crash scenarios

- What if only two writes suceed before a crash 
  1. Only inode and bitmap writes suceed 
    + Inode and bitmap agree => FS metadata is consitent 
    + However, data block contains garbage 
  2. Only inode and datablock write suceeds 
    + inode points to correct data but clashes with bitmap => FS inconsistency 
  3. Only bitmap and datablock write suceeds 
    + inode and bitmap do not match 
    + even though datablock was written, no inode points to it 

### Solution: fsck 

- unix tool for finding inconsitencies and repairing them 
- cannot fix all porblems 
  + when DB is garbage - cannot know that's the case
  + only cares that FS metadata is consistent 
- What does it check 
  1. Superblock: sanity checks 
    + use another superblock copy if suspected corruption 
  2. Free blocks: scan inodes (including indirect blocks), build bitmap 
    + inodes and data bitmap inconsistency => resolve by trusting inodes 
    + ensure inodes in use are marked in inode bitmaps 
  3. Inode state: check inode fields for possible corruption 
    + e.g must have a valid mode field (file, dir, link etc) 
    + if cannot fix => remove inode and update inode bitmap 
  4. Inode links: verify links num for each inode 
    + traverse directory tree, compute expected links num, fix if needed 
    + if inode discovered, but no dir refers to it => move to "lost + found" 
  5. Duplicates: check if two different inodes refer to same block 
    + clear one if obviously bad, or give each inode its own copy of the block 
  6. Bad blocks: bad pointers (outside valid range) 
    + just remove the pointer from the inde or indirect block 
  7. Directory checks: integrity of directory structure 
    + make sure that "." and ".." are the first entries, each indoe in a directory entry is allocated, no
    directory is linked more than once 
    
### fsck limitations 

- cannot do anything about lost data 
- bigger problem: too slow 
  + scanning an entire disk can take hours 
  
### Alternative solution: Journaling 

- aka write ahead logging 
- basic idea:
  + when doing an update, before overwriting structures, first write down a little note elsewhere on disk, 
  saying what you plan to do 
- if a crash takes place during the actual write => go back to journal and retry the actual writes 
  + don't need to scan the entire disk, we know what to do 
  + can recover data as well 
- if a crash happens before journal write finishes, then it doesn't matter since the actual write has not happened at all,
so nothing is inconsistent

### Linux Ext3 File System
  + extends ext2 with journaling capabilities 
  
### What goes in that "note" 

- Transaction structure: 
  + starts with a transaction begin block, containing a transaction ID 
  + Followed by blocks with the content to be written 
    * Physical logging: log exact physical content 
    * Logical logging: log more compact logical representation 
  + ends with transaction end block, containt the corresponding TID 
  
- write TxEnd only when txbegin, inode and bitmap are written 
  + if crash happens, we know this is not a valid entry 
- now that journal entry is safe, write the actual data and metadata to their right locations on the FS (Checkpoint step)
- Mark transaction as free in journal (free step)

### Journalling Space Requirements 
