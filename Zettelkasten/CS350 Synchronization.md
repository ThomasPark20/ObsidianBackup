2022-09-29 20:06
source: [source]()
tags: #pages #CS350 #thread #threads #synchronization #lock #spinlock #semaphore


# CS350 Synchronization


## Overview
Synchronization. How threads work together and mitigate danger.

## Details

==**[[Concurrent]]**== threads interact with each other in a variety of ways, sometimes sharing access to the program's code, global variables, and heap

Danger exists, where a shared object is accessed which must not be concurrently executed by more than one thread. This is called ==**critical section**==

We try to enforce ==**mutual exclusion (also refered to as mutex)**== to mitigate this danger.

We could try and use **[[volatile]]** keyword, but this is not enough. 

**Good illustration on typical race conditions we face**

![[Racecondition.png]]

## Spinlocks, locks, and semaphores

A ==**lock**== is a mechanism to provide mutex.

We have many Approaches to achieving locks, but they all follow the same fashion:
==**test-and-set**==. We test if the lock is held. If held, keep trying to acquire lock. If acquired, exit.

### Approach #1 (x64):

x64 provides an ==**atomic**== (i.e. indivisible or *==uninterruptable==*) test-and-set operation used to implement synchronization primitives such as locks.

we have... `xchg`. 

```c
// True, if lock is held. False, if lock was released and we now have the lock.
xchg(addr, new_value) {
	old_value = *addr; // load lock value
	*addr = new_value; // try to set lock value to new_value
	return(old_value); // return the lock value.
}
```

Therefore, utilizing this,

```c
// so while xchg is true, (lock held by other thread) this continues to SPIN 
Acquire(bool *lock) {
	while (xchg(lock,true) == true) {}; // test and set
}

Release(bool *lock) {
	*lock = false; // give up lock
}
```

This construct is known as a ==**Spinlock**==, since a thread just loops in Acquire() until the lock is free...

### Approach #2 (ARM):
Rather than a single `xchg`, ARM provides a *pair of instructions*. 

`LDREX` (load register exclusive) and `STREX` (store register exclusive). They are a pair, meant to be used ==together==. (reasonably close.. specifically within 128 bits)

```c
ARMTestAndSet(addr, new_val) {
	old_val = LDREX addr // load old value
	status = STREX new_val, addr // try store new value
	if (status == SUCCEED) return old_val // if succeeded, old_val would be false!
	else return true // lock is being held by other thread
}

// utilizing this,

Acquire(bool *lock) { // this also spins until lock obtained
	while(ARMTestAndSet(lock, true) == true) {};
}

```

### Approach 3 (MIPS):

we use two instructions. `ll` and `sc`.

`ll` : load linked. load value at address *addr*
`sc` : store conditional. store new value at *addr* if the value at *addr* has not changed since the instruction `ll` was executed (this returns SUCCESS)

Weird isnt it? `sc` does not care about what ==32-bit== value was store at *addr*. It only cares if it changed or not. 

**Even if `sc` returns SUCCESS, the lock can be acquired or not acquired**

![[MIPSlock.png]]

1. `ll` addr was false and stayed false until `sc`. This case, old_val is `false` and new_val is `true`. LOCK ACQUIRED
2. `ll` addr was true and stayed true until `sc`. This case, old_val is `true` thus we keep spinning.
3. `ll` changed. This means `status == fail` which executes `else return true` thus we keep spinning.
4. same as 3.


## Spinlocks

So, it was all lies. The above "locks" were actually ==**Spinlock**==s

What are **==spinlock**==s? easy. They are above definitions.. they **SPIN(or loop)** until they acquire lock. This process is formally called: ==**busy-wait**==. This *uses the processor while they wait*.

Spinlocks are efficient for short waiting times (a few instructions)

Usage in OS/161
- updating a shared variable or data structure
- implementing other synchronization primitive such as ==locks==, ==semaphores== and ==condition variables==

```c

// REF: /kern/include/spinlock.h
// REF: /kern/arch/mips/include/spinlock.h

struct spinlock {
	volatile spinlock_data_t lk_lock;
	struct cpu *lk_holder;
};

void spinlock_init(struct spinlock *lk);
void spinlock_acquire(struct spinlock *lk);
void spinlock_release(struct spinlock *lk);


// spinlock_acquire calls spinlock_data_testandset in a loop until the lock is acquired.

// function definitions

void spinlock_init(struct spinlock *lk);
void spinlock_acquire(struct spinlock *lk);
void spinlock_release(struct spinlock *lk);
bool spinlock_do_i_hold(struct spinlock *lk);
void spinlock_cleanup(struct spinlock *lk);

```


Again, **spinlock_acquire() calls spinlock_data_testandset() in a loop until the lock is acquired.**

Welcome back to MIPS.. I mean OS/161. We also use `ll` and `sc` to implement `spinlock_data_testandset()`

`ll` loads data from `sd` (spinlock data) into `x`, and uses `y` to store 1 (true, or means locked).

