### Optional !!! - Automating linux command line commands using Python

### Concurrent execution using Python in Linux

There are 3 main ways to produce concurrent execution using Python in Linux

#### Process Based Parallel Processing

In Linux, each process has its own stack and heap.  

The stack is used to store the calling frame.  Each time a function is called, the current function's local variables, registers, instruction pointer, etc. are placed on the stack.  The stack for a 32 bit point has a max size of 2 GiB and in languages which don't build efficients stacks (for example, Java), the stack can easily overflow.  For Python, pointers to memory allocated in the heap are placed on the stack, allowing for a very efficient use of stack.

The heap is used to store the bulk of memory and in a 64 bit operating system, can store more memory than we could at present buy in RAM.  

If processes want to communicate with each other, they would need to use Linux Inter Process Communications (IPC), such as shared memory segments, message queues, etc.  They would also need use lock management, which can get tricky.

For more information, please check out this link:
https://docs.python.org/3/library/multiprocessing.html

#### Thread Based Parallel Processing aka "threading"

In Linux, all threads in the same process have their own stack while sharing the heap.

This makes communications between threads much easier, as all threads have access to all data in the process's heap.  However, the threads do still have to take locks, but locks are also much simpler.

For more information, please check out this link:
https://docs.python.org/3/library/threading.html

#### Subprocess Management  

In Linux, subprocesses are child processes spawned from the current process.  They will execute at the same time as the current process, however, it is customary for the current process to block (aka wait or suspends executation) until the child process has completed.  It's also customary for the current process to communicate with the child process using Standard I/O: Standard Input (STDIN), Standard Output (STDOUT), and Stardard Error (STDERR).

Note that I said "customary".  Subprocesses are real process and can do anything that a process can do.  But if you want to do that kind of stuff, it's best left to Process Based or Thread Based Parallel Processing, as they are better fits.

Using subprocesses would be the best way to automate bash shell commands. 

### Specific to Project 2

If you would like to try to automate your bash shell commands, such as bringing up the cluster, checking to make sure that the cluster is up all the way, creating the kafka topic, and writing the assessments to the kafka topic, the subprocess management package of Python would probably be your best bet.

Here is some very raw sample code to get your started.  This is very raw.  There are a lot of improvments that can be made to make it better:


