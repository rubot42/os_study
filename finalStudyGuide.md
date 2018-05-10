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
Comparison
Processes for concurrency (bad)
+ not efficient (own PCB and OS resources & high overhead)
+ don't directly share memory (due to individual address space)
+ can contain multiple threads (multi-threading)
Threads for concurrency (good)
+ 10-100x faster than processes
+ one program can access multiple CPUs or cores
+ allows overlap of I/O and computation
+ a process is a container for threads

Multithreading
+ share same address space
+ share global variables
+ owner by a single user
+ no protection between threads

POSIX Thread commands: Pthread_...
+ create: create a new thread
+ exit: terminate the calling thead
+ join: wait for a specific thread to exit
+ yeild: release the CPU to let another thread run
+ attr_init: create and initializa a thread's attribute structure
+ attr_destroy: remove a thread's attribute structure

Thread Implementation (Kernel vs User Space)
User Space
Pros
+ can be implemented on a OS that does not support threads
+ thread switching is faster than kernel
+ thread scheduling is very fast (no context switching, no kernel trap, no flushing memory cache)
+ each process can have it's own scheduling algorithm
Cons
+ blocking system calls (waiting for keyboard input)
+ page faults (partial loads of programs into memory)
+ threads need to voluntarily give up cpu
Kernel Space
Pros
+ no run time system needed in each process
+ no thread table in each process
+ blocking system calls not an issues
Cons
+ if thread operations are common, much more kernel overhead
+ fork a multithreaded process?
+ signals sent to processes. Should the kernel assign it to a specific thread to handle?
+ slower than user space threads

Hybrid implementations use both
Pop-up threads: new threads created to handle incoming messages

Process Control Block (PCB)
+ process state
+ process id
+ program counter (PC)
+ current cpu register (if not executing)
+ cpu scheduling info (ie: priority)
+ memory-management info
+ resources allocated to it
+ resources it needs

Thread Control Blocks (TCB)
+ break the PCB into two pieces:
+ Thread Specific: process state
+ Process Specific: address space and os resources
+ smaller and cheaper than processes
+ threads in a process can execute different parts of the program code at the same time.
+ threads can execute the same parts of the code at the same time, but with different execution state

Context Switching with TCB
between two threads in the same process:
+ no need to change address space
threads in different processes:
+ must change address space, sometimes invalidating cache (has to do with virtual memory)

A thread is an independent sequential execution stream within a process.
This is because it maitains its own:
+ stack pointer
+ registers
+ scheduling priority

## Ipc: Inter Process Communication
Why IPC?
Processes may need to share data:
+ Exchange Data between multiple processes
+ processes do not get in each other's way
+ maintain proper sequencing of actions in multiple processes

Three IPC Approaches
1. passing messages through the kernel
2. sharing a region of physical memory
3. asynchronous signals or alerts

Race Conditions
+ two or more processes are reading or writing the shared data
+ result depends on the order
+ these are bad!

Critical Section
+ part of the program where the shared memory is accessed
+ read/write of data in these sections can lead to errors and race conditions

Requirements to avoid race conditions (same as mutex properties)
1. no two processes may be simultaneously inside their critical regions
2. no assumptions may be made about speeds or the number of cpus
3. no process running outside its critical region may block other processes
4. no process should have to wait forever to enter its critical region

Mutual Exclusion: only one thread can be in critical section at a time
4 properties of mutex in a critical section:
1. No 2 processes may be simultaneously inside their critical section
(correctness)
2. No process should have to wait indefinitely long to enter its critical section
(liveness - freedom from starvation)
3. No process stopped outside its critical section should block other processes
4. No assumptions about relative speeds of processes or number of processes

Mechanisms for Mutal Exclusion
1. Software solution
  + strict alternation
  + Peterson's solution
  + sleep and wakeup
2. Hardware solution
  + Interrupt disabling
  + Test-and-Set lock (TSL)
3. Higher level solutions
  + mutex
  + semaphores
  + ...
  
Strict Alternation (Software 1):
+ Use a shared variable "Turn" to strictly alternate between processes
+ Waiting process continually reads "Turn" to see if it can proceed
+ "Spin-lock": lock wherein a process busy waits
+ Disadvantages
  + horribly wasteful (process waiting to aquire locks spin on the CPU)
  + if process 0 is faster than process 1, then process 0 will constantly get blocked
    by waiting for process 1 to finish critical section
  + only allows two processes
  + violates condition 3 of mutual exclusion/race conditions
  
Peterson's Solution (Software 2):
+ like strict alternation but with a flag to show interest in entering the critical section
+ still for 2 processes
+ can be generalized to n processes, for n has to be a fixed number

Sleep and Wakeup (Software 3):
+ uses buffer
+ sleeps or wakes up based on buffer