As always, on failure (lock is held by other thread), return 1 (true). 
On Success, we return old value of the lock
- If old value was 0 -> lock has been acquired (return false to exit)
- If old value was 1 -> lock is still held by another thread. (return true to spin)

```c
/* return value 0 indiciates lock was acquired*/
spinlock_data_testandset(volatile spinlock_data_t *sd) {
	spinlock_data_t x,y;
	y = 1; // value to store (we try to set true)
	__asm volatile( // begin asm
		".set push;" // save assembler mode
		".set mips32;" // allow MIPS32 instructions
		".set volatile;" // avoid unwanted optimization (get off my code!!)
		"ll %0, 0(%2);" // get value of lock x = *sd
		"sc %1, 0(%2);" // *sd = y; y = success?
		".set pop" // restore assembler mode
		: "=r" (x), "+r" (y) : "r" (sd)); // : outputs : inputs :
	if (y == 0) {return 1;} // if unsuccessful, still locked
	return x; // if success return lock value
}

/* Notes: outputs are x (%0) and y (%1) and the input is sd (%2).
	Switches describe how a value is handled:
		"=r" means write only, stored in a register.
		"+r" means read and write, stored in a register.
		"r" means input, stored in a register.
*/

```

## Locks

Again, A ==**lock**== is a mechanism to provide mutex.

Unlike spinlocks however, it ==**blocks**==

```c
struct lock *mylock = lock_create("LockName");

lock_acquire(mylock);
// critical section
lock_release(mylock);

// function definitions

struct lock *lock_create(const char *name);
void lock_acquire(struct lock *lk);
void lock_release(struct lock *lk);
bool lock_do_i_hold(struct lock *lk);
void lock_destroy(struct lock *lk);

```

***locks can be used to protect larger critical sections without being a burden on the processor***

#### Spinny boi vs lockky boi

**For both**
- thread would acquire, release
- must be initialized or created before using them
- have funtion `do_i_hold` it?
- have *owners* and can only be released by their owner.
	- why? in OS/161 and modern OSs, interrupts are disabled on the CPU that holds the spinlock but not other CPUs to minimize spinning
	- spinlock - *CPU*
	- lock - *thread*
	

#### So what is thread ==**blocking?**==

- Sometimes a thread will need to wait..
	- wait for lock to be released by another thread
	- wait for data from a slow device
	- wait for input from a keyboard
	- etc

==**when a thread blocks, it stops running**==, i.e. does not use the CPU.
- The scheduler chooses new thread to run
- context switch occurs
- blocking thread is queued in a ==**wait channel**==

```c
void wchan_sleep(struct wchan *wc);
	- blocks calling thread on wait channel wc
	- causes a context switch, like thread_yield()
void wchan_wakeall(struct wchan *wc);
	- unblock all threads sleeping on wait channel wc
void wchan_wakeone(struct wchan *wc);
	- unblock one thread sleeping on wait channel wc
void wchan_lock(struct wchan *wc);
	- prevent operations on wait channel wc


// wait channels are implemented with queues
```


## Semaphores

A ==**Semaphore**== is a synchronization primitive that can be used to enforce mutex and also other kinds of synchro problems. It has an ==*integer*== value that supports two ==*atomic*== operations:
- **P ( Procure )**: if the semaphore value is greater than 0, decrement it.
	- Otherwise, wait until value is > 0 and then decrement it.
- **V ( Vacate )**: increment the value of the semaphore

#### 3 types of semaphores:
1. **binary semaphore**: a semaphore with a single resource; behaves like a lock, but does not track ownership
2. **counting semaphore**: a semaphore with an arbitrary number of resources
3. **barrier semaphore**: a semaphore used to force one thread to wait for others to complete; initial count is typically 0

#### Differences between a lock and a semaphore:
- **Vacate** does not have to follow **Procure**
- A semaphore can start with a count of 0 (no resources avail)
- Calling **Vacate** increments the semaphore by 1
- This sequence of operations forces a thread to wait until resources are produced before continuing
- Semaphores do not have owners. Locks do.

#### Mutex using a semaphore

```c
volatile int total = 0;
struct semaphore *total_sem;
total_sem = sem_create("total mutex", 1); // initial value is 1

void add() {
	int i;
	for (i=0; i<Nl; i++) {
		P(sem); // decrement sem! (now sem is 0)
		total++;
		V(sem); // increment sem! (now sem is 1)
	}
}

void sub() {
	int i;
	for (i=0; i<N; i++) {
		P(sem); // decrement sem! (now sem is 0)
		total--;
		V(sem); // increment sem! (now sem is 1)
	}
}

"P(sem) won't go thru unless there is at least 1 resource avail. Thus, mutex!"

```

#### Producer/Consumer Synchronization with Bounded Buffer
1. ==**Producers**==: threads that (create and) ==*add*== items to a buffer
2. **==Consumers==**: thread that *==remove==* (and process) items from the buffer

