# Concurrency Bugs, Deadlock and Transactions 

## Non-Deadlock bugs 

- Atomicity violation bugs 
  + when a code region intended to be atmoic, bu the atomicity is no enforced during execution 
- order violation bugs 
  + when the desired order between memory accesses is flipped
  
### Deadlocks 

- Definition: the permanent blocking of a set of processes that either:
  + compete for system resources or communicate with each other 
  
- each process in the set is block, waiting for an event which can only be caused by another process in the set
- Root causes:
  + resources are finite 
  + processes wait if a resource they need is unavailable 
  + resources may be hel by other waiting processes 
  
### What do we mean by Resource?

- any object that might be needed by a process to do its work 
  + hardware like printers, memory, processors, disk drive etc
  + Data such as variables, record in a database or a file
  + synchronization objects (or equivalently the critical regions they protect) 
  
### Conditions for Deadlock
1. Mutual Exclusion
  - only one process may use a resource at a time
2. Hold and Wait 
  - A process may hold allocated resources while awaiting assignment of others 
3. No preemption 
  - No resource can be forcibly removed from a process holding it 

- These are necessary conditions 

One more Condition:
4. Circular Wait 
  - a closed chain of processes exists, such that each process holds at least one resource needed by the 
  next process in the chain 

- together these four conditions are necessary and suffiicent for deadlock 
- circular wait implies hold and wait, but the former results from a sequence of events, while the latter 
is a policy decision 

### Deadlock Prevention

- ensure one of the four conditions doesn't occur 

1. Break Mutual Exclusion 
  - not much help here, as it is often required for correctness 

### Lock free data structures 

- use architectural support to implement lock-free data structures 

```
void AtomicIncrement(int *value, int amount) 
{
  do 
  {
    int old = *value; 
  } while (CompareAndSwap(value, old, old + amount) == 0)
}
```

### Preventing Hold and Wait 

- Break hold and wait - processes must request all resources at once and will block until entire request can
be granted simultaneously 

```
lock(master_lock);
lock(L1); 
lock(L2); 
...
unlock(master_lock); 
```

- all or nothing approach: depending on whoe gets the master_lock first => no deadlock can happen
- problematic because it makes encapsulation difficult because we always need to know which locks must be 
acquired at which times 

- Problems:
  + May wait a long time for all resources to be available at the same time 
  + must acquire all locks at the start, rather than when they are really needed => limits concurrency 
    - sme longer processes may hold locks for a long time before they end up using them 
  + may not know all resoruce requirements in advance 

- An alternative is to release all currently held resoources when a new one is needed, then make a request
for the entire set of resources 

### Preventing No-Premption

- break "no preemption" - forcibly remove a resource from one process and assign it to another 
  + need to save the state of the process losing the resource so it can recover later 
  + may need to rollback to an earlier state 
  + impossible for consumable resources 
- Alternative: some thread libraries offer trylock() function: grab a lock if it's available, otherwise try
later 

```
while (!ready) {
  lock(L1);
  if (trylock(L2) == -1) {
    unlock(L1);
    ready = false;
  }
}
```

- livelock vs deadlock
  + livelock is similar to deadlock, except that the states of the processes constantly change with regard
  to one another, none progressing 
  
### Preventing Circular wait 

- break "circular wait" - assign a linear ordering to resource types and require that a process holding 
resource of one type R, can only request resources that follow R in the ordering 
  + e.g Ri precedes Ri+1
  + for deadlock to occur, need P to hold Ri and request Rj while Q holds Rj and requests Ri 
  + if we enforce ordering of requests to resources, Q's request order cannot happen => no deadlock 
  
### Deadlock Avoidance 

- all prevention strategies are unsatisfactory in some situations 
- avoidance allows the first three conditions, but orders events to ensure circular wait does not occur 
- requires knowledge of future resource requests to decide what order to choose
  + amount and type of information varies by algorithm 

### Two Avoidance Strategies 
1. Do not start a process if its maximum resource requirements, together with the maximum needs of all 
processes already running, exceed the total system resources 
  + pessimistic, assumes all processes will need all their resource at the same time 
2. Do not grant an indiviudal resource request, if any furture resource allocation "path" leads to deadlock 

__Processes must declare maximum resource needs up front__

### Restrictions on Aviodance 

1. Maximum resource requirements for each process must be known in advance 
2. Processes must be independant 
  + if order of execution is constrained by synchronization requirements, system is not free to choose a 
  safe execution 
3. There must be a fixed number of resources to allocate 
  + tough luck if printer goes offline 
  
### Banker's Algorithm 
- System can be in one of three states:
  - Safe: for any possible sequence of requests, there is at least on safe dequence that eventually succeeds in
  granting all pending and future requests 
  - Unsafe: if all threads request there max resources at this point, system will deadlock
  - deadlocked

