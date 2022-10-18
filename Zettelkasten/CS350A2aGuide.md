2022-10-18 14:32
source: [source]()
tags: #scratch #CS350 #process #processes 

#  2022-10-18 2crpmtc2Ppm2

exit needs to keep track of exit code (leave behind for parent to call waitpid)
exit code clean up? (waitpid call. Anything left is cleaned when parent dies)
code for waitpid and exit are very coupled

Fork
create new process struct
	add new fields to process struct (pid etc)
	proc_create_runprogram
		this sets up VFS and console (don't worry now)
	check for errors
		what if proc_create_runprogram returns NULL? 
			modify trapframe of parent process a3 v0 (register values) (example in course notes?)
			

create and copy address space (already a function to help exists)
	child process must == parent
	as_copy() copies a new address space and copies pages from old to new
	check for errors
		return error code
	Associate address space not associated with new process yet
		look at curproc_setas() (use as template) to figure out how to give a process an address space
	check for errors
		
attach/set child's address space
	
assign PID to the child process (unque) (no need to worry about reusing PID)
	give unique PID and PID>0 (as 0 is what is returned in child code thingy in lecture notes)
		can assume max of 64 PIDs needed and just have a global counter that gets incremented
		proc_bootstrap() would be a good plaace to initialize that counter (what initializes shit before os start..?)
		provide mutex for global structure
			where to initialize lock?? must be created before everything
				proc_bootstrap(). (proc.h) 
					edit proc_bootstrap() at the end.
	could use process's address as PID (after making it positive etc) (unique by design if positive done correctly)

spawn a thread for chlld process (thread fork) -> needs an entry point (a special function we create which grabs/copy trapframe from the parent and calls mips usermode)
	copy trapframe on child kernel stack
	update a3 v0 (0,0 as child creation was sucess)
	increment PC by 4
enter_forked_process (change to void pointer in def as thread_fork only consumes void pointers)
before calling thread_fork, in the parent process:
struct trapframe * tf_copy = Kmalloc(size of trapframe struct)
lock
	*tf_copy = current process's trapframe
unlock
then enter_forked_process runs with copied trapframe (free it after enter_forked_process)
struct trapframe tf_copy; (as we are running in kernel if inside enter_forked_process)
tf_copy = *tf; //cast in to a trapframe
// free tf not tf_copy
// modify tf_c: a3, v0, epc (epc is probably counter)
mips_usermode(...);
put trapframe on the stack 
usermode time

waitpid
condition variable for the condition is child dead?

getpid
lock it

exit
if parent dead, don't generate status code data
if parent not dead, generate data