Disabling Interrupts (Hardware solution 1):
+ right before the critical section, the process disables system interrupts
+ before leaving, it reenables them
+ the CPU cannot be switched to other processes when a process is in the critical section
+ doesn't allow for user input during critical section
+ system could lock up

Locking Test-and-Set Lock (TSL) (Hardware Solution 2):
+ test and modify the content of a word atomically
+ the word here is the lock
+ TSL REGISTER,LOCK
+ function returns the current value of memory word lock, then set lock to be true
+ this is an atomic operation

## Semaphore
Disadvantages to prior "solutions":
+ they require busy wait (continually testing a variable until some value appears)
+ wastes cpu time
+ low priority job blocks high priority ones

Improvement
+ instead of blocking with busy wait, sleep instead
+ sleep(): caller gives up cpu for a duration until it is woken up
+ wakeup(): caller wakes up a sleeping process
+ still need to have mutual exclusion with this

Semaphores
purpose: count the number of wakeups saved to solve the lost wakeup problem
How:
+ define a count variable (a semaphore)
+ semaphore is initialized to zero
+ down/--: use one saved wakeup
+ up/++: save a wakeup
+ up and down are atomic
+ semaphores can't be negative
+ can't read or write to semaphore except initial set, as well as up/down

Types of Semaphores
+ Counting: any non-negative num
+ Binary/Mutex: only 1 or 0

How do processes share lock variables, semaphores, or a common buffer?
+ stored in the kernel and only accessed via system calls
  or
+ modern Oss offer a way for processes to share a portion of their address space with other processes

Mutexes in Pthreads:
Pthread_mutex_....
+ init: create a mutex
+ destroy: destory an existing mutex
+ lock: aqurire a lock or block
+ trylock: aquire a lock or fail
+ unlock: release a lock
Pthread_cond_...
+ init: create a condition variable
+ destroy: destroy a condition variable
+ wait: block waiting for a signal
+ signal: signal another thread and wake it up
+ broadcast: signal multiple threads and wake all of them
## Monitor
Monitor (they use condition variables)
Rules
1. only one process can be active in a monitor at any time
2. processes cannot access internal data of monitor
3. put all critical sections inside monitor
Operations
+ Wait (&condVar): causes the calling process to block. Reacquire lock later, before calling.
+ Signal(&condVar): wake up one blocked process, if any
+ (Rule: must hold condition variable when doing condition variable operations!)


## Prelab
Dining Philosopher Problem (we should know this by now lol)
## Pthread
commands are in prior notes
## Scheduling
First Come, First Served
+ as stated. tasks are queued up

Round Robin
+ does each task for a set amount of time denoted by q (quantum)

Non-Preemptive
+ for each clock interrupt, running process keeps going

Preemptive
+ for each clock interrupt, running process is suspended and switched with another (if any)
+ maximum or fixed time execution of processes (timeslice)


