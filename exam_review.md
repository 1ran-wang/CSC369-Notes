# Exam Review 

### The Basics 

- What is an operating System? 
  + An abstraction of the hardware, supporting basic functions of the computer such as scheduling tasks, executing apps, and controlling peripherals
  + allow programs to share memory, allow them to interact with devices etc 
- Purpose of an Operating Sysem?
  1. Virtualizaion
    * Makes the system easier to use, because you do not need to worry about hardware specifics such as where memory is, etc 
    * Takes the physical resource such as the processor or disk and transforms it into a more general, easy to use virtual form of itself
  2. Provides a standard Library for applications 
  3. Manages resources 
  
### Processes & Threads 

- What is a process? 
  + a running program 
- What is a thread?
  + an abstraction of a single running program 
  + A multi-threaded program has multiple points of execution 
  + Like a seperate process but shares the same address space 
- __TO DO__
- What is the difference betweem user-level threads and kernel-level threads?
- How are new processes created? Deleted? Zombies? 
- What does the address space look like? PCB?
- What state can a process be in?
- How do threads relate to virtual address spaces? 

### System Calls 

- What are protection domains? Why do we need them?
- How do interrupts work? Why do we need them?
- What happens when a proces makes a system call?
- How and when does a context switch happen?

### Concurrency 

- What is the critical section problem?
- What properties does a solution need to have?
- What is a race condition?
- Synchronization primitives
  + Software solutions (no HW support): Peterson's algorithm, Bakery Algorithm 
  + Hardware instructons - Test and Set, Swap/Exchange 
  + Locks (Spinlocks vs Sleep locks), semaphores, CVs, Monitors 
- Revisit synchronization problems 
  - Producer/Consumer, Reader/Writer problems 
- How would you write a monitor (see last tutorial too) 

### Scheduling 

- Goals in developing a good scheduling algorithm 
- Know the properties of different algorithms we discussed 
  + FCFS, SJF, RR, MLFQ, etc 
  + Which schedulers are best suited for different workloads 
  + which schedulers may cause starvation 
  + How can process properties (compute-bound, I/O bound_ be used by schedulers?
  
### Memory Management - Overview 

- Why is memory management important? 
  + What are the goals of virtual memory? 
  - Why do we have virtual memory if its so complex?
- What are the mechanisms for implementing mem management? 
  + physical and virtual addressing
  + paritioning types, paging 
  + page tables, TLB 
- What are the overheads related to providing memory management?
- What are the policies related to MM?
  + page replacement 

### Virtualizing Memory 

- What is the difference between a physical and virtual address?
- What is the difference between fixed and variable partitioning?
  + How do base and limit registers work?
- What is internal fragmentation?
- What is external fragmentation?
- What is a protection fault? 

### Paging 

- How is paging different from partitioning?
- What are the advantages/disadvantages of paging?
- What are page tables?
- What are page table entries?
- What are all the PTE bits used for 
- Know these terms very well
  + Virtual page number (VPN), page frame number (PFN), offset 
- Know how to break down virtual addresses into page numbers, offset
  + and how to translate virtual to physical 

### Page Tables 

- Page tables introduce overhead
  + Space for storing them 
  + Time to use them for translation
  + What techniques can be used to reduce their overhead?
- How do linear/multi-level page tables work?
- Know the terminology: PTE, PDE< PTBR, PDBR, etc 
- You should be able by now to translate manually between hexadecimal, binary and decimal and work with them
  + hex digits: 4 bits, 0-9, a-f 

### TLBs 

- What problem does the TLb solve?
- How do TLB's work?
- Why are TLBs effective?
- How are TLBs managed?
  + What happens on a TLB miss fault 
- What is the differece between a hardware and a software managed TLB 

### Page Faults 

- What is a page fault 
- How is it used to implement demand paged virtual memory?
- What is the complete sequence of steps, from a TLB miss to paging in from disk, for translating a virtual
address to a physical address?
  + Whta is done in hardware, what is done in software? 
  
### Page Replacement 

- What is the purpose of the page replacement algorithm?
- What appication behaviour does page replacement try to exploit?
- When is the page replacement algorithm used? 
- Refresh these thoroughly 
  + Belady's (Optimal), FIFO, LRU, Clock (second chance), Belady's anomaly, Working Set model, Page Fault Frequency
  
### Advanced Memory Management 

- What is thrashing? Possible Solutions?
- Multiprogramming correlation with CPU utilization?
- What is shared memory?
- What is copy on wirte?
- How does the operating system leverage virtual memory to provide these? 

### File Systems 
- Some Topics 
  - Files
  - Directories
  - Sharing
  - Protection
  - Layouts
  - Buffer Cache 
- What is a file system 
- Why are file systems useful (why do we have them?) 

### Files and Directories 

- What is a file 
  + What operations are supported?
  + What characteristics do they have? 
  + What are file access methods> 
- What is a directory 
  + What are they used for?
  + How are the implemented?
  + What is a directory entry? 
- How are directories used to do path name translation?
- What is a hard link? A symbolic link?
- What's the content of a file/directory/link? 

### File System Layouts 

- What are file system layouts used for?
- What are the general strategie?
  + Contiguous, linked, indexed 
- What are the tradeoffs for those strategies? 
  + In what special circumstances might you prefer a method that is not suitable for a general purpose 
  file system? (e.g contiguous allocation)
- What is an inode?
  + How are inodes different from directories? 
  + How are inodes and directories used to do path resolution, find files? 

### More FS concepts 

- How do we go about building a file system? VSFS.. 
  + What are the levels of indirection in inodes?
  + Advantages/disadvantages of extent-based approach 
- Performance optimizations 
  + What's the file buffer cache, and why do operating systems use one? 
  + Why is buffering writes useful?
  + What is a major tradeoff when it comes to caching and buffering?
  + What is read ahead and why it is important?
  
### Disk 

- Understand the memory hiearchy concept, locality 
- Physical disk structure
  + platters, surfaces, tracks, sectors, cylinders, arms, heads 
  + What are seek, rotation, transfer? 
  + Why try to allocate related data close together? 
  + What is FFS, and how is it an improvement over the original Unix file system?

### Disk Scheduling 
  
- How can disk scheduling improve performance? 
  + What are the components of disk access time?
  + What effect does disk scheduling have on each?
- What are the issues in disk scedhuling?
  + Response time, throughput, fairness
- Refreash disk scheduing algorithms 
  + FCFS, SSTF, SCAN, C-SCAN, LOOK, C-LOOK 
  
### Advanced FS Topics 

- THEME: crash consistency, optimizing writes, redundancy 
- What are some crash recovery mechaniss? 
- What is fsck? What are its limitations?
- How is hournaling performed in modern file systems like ext3? Advantages over fsck? Metadata journaling?
- What is LFS? What was the key idea in its design? Advantages and drawbacks? 
- Can we handle complete disk crashes? What's the idea behind RAID?
- SSDs (probably no on the exam but highly relevant)

### Deadlocks 

- What is the definition of a deadlock?
- What are the conditions for deadlock? 
- What is deadlock prevention/avoidance/detection & recovery 
- How does the banker's algorithm work? Safe states? 
- What is a resource allocation graph, what is it used for? 
- No questions on concurrent transactions on this topic 

