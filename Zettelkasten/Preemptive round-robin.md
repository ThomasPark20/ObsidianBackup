2022-09-28 20:44
source: [waterloo]()
tags: #scratch #OS #CS350 #thread #threads 

#  2022-09-28 8crpmtc8Ppm3

## Preemptive round-robin

Preemptive round-robin scheduling:

- scheduler maintains a queue of threads, called ==*ready queue*==
- on context switch, the running thread is moved to the end of ready queue, and the first thread on the queue is allowed to run
- Newly created threads are placed at the end of the ready queue