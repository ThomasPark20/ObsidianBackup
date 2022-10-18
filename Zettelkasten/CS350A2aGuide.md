2022-10-18 14:32
source: [source]()
tags: #scratch #CS350 #process #processes 

#  2022-10-18 2crpmtc2Ppm2

exit needs to keep track of exit code (leave behind for parent to call waitpid)
exit code clean up? (waitpid call. Anything left is cleaned when parent dies)
code for waitpid and exit are very coupled

Fork
create new process struct
	


create and copy.... (already a function to help exists)
attach/set child's address space
assign PID to the child process (unque) (no need to worry about reusing PID)
spawn a thread for chlld process (thread fork) -> needs an entry point (a special function we create which grabs/copy trapframe from the parent and calls mips usermode)
put trapframe on the stack 
usermode time

