# Disks & File Systems 

### Data Region 

__Overall Organization__

- The whole disk size is 256KB, divided into 4KB blocks 

- Most of the disk should be used to store user data, leaving some space for storing things like metadata
  + In VSFS we reserve the last 56 blocks as data region 

__Metadata: Inode Table__ 

- The FS needs to track information about each file 
- In VSFS, we keep the info of each file in an inode, and we use 5 blocks for storing all inodes. 

__Allocation Structures__ 

- keep track of which blocks are being used and which ones are free 
- we use a bitmap for this purpose 
  + each bit indicates if one block is free or in use 
- A bitmap for the data region and a bitmap for the inode region 
  + Reserve one block for each bitmap

__Superblock__

- superblock contains information about the file system 
  + What type of file system it is 
  + how many inodes and data blocks there are 
  + where the inode table begins 

Example: Read a file with inode number 32 

* From the superblock we know the inode table begins at block 3 and each inode is 128B 
* The address of inode 32 is 12Kb*32*128B = 16KB
* say the inode contains an array of 15 direct pointers to 15 data blocks that belong to the file
  + Maximum file size is 15*4Kb = 60 KB
  + If we need a file larger than 60 KB, we need to do something more sophisticated 
  
__Multi-Level Index with Indirect Pointers__ 

- direct pointers to disk blocks do not support large files
- an __indirect pointer__ points to a block with more pointers 
  + from the 15 pointers in an inode, use the first 14 as direct and the last one as an indirect pointer
- We can support much larger files now (4K*(12 + 1K) = 4152 KB)

- for even larger files we can use a double indirect pointer or a triple indirect pointer 

__ A tree-view of multi-level indirect pointers__

- the trees are often super imbalanced 
- if big files are accessed frequently, the blocks with double and triple indirect pointers get accessed many times 
- but in practice, it doesn't make much of a difference because most files are small 

### Another Approach: Extent Based 

- An __extent__ is a disk pointer plus a length (in num of blocks)
  + It allocates a few blocks in a row 
- Instead of requiring  a pointer to every block of a file, we just need a pointer to every several blocks
- Advantages: uses a smaller amount of metadata per file, and file allocation is more compact 
- Disadvantages: Less flexible than the pointer based approach 

### Another Approach: Linked-Based 

- instead of pointers to all blocks, the inode just has one pointer to the first data block of the file, then
the first block points to the second etc
- works poorly if we want to access the last block of a big file
- use an in memory File Allocation Table, indexed by the address of data block

## Summary 

- Inodes
  + Data Structure representing a file system object (file or dir)
  + Attributes, disk block locations 
  - No file name, just metadata 
- Directory 
  + list of (name, inode) mappings 
  - each directory entry: a file, directory, link, itself (.), oarent dir (..) etc 

### Links: Examples 

- Hard links 
  + Multiple file names (and directory entries) mapped to the same inode 
  + Reference count - only remove file when it reaches 0 
  
- Soft (Symbolic) links 
  + "Pointer" to a given file 
  + Contains the path 

- A hard link points to a file by inode number. The file you link must exist in the file system
  + If you delete the original name, then the hard link still points to the same file
- A soft link points to a file by _name_. The name does not actually have to exist or may exist on a different filesytem
  + If you replace the named file (without changing name) then the link points to the new file
  
### The content of a data block 

- If it belongs to a regular file 
  + Data of the file 
- If it belongs to a directory 
  + list of directory entries: (name, inode number) pairs, which are entries under the directory 
- If it belongs to a symbolic link 
  + The path of the file that it links to 

### Unix Inodes and Path Search 

- Unix inodes are not directories, they descrive where on the disk the blocks for a file are placed 
  + Directories are files, so inodes also describe where the blocks for the directories are placed on disk 
- Directory entries map file names to inodes 
  + to open "/somefile", use Master block to find inode "/" on disk and read inode into memory
  + inode allows us to find the data block for directory "/"
  + Read data block for "/", look for entry "somefile"
  + This entry identifies the inode for "somefile"
  + Read the inode for "somefile" into memory 
  + The inode says where the first data block is on disk 
  + Read that block into memory to access data in the file 
  
## Performance Optimizations 

### Caching 

