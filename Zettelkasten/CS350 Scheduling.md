2022-12-13 17:47
source: [source]()
tags: #pages


# CS350 Scheduling


## Overview
aaaaaaaaaaaaaaaaaaaaaaaa

## Details

#### Outputs & Performance Metrics

- Scheduler decides which job should be running on the server at each point of time
- For each job,
	- *Start time s_i*, when ith job starts running
	- *Finish time f_i*, when ith job finishes running
- ==Performance Metrics==
	- response time: s_i - a_i (time between arrival and run)
	- turnaround time: f_i - a_i (time between arrival and finish)

### FCFS (Round-Robin)
- simple, avoids starvation


### SJF (SRTF - Shortest Remaining Time First)
- minimizes avg turnaround time
- LONG JOBS MAY STARVE


### CPU Scheduling
- Difference between simple scheduling model:
	- run time not normally known
	- threads sometimes not runnable (blocked)
	- ==threads may have different priorities==
		- may be specified by the app or user
		- chosen by scheduler
		- combination of above
		- 
- Objective of the Scheduler
	- responsiveness: ensure that threads get to run regularly
	- fairness: sharing of the CPU
	- efficiency: account for the fact that there is a cost to switching

### MLFQ
- Good responsiveness for interactive threads - frequently blocked, devices etc
- how? give them higher priority so they run whenever ready

