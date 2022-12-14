2022-12-13 22:07
source: [source]()
tags: #pages


# CS350 Devices & Disks



## Details

### Kind of Device
- keyboard -> input
- printer -> output
- touch screen -> both

### Bussin
- internal bus (memory bus) -> cpu and ram communication. fast
- peripheral bus (expansion bus) -> devices communication
- bridge -> connects two busses.

### Registers
- status -> device's current state (typically read)
- command -> issue command (write)
- data -> transfer larger blocks of data
![[clock.png]]
![[serial.png]]

### Device Driver

1. ==Polling==
	- Repeat (Poll) for completion
	- Waste of CPU cycle
	 ```
	 P(output device write semaphore)
	 write character to device data register
	 repeat {
		 read writeIRQ register
	 } until status complete
	 clear Write IRQ
	 V(output device write semaphore)
	```
2. ==Interrupts==
	- No repeat (poll)
```
	P(output device write semaphore)
	write character to device data register
	(interrupt after completion)

	#Interrupt Handler
	Write Write IRQ ack completion
	V(output device write semaphore)
```

- Why use semaphore? It doesn't matter which thread releases the device.
	- May be different threads that initiate write and recognize end write

### Accessing Devices

1. Port-Mapped I/O
	- Uses special assembly language I/O operations
	- device registers are assigned port numbers
	- FAST as # of instr is less
	- VERY restrictive as special I/O instructions are forced.
2. Memory-Mapped I/O
	- Each device register has a physical memory address
	- devices read write from these devices registers as if accessing memory
	- Supports more devices due to lack of special operations
	- SLOW as all devices must decode all instr that comes from mainbus to check if its for them... SLOW
### LARGE data transfer
1. Program-controlled I/O (PIO)
	- Device driver moves data between mem and buffer of device
	- USES CPU
2. Direct memory access (DMA)
	- Device it self moves data to/from mem
	- NO CPU
	- Used for block data transfers between devices (i.e. disk controller and primary memory)


### HDDDDDDDD
- array of numbered blocks (sectors)
	- Blocks are same size (512 bytes typical)

#### formula
1 to 4 platters
	2 surfaces per platter
		split into circles called tracks
			each track broken into sectors (blocks)

#### Cost model
1. seek time
	- Most dominant cost
	- time it takes read/write head to the appropriate ==track==
2. roational latency
	- time it takes for desired sectors to spin to the read/write heads
3. transfer time
	- time it takes until desired sectors spin past the read/write heads
==Request Service Time==
- SEEK TIME + ROTATION LATENCY + TRANSFER TIME

#### Some examples

Params
- Disk capa: 2^32 bytes
- Num of Tracks: 2^20 tracks
- Num sectors track: 2^8 sectors
- RPM: 10000
- MAX SEEK: 20 ms

a) How may bytes per track?
- Disk capa / Num of Track -> 2^12 bytes

b) How many bytes in sector?
- Bytes per track / Num sectors track = 2^4 bytes

c) Max rotational latency?
- 60 / RPM = 0.006 or 6 ms

d) What is the average seek time?
- Max SEEk / 2 = 10 ms avg seek time

e) AVG rotational latency?
- Max ROT / 2 = 3ms avg rot lat

f) What is the cost to transfer 1 sector? (==transfer== meaning track was already found) (This is actually asking for transfertime)
- MAX ROT / Num sectors track = 6 / 2^8 = 6 / 256 = 0.0195 ms per sector

g) expected cost to read 10 consecutive sectors? (expected -> avg like stat)
- AVG SEEK + AVG ROT + 10(TRANSFER TIME) = 10 + 3 + 10(0.0195) = 13.195ms

#### Performance characteristics
- Larger tranfers (many blocks) more efficient
	- cost per byte is smaller
		- because the time it takes to get there is amortized over many blocks of data
- Sequential I/O faster than non-sequential I/O
	- Eliminate seek time!


#### Disk Head Scheduling

- Goal: reduce seek time.

##### FCFS
- fair and simple, no optimization

##### SSTF (Shortest Seek Time First)
- Seek time reduced, STARVATION exists

##### SCAN (Elevator)
- moves in one direction until no more request in front of it, then reverse
- Seek time reduced (relative to FCFS), NO starvation
#### Disk Controller driver
![[discon.png]]

- Constraints
	- OS/161 allow one thread at a time to access disk
	- OS/161 thread that initiates a write/read should wait until that write/read is completed

##### Write
```c
// allow only one disk request at a time
P(disk)
copy data from RAM to device transfer buffer
write target sector number to sector number register
write write command to the status register
// wait for request to complete
P(disk_completion)
// when completed
V(disk)

// Interrupt hander for Disk Device
write disk status register to ack completion
V(disk_completion)
```

##### Read
```c
// allow only one disk request at a time
P(disk)
write target sector number to sector number register
write read command to the status register
// wait for request to complete
P(disk_completion)
copy data from device transfer buffer to RAM
// when completed
V(disk)

// Interrupt hander for Disk Device
write disk status register to ack completion
V(disk_completion)
```


### SSD and flash
- no disk only integrated circuits

- read/write at page level
- overwrite/delete at block level
	- due to high voltage required to 0 -> 1
	- can only apply high voltage at block level

#### Writing and Deleting
- Naive solution (SLOW)
	- read whole block
	- re-initialize block
	- update block in RAM and write back to SSD
- SSD controller handles requests (faster)
	- page to be deleted/overwritten is marked as invalid (not del lmao)
	- write to an unused page
	- update translation table
	- requires garbo collection (use naive)

#### Limitations of SSD
- Each block has a limited number of write cycles before it becomes read-only
- SSD controllers perform wear leveling, distributing writes evenly


## Summary
