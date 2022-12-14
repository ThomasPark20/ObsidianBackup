2022-12-14 00:42
source: [source]()
tags: #pages #CS350 #OS #kernel 


# CS350 File Systems


## Details

### Files
- when we edit, change is not persistent until explicit save
- "c = 2*(a+b)" -> a,b,c are named data objects, (a+b) is an intermediate result
- data consists of a sequence of numbered bytes
	- can ask for ith byte
- have associated meta-data (owner, date created, date modified etc)

### File Systems
1. logical file system (top layer)
	- (fopen, fprintf, etc)
	- Manages file system information (which files are open for read/write)
2. virtual file system (mid)
	- abstraction of lower level file systems, presetns multiple different underlying file systems to the user as one.
3. Physical file system (low)
	- how they are actually stored (track, sector, magnets)


### File interface basics
common operations
- open
	- returns a ==file descriptor== 
		- has bunch of parameters which other file operations use
- close
	- invalidates a valid ==file descriptor==
	- kernel tracks which file descriptors are currently valid for EACH process
- read, write, seek
	- read
		- copies data from file in to a virtual address space
	- write
		- copies data from a virtual addresss space into a file
	- seek
		- enables non-sequential reading/writing
		- changes the ==file position== associated with a descriptor.
			- next read/write will use this new position

### Directories and i-numbers

- A directory maps file names to ==i-numbers==
	- ==i-numbers== are unique index numbers which file system uses to locate files
- Q: Only the kernel is permitted to directly edit directories. Why?
- A: ==Kernel should not trust the user with direct access to data structures that the kernel relies on==

### Links
- hardlink
	- association between a name (string) and an i-number
	- open("/docs/a.txt", "O_CREAT|O_TRUNC")
		- opens file and creates a hardlink to that file in the directory /docs
	- link("/docs/a.txt", "/foo/myA")
		- creates a new hardlink myA in directoy /foo
		- link refers to the i-number of file /docs/a.txt which must exist
		- CANNOT LINK DIREC TO AVOID CYCLING
![[Pasted image 20221214014120.png]]

==File is deleted when its last hard link is removed and the count of the number of hard links maintained by kernel becomes zero.==
- softlink
	- symlink(/z/a, /y/k/m) creates symlink m in directory /y/k
	- If we open /y/k/m
		- recognize /y/k/m as symlink
		- attempt to open /z/a instead
	- ==referential integrity not preserved==
		- /z/a need not exist. (It wiill create error tho)
![[Pasted image 20221214014416.png]]

### Multiple File Systems




## Summary
