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
