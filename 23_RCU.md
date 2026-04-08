To study the RCU mechanism in Linux kernel



# locks



## deadlock

How do you avoid deadlock?
Banker's algorithm is theory; real answer = lock ordering + lockdep



4 Coffman conditions

- mutual exclusion
- hold and wait
- no preemption (for the resource held)
- and circular wait



There are only detection, resolution, and prevention techniques. But, there are no Deadlock-stopping techniques



### Deadlock Prevention

**Lock Ordering**:

All threads must acquire locks in the same, predefined order.

Assgin a level (0, 1, 2...) to every lock and ensure locks are acquired in increasing order of these levels.

When acquiring multiple resources (e.g., account A and B), sort them by ID (hash code or memeory address) to determine te locking order.



**Lockdep**:

Deadlock Detection

ref: https://docs.kernel.org/locking/lockdep-design.html

Lockdep is the Linux kernel's lock validator, which checks for potential deadlocks by analyzing lock dependency chains at runtime, even if the deadlock hasn't technically occurred yet.

Class-Based Checking: Lockdep checks dependencies between **classes** of locks rather than specific instances to detect potential future deadlocks.

Dependency Graph: It constructs a **global graph of lock acquisition order**.

Validation: Whenever a lock is acquired, it checks if any locks in the current "after" list are already held, signalling a potential cycle.



Actual Deadlocks: Pinpoints where a deadlock has occured.

Potential Deadlocks: Warns developers of risky code paths that could lead to a deadlock (e.g., inconsistet ordering).



**Comparison**

Lock Ordering is a defensive, design-time prevention technique.

Lockdep is a runtime detection tool, often used during development to identify bugs in the locking strategy.

Best Practice: Combine strict lock ordering with Lockdep (enabled via `CONFIG_PROVE_LOCKING` in Linux) to guarantee that the locking hierarchy is strictly followed and to catch complex, non-obvious deadlock scenarios.



**Common Pitfalls**

Inconsistent Ordering: The main cause of deadlocks in multi-threaded systems, often occuring when two threads acquire the same locks in different orders.

Overhead: Lockdep has performance overhead; it should not be enabled in production environments.

Nested Lock Types: Care must be taken with nested locks, especially regarding sleeping locks (`mutex`) inside spinlocks.



### Dealing with deadlock



There are several ways to detect and deal with deadlock.  In increasing order of complexity: 

1. **Do nothing**.  If the system locks up, that will teach the user not to try doing what they just did again.  This is approach taken in practice for many  resource constraints. 
2. **Kill processes**.  We can detect deadlock by looking for waiting cycles (which can be  done retrospectively once we notice nobody is making as much progress as we like).  Having done so, we can either kill every process on the  cycle if we are feeling particularly bloodthirsty or kill one at a time  until the cycle evaporates.  In either case we need to be able to reset  whatever resources the processes are holding to some sane state (which  is a weaker than full preemption since we don't care if we can restore  the previous state for the now-defunct process that used to be holding  it).  This is another reason why we design operating systems to be  rebooted. 
3. **Preempt and rollback**.  Like the previous approach, but instead of killing a process restore  it to some earlier safe state where it doesn't hold the resource.  This  requires some sort of checkpointing or transaction mechanism, and again  requires the ability to preempt resources. 



### Banker's Algorithm

Let **'n'** be the number of processes in the system and **'m'** be the number of resource types.

**1. Available**

-  It is a 1-D array of size  **'m'**  indicating the number of available resources of each type. 
-  Available[ j ] = k means there are  **'k'**  instances of resource type  **R<sub>j</sub>** 

**2. Max**

-  It is a 2-d array of size '**n\*m'**  that defines the maximum demand of each process in a system. 
-  Max[ i, j ] = k means process  **P<sub>i</sub>**  may request at most  **'k'**  instances of resource type  **R<sub>j</sub>** .

**3. Allocation**

-  It is a 2-d array of size  **'n\*m'**  that defines the number of resources of each type currently allocated to each process. 
-  Allocation[ i, j ] = k means process  **P<sub>i</sub>**  is currently allocated  **'k'**  instances of resource type  **R<sub>j</sub>** 

**4. Need**

-  It is a 2-d array of size  **'n\*m'**  that indicates the remaining resource need of each process. 
-  Need [ i,  j ] = k means process  **P<sub>i</sub>**  currently needs  **'k'**  instances of resource type  **R<sub>j</sub>** 
-  Need [ i,  j ] = Max [ i,  j ] – Allocation [ i,  j ] 

Allocation specifies the resources currently allocated to process Pi and Need specifies the additional resources that process Pi may still request to complete its task.



### Key Concepts in Banker's Algorithm

- **Safe State:** There exists at least one sequence of processes such that each process  can obtain the needed resources, complete its execution, release its  resources, and thus allow other processes to eventually complete without entering a deadlock.
- **Unsafe State:** Even though the system can still allocate resources to some processes,  there is no guarantee that all processes can finish without potentially  causing a deadlock.



Tanenbaum observes that in practice nobody uses this algorithm given the **unreasonable requirement** that each process **pre-specify** its maximum demand.



spinlock vs mutex

When to use what in drivers (interrupts = spinlock; user-facing = mutex)



## RCU

RCU (read-mostly)

Writer will make a copy of the protected data, and do changes to that copy. Finially, it does a atomic switch to make the pointer point to the new copy. The release of the old copy will be delayed until all readers depending on it finished. As the critical section (implementations may change to preempt the "critical section", then the occasion of reclaiming old copy changes too) prevents preemption, this can easily be determined by observing all CPU have a context switch.



RCU cannot substitute `rwlock`, in write-heavy occasions, as the improvents for readers cannot compensate preformance loss of writers( of the copy-update operation).



## spin lock

Spinlock (no sleep, atomic context)

Acquiring spin lock will disable preempt first. This is why it can't sleep and won't sleep. Busy waiting a spin lock is wasting CPU time, but this is needed to prevent race conditions, especially useful in interrupt context, where sleep lock is not applicable, or context switch is considered inefficient. It's a light weight lock, which should not be hold for a long time.

If a spin lock is shared between Process(Thread) & Interrupt Context, it should use `spin_lock_irqsave` and `spin_unlock_irqrestore` to avoid deadlock. Or, using `spin_lock` and `spin_unlock` is enough.

A interrupt context is hardirq context if IRQ disabled, softirq context if IRQ enabled. 

In the hardirq context or explicitly disabling IRQ before entering a critical section, it won't be interrupted by other interrupts (as IRQ is disabled), we can say it's a atomic context. Before SMP, the atomic context is enough for the scenarios requiring a spin lock. It's like a spin lock covers a section for atomic operations which can't be interrupted by others.



## rwlock

Reader-Writer Lock

It allows multiple threads to read a shared resource simultaneously while ensuring exclusive access for write operations.

Suitable for Read-heavy workloads.



In User-Space (Pthreads):

`pthread_rwlock_t`

`pthread_rwlock_init`



In Kernel-Space (Spinlocks):

Defined in `<linux/rwlock.h>`:

`read_lock()`

`write_lock()`

`read_lock_irqsave()`



Modern Linux kernels ofter use a 32-bit integer to track state:

- Readers: use the lower bits as a counter for active readers
- Writers: use the highest bit (bit 31) as a 'write-locked" flag
- Atomic Operations: Operations use architecture-specific atomic instructions (e.g., `ldxr` / `stxr` on ARM64 or `LOCK` prefixed instructions on x86) to ensure thread safety without manual software locking.



Implementation in Linux kernel: 

```c
static inline void __raw_read_lock(rwlock_t *lock)
        __acquires_shared(lock) __no_context_analysis
{
        preempt_disable();
        rwlock_acquire_read(&lock->dep_map, 0, 0, _RET_IP_);
        LOCK_CONTENDED(lock, do_raw_read_trylock, do_raw_read_lock);
}

static inline unsigned long __raw_read_lock_irqsave(rwlock_t *lock)
        __acquires_shared(lock) __no_context_analysis
{
        unsigned long flags;

        local_irq_save(flags);
        preempt_disable();
        rwlock_acquire_read(&lock->dep_map, 0, 0, _RET_IP_);
        LOCK_CONTENDED(lock, do_raw_read_trylock, do_raw_read_lock);
        return flags;
}

static inline void __raw_write_lock(rwlock_t *lock)
        __acquires(lock) __no_context_analysis
{
        preempt_disable();
        rwlock_acquire(&lock->dep_map, 0, 0, _RET_IP_);
        LOCK_CONTENDED(lock, do_raw_write_trylock, do_raw_write_lock);
}

static inline unsigned long __raw_write_lock_irqsave(rwlock_t *lock)
        __acquires(lock) __no_context_analysis
{
        unsigned long flags;

        local_irq_save(flags);
        preempt_disable();
        rwlock_acquire(&lock->dep_map, 0, 0, _RET_IP_);
        LOCK_CONTENDED(lock, do_raw_write_trylock, do_raw_write_lock);
        return flags;
}
```





## mutex

Mutex (can sleep)

A mutex (mutual exclusion) is a sleeping lock used to serialize access to a shared resources. It won't busy-wait. It allows a task to sleep (suspend itself) if the lock is unavailable, freeing the CPU for other tasks. So it can't be used in interrupt context, which has no its own task context for swapping.

In **Fastpath**, if the lock is entirely free, it is acquired atomically via `cmpxchg()`, which should be supported by atomic instructions in most CPU

In **Midpath**, the requesting task will do a brief spin if the lock is likely to be released soon.

In **Slowpath**, the task is added to a wait-queue and enters a sleeping state (`TASK_UNINTERRUPTIBLE`) until the owner wakes it up.



`DEFINE_MUTEX(name)`

`mutex_init(&lock)`

`mutex_lock(&lock)`

`mutex_lock_interruptible(&lock)` can be interrupted by signals

`mutex_trylock(&lock)` won't sleep, return 1 if acquired, or 0 if not

`mutex_unlock(&lock)`



## semaphore

Semaphore (for long-duration tasks, can sleep)

Semaphores are used for critical sections that may take a long time to execute. Because they can cause the calling process to sleep, they cannot be used in **interrupt context** where sleeping is prohibited.

The semaphore structure is defined n `<linux/semaphore.h>`:

```c
struct semaphore {
    raw_spinlock_t      lock;       // Protects the semaphore's internal data
    unsigned int        count;      // Number of available resources
    struct list_head    wait_list;  // List of sleeping tasks waiting for the lock
};
```



`sema_init(sem, val)`

`down(sem)`

`down_interruptible(sem)` Most common; sleeps but can be interrupted by signals (returns `-EINTR`)

`down_trylock(sem)` Attempts to acquire without sleeping; return 0 on success

`up(sem)`



vs. Mutex: A mutex is a specialized binary semaphore with stricter rules (only the owner can unlock it) and support for **priority inheritance**, which helps prevent priority inversion. Modern Linux development prefers mutexes for simple mutual exclusion.

In modern Linux kernel development, the use of semaphores has significantly decreased in favor of mutexes (for mutual exclusion) and completions (for task synchronization). However, they remain essential for managing **pools** of multiple identical resources.



## rw_semaphore

Defined in `include/linux/rwsem.h`

In standard kernel configurations, the implementation is designed to be fair, preventing "writer starvation" by ensuring that waiting writers take **precedence** over new incoming readers.



Lock: `down_read(sem)` `down_write(sem)`

Try Lock: `down_read_trylock(sem)` `down_write_trylock(sem)`

Unlock: `up_read(sem)` `up_write(sem)`

Killable: `down_read_killable(sem)` `down_write_killable(sem)`

Getting rwsem lock failure will lead to KILLABLE state



The `rw_semaphore` is best suited for scenarios where data is **frequently read but rarely modified** (e.g., protecting the `mmap_lock` for a process's memory address space). For short, non-sleeping critical sections, a `rwlock_t` (a spinning variant) is used instead.



## condition variables

Condition variables are another synchronization mechanism available to threads

A **condition variable** is an explicit queue that threads can put themselves on when some state of execution(i.e., some **condition**) is not as desired (by **waiting** on the condition); some other thread, when changes said state, can then wake one (or more) of those waiting threads and thus allow them to continue (by **signaling** on the condition). The idea goes back to Dijkstra's use of "private semaphores"; a similar idea was later named a "condition variable" by Hoarse in his work on monitors.



`pthread_cond_init`

`pthread_cond_wait`

`pthread_cond_timewait` Blocks for a specific amount of time before timeout

`pthread_cond_signal`

`pthread_cond_broadcast`

`pthread_cond_destroy`

```c
pthread_cond_t cond; // declare a condition variable
pthread_mutext_t lock; // declare a mutex to protect condition variable

pthread_mutex_lock(&lock);
while (!condition_is_met) { // check the waited condition
    // unlock and wait atomically
    pthread_cond_wait(&cond, &lock);
}
// Process the data
pthread_mutex_unlock(&lock);

```





# pthread vs std::thread

`pthread` will lead to memory leakage if neither called with `pthread_join` to wait for its finish and release resources, nor with with `pthread_detach` to detach it, then it will release its resources at its finish ( without these operations, its resources will not be release at its return)

`std::thread` in C++ needs the corresponding operation by `std::thread::join` and `std::thread::detach`

`std::jthread` will join the thread at the exiting of the process

