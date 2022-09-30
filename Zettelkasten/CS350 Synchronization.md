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
void 
```

## Semaphores