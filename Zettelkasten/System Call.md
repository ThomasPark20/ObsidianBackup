2022-09-28 17:31
source : [source]()
tags: #tips #syscall #cybersec #pwn #CS350 #process #processes 

# System Call


## What
- interface between user process and the kernel
- various languages have their own system call library for using OS services

## When
- When user processes want to say hi to the kernel

## How

1. Interrupts
	- ==generated by devices== when they need attention
2. Exceptions
	- ==caused by instruction execuition== when running program needs attention

#### Interrupts
- raised by devices (hardware)
	- e.g. system clock , WIFI card got a packet, SSD accessed some data
- causes the processor to ==transfer control== to a fixed location in memory (depends on the processor) where an ==**interrupt handler**== must be located
- causes the processor to switch in to priviledged mode and use kernel stack.

#### Exceptions
- ==conditions== (not necessarily errors) ==that occur during the execution of a program instruction== 
	- e.g. overflow, illegal instructions, illegal address, page faults, system calls.
- Same as interrupts, sent to a specific ==**exception handler**==

In both cases, processor stores the cause of this in the *cause* register. Both handlers use this code to determine what happened.

### System Call Software Stack (OS/161 MIPS)

1. The *application* calls a library wrapper function for the sytem call.
2. The *library wrapper function* executes a ==syscall== instruction causing...
3. ... the kernel's *exception handler* to run. (MIPS uses same routine for both interrupts and exceptions so everything goes to the exception handler)
	1. creates trap frame to save application program state
	2. determines that this is a system call exception (==EX_SYS 8==) (REF: kern/arch/mips/include/trapframe.h)
	3. determines which system call is being requested (==Syscall codes== saved in register v0) (REF: kern/include/kern/syscall.h)
	4. does the work for requested syscall
	5. restores the application program from the trapframe
	6. returns from the exception
4. *library wrapper function* finishes and returns
5. *application* can continue execution.
