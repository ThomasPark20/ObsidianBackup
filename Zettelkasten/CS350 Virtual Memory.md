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
	- ==support virtual memory that is larger than physical memory==
		- allows greater support for multiprocessing
- ==protect one process's address space from other (perhaps buggy) processes==

### Dynamic Relocation

- Vaddr of each process is translated using:
	1. offset: stored in ==relocation register==, ==addr in pmem where the process's memory begins==
	2. limit: the amount of memory used by the process
- ==MMU has these==
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
	- large amounts of wasted space results in less effeciency due to large size address spaces.

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
- MMU Now has Seg table base reg and len reg![[SegmentedAddressTranslationB.png]]
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
- Number of Bits for offset = log(Page size)
- Number of Bits for Page number = log(Number of PTEs)
- Page Table Size = Num of Pages x size of a PTE
#### Translation
- MMU has ==page table base register== that points to the page table for the current process
1. determine the page number and offset of the Vaddr
	- page number = v address divided by page size
	- offset = v address module page size
2. look up the PTE using page number
3. check if PTE valid if not, exception
4. combine the corresponding frame number with the offset to get paddr
	- paddr = (frame number x frame size) + offset
![[Pasted image 20221123194822.png]]![[Pasted image 20221123194836.png]]
0x102C -> page 0x1 -> valid -> add -> 0x2602C
basically find how many bits each part has and look at table

#### Other Info in PTE
1. write protection bit
	- If a write operation uses vaddr on read-only page, MMU will raise exception when it translate the vaddr

### Multi-level Paging
- split the page table into multiple levels
- no valid PTE? do not create that table

==address translation is the same as single-level but lookup is different==

Example. 0x58B4, 2 levels, each page num uses 2 bits

- need to know the number of bits for each page and the num of levels (given above)
- vaddr has 3 parts: p1, p2. offset
- 0x58B4 -> 0101 1000 1011 0100
- Thus, p1 -> 01 p2 -> 01 offset 1000 1011 0100
- find entry 01 from table 1
- find entry 01 from table 2, add offset.

