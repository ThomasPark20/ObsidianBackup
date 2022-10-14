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
- 