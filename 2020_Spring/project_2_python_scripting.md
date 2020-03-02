### Optional !!! - Automating linux command line commands using Python

### Concurrent execution in Python in Linux

There are 3 ways to produce concurrent processing in Pytyhon

#### Process Based Parallel Processing aka "multiprocessing" 

In Linux, each process has its own stack and heap.  

The stack is used to store the calling frame.  Each time a function is called, the current function's local variables, registers, instruction pointer, etc. are placed on the stack.  The stack for a 32 bit point has a max size of 2 GiB and in languages which don't build efficients stacks (for example, Java), the stack can easily overflow.  For Python, pointers to memory allocated in the heap are placed on the stack, allowing for a very efficient use of stack.

The heap is used to store the bulk of memory and in a 64 bit operating system, can store more memory than we could at present buy in RAM.  

If processes want to communicate with each other, they would need to use Linux Inter-Process Communications (IPC), such as shared memory segments, message queues, etc.  They would also need use lock management, which can get tricky.

For more information, please check out this link:
https://docs.python.org/3/library/multiprocessing.html

#### Thread Based Parallel Processing aka "threading"

In Linux, all threads in the same process have their own stack while sharing the heap.

This makes communications between threads much easier, as all threads have access to all data in the process's heap.  However, the threads do still have to take locks, but locks are also much simpler.

For more information, please check out this link:
https://docs.python.org/3/library/threading.html

#### Subprocess Management - NOT Parallel Processing - allows the current process to create sub processes 

