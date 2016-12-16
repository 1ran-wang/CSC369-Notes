# Exam Review 

### The Basics 

- What is an operating System? 
  + An abstraction of the hardware, supporting basic functions of the computer such as scheduling tasks, executing apps, and controlling peripherals
  + allow programs to share memory, allow them to interact with devices etc 
- Purpose of an Operating Sysem?
  + Virtualizaion
    * Makes the system easier to use, because you do not need to worry about hardware specifics such as where memory is, etc 
    * Takes the physical resource such as the processor or disk and transforms it into a more general, easy to use virtual form of itself
  + Provides a standard Library for applications 
  + Manages resources 
  
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
