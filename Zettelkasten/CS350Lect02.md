2022-09-28 17:32
source: [waterloo]()
tags: #pages #CS350 #OS #thread


# CS350Lect02


## Overview
Introduction to Threads

## Details

**Thread** is a ==*sequence of instructions*==

Normal sequential program will use 1 thread.

A single program however, can have 
 - ==*Different threads responsible for different roles*== (web browsers)
 - ==*Multiple threads responsible for the same roles*== (my tool at AWN)

Why Threads? [[Concurrent]] ! (This is different but includes Parallelism which is strictly hardware and is actually working at the same time using multiple cores)

Benefits include...
1. ==*Efficient Use of Resources*==
2. ==*Parallelism*== (multiple cores using multiple threads)
3. ==*Responsiveness*== (imagine having to wait for a website to load to watch video on another page ew)
4. ==*Priority*== can be set to get more processor time for certain events
5. ==*Modular code*== (separete UI from Web loading code)

`thread_fork` allows a thread to create new threads
```c
int thread_fork(const char *name, struct proc *proc,

void (*func)(void *, unsigned long),

void *data1, unsigned long data2);
```