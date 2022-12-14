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
Why the separation?
	1. Security. If a user program can directly access data, user may be able to bypass permissions
	2. Good abstract design. The OS can change the implementation or hardware without affecting user program
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
		- doesn't validate the bit. can set bit wayyyy off. Won't get error until read/write

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
virtual file sys.
How do we present multiple filesystems to logical filesystem as one?

- DOS/Windows -> use two part file names
	- ==C:== \\user\\cs350\\schedule.txt
- Unix -> create single hierarchical namespace
	- Unix mount system call
		- does not actually make two in one. merely creates a namespace that combines namespace
![[Pasted image 20221214014844.png]]

### Implementation
- What nees to be stored persistently?
	- file data
	- file meta-data
	- directories and links
	- file system meta-data
- non-persistent (may be around if hibernation (not OS/161))
	- per process open file descriptor table
		- file handle
		- file position
	- system wide
		- open file table
		- cached copies of persistent data

### Example Disk
- 256KB disk
- sector 512 bytes
- block -> 8 sectors -> 4KB block
	- ==better spatial locality==
	- ==Reduces number of block pointers==
	- ==4 KB is convenient size for paging==
- 64 total blocks on disk

### VSFS
- last 56 blocks is data.
	- data block is not shared by files
- first 8 blocks metadata.
	- 5 blocks for i-node array
		- inodes (max 80) -> one per file (256 bytes)
			- contains 
				- i number
				- file permission
				- file type
				- file length
				- modification table
				- those damned pointers
				- etc
		- basically page table for where in data can I find this file

Need to know which i-nodes and blocks are unused to create new file
==searching i-node array is linear time and slow==
We use bitmap for this. (still linear search, but cheaper to read in binary int)
- 2 blocks for i and d bitmap
	- block size of 4KB means 32K i-nodes and 32K blocks trackable

- first block is superblock
	- contains meta-info about entire file system
		- how many i-nodes and blocks, where i-node table begins etc
![[Pasted image 20221214015808.png]]

### i-node and its damned pointers
![[Pasted image 20221214015946.png]]

Assume disk blocks can be referenced based on 4 byte address (32bits)
- 2^32 blocks, 4KB sized blocks
- max disk size is 16TB

i-node is 256 bytes
- 12 direct pointers only
	- 12 * 4 kb = 48 kb is max size
	- oK so how get more size?

#### Indirect Pointers
- points to a block full of direct pointers (these eat up data space)
- 4 KB block of direct pointers = 2^12 / 2^2 (size of pointer) = 2^10 = 1024 pointers
- Thus, as we have 12 direct and 1 indirect that points to block full of direct pointers,
	- max file size -> (12+1024) * 4kb = 4144 kb
- double, triple
	- points to a 4KB block o single indirect pointers 
		- \[12(direct) + 1024(single) + 1024 * 1024 (double)\] * 4kb
	- points to a 4KB block of double indirect pointers
		- \[12(direct) + 1024(single) + 1024 * 1024 (double) + 1024^3 (triple)\] * 4kb

Example:
Open & Read /foo/bar

Open (5 reads)
1. root inode read
2. root data read (lookup i number for foo)
3. foo inode read
4. foo data read (lookup i number for bar)
5. bar inode read 
	- permission checked
	- file descriptor returned and added to processes's file descriptor table
	- file is added to kernel's list of open files

Read (2 reads)
1. bar inode read again
	- find pointer to correct data block to read (after checking file descriptor for where to read and how much to read)
2. read bar data
3. bar inode write (atime)

Create & Write /foo/bar

Create (6 reads, 10 disk I/O)
1. root inode read
2. root data read
3. foo inode read
4. foo data read (no bar)
5. inode bitmap read
6. inode bitmap write (0 -> 1 for a new i number)
7. foo data write (hardlink new inumber)
8. bar inode read (as we have to read entire block to access this small inode)
9. bar inode write (new inode stuff: permissions, type, pointers init etc)
10. foo inode write (mtime ==modification== time!!)

Write (2 reads)
1. read bar inode (we already know where this is as we created right before this)
	- check if we need direct single or double pointers etc
	- we know that no data yet for this inode
2. read data bitmap (find space)
3. write data bitmap (get space)
4. write bar data
5. write bar inode (filesize, pointers, mtime)

If new write 49KB, we need 12 direc and 1 indirec


Delete (may be incorrect) /foo/bar
1. root inode read
2. root data read
3. foo inode read
4. foo data read
5. bar inode read
6. bar inode write (decrement hardlink counter.)

If hardlink was 0,
7. data bitmap read
8. data bitmap write (1 -> 0)
9. inode bitmap read
10. inode bitmap write (1-> 0)


### Chaining
- VSFS uses per-file index (direct and indirect pointers) to access blocks
- Two alterative approaches
	- Chaining![[Pasted image 20221214070517.png]]
		- Directory table contains the name of the file, and each file's STARTING block
		- Acceptable for sequential access, very slow for random access
			- if a file had 3 blocks, we have to read all 3 blocks to access the last block... imagine if we wanted to access last blocks of 100 files. 
	- External Chaining![[Pasted image 20221214070538.png]]
		- speical file access table that specifies all of the file chains
			- instead of seq reading actual files, we read smaller data structure that tells us the required info to find desired block of data.

### File System Design
- File system params:
	- How many i-nodes should a file system have?
	- How many direct and indirect blocks should an i-node have?
	- What is right block size?
- For general purpose fs, design it to be efficient for the common case
	- most files are small, 2kb
	- average file size growing
	- on average 100 thousand files
	- typically small direcs

### Faults and Failures
- special-purpose consistency checkers (fsck, etc)
	- runs after a crash, before normal ops resume
	- find and attempt to repair inconsistent file system data structures
- journaling, write-ahead logging
	- record file system meta-data changes in journal in sequence
	- UPDATE only AFTER journaling (write-ahead logging)
	- after failure, redo journaled updates.

## Summary
