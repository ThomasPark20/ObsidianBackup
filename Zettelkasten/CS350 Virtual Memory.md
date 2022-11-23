2022-11-23 15:08
source: [source]()
tags: #pages #CS350 #virtual_memory #OS 


# CS350 Virtual Memory


## Overview

sad memory time

## Details

### Physical address
==provided directly by hardware, this is the actual memory==

### Virtual address
==fake. provided by OS to the processes==

#### Why use VMEM?
- provide each process with the *illusion* that it has a large amount of contiguous memory available exclusively to itself. i.e. an address space
- protect one process's address space from other (perhaps buggy) processes

### KMG
- ==B = byte, b = bit==
- == 1K = 2==