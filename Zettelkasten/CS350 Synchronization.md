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

## locks

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
Rather than a single ``