2022-09-29 20:06
source: [source]()
tags: #pages #CS350 #thread #threads #synchronization #lock #spinlock #semaphore


# CS350 Synchronization


## Overview
Synchronization. How threads work together and mitigate danger.

## Details

==**[[Concurrent]]**== threads interact with each other in a variety of ways, sometimes sharing access to the program's code, global variables, and heap

Danger exists, where a shared object is accessed which must not be concurrently executed by more than one thread. This is called ==**critical section**==

We try to enforce ==**mutual exclusion (also refered to as mutex)**== to mitigate this danger.

We could try and use **[[volatile]]** keyword, but this is not enough. 

**Good illustration on typical race conditions we face**

![[Racecondition.png]]

## locks

A ==**lock**== is a mechanism to provide mutex.
