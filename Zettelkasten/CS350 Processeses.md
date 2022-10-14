2022-10-05 18:25
source: [CS350]()
tags: #pages #CS350 #OS #kernel  #process #processes #syscall


# CS350 Processeses



## Overview

Processes in CS350 and how they are implemented
Syscalls in CS350 and how they are implemented

## Details


### Process - an ==environment in which application program runs in isolation== 
Process contains...
- ==Threads== to execute code on processors
- ==address space== for code and data (VMEM)
- a file system
- I/O: Keyboard, display, etc

Processes are ==created and managed by the kernel==


#### Variety of functions ([[System Call]]) regarding Processes

![[Processfunction.png]]


### fork
- creates new process ==**child**==, ==*that is a clone*== of the ==**parent**==.
	- copies P's address space (==code, globals, heap, stack==) in to C's
	- makes a new thread for C
	- copies P's ==trap frame== to the C's kernel stack
	- ==address space is identical at the time of fork, though it may diverge afterwards==
	- different return values
		- C = 0
		- P = C's pid

### \_exit
- terminates the process that calls it
	- process that dies can supply an exit status code
	- kernel records the exit status in case another process wants to hear it

### waitpid
- lets a process wait (block) for another to terminate (\_exit's exit status code thingy)

### getpid
- returns the ==process identifier== of the current process
- each existing process has its own unique pid

### execv
- changes the program that a process is running
- ==calling process's VMEM is destroyed==
	- Thus process gets new VMEM (code, heap, stack, etc) initialized with code and data of the new program to run.
- ==pid remains the same==
- can pass arguments to the new program, if required
- new program starts executing after execv

### [[System Call]] 

### Stacks

#### User Stack
 - used while the thread is executing application code
 - our usual stack in user space
 - holds stack frames (activation records)

#### Kernel Stack
- used while the thread is executing kernel code
- located in kernel space
- OS/161, the ==t_stack== field of the ==thread== structure points to this stack
- holds stack frames (activation records) for the kernel functions
- holds *trap frames* and *switch frames* (as ==it is the kernel that creates trap frames and switch frames==)

For diagram, check lecture 9 notes

### IPC: Inter-Process Communication
- family of methods used to *send data between processes*

- **File**: data to be shared is written to a file, accessed by both processes.
- **Socket**: data is sent via network interface between processes
- **Pipe**: data is sent, unidirectionally, from one process to another via OS-managed data buffer.
- **Shared Memory**: data is sent via block of shared memory visible to both processes.
- **Message Passing/Queue**: a queue/data stream provided by the OS to send data between processes.

