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
  
### Disk Interaction 

- specifying disk requests require a lot of info 
  + cylinder num, surface num, track num, sector num, transfer size ...
- modern disks are more complication 
  + not all sectors are the same size, sectors are remapped 
- Current disks provide a higher level interface (SCSI) 
  + the disk exports its data as a logical array of blocks 
    * disk maps logical blocks to cylinder/surface/track/sector 
  + only need to specify the logical block # to read/wrtie 
  + but now the disk parameters are hidden from the OS 

### Back to File Systems ... 
- Key idea: File Systems need to be aware of disk characteristics for performacne 
  + allocation algorithms enhance performance 
  + request scheduling to reduce seek time 

### Enhancing achieved disk performance 

- high-level disk characteristics yield two goals:
  + closeness
    * reduce seek times by putting related things close to each other 
    * generally, benefits can be in the factor of 2 range 
  + Amortization 
    * amortize each positioning delay by grabbing lots of useful data 
    * generally, benefits can reach into factor of 10 range 
    
### Allocation Strategies 

- disk performs best if seeks are reduced and large transfers are used 
  + scheduling requests is one way to achieve this 
  + allocating related data "close together" on the disk is even more important 

## FFS: A disk-aware file system 

### Original Unix File System 

- recall FS sees storage as linear array of blocks (each block has logical block number (LBN)) 
- default usage of LBN space is (superblockm bitmap, inodes, data blocks) 
  + simple and straightforward by very poor utilization of disk bandwidth 

### Data and Inode Placement - Problem \#1
  - on a new FS, blocks are allocated sequentially, clost to each other 
  - as the FS gets older, files are being deleted and create random gaps 
    + data blocks end up allocated far from each other 
    + fragmentation causes more seeking 

### Data and Inode Placement - Problem \#2

- inodes allocated far from blocks 
- traversing file name paths, manipulating files and directories require going back and forth from inodes
to data blocks 
  + again lots of seeks 

### FFS (Fast File System)

- improved disk utilization, decreased response time
- All modern FSs draw from the lessons learned from FFS

### Cylinder Groups 

- BSD FFS addressed placement problems using the notion a cylinder group
  + data blocks in same file allocated in same cylinder group 
  + files in same directory allocated in same cylinder group
  + inodes for files are allocated in same cylinder group as file data blocks 
- allocation in groups reduces number of long seeks 
- free space requirement 
  + to be able to allocate according to cylinder gropus, the disk must have free space scattered across cylinders 
  + 10% of disk reserved just for keeping the disk partially free all the time 
  + when allocating large files, break it into large chunks and allocated from different cylinder groups, 
  so it does not fill up one cylicder roup 
  + if preferred cylinder group is full, allocated from a "nearby" group 
  
### More FFS solutions 

- small blocks in original UNIX FS caused 2 problems 
  + low bandwidth utilization and small max file size (function of block size) 
- fix using a larger block
  + very large files only need two levels of inderection 
  + new problem: internal fragmentation 
  + fix: introduce fragments 
- Problem: media failures 
  + replicate master block(superblock) 
- Problem: Device oblivous 
  + paramterize according to device characteristics 

### Disk Scheduling Algorithms 

- because seeks are so expensive, OS tries to echedule disk requests that are queued for the disk
- Policies for minimizing seeks:
  1. FCFS (do nothing)
    * Reasonable when load is low
    * long waiting times for long request queues 
  2. SSTF (shortest seek time first)
    * minimize arm movement (seek time), maximize request rate 
    * favors middle blocks 
  3. SCAN (elevator) 
    * service requests in one direction until done, then reverse
  4. C-SCAN 
    * like scan, but only go in one direction (typewriter) 
  5. Look/C-Look 
    * like SCAN/C-SCAN but only go far as last request in each direction (not fill width of the disk) 
- In general, unless there are request queues, disk scheduling does not have much impact 
  + important for servers, less so for PCs 
- Modern disks often do the disk scheduling themselves 
  + disk know their layout better than OS, can optimize better 
  + if so, ignore/undoes any scheduling done by the OS 

## Summary 

- file systems overview 
  + disks are large, persistent, but slow   
  + Operations on files and directories, sharing 
- File system organization 
  + VSFS example, ext2, other strategies and design choices 
- Performace enhancements: caching, read-ahead
- Disks
  + physical structure 
  + Placement problems and strategies, FFS
  + Disk Scheduling algorithms 
