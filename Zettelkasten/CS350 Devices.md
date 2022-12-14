2022-12-13 22:07
source: [source]()
tags: #pages


# CS350 Devices



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




## Summary
