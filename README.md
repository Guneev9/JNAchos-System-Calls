# JNAchos-System-Calls

Pre-Requsites:
Use muliprogramming JNachos file for this system calls implementation.

1) Fork System Call:

Understanding:
Fork creates and exact copy of the currently running process. Notice in user space this implies copying. The return value 
for the parent should be the process ID of the newly created child process.The return value from the child should be 0. 

Consider the following general outline:
(1) Create a new NachosProcess (the child).
(2) Copy the memory space of the parent to a new section of RAM for the child, in the process setting up the childs memory map.
(3) Copy the parents registers into the childs set of registers with the notable exception that the return value of the 
    Fork system call should be different.
(4) Call the NachosProcess::fork member function make the child Ready

2) Join System Call:

Understanding:
The Join system call will should make the invoking process wait until the process with the specified PID has completed. 
Notice that there is currently no PID in JNachos, you will need to add one.If the processs passes in its own PID, join should
return immediate. If the process passes in an invalid PID (process does not exist or has completed), join should return
immediately. The specified process is not required to be a child of the calling process.

(1) Ensure that the specified process exists.
(2) Save a pointer to the invoking process some where that can be accessed later (new data structure)
(3) Put the invoking process to sleep.
(4) Every time a process calls the Exit System call, check if any other process is waiting for it to finish.
(5) If so (previous), save the input to Exit (exit status) to the waiting process so that it is the return value from Join.
(6) Call readyToRun on the waiting process so that it will run again.

3) Exec System Call:

Understanding:
The Exec system call takes as a parameter one of the user programs stored in the test directory. The processs memory space 
should be replaced with the address space of the specified executable program. 

Consider the following general outline:
(1) Get the String parameter from the processs address space (only a pointer passed in).
(2) Open the file and overwrite the contents of this processs address space with its content.
(3) Reset the processs registers to their initial default state.

Implemention:

I have modified the following files:

1)SystemCallHandler.java
2)NachosProcess.java

Changes made to the above files:

1) SystemCallHandler.java
The very first thing to handle is program counters because there are system calls, means we are shifting from. A user
mode to kernel mode, otherwise it will result in the execution of same program again and again.

I got the idea of program counter updating from MipSims.java file.
For that, update the following:
PrevPCReg to PCReg
PCReg to NextPCReg
NextPCReg to NextPCReg + 4

Switch Case: Fork

1) Since it is a system call initiated by kernel, we must disable the interrupt. So, I have saved its present state in a
variable and then disabled it, later once fork process is done, we will re-enable the interrupt.
2)As, it is mentioned that output of user program is written in Register r2, so write 0 to r2 as per project instructions
since, it’s a signal to child process that it is child.
3)The next step is to create a new process for child.
4)Then, copying the address space of parent to child process
5)Then, saving the user state for the child process just now created so that is has current register values copy.
6)Now, child has its own registers, so I will just replace r2 value with this child process id. This makes clear about the
child process id to its parent
7) I have created a new java list “process_list” just to keep track of processes, I have added the child process to the
list
8)Finally, I have invoked fork function for child process so that it will be put in ready queue and will run
9)At last, restore the initial state of interrupt.

Switch Case: Join

1) Since it is a system call initiated by kernel, we must disable the interrupt. So, I have saved its present state in a
variable and then disabled it, later once fork process is done, we will re-enable the interrupt.
2)Retrieve the process id, P_id on which join is called (from register r4)

3)Then, I have checked some conditions, first checking whether this id is valid id or not. As earlier I have saved process details
in a list. So, I have checked that whether this id is present in
that list or not.
4)I am also checking that the process id which is passed should not be the process id of current process
5)Also, the process id should not be zero
6)if any of the above conditions are failing, join will simply return, so I have put break for that and restoring the interrupt state
before that.
7)Now, we have the process which must wait on before it finishes, so I have put the process on to hash map
8)Then, restore the interrupt state
9)Put the calling process to sleep

Switch Case: Exec

1)Again, the first thing is to disable the interrupts
2) now, to get the start, point of memory location of file name, read register r4
3)Then I have called FileRead function to fetch name of file from memory
4)Pass this file name to mFileSystem.open
5)Override the current process address space with this new file address space if the file is not null. Then initialize the
registers.
6)Restore state from address space
7)Restore the interrupt state
8)Call machine.run() function to execute instructions
Note that if the file is empty, join will simply return from system call (that is break).

Switch Case: Exit (modifications)

As mentioned in project deadlines, join require some changes in the existing EXIT system call.
1) I have made changes to the EXIT system call to test hash map for existing process id’s.
2) If there is some process id existing in the hash map, then, it means that there is atlas one process to finish.
3)Also, in such case, remove the process which is exiting from process’s list
4)Now, if the list is empty after removing the process, I will remove that entry from hash map.
5)Then, search process list, to find what all processes are waiting on exit process to finish.
6)For all such processes, put the result of exit process in r2 register (WritetouserRegister) for the waiting process to
access.
7)then, wake all these processes
8)Restore the interrupt state
9)Invoke finish on the process which is exiting
10)Also, I have remove the exit process from my process list.

Also,
I have added following to this file:

1)Count variable-this is for creating a unique process id’s. It will increment once current count value is assigned to new
process.
2)process_list- it maintains a list of all the processes. This is used to check process is there or not.
3)map-it is a hash map which shows the linkage from processes that are waiting on other processes to finish to a list
of the processes it's waiting on.
2) NachosProcess.java
1)FileRead function- it will read from a starting memory address and stops once a terminating character is fetched.
2)writetoUserReg function-it will write a value to a register
3)P_id variable- it is a unique Process identifier used to identify each process.

New ForkedProcess.java file:
This new class is created for the fork call function of NachosProcess.
This function will restore the processes registers and state so that the address space can run. Then Machine.run is
invoked to begin executing instructions for this process.

  