- file operations (open, read and write) incur a good amount of disk I/O 
- caching helps the file system perform reasonably well 

### File Buffer Cache 

- Key observation: Applications exhibit significant locality for reading and writing files
- Idea: Cache file blocks in memory to capture locality 
  + This is called the __file buffer cache__
  + Cache is system wide, used and shared by all processes 
  + Reading from the cache makes a disk perform like memory 
  + Significant reuse: spatial and temporal locality 
  + Even a 4 MB cache can be very effective 
- What do we want to cache? 
  + Inodes, directory entries, disk blocks for "hot files", even whole files if small 
  
### Caching and Buffering 

- Static Partitioning: at boot time, allocate a fixed size cache in memory (typically 10%) 
  + Can be wasteful if the file system doesn't really need 10% if memory 
- Dynamic Partitioning: Integrate virtual memory pages and file system pages into a unified page cache, so pages of memory
can be flexibly allocated for either virtual memory or file system, used by modern systems 

- Replacement policy: typically use LRU 
- The tradeoff between static and dynamic partitioning 
  + To be considered on any resource allocation kind of problem 

- works well for reads but not for writes 
  + writes still have to go to disk to become persistent anyways 
  + once a block is modified in memory, the write back to disk may not be immediate (synchronous) 
  
### Tradeoff: speed vs durability 

- caching and buffering improves the speed of file system reads and writes 
- however it sacrifices the durability of data 
  + crash occurs => buffered writes not written to disk yet so they are lost 
  + better durability => sync to disk more frequently => worse speed 
- When to favour speed? when to favour durability?
  + depends on the application and its needs (e.g web browser cache or bank database) 
  
### Approaches? 

- delay writes only for a specific amount of time 
  - how long do we hold dirty data in memory?
- asynchronous writes ('write-behind) 
  + maintain a queue of uncomitted blocks 
  + periodically flush the queue to disk 
  + unreliable 
- Battery backed up RAM (NVRAM) 
  + as with write-behind, but maintain queue in NVRAM 
  + expensive 
- log-structured file system 
  + always write contiguously at end of previously write 
  
### Read Ahead 

- many file systems implement "read ahead"
  + FS predicts that the process will request next block 
  + FS goes ahead and requests it from disk 
  + This can happen while the process is computing on previous block 
    * overlap I/O with execution 
  - when the process requests block, it will be in the cache 
  - compliments the on-disk cache, which also is doing read ahead 
- for sequentially accessed files, can be very effective 
  * unless blocks for the file are scattered across the disk 
  * file systems try to prevent that during allocation 
  
## Disk's Physical Characteristics 

### Secondary Storage Devices 

- Drums 
- Magnetic disks 
  + fixed disks 
  + removable disks 
- Optical disks 
  + write once, read many (CD-R, DVD-R)
  + write many, read many (CD-RW) 
  
### Disk Performance 

- disk request performance depends on a number of steps 
  + Seek - moving the disk arm to the correct cylinder 
    * depends on how fast disk arm can move
    * typical times: 1-15 ms, depending on distance (avg 5-6 ms)
  + Rotation - waiting for the sector to rotate under the head 
    * depends on rotation rate of disk 
    * avaerage latency 1/2 rotation 
  + transfer - transferring data from surface into disk controller electronics, sending it back to the host
    * depends on density 
    * improving rapidly (40% per year) 

### Some hardware optimizations 

- Track skew 
  + if arm moves to the correct track to slowly, we may miss the sector we want and we need to do another 
  entire rotation to reach it 
  + instead, skew the track location, so we have enough time to position 
- Zones 
  + outer tracks are larger so they should hold more sectors 

- Cache, aka Track Buffer 
  + A small memory chip, part of the hard drive (usually 8-16 MB)
  + different from cache that OS has 
    * unlike OS cache, it is aware of disk geometry  
    * when reading a sector, may cache the whole track to speed up future reads on the same track 

### Disk and the OS 

- disks are messy physical devices:
  + errors, bad blocks, missed seeks 
- the job of the OS is to hide this mess from higher level software   
  + low level device control 
  + higher level abstractions 
- The OS may provide different levels of disk access to different clients 
  + physical disk (surface, cylinder, sector)
  + logical disk (disk block #)
  + logical file (file block, record, or byte #) 
  