#### calculations.. again
If V = 40 -> 2^40, page size = 4KB = 2^12, and PTE size 4 bytes...
- Num of pages and PTEs needed = 2^40 / 2^12 = 2^28 pages
- Num of PTE's that can be stored on each page = 2^12 / 2^2 = 2^10 PTEs
- How many tables are needed to store all 2^28 PTEs? (as 2^28 pages)
	- 2^28 / 2^10 = 2^18 page tables
	- Thus top page table (==directory==) must hold 2^18 references to page tables
	- requires 2^18(# references) x 2^2 (PTE size) = 2^20 or 1MB of space

More formulas:
how many levels?
- ceil((Vbit - offset bit) / (page table bit))
How many table exist each level, how many valid entries each table and per level?
- 64 bit. 1 -> 12; 2,3 -> 15; offset = 20
- Process uses bytes 2^58 - 1.
	- Level 3
		- valid entries per level -> 2^58 / 2^20 = 2^38
		- valid per table -> min(2^L3bits, valid per level) -> 2^16
		- Num table -> 2^38 / 2^16 = 2^22
	- Level 2
		- 2^22
		- 2^16
		- 2^6
	- level 1
		- 2^6
		- 2^6
		- 1

### Kernel vs MMU
- Kernel (software)
	- manages MMU on address space switches
	- creates and manages page tables
	- manages (allocate/deallocate) physical memory
	- handle exceptions raised by the MMU
- MMU (hardware)
	- translate vaddr to paddr
	- check for and raise exceptions when necessary

### TLB
- ==No TLB can be slow due to==
	- So, at least one memory operation is required for all asm instruction.
	- Doing this with only page table adds minimum of one extra memory operation
- What is TLB?
	- small. fast, dedicated cache of addr translation in the MMU
	- 1 entry 1 mapping (page # -> frame #)
- ==How does MMU translate a vaddr on page p?==
	- Hardware TLB
		- MMU handle TLB misses, including page table look up and replacement of TLB entries. 
		- No room for you to change it up
	- Software TLB
		- MMU only ever reads from TLB.
		- you can change up page table structure
#### OS/161 (MIPS R3000)
- 64 entry room
- each entry is 64 bits
- TLBLO_DIRTY (write permission) = 1: you can write to this page
- v,paddr -> 32 bits. 
- page size -> 4KB (2^12 thus 12 bits)
	- Therefore, each addr need to have 12 bits front to store offset
		- Meaning frame number and page number are 20 (32 - 12) bits each

### Paging Summary
Benefits
- paging does not introduce external fragmentation
- Multi-level paging reduces the amount of mem required to store page-to-frame mappings
Costs
- TLB misses are increasingly expensive with deeper page tables.
	- TLB miss for a 3 level page table requires minimum three mem access

### dumbvm
- ==OS/161 uses segmentation for user programs (processes) need to convert them to pages==

#### addrspace
![[addrspace.png]]
- as_vbase: starting address of each segment in vmem
- as_pbase: starting address of each segment in pmem (relocation)
- as_npages: size of each segment in number of pages
- stackpbase: base of stack in phys mem only (==as size, top and bottom of stack in vmem is fixed for all processes==)
#### Translation
1. calculate the ==top== and ==bottom== address for each segment
2. top -> base + npages * PAGE_SIZE (predefined as 0x1000 or 4096 or 4K)
3. bottom -> just base
4. ==different for stack== as it grows the opposite way!
	1. base -> USERSTACK (0x8000 0000) - DUMBVM_STACKPAGES (0xC) * PAGE_SIZE
	2. stacktop = USERSTACK (0x8000 0000)
```python
if (faultaddress >= vbase1 && faultaddress < vtop1): # in code segment?
	paddr = (faultaddress - vbase1) + as->as_pbase1
else if (faultaddress >= vbase2 && faultaddress < vtop2): # in data segment?
	paddr = (faultaddress - vbase2) + as->as_pbase2
else if (faultaddress >= stackbase && faultaddress < stacktop): # in stack?
	paddr = (faultaddress - stackbase) + as->as_stackpbase
else:
	return EFAULT
```
![[dumbvmexample.png]]

0x0040004 is code segment -> 0x0040004 - 0x0040000 + 0x0020000 -> 0x0020004

0x8000000 - 0xC * 0x1000 = 0x7FFF4000 <= ==0x7FFF41A4== < stacktop -> is stack -> 0x7FFF41A4 - 0x7FFF4000 + 0x00100000 = 0x001001A4

Where do these vaddrs come from?
- ==from the program's object file (elf format in our case)==
- not stack

### Initializing an Addr Space
- When kernel creates a process
	- create an address space
	- load the program's code and data
- OS/161 pre-loads addr space before program run
	- Other OS load on demand
		- faster, not use up physical memory for unused features
### ELF Files
- contain addr space segment desriptions
	- Header describes the segment ==images==
		- vaddr of the start of segment
		- length of segment in vaddr
		- location of segment in ELF file
		- length of segment in ELF file
	- Identifies vaddr of the program's entry point: the first instruction
	- contains lots of other info too
- OS/161 dumbvm assumes...
	- a ==text (code) segment exists==
		- contains program code and read-only data
	- a ==data segment exists==
		- contains other global program data
- OS/161 dumbvm creates...
	- ==stack segment==
		- Why not in ELF file?
			- ==Initial contents of stack are unkown until run time (think user input!)==
		- 12 pages long, running from vaddr 0x7FFFC000 to 0x7FFFFFFF
	- ==If ELF smaller than segment, zero fill the rest==
### Kernel Vmem
Two challenges
- Bootstrapping: Since the kernel helps to implement vmem, how can kernel run in vmem when it is just starting?
	- ==Solution is architecture-specific==
- Sharing: Sometimes data needs to be copied between kernel and apps. How can this happend if they are in different vaddr?
	- ==make the kernel's vmem overlap with process' vmem.==

OS/161
- sys/161 only supports 1GB of pmem
- kernel Vmem has 3 segments
	1. ==kseg0 ==(512MB) : kernel data structure, stacks, code, page table, etc
		- subtract 0x8000 0000 to translate (TLB not used)
		- maps to first 512MB of Pmem
	2. ==kseg1== (512MB) : for addressing devices (like hdd)
		- subtract 0xA000 0000 to translate (TLB not used)
		- maps to first 512MB of Pmem
	3. kseg2 (1GB) : not used
- kernel Pmem is divided into frames managed by the kernel in the coremap
	- coremap -> Which frames are in use? checker. (Which process using this frame?)

### Swap

Goals:
1. Allow vaddr space larget than installed RAM
2. Allow greater multiprogramming levels by using less RAM for each process

How?:
1. Allow vmem pages to be stored in secondary storage
2. ==Swap== pages (or segments) between secondary and primary so that its in primary when needed.

What do we need?:
- present bit -> if in pmem, 1

### Page Fault
- present bit is zero

- Hardware-managed TLB
	- MMU detects this and raises exception, which kernel must handle
- Software-managed TLB
	- Kernel detects this after TLB miss and checking the missing page's PTE... and?
	- TLB SHOULD NOT CONTAIN ANY ENTRIES THAT ARE NOT PRESENT IN RAM

- What should the Kernel do?
	 1. Swap the page into memory from secondary storage (possibly evicting other page to secondary)
	 2. Update PTE (set the present bit)
	 3. Return from exception
	 ![[Pasted image 20221124172354.png]]

### Optimal Page Replacement
- MIN
	- Replace the page that will not be referenced for the longest time
	- This is a theoretical policy, as it needs knowledge of future to implement
- Locality
	1. temporal locality
		1. likely access recently accessed pages
	2. spatial locality
		1. likely access parts close to parts recently accessed
- LRU
	- evict page that has not been used in the longest period of time
- ==Challenges using LRU==
	- must track usage and find a maximum value
	- kernel not aware which pages a program is using until exception (TLB miss)
- ==Solution to this==
	- Have the MMU track page accesses in hardware
		- use bit (reference bit) to each PTE.
			- set by MMU each time the page is used.
			- read and ==cleared== by the kernel.
- Clock Algorithm
	- a) detected on TLB miss when TLB is full
	- b) if potential victim has use bit 1, reset to 0 and iterate to next victim
	- c) If use bit 0, remove entry from TLB and bring in new address translation
	- d) set the PC to re-execute the instruction
	- e) return from exception
```python
while (use bit of victim is set) {
	clear use bit of victim # 2nd chance
	victim = (victim + 1) % num_frames
}
evict victim # use bit was 0
victim = (victim + 1) % num_frames
```
![[Pasted image 20221124185739.png]]