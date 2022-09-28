2022-09-28 17:32
source: [waterloo]()
tags: #pages #CS350 #OS #thread


# CS350Lect02


## Overview
Introduction to Threads

## Details

**Thread** is a ==*sequence of instructions*==

Normal sequential program will use 1 thread.

A single program however, can have 
 - ==*Different threads responsible for different roles*== (web browsers)
 - ==*Multiple threads responsible for the same roles*== (my tool at AWN)

Why Threads? [[Concurrent]] ! (This is different but includes Parallelism which is strictly hardware and is actually working at the same time using multiple cores)

Benefits include...
1. ==*Efficient Use of Resources*==
2. ==*Parallelism*== (multiple cores using multiple threads)
3. ==*Responsiveness*== (imagine having to wait for a website to load to watch video on another page ew)
4. ==*Priority*== can be set to get more processor time for certain events
5. ==*Modular code*== (separete UI from Web loading code)

`thread_fork` allows a thread to create new threads **This copies the stack and register values into the newly created thread**

![[ThreadFork.png]]

```c
// from kern/include/thread.h

int thread_fork(
const char *name, // name of new thread
struct proc *proc, // thread's process
void (*func) // new thread's function
	(void *, unsigned long),
void *data1, // function's first param
unsigned long data2); // function's second param
```

We can ==*Terminate*== threads by `void thread_exit(void);`
We can ==*Yield*== threads by `void thread_yield(void);`

### Register Definitions
![[Regdefs.png]]

**Register Usage**

==**Register Names**==: Referred by their default function e.g. v0, a0, t0

==**Passing Arguments**==: Pass first 4 arguments in registers a0-a3 and if more, pass on to stack

==**Return Values**==: v0. v1. CS241 `jalr` is `jal` here.

==**Saving Registers**== - Minimizing situations where the callee is saving unneccessary stuff:
- ==*Caller-save*==, if the ==calling== subroutine has a value in one of these registers that it wants preserved (t0-t7), it stores the value on the stack before calling other subroutine and restores it after.
- ==*Callee-save*==, if the ==called== subroutine uses one of these registers (s0-s7) it must first store its current value on the stack and restore it before exiting

**Concurrent Program Execution**
![[Twothread.png]]

Everything is shared among threads using the same code,
**EXCEPT STACK + REGISTER CONTENTS**

**Some things that I already understand so skipping**

==**Timesharing**==: Multiple threads take turns on the same hardware; Rapidly switching between threads so all make progress

==**Multicore Processor**==: a single die containing more than one processor unit (parallelism)

==**Multithreading**==: having multiple threads run on the same core
- special hardware required. (two sets of registers, but others shared like ALU and Data Mem)
