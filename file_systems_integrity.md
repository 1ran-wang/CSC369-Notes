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
  
- write TxEnd only when txbegin, inode and block are written 
  + if crash happens, we know this is not a valid entry 
- now that journal entry is safe, write the actual data and metadata to their right locations on the FS (Checkpoint step)
- Mark transaction as free in journal (free step)

### Metadata Journalling 
- recovery is much faster with journaling 
  + replay only a few transactions instead of checling the whole disk
- However normal operations are slower 
  + every update must write to the journal first, then do the update 
    * writing time is at least doubled 
  + journal writing may break sequential writing 
    * jump back and forth between writes to journal and writes to main region 
  + Metadata journaling is similar, except we only write FS metadata (no actual data) to the journal 

- to make sure inodes are not pointing to garbage, we write data before writing metadata to journal 
  1. Write data, wait until it completes
  2. Metadata journal write
  3. Metadata journal commit
  4. Checkpoint metadata
  5. Free

## Summary: Journaling 

- jorunaling ensures file system consistency 
- complexity is in the size of the journal, not the size of the disk 
- widely adopted in most modern file systems 

### Ext3 Final notes 

- lacks modern FS features (e.g extents) 
  + for recoverability, this may actually be an advantage 
  + FS metadata is in fixed, well know locations, and data structures have redundancy 
  + When faced with significant data corruotion, ext2/3 may be recoverable while a tree based file system 
  may not 


### Log-Structured File System (LSF) 
- different approach then FFS
  + memory is increasing => don't care about reads, most will hit mem
  + assume writes will pose the bigger I/O penalty 
  + treat storage as a circular log 

- Advantages
  + write throughput improved (batched into large sequential chunks) 
  + crash recovery - simpler 
- Disadvantages 
  + initial assumption may not hold => read much slower on HDDs 
  
- writes all file system data in a continuous log 
- use inodes and directories from FFs 
- segments: each has a summary block
- summary block contains pointer to next one 
- need a fresh segment => first clean an existing partially-used segment (garbage collection) 
- LFS not so easy
  + updates are sequential => inodes all over disk 
  + also inodes not static keep moving 

- needs an inode map to find inodes 
  + inode number is no longer a simple index from the start of an inode table as before 
  + must be persistent so must also be on disk 
    * on a fixed part of the disk 

- check point region (CR) 
  + pointers to the latest pieces of the inode map 
  + so find imap pieces by reading the CR 

### Crash Recovery 

- what if the system crashes while LFS is writing to disk 
- LFS normally buffers writes in segment 
  + when fill (or at periodic intervals), writes segment to disk 
- LFS also updates CR 
- crashs can happen at any point, how do we handle crashes during these writes 
  + Solution
    1. Uncommitted segments: reconstruct from the log after reboot 
    2. CR: Keep two CRs, at either end of the disk; alternate writes
      + update protocol (header, body, last block) 
      
### Garbage Collection 
- must periodically find obsolete versions of file data ad cleen up
- cleaning done on a segment by segment basis 
  + since segments are large chunks, it avoids the situation of having small holes of free space 

## LFS Summary 

- instead of overwriting in place, append to log and reclaim (garbage collect) obsolete data later 
- Advantage: very efficient writes 
- Disadvantage 
  + less efficient reads, but assumes most reads hit in memory anyway 
  + garbage collection is a tricky problem 

### Virtual File System (VFS) Concept 

- provides an abstract file system interface 
  + seperates abstraction of file and collections of files from specific implementations 
  + System calls such as open, read, write etc. can be implemented in terms of operations on the abstract 
  file system
    * vfs_open, vfs_close 
  + Abstraction layer is the OS itself 
    * User level programmer interacts with the file systems through the sytem calls 
    
### Redundancy (Another approach to data persistency) 

- redundant array of independant disks (RAID) 
- reliability strategies: 
  - data duplicated - mirror images, redundant full copy => one disk fails, we have the mirror
  - data spread out accross multiple disks with redundancy => can recover from a disk failure, by reconstructing the data
- Concepts: 
  + redundancy/mirroring: keep multiple copies of the same bock on different drives just in case a drive fails
  + Parity information: XOR each bit from 2 drives, store checksum on 3rd drive 
  
### Solid State Disks (SSDs) 

- replace rotating mechanical disks with non volatile memory 
  - battery backed ram and NAND flash
- advantage: faster 
- disadvantages  
  + expensive
  + wear out (flash based) 

### SSD Characteristics 

- data cannot be modified in place 
  + no overwrite without erase 
- Terminology 
  + page(unit of read/write), block (unit of erase operation) 
- Uniform random access performance 
  + disks typically have multiple channels so data can be split across blocks, speeding access time 
  
### Writing 

- Consider updating a file system block (e.g a bitmap allocation block in ext2 file system)
  + find the block containing the target page 
  + read all active pages in the block into controller memory 
  + update tatget page with new data in controller memory 
  + erase the block (high voltage to set all bits to 1)
  + write entire block to drive 
- Some FS blocks are frequently updated 
  + and ssd blocks wear out (limited erase cycles) 

### SSD Algorithms 

- Wear levelling 
  + always write to new location 
  + keep a map from logical FS block nuber to current SSD block and page location 
  + Old versions of logically overwritten pages are "stale" 
- Garbage collection 
  + reclaiming stale pages and creating empty erased blocks 
- RAID 5 (with parity checking) stripping across I/) chanels to multiple NAND chips 

## Summary File System Goals 

- Efficiently translate file name into file number using a directory 
- Sequential file access performance 
- Efficient random access to any file block 
- Efficient support for small files (overhead in terms of space and access time) 
- support large files 
- efficient metadata storage and lookup
- crash recovery 

## Summary: File System Components 

- Index structure to locate each block of a file
- Free space management 
- locality heuristics 
- crash recovery 

