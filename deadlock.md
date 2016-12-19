# Concurrency Bugs, Deadlock and Transactions 

## No-Deadlock bugs 

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

