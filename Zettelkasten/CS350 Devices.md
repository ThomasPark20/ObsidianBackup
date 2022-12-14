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
	- 
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
	- 
```
	P(output device write semaphore)
	write character to device data register
	(interrupt after completion)

	#Interrupt Handler
	Clear Write IRQ
	V(output device write semaphore)
```




## Summary
