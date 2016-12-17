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
  
### The content of a data block 

- 
