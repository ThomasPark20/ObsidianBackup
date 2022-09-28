
2022-09-28 17:04
source: [Waterloo]()
tags: #pages #CS350 #OS #kernel 


# CS350 Intro to OS


## Overview
Introduction to OS.


## Details

### Three views of an OS
1. **Application View - What services does an OS provide?**
	- Provides Program with ==*resources*== (processor time, memory space, access to I/O devices like the keyboard and monitor)
	- Provides Program with ==*interfaces*== (networks, storage, I/O devices, and other system hardware components)
	- ==*Isolates running programs*== (prevent undesirable interactions)
2. **System View - What problems does an OS solve?**
	- ==*Manages the hardware resources*== of a computer system.
	- ==*Allocates resources*== among running progrmas
	- ==*Controls the access to*== or sharing of resources among programs
3. **Implementation View - How is it built?**
	- The OS is.. [[Concurrent]] , [[real-time]] program

**How does the OS implement concurrency and hardware interactions?**

==**[[kernel]]:**== Part of the OS that responds to syscalls, interrupts and exceptions

![[Schematic View of an Operating System.png]]

**Bunch of Terms related to OS**
- ==*User Programs*== are programs that the user intearcts with directly
- ==*User Space*== are all programs that run outside the kernel (no direct interaction with hardware)
- ==*[[System Call]]*== how a user process interacts with the OS. A programmer uses the C function `fopen` , which is implemented by making a syscall to the OS (in linux. `sys_open`)
- ==*Kernel Space*== region of memory where the kernel runs