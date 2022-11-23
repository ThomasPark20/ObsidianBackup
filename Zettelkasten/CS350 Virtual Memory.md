2022-11-23 15:08
source: [source]()
tags: #pages #CS350 #virtual_memory #OS 


# CS350 Virtual Memory


## Overview

sad memory time

## Details

### KMG
- ==B = byte, b = bit==
- ==1K = 2^10, 1M 2^20, 1G=2^30==
Key skills:
1. ![[Addrspacebitsconversion.png]]
2. ![[HowmanyXfitY.png]]


### Physical address
==provided directly by hardware, this is the actual memory==

- If ==P== bits to specify the physical address of each byte, max phys mem = 2^==P== bytes

### Virtual address
==fake. provided by OS to the processes==
- If ==V== bits to specify the virtual address of each byte, max ==V==mem = 2^==V== bytes

#### Why use VMEM?
- provide each process with the *illusion* that it has a large amount of contiguous memory available exclusively to itself. i.e. an address space
	- allows greater support for multiprocessing
- protect one process's address space from other (perhaps buggy) processes

### Dynamic Relocation

- Vaddr of each process is translated using:
	1. offset: stored in ==relocation register==, ==addr in pmem where the process's memory begins==
	2. limit: the amount of memory used by the process
To translate:
```c
if (v < limit) then
	p = v + offest
else
	raise memory exception
```
- Offset and limit are maintained by kernel for each process. 
- Kernel changes these values in the MMU register in case of context switch between processes
Example:
- if offset = 0x24000 and limit = 0x7000, phys mem \[0x24000, 0x2AFFF\]
#### Properties of Dynamic Relocation
- uses contiguous range of physical address
- potential ==fragmentation of physical memory==
	- OS must allocate/deallocate variable-sized chunks of physical mem since addr spaces may have different sizes
	- ==Example==
		- 