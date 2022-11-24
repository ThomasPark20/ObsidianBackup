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
		- 100MB free, not contiguous.
		- process needs 100MB. 
		- Even though free, not contiguous. cannot assign
- In the end it allows ==multiple processes to share RAM== and ==isolates each process's address space==, but it ==does not use phys mem efficiently==

### Segmentation

#### Properties of Segmentation
- we segment the addr space
- ==kernel== maintains an ==offest and limit value for each segment==
- Vaddr has two parts: (==segment ID. offset within segment==)
- With K bits for the segment ID, we can have up to:
	- ==2^K== segments
	- ==2^{V-K}== bytes per segment
- kernel decides where each fragment is placed in phys mem
- ==Fragmentation still possible==
- ==Limitation:== space for heap and stack cannot grow beyond a certain point.
#### Approach 1
- For each segment i, the MMU has
	- a relocation offset register: offset\[i\] and
	- a limit register: limit\[i\]
- To translate
	```python
	s = SegmentNumber(v) # segment id
	a = OffsetWithinSegment(v) # segment's offset
	if (a >= limit[s]):
		raise memory exception
	else
		p = a + offset[a]
```
	![[SegmentedAddressTranslationA.png]]
- 0x1240 -> 0001 0010 0100 0000
	- Segment (1bit) -> 0
	- Offset (rest)-> 001 0010 0100 0000 -> 0x1240
	- segment offset < limit -> no exception. Thus,
	- 0x38000 + 0x1240 = 0x39240
#### Approach 2
- we now have a table.
- ![[Pasted image 20221123185713.png]]
- 0x00002004 -> 0000 0000 0000 0000 0010 0000 0000 0100
- seg num = 0x0
- offset = 0x 000 0000 0000 0010 0000 0000 0100
- 0x0 < 0x4 (segment number) thus no exception
- 2004 < 6000 thus no exception
- p = 0x72004
### Paging
- Divide Physical memory into fixed-size chunks called frames (physical pages)
- Page size must equal frame size
- ==Each process has its own page table==
Calculations:
- physical address 18 bits
	- size of phys mem -> 2^18 = 256KB
- frame size = 2^12 = 4KB
	- phys mem has 2^18 / 2^12 = 2^6 = 64 frames
- v address 16 bits
	- 2^16 / 2^12 = 2^4 = 16 pages
- N of PTEs = Max Vmem size / Page size
#### Translation
- MMU has ==page table base register== that p