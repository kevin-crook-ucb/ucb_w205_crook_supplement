### Optional !!! - Automating linux command line commands using Python

### Concurrent execution using Python in Linux

There are 3 main ways to produce concurrent execution using Python in Linux

#### Process Based Parallel Processing

In Linux, each process has its own stack and heap.  

The stack is used to store the calling frame.  Each time a function is called, the current function's local variables, registers, instruction pointer, etc. are placed on the stack.  The stack for a 32 bit point has a max size of 2 GiB and in languages which don't build efficients stacks (for example, Java), the stack can easily overflow.  For Python, pointers to memory allocated in the heap are placed on the stack, allowing for a very efficient use of stack.

The heap is used to store the bulk of memory and in a 64 bit operating system, can store more memory than we could at present buy in RAM.  

If processes want to communicate with each other, they would need to use Linux Inter Process Communications (IPC), such as shared memory segments, message queues, named pipes, etc.  They would also need use lock management, which can get tricky.

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

UPDATE: 3/2/2020

I tried using the subprocess management package of Pythyon, but it appears there is a bug with docker-compose.  docker-compose does not check and reset the ioctl device when it's not a pseudo terminal.  It makes it impossible to script any docker-compose commands from anything other than a terminal.

Here is the actual error message I'm getting:
```
stty: 'standard input': Inappropriate ioctl for device
```

### A Basic example of using Subprocess Management 

```python
import subprocess
import shlex

my_command = ["ls", "-lah"]
my_process = subprocess.run(my_command, stdout=subprocess.PIPE, stderr=subprocess.PIPE, universal_newlines=True)
my_process
```

The output should look something like this:

```
CompletedProcess(args=['ls', '-lah'], returncode=0, stdout='total 9.1M\ndrwxr-xr-x  5 jupyter jupyter 4.0K Mar  2 23:47 .\ndrwxr-xr-x 18 jupyter jupyter 4.0K Feb 28 02:39 ..\n-rw-r--r--  1 jupyter jupyter 8.9M Feb 28 04:14 assessment-attempts-20180128-121051-nested.json\n-rw-r--r--  1 jupyter jupyter  713 Feb 28 04:13 derby.log\n-rw-r--r--  1 jupyter jupyter 1.1K Feb 28 04:07 docker-compose.yml\ndrwxr-xr-x  8 jupyter jupyter 4.0K Mar  2 23:48 .git\n-rw-r--r--  1 jupyter jupyter  41K Feb 28 04:25 history.txt\ndrwxr-xr-x  2 jupyter jupyter 4.0K Mar  2 23:11 .ipynb_checkpoints\ndrwxr-xr-x  5 jupyter jupyter 4.0K Feb 28 04:13 metastore_db\n-rw-r--r--  1 jupyter jupyter  11K Mar  2 23:47 Project_2_Automate_Linux_Command_Line.ipynb\n-rw-r--r--  1 jupyter jupyter  34K Feb 28 04:27 Project_2.ipynb\n-rw-r--r--  1 jupyter jupyter 3.7K Feb 21 03:44 README.md\n', stderr='')
```

### Issue with docker-compose commands

```python
my_command = ["docker-compose", "ps"]
my_process = subprocess.run(my_command, stdout=subprocess.PIPE, stderr=subprocess.PIPE, universal_newlines=True)
my_process
```

Gives the following error message:
```
CompletedProcess(args=['docker-compose', 'ps'], returncode=0, stdout='Name   Command   State   Ports \n------------------------------\n', stderr="stty: 'standard input': Inappropriate ioctl for device\n")
```

