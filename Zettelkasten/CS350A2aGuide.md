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
			

create and copy.... (already a function to help exists)
	
attach/set child's address space
assign PID to the child process (unque) (no need to worry about reusing PID)
spawn a thread for chlld process (thread fork) -> needs an entry point (a special function we create which grabs/copy trapframe from the parent and calls mips usermode)
put trapframe on the stack 
usermode time

