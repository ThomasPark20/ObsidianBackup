2022-09-28 20:30
source: [waterloo]()
tags: #scratch #CS365 #OS #thread #threads 

#  2022-09-28 8crpmtc8Ppm3

## Interrupt

- an event that occurs during the execution of a program
- caused by system devices (timer, disk controller, nic, etc)

1. Hardware automatically transfers control to a fixed location in memory
2. ==Interrupt handler== procedure is placed by the thread library in that memory location

### I==nterrupt hander==
- creates a [[trap frame]] to record the thread context at the time of the interrupt
- determines the devices that caused the interrupt and performs device-specific processing
- restores the saved thread context from the [[trap frame]] and resumes execution of the thread