Algorithm 
  - For every resource request 
    1. Can the request be granted?
      - if not, request is imposisble at this point => block the process until we can grant the request 
    2. Assume that the request is granted
      + update state assuming request is granted 
    3. Check if new state is safe 
      - if so, continure
      - if not, restore old state and block processes until it is safe to grant the request 
 
 ### Deadlock Detection & Recovery 
 
 - prevention and avoidance are awkward and costly
 - instead, allow deadlocks to occur, but detect when this happens and find a way to break it 
 - finding circular waits is equivalent to finding a cycle in the resource allocation graph 
  + nodes are processes and resources 
  + Arcs from a resource to a process represent allocations 
  + arcs from a process to a resource represent ungranted requests 

### Deadlock Recovery 
- basic idea is to break the cyle 
  + Drastic: kill all deadlocked processes
  + Painful - backup and restart deadlocked processes
  + better - selectively kill deadlocked processes until cycle is broken 
    * rerun detection alg after each kill 
  + tricky - selectively preempt resources until cycle is brokern 
    * processes must be rolled back 

### Why does the ostrich alg work?

- Algorithm: ignore the problem and hope it doesn't happen often 
- recall the causes of deadlock:
  + resources are finite
  + processes wait if a resource they need is unavailable
  + resources may be held by other waiting processes 
- modern operating systems virtualize most physical resources, eliminating the first cause of deadlock
  + some logical resources can't be virtualized (there has to be exactly one), such as bank accounts or 
  process table
    * These are protected by synchronization objects, which are now the only resources that we can deadlock
    on 
    
#### Starvation

- a thread is suffering from starvation if it is waiting indefinitely because other threads are in some way 
preferred

### Communicating Deadlocks
- messages between communicating processes are a consumable resource 
- example 
  + process B is waiting for a request
  + process A sends a request to B, and waits for reply 
  + The request message is lost in the network 
  + B keeps waiting for the request, A keeps waiting for a reply => deadlock 
- Solution?
  + use timeouts, resend message and use protocols to detect duplicate messages 
  
## Transactions and Atomicity 

### Atomic transactions 

Def: Transaction 
  - A collection of operations that performs a single logical function 
  - we will consider a sequence of read and write operations, terminated by a commit or abort 
Def: Committed
  - A transaction that has completed successfully; once committed, a transaction cannot be undone 
Def: Aborted
  - A transaction that did not complete normally; typically rollback and start again 
  
### Write ahead logging 
- before performing operations on data, write intended operations to a log on stable storage 
- log records id transaction, the data item, the olde value, and new value 
- special records indicate the start and commit(or abort) of a transaciton 
- log can be used to undo/redo the effect of any transaction, allowing recovery from arbitrary failures 

### Checkpoints 
- limitations of basic log strategy
  + time consuming to process entire log after failure 
  + large amount of space required by log 
  + performance penalty - each write requires a log update before actual data update 
- Checkpoint help with the first two problems 
  + write all updates to log and data to stable storage; periodically write a checkpoint entry to the log
  + recovery only needs to look at log since last checkpoint 
  
### Concurrent Transactions 
- transactions must appear to execute in some arbitrary but serial order 
  + Solution 1: All transactions execute in a critical section, with a single common lock to protect access
  to all shared data
    * but most transactions will access different data 
    * limits concurrency unnecessarily 
  + Solution 2: Allow operations from multiple transactions to overlap, as long as they don't conflict 
    * end result of a set of transaction must be indistinguishable from Solution 1

### Conflicting Operations 

- operations in two different transactions conflict if both access the same data item and at least one is a write 
  + non conflicting operations can be reordered without changing the outcome 
  + if a serial schedule can be obtained by swapping non-conflicting operations, then the original schedule
  is conflict-serializable 
  
### Ensuring Serializability 
- Two phase locking 
  + individual data items have thier own locks 
  + each transaction has a growing phase and shrinking phsae:
    * Growing: a transaction may obtain locks, but may not release any lock 
    * Shrinking: a transaction may release locks, but not acquire any new ones 
  + doesn't guarantee deadlock-free 
    + Fix: prevent hold and wait by aborting and retrying transaction if any lock is unavailable 

### Timestamp Protocols

- each transaction gets unique timestampe before it starts executing 
  + transaction with earlier timestamp msut appear to complete before any later transactions 
- each data item has two timestamps 
  + W-TS: the largest timestampt of any transaction that successfully wrote the item 
  + R-TS: The largest timestampt of any transaction that successfully read the item 

### Timestamp ordering 
- reads on data X:
  - if transaction has earlier timestamp than W-TS on data X, then transaction needs to read a value that was
  already overwritten 
    + Abort transaction, restart with new timestamp
- Writes 
  - if transaction has earlier timestamp than R-TS (W-TS) on data, then the value produced by this write shouldhave been read (overwritten) already 
    + abort and restart 

- some transaction may starve (abort and restart repeatedly) 