**We use semaphores to provide the necessary synchronization for**...
1. ensure consumers don't consume if buffer is empty (wait until at least one in buffer)
2. if buffer has limited capacity, ensure producers wait if buffer is full

```c

struct semaphore *Itmes, *Spaces, *Binary;
Items = sem_create("Buffer Items", 0); // initially = 0
Spaces = sem_create("Buffer Spaces", N); // initially = N
Bianry = sem_create("Binary Mutex", 1); // initally = 1

// Producer's pseudo-code:
	P(Binary); // mutex. If this is 1, continue (decrement 1). if this is 0, we wait until other function is over (race-condition avoided).
	P(Spaces); // block if there is no space in buffer
	add item to the buffer
	V(Items); // increase the number of items available
	V(Binary); // release mutex (increment by 1)

// Consumer's Pseudo-code:
	P(Binary); // mutex. If this is 1, continue (decrement 1). if this is 0, we wait until other function is over (race-condition avoided).
	P(Items); // block if there are no itesm to consume
	remove item from buffer
	V(Spaces); // increase the amount of buffer space available
	V(Binary); // release mutex (increment by 1)
```

#### Semaphore Implementation
```c
Ref: kern/include/synch.h

struct semaphore {
	char *sem_name; // for debug purposes
	struct wchan *sem_wchan*; // queue where threads wait
	struct spinlock sem_lock; // to synch access to this struct
	volatile int sem_count; // value of the semaphore
}
```

```c
Ref: kern/thread/synch.c

1. P(struct semaphore * sem) {
2.	spinlock_acquire(&sem->sem_lock);
3.	while (sem->sem_count == 0) { // test
4.		wchan_lock(sem->sem_wchan); // prepare to sleep
5.		spinlock_release(&sem->sem_lock);
6.		wchan_sleep(sem->sem_wchan); // sleep
7.		spinlock_acquire(&sem->sem_lock);
8.	}
9.	sem->sem_count--; // set
10.	spinlock_release(&sem->sem_lock);
11.}

2. "You need to acquire a spinlock to access the semaphore structure"
3. "If there are no resources available, (sem_count == 0) time to sleep"
4. "Need to modify wait channel, so lock it (4) before releasing the spinlock (5)
6. "Calling" wchan_sleep() "will release the wchan spinlock for the thread and then the thread goes to sleep. When the thread is woken," wchan_sleep() "returns"
7. "Another thread may have gotten to the spin lock between the time that you woke up and tried to acquire the lock.
9. "If you've broken out of the while-loop, you have the lock and sem_count is greater than 0, so claim an item.
```

```c
Ref: kern/thread/synch.c

V(struct semaphore * sem) {
	spinlock_acquire(&sem->sem_lock);
	sem->sem_count++;
	wchan_wakeone(sem->sem_wchan);
	spinlock_release(&sem->sem_lock);
}

"Acquire the lock, increase the count, wake up the first thread (if any) sleeping on the semaphore's wait channel"
```

#### Example run
| Thread 1 | Thread 2 | Explanation|
| ---------- | ---------| --------|
| calls P() | ...| ... |
| spinlock_acquire | calls v()| ... |
| sem_count == 0 | spinlock_acquire| Thread 2 tries to get spinlock|
| wchan_lock | spins| Thread 2 spins and Thread 1 locks wchan|
| spinlock_release| spins| Thread 1 releases spinlock|
| wchan_sleep | sem_count++| Thread 2 gets spinlock and releases resource |
| sleeps| wchan_wakeone| Thread 1 is sleeping, Thread 2 wakes it up |
|ready | spinlock_release| Thread 1 is ready, Thread 2 releases spinlock |


## Condition variables

**condition variable is intended to work together with a lock.** 
**It is only used within the critical section that is protected by the lock.**

**Condition variable allows threads to wait for arbitrary conditions to become true inside of a critical section**

#### Operations:
1. ==**wait**==: Causes calling thread to ==*block*==, and releases the lock associated with the condition variable. Once the thread is unblocked, it ==*reacquires*== the lock.
2. ==**signal**==: If threads are blocked on the signaled condition variable, then ==*one*== of those threads ==*is unblocked*==
3. ==**broadcast**==: LIke signal, but unblocks ==*ALL*== threads that are blocked on the condition variable.


#### Example 1:

```c
int volatile numberOfGeese = 100;
lock geeseMutex;
cv zeroGeese;

int SafeToWalk() {
	lock_acquire(geeseMutex);
	while (numberOfGeese > 0) {
		cv_wait(zeroGeese, geeseMutex);
	}
	GoToClass();
	lock_release(geeseMutex);
}

// cv_wait() will handle releasing and reacquiring the lock passed in.
// It will put the calling thread in the cv's wait channel to block.
// cv_signal() and cv_broadcast() wake threads waiting on the cv.
```


#### Example 2:
```
int volatile count = 0; // must be 0 i
```