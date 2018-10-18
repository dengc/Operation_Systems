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
- Guarded Commands
    - evaluting the guard and executing the command sequence altogether is an atomic operation if the guard is true
        - inside one critical section
    - the guard is complicated: keeps changing its value, continuously
        - need condition variables, which is a queue of threads
    - steps:
        1. pthread_cond_wait(cv, mutex)
            - when mutex locked, wait
            - atomically unlocks mutex and wait for the "event"
        2. pthread_cond_broadcast: for all threads; pthread_cond_signal: for one thread
            - for some Readers-Writers 
- Semaphores
    - usually 2 operations
    - can be binary or counting
        - counting: solve the producer-consumer problem
            - if producer is fast and consumer slow, producer may wait
    - looks like mutex
        - diff: one thread performs a P operation on a semaphore, another thread performs a V operation on the same semaphore
- Barriers
    - one thread reaches a barrier, wait until all threads reach, last thread broadcast, then all move forward

### Safety
- can be reentrant
- To suspend your thread for a certain duration

### Deviations
Unix’s signal mechanism
- Signals
    - Ex:  <Cntrl+C>
    - callback/ upcall
    - A signal is pending if it’s generated but blocked, can be delivered if unblocked
- handling signals
    - asynchronously
        - signal handler
            - each signal in a process can have at most one handler
            - sigset(), sigaction()
            - handler() is not async-signal safe
    - synchronously
        - use sigwait()
- masking signals
    - sigset_t
    - if a mask bit is 1, the corresponding signal is blocked
- POSIX Cancellation Rules
    - when pthread_cancel() gets called, the target thread is marked as having a pending cancel, not wait for the cancel to take effect
    - when a thread acts on the cancel, walks through a stack of cleanup handlers


## Context Switching
-------------------------------------
what are we switching -- Execution context, which is the current state of our thread

Context of main(): CPU registers, global var, local var
Context of sub(): global, local var, arguments
Stack frame 从上到下：local var (from callee), ebp, eip, args (from caller)

### Coroutine Linkage
- Threads are independent, but they can be made aware of each other and be able to transfer control from one thread to another
- Switch: To transfer control from one thread to another is equivalent to copying the thread control block of the target thread into the "Current Thread context"

### System Calls
A system call involves the transfer of control from user code to system/kernel code and back
- there is no thread switching!
- Most systems provide threads with two stacks
    - executes in priviledged mode (kernel)
    - it then changed back to a user thread eventually
- Trap into the kernel with all interrupt disabled and processor mode set to privileged mode

### Interrupts
Different to Signals:
- signals are generated by the kernel
    - they are delivered to the user process
    - signal ≠ software interrupt
- interrupts are generated by the hardware
    - they are delivered to the kernel

When an interrupt occurs, the processor puts aside the current context and switch to an interrupt context
- the current context can be a thread (user or kernel) context or another interrupt context need to be able to mask/block individual interrupts (similar to signal masking/blocking)

If interrupt occurs when a user thread is executing in the CPU
- Disable interrupt and set processor mode to privileged mode

once you have interrupt handlers running, a normal thread cannot run until all interrupt handlers complete

Interrupt Mask
- Individual interrupt can be masked, if an interrupt occurs while it is masked, the interrupt indication remains pending
- once that interrupt is unmasked/unblocked, the interrupt is delivered and the CPU is interrupted to execute a corresponding interrupt service routine
- hardware register implements a bit vector / mask
- hierarchical interrupt levels (more common)

## Input/Output Architectures
-------------------------------------
memory-mapped I/O
- all controllers listen on the bus to determine if a request is for itself or not

two categories of devices
- programmed I/O
    - perform I/O operations by reading or writing data in the controller registers one byte or word at a time over the bus
- DMA (direct memory access)
    - the controller performs the I/O itself
    - the processor writes to the controller to tell it where to transfer the results to
    - the controller takes over and transfers data between itself and primary memory

use I/O processors (a.k.a. channels) to handle much of the I/O work

## Dynamic Storage Allocation
-------------------------------------
Goal: allow dynamic creation and destruction of data structures
- use "memory allocator" to manage a large block of memory (i.e., a collection of contiguous memory addresses)
- first-fit vs. best-fit allocation
    - first-fit works better
    - best-fit tends to create smallest left-over blocks
    - Memory allocator must run fast

coalescing: need to know that size of the blocks above and below the block being freed

### Buddy System
blocks get evenly divided into two blocks that are buddies with each other
- can only merge with your buddy if your buddy is also free
- internal fragmentation

BUDDYk(x) =x+2 k if x mod 2 k+1=0 
BUDDYk(x) =x-2 k if x mod 2 k+1=2 k 
Ex: BUDDY2(1010100) =1010000

### Slab Allocation
- sets up a separate cache for each type of object to be managed
- contiguous sets of pages called slabs, allocated to hold objects
- Whenever a slab is allocated, a constructor is called to initialize all the objects it hold
- As objects are being allocated, they are taken from the set of existing slabs in the cache

## Linking & Loading
-------------------------------------
if everything can be accessed relative to the frame pointer, then you don’t need to know the actual address of an object, just use relative-addresses to access variables

compiler figure out how many bytes each object is
linker figures out where each object is

Linker Functions:
- relocation
- symbol resolution

o- file
An object file describes what’s in the data, bss, and text segments in separate sections
- global symbols
- undefined symbols
- instructions for relocation

## Booting
-------------------------------------
to pull yourself up by your bootstraps
- load a tiny OS into memory
- known as the bootstrap loader
    - it would handle: disk device, on-disk file system, device driver

### Basic Input-Output System (BIOS)
- code stored in read-only memory (ROM)
- configuration data in non-volatile RAM
- 3 primary functions
    - power-on self test (POST)
    - knows where to load the boot program
    - load and transfer control to boot program provide drivers for all devices