Scheduling Criteria
User-oriented
+ minimize response time (time from issuing request to get the first response )
+ minimize turaround time (time btween submission and termination
+ maximize throughput (# of jobs completed per unit time)
+ freedom from startvation
+ fairness
System-oriented
+ maximize cpu utilization (keep the cpu busy)
+ minimize overhead (keep the cpu doing useful stuff, not context switching)

Batch System
+ off-line, jobs come with resource estimates
+ assumption: know how much time is needed to complete the job ahead of time
+ scheduling goals: throughput, turnaround time, cpu utilization
+ Algorithms: First come first served (FCFS), Shortest job first (SJF)

Shortest-Job-First
+ non-preemptive: run shortest process after each completion
  (reduces context switches)
+ preemptive: choose program with shortest time left after each completion or when a new process arrives
  (optimal, gives minimal avg turnaround time)

Scheduling in Interactive Systems
Goals:
+ Response time and proportionality
Algs:
+ round robin
+ priority
+ multiple queues

Round Robin
+ allowed to run a set amount of time (quantum)
+ solution to fairness and starvation
+ fair allocation of cpu across jobs
+ low average waiting time when job lengths vary
+ good for respoonsiveness if small number of jobs
+ typically, higher avg turnaround than SJF but better response times

Priority Scheduling
Plan:
+ always execute highest-priority runnable jobs to completion
+ each queue can be processed with RR with some quantum
Problems:
+ starvation
+ deadlock: Priority inversion (happens when low priority task has lock needed by high-priority task)

Ways to assign priorities:
Statically based on process type
+ Priority = 1/f
+ f = size of quantum last used
+ the longer a process ran, the lower its priority
+ the process that runs the shortest gets the highest priority to run next
Lottery Scheduling
give tickets and pick a random one
you can assign tickets based on size
give each process at least one ticket to avoid starvation
behaves gracefully as load changes
## Deadlocks
Types of resources (objects granted to the processes)
+ Preemptable: can be taken away from a process with no ill effects (ie: memory)
+ Nonpreemptable resources: will cause the process to fail if taken away

Deadlocks are caused by each process needing what another process has.

Dining lawyer/philosopher example

In a deadlock, no processes can run, release resources, or be awakened

Conditions for resource deadlocks (all 4 must be present)
1. mutual exclusion
2. hold and wait
3. no preemption
4. circular wait

Starvation vs Deadlocks
+ starvation: thread waits indefinitely
+ deadlock: circular waiting for resources
+ deadlock is a type of starvation but not all starvation is deadlock

Dealing with Deadlocks
Allow it to happen
+ ostrich alg. (ignore it)
+ detection and recovery (allow it, detect it, fix it)
Ensure it doesn't happpen
+ prevention (negate one of the 4 nessesary conditions)
+ dynamic avoidance (careful resource allocation. analysis on request to see if it will cause an issue)

Resource allocation graphs and how to detect if there are issues.

Detecting Deadlocks (single type of resources)
+ use resource graph
+ perform depth first search at each node looking for cycles/recurring nodes

Recovering from Deadlocks
Preemption: take a resource from some other process (depends on nature of resource)
Rollback: checkpoint a process periodically, use save state and restart deadlocked process
Kill the process: self explanitory

Detecting deadlocks (multiple types of resources)
compare vectors with alg:
vectors and matrices:
+ resources in existence
+ resources available
+ current allocation matrix
+ request matrix
Alg:
1. look for an unmarked process, Pi for which R[i] (ith row of R) <= A
   (Pi's request can be granted with available resources)
2. if such a process is found, A = A + C[i], mark the process and go back to step 1
   (Pi releases resources)
2. if no such process exists, the alg is done

Results:
+ alg outputs a sequence of processes = good
+ else = deadlock


Preventing deadlock by attacking conditions
1. Mutual Exclusion
   Allow resources to be shared
   (spooling)
2. Hold and Wait
   Prevent processes that hold resources from waiting for more
   (request everything initially)
3. No preemption
   If a process is holding some resources and request another that is no available, it releases all.
   Applied to resources whose state can easily be saved and restored later
   (take resources away)
4. Circular Wait
   all requests of a process must be done in numerical order
   (order resources numerically)

Resource allocation graph

safe state   => no deadlocks
unsafe state => possibility of deadlocks
avoidance    => ensure that a system will never reach an unsafe state
## Memory
Goals of memory management
+ convenient abstract for programming
+ provide isolation between different processes
+ allocate scarce physical memory resources across processes


Address Space: the set of addresses that a process can use to address memory
Base and limit registers can be used to give each process a seperate address space.

Memory Holes: block of available memory.
Fragmentation: space exists in memory but hole is not continuous

Paging:
+ solves the external fragmentation by using fixed-size chunks of virtual and physical memory
+ virtual memory unit is called a "page"
+ physical memory unit is called a "frame"/"page frame"
+ Pages and page frames are the same size (a power of 2)
+ Virtual memomory is a sequence of pages
+ physical memomory is a sequence of page frames
+ the program believes it has a single continous block but the actual scattered memory is hidden by the
  mapping of virtual to physical memory
+ programs do not control mapping

Address Translation
+ any page can be translated to any page frame
+ this is doene by MMU
+ virtual address has 2 parts: virtual page number & offset
+ mapping from page to frame provided by the page table
+ number of entries in page table = number of virtual pages
+ one page table per process (located in memory)

Page Table Entry (PTE)
+ page frame # - physical frame where virtual page is loaded
+ present/absent bit - is the page loaded into main memory
+ Protection - read/write/exec access
+ "dirty bit" -  has it been modified (if so needs to write back to disk when swapped out)
+ referenced - set when page is accessed (used in page replacement algs)
+ disable caching - if set do not use cached copy
  (may be set if page is mapped to an i/o device and will change often (memory-mapped i/o))

Size Facts

A n-bit virtual address space
+ the virtual address consists of n bits
+ there are 2^n virtual addresses

A n-bit virtual address space with k bits for offset
each page consists of 2^k virtual addresses
teh virtual page number takes (n-k) bits
there are 2^(n-k) virtual pages
there are 2^(n-k) entries in the page table

The same is true for physical address spaces

Translation Lookaside Buffer (TLB)
+ is part of the MMU
+ is very small and fast
+ caches most recent virtual to physical address translations

Multi-level page tables: page the page tables
Inverted Page Table: page frame -> virtual frame rather than the other way around
Hash table in inverted page tables: better version of inverted page table, better performance

Summary of page table and alternatives:
the standard:
+ page table
solution for faster access:
+ translation lookaside buffer (TLB): reduces thenumber of memory references to get page table entry
solution to minimizing size of page tables:
+ multi-level page table
+ inverted page table
+ hashed page table

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

# Equations/Calculations/Other stuff
cpu utilization
Deadlock stuff:
+ deadlock picture diagrams
+ deadlock detection matrix stuff
+ resource allocation graph
+ bankers algorithm
Memory Diagrams/Drawings
