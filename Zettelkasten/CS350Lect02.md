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

**Skipping end**

### ==**Context switch**== - the switch from one thread to another

**What happens here?**
- Decide which thread will run next (==Scheduling==)
- ==Save== register contents of the current thread
- ==Load== register contents of the next thread

==**Thread context**==: the register values that must be saved/restored

```c
/*

* a0 contains the address of the switchframe pointer in the old thread.

* a1 contains the address of the switchframe pointer in the new thread.

*

* The switchframe pointer is really the stack pointer. The other

* registers get saved on the stack, namely:

*

* s0-s6, s8

* gp, ra

*

* The order must match <mips/switchframe.h>.

*

* Note that while we'd ordinarily need to save s7 too, because we

* use it to hold curthread saving it would interfere with the way

* curthread is managed by thread.c. So we'll just let thread.c

* manage it.

*/

  

/* Allocate stack space for saving 10 registers. 10*4 = 40 */

addi sp, sp, -40

  

/* Save the registers (callee saved)*/

sw ra, 36(sp)

sw gp, 32(sp)

sw s8, 28(sp) /* Frame Pointer */

/* s7 is not used as OS needs it to keep track of stack pointer in the current thread */

sw s6, 24(sp)

sw s5, 20(sp)

sw s4, 16(sp)

sw s3, 12(sp)

sw s2, 8(sp)

sw s1, 4(sp)

sw s0, 0(sp)

  

/* Store the old stack pointer in the old thread */

sw sp, 0(a0)

  

/* Get the new stack pointer from the new thread */

lw sp, 0(a1)

nop /* delay slot for load */

  

/* Now, restore the registers */

lw s0, 0(sp)

lw s1, 4(sp)

lw s2, 8(sp)

lw s3, 12(sp)

lw s4, 16(sp)

lw s5, 20(sp)

lw s6, 24(sp)

/* s7 is not used as OS needs it to keep track of stack pointer in the current thread */

lw s8, 28(sp) /* Frame Pointer */

lw gp, 32(sp)

lw ra, 36(sp)

nop /* delay slot for load */

  

/* and return. */

j ra

addi sp, sp, 40 /* in delay slot */

.end switchframe_switch
```