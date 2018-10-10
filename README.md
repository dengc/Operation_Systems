# Operating Systems

@(IT Studies)


## Intro
-------------------------------------
### Abstraction
execution context
- what you need to execute code, like 'sort()' for object
- represents the state of a process and its threads

examples: 
- files: in disk 
- programs: memory (up-side-down): 上low下high
	- memory map - VM
		- convert Virtual Address to Physical Address
		- part hw, part os
		- each program thinks has own full address space
- threads: virtual processor


-----build an os

Kernel: the portion of the OS that runs in privileged mode

Traps: App calls OS
- means for invoking the kernel from user code
- system calls through which user code can access the kernel in a controlled manner
- for programming errors, the default action is to terminate the user program

Interrupts: External devices call OS
- a request from an external device for a response from the processor
- often has no direct effect on the currently running program

Upcall: OS call App
- A program may establish a handler (i.e., a signal handler) to be invoked in response to the error
- set up a timer

Bus: a collection of signals (address in processor) who connect to each other

Process Control Block (PCB) is a kernel data structure
- has return code & PID
- the "Link" field points to the next PCB

run a program:
- make a copy of a process
- replace the child process with a new one



## Multithreaded Programming
-------------------------------------
### Why

If only one processor, make it look like things are running in parallel
- easier & faster

### Creation
Creating a POSIX Thread
    - every thread needs a separate stack
    - first stack frame in every child thread corresponds to start_routine
Multiple Arguments
- Be careful how to pass argument to new thread when you call pthread_create()
    - passing address of a local variable only works if we are certain the this storage doesn’t go out of scope until the thread is done with it
    - passing address of a static or a global variable only works if we are certain that only one thread at a time is using the storage
    - passing address of a dynamically allocated storage only works if we can free the storage when, and only when, the thread is finished with it

- To wait for a child thread to die, use pthread_join()

### Termination
- Return Value
- self-terminate
    + return from its "first procedure"
    + pthread_exit: only for this thread (only way for not affecting others)

### Synchronization
avoid Mutual Exclusion
- Atomic operation
    + pthread_mutex_lock()
    + critical section
    + pthread_mutex_unlock()
- multiple locks may cause deadlock
    + Prevention: Lock Hierarchies: must not try locking a mutex at level i if already holding a mutex at equal or higher level
