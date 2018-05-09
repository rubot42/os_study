# Operating Systems Study Guide

# Lectures
## Intro
## Processes
Program: a set of instructions, no need to be active
Process: an active program
A process contains:
1. an executable program
2. associated data needed by the program
   + program counter (PC)
   + data segment
   + text segment
   + stack pointer (SP)
3. Execution context of the program

Using multiple processes increases CPU utilization and reduces latency

Stack
+ static
+ last in first out
Heap
+ Dynamic
+ First in first out

"A process's view of the world"
+ multiple processes can run at the same time
  + each process owns its own image of the program
  + address space (memory) the program can use
+ state (registers, including PC & SP)
+ address space and memory protection
  + each process has its own exclusive address space
  + physical memory is divided into user memory nad kernel memory
  + (kernel memory can only be accessed in kernel mode)
  
Reasons for Process Creation
1. system initialization
2. running process creates another process
3. user creates a new process
4. initialization of a batch job

Unix Process Creation: fork()
+ child and parent are almost identical
+ fork() returns 0 for child and the child's pid for the parent (a non-zero)
+ child often executes execvs() to change memory image and run a new program
+ parent and child have seperate copies of data and PCs
+ (if one changes the global variable, the other can't see it)

Fork bombs
+ recursively call fork on parent and child to overload the system
+ prevent by limiting the max number of processes a single user can own (command: ulimit)

Windows Process Creation: CreateProcess()
+ loads program that is specified in the call
+ child and parent are not necessarily the same
+ child and parent still executed concurrently

Process Hierarchies
Unix:
+ parent and childrem for a hierarchy
+ if process exits, its children are "inherited" by the exiting process's parent
Windows:
+ no concept of hierarchy
+ all processes are created equal

Process Termination
Reasons:
1. normal exit
2. error exit (ie: file doesn't exist)
3. fatal error (ie: program bug)
4. killed by another process
How:
+ Unix: kill()
+ Windows: TerminateProcess()
+ Neither kills children processes of process killed

Process State
1. Running: has control of the cpu
2. Ready: ready to execute, but another process is running
3. Waiting/Blocked: cannot make process until event is signaled

+ Transitions: Waiting/Blocked <-- Running <--> Ready <-- Waiting/Blocked

Checking process state in Unix/Linux:
+ command: ps
+ STAT column indicates execution state

Process Data Structures
+ each process represented with a Process Control Block (PCB) which contains all the process info
+ PCBs contained in the Process Table, one entry per process

Process Control Block
+ represents a single process
+ contains all the process info
+ stored in the Process Table
+ Parts:
  + PID
  + register values for the process (including PC and SP values)
  + address space
  + priority
  + pointer to the next pcb
  + I/O information
  
Context Switch (changing running process)
+ saves and loads from PCB of respective process

CPU Utilization = 1-p^n
p = waiting time for I/O
n = number of processes
## System calls
System calls are how processes communicate to the OS
Major System calls
+ pid = fork(): create a child process identical to the parent
+ pid = waitpid(pid, &statloc, options): wait for a child to terminate
+ s = execve(name, argv, enviornp): replace a process' core image
+ exit(status): terminate process execution and return status
+ fd = open(file, how, ...): open a file for reading, writing, or both
+ s = close(fd): close an open file
+ n = read(fd, buffer,nbytes): write data from a buffer to a file
+ position = lseek(fd, offset, whence): move the file pointer
+ s = stat(name, &buf): get a file's status information
+ s = mkdir(name, mode): create a new directory
+ s = rmdir(name): remove an empty directory
+ s = link(name1, name2): create a new entry, name2, pointing to name1
+ s = unlink(name): remove a directory entry
+ s = mount(special, name, flag): mount a file system
+ s = umount(special): unmount a file system
+ s = chdir(dirname): change the working directory
+ s = chmod(name, mode): change a file's protection bits
+ s = kill(pid, signal): send a signal to a process
+ seconds = time(&seconds): get the elapsed time since Jan 1st, 1970
(s = -1 if an error occurs)

windows has calls that roughly correlate to unix ones but I don't think we'll need to memorize them

## Threads
## Ipc
## Semaphore
## Monitor
## Prelab
## Pthred
## Scheduling
## Deadlocks
## Memory
## Page replacement
## Filesystems
## IO
## Disks


# Quizzes
Fork
Race Conditions
Scheduling
Virtual Memory
File Systems

# Midterm
