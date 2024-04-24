# Parallelism and Concurrent Programming

## Table of Contents

- [Parallelism and Concurrent Programming](#parallelism-and-concurrent-programming)
  - [Table of Contents](#table-of-contents)
  - [Intro](#intro)
  - [Defining Concurrency and Parallelism](#defining-concurrency-and-parallelism)
    - [Concurrency](#concurrency)
    - [Parallelism](#parallelism)
      - [Implicit vs Explicit Parallelism](#implicit-vs-explicit-parallelism)
    - [Task vs Data Parallelism and Flynn's Taxonomy](#task-vs-data-parallelism-and-flynns-taxonomy)
      - [GPU: SIMT](#gpu-simt)
    - [Orthogonality of Concurrency and Parallelism](#orthogonality-of-concurrency-and-parallelism)
  - [Implicit Parallelism](#implicit-parallelism)
    - [Pipelining](#pipelining)
    - [Latency vs Throughput](#latency-vs-throughput)
    - [Stall And Data Dependencies](#stall-and-data-dependencies)
      - [Data Dependencies](#data-dependencies)
    - [Branch Dependencies](#branch-dependencies)
      - [branch assembly](#branch-assembly)
    - [Superscalar CPUs](#superscalar-cpus)
      - [Resource Dependencies](#resource-dependencies)
    - [VLIW](#vliw)
  - [Explicit Parallelism](#explicit-parallelism)
    - [Hyperthreading](#hyperthreading)
    - [Multicore CPUs](#multicore-cpus)
    - [Symmetric vs Asymmetric Multiprocessing](#symmetric-vs-asymmetric-multiprocessing)
    - [Distributed Computing](#distributed-computing)
  - [Operating System Fundamentals](#operating-system-fundamentals)
    - [OS kernel](#os-kernel)
      - [Kernel Mode vs User Mode](#kernel-mode-vs-user-mode)
      - [Kernel Mode Privileges](#kernel-mode-privileges)
    - [Interrupts](#interrupts)
    - [Kernel Calls](#kernel-calls)
    - [Preemptive Multitasking](#preemptive-multitasking)
    - [Processes](#processes)
      - [Anatomy of a Process](#anatomy-of-a-process)
      - [Virtul Memory Map of a Process](#virtul-memory-map-of-a-process)
    - [Threads](#threads)
      - [Thread Libraries](#thread-libraries)
      - [Thread Creation and Termination](#thread-creation-and-termination)
      - [Polling, Blocking and Bielding](#polling-blocking-and-bielding)
      - [Context Switching](#context-switching)
      - [Thread Priorities and Affinity](#thread-priorities-and-affinity)
      - [TLS: thread local storage](#tls-thread-local-storage)
    - [Fibers](#fibers)
      - [Fiber Creation and Destruction](#fiber-creation-and-destruction)


## Intro

Computing performance measurements:

- MIPS (Million Instructions Per Second)
- FLOPS (Floating Point Operations Per Second)

## Defining Concurrency and Parallelism

### Concurrency 

Feature: concurrent programs utilize multiple flows of control

Primary factor: reading and/or writing of shared data

### Parallelism

Definition: two or mode hardware components are operating simultaneously

#### Implicit vs Explicit Parallelism

- implicit: parallel hardware components within a CPU
  - improves the performance of a *single* instruction stream
  - i.e., instruction level parallelism (ILP)
- explicit: multiple instruction streams
  - e.g., hyperthreaded or multicore CPUs

### Task vs Data Parallelism and Flynn's Taxonomy

Two orthogonal axes:

- Instruction streams (one to multiple)
- Data streams (one to multiple)

Four quadrants:

- SISD
- MISD
  - multiple instruction streams on a single data stream
  - provides fault tolerance via redundancy
- SIMD
  - e.g., vector processor
- MIMD
  - e.g., multiprocessor

#### GPU: SIMT

- Single Instruction, Multiple Threads
- _manycore_: refers to GPU design, where a GPU consists of many lightweight SIMD cores
- _multicore_: a CPU with a relatively small number of heavyweight, general-purpose cores

### Orthogonality of Concurrency and Parallelism

- A concurrent multi-threaded can run on a single, serial CPU core

## Implicit Parallelism

### Pipelining

Every CPU needs to implement these stages in some form:

- Instruction fetch: CU and MC
- Instruction decode: get opcode, addressing mode and operands
- Execute: select appropriate functional unit and dispatch the instruction
- Memory access: MC
- Register write-back: writes the result to the destination register

pipelining: begin a new instruction every clock cycle to overlad the stages

### Latency vs Throughput

- Latency: time to complete a single instruction
  - sum of the latencies of all stages
- Throughput of bandwidth: how many instructions a pipeline can process per unit time
  - kind of a frequency, e.g., instructions per second
  - $f = \frac{1}{max(T_i)}$, where $T_i$ is the time to complete some single stage
  - therefore, we want all stages to have the same latency, e.g., by breaking down slow stages
- tradeoff: more stages means higher overall latency

### Stall And Data Dependencies

At some clock cycle the CPU cannot issue a new instruction, causing a "bubble" which passes in the pipeline.

Stalls are typically caused by dependencies:

- data dependency
- control dependency (branch dependency)
- structural dependency (resource dependency)

#### Data Dependencies

Here is some ways to avoid data dependencies. However, they can cause bugs in concurrent programs as well.

- Instruction Reordering
  - For any pair of interdependent instructions, we can reorder them with nearby independent instructions
  - Compilers does this automatically
- Out-of-Order Execution
  - CPUs can detect data dependencies and execute instructions out of order
  - They look ahead in the instruction stream, analyzing the use of registers

### Branch Dependencies

#### branch assembly

- consider `(b != 0) ? a / b : someVal;`
  - This will be translated to `cmp b, 0; jz SkipDivision`
  - The conditional jump cannot happen until `b` is calculated
- Speculative Execution
  - i.e., `branch prediction`
  - CPU guesses which branch to take
  - If the guess is wrong, the pipeline has to be flushed and restarted
  - E.g., backward branches are always taken (e.g., `while`), and forward branches are never taken (e.g., `else`)
- Predication
  - Attempts to avoid branches
  - Idea: run both paths, each predicated by a condition
  - In the case above, we just calculate both result and do some bitwise operations to get the correct result
  - Problem: It may not be safe to run both paths, e.g., `(b != 0) : a / b : 0`

### Superscalar CPUs

- duplicate (most parts of) the CPU to issue multiple instructions in a single clock cycle
  - still only one instruction stream, but fetches multiple instructions at once
  - requires complicated dispatch logic to deal with previous 2 kinds of dependencies
  - it also introduces a new kind of dependency, described below

#### Resource Dependencies

- multiple consecutive instructions all require the same functional unit (i.e., resource)
  - there may be 2 ALUs but 1 FPU, so the CPU cannot perform two floating point operations in a single clock cycle

### VLIW

- Problem: Dispatch logic is complicated and expensive
  - CPU cannot see very far in the instruction stream, so it may not predict well
  - complex dispatch logic unit requires much real estate on the chip, expensive
- Idea: Leave the dispatch logic to the programmer and/or compiler
- VLIW: Very Long Instruction Word
  - A instruction word contains multiple slots, each corresponding to a compute element
  - Programmers/Compiler dispatch instructions to multiple computing units per clock cycle
  - Problem: hard to convert serial code to VLIW code

## Explicit Parallelism

Recall that it refers to the ability of hardware processing mulitple instruction streams at the same time.

### Hyperthreading

- Idea: CPU utilizes out-of-order execution, selecting independent instructions to fill delay slots (bubbles in the pipeline)
  - However, CPU has limited choice from only a single instruction stream
- Hyperthreading: CPU has multiple instruction streams
  - CPU can select from multiple streams
  - An HT core has two register files (and other necessary components) and two instruction decode units, and a single backend and L1 cache for execution.
    - Recall that each thread need to have its own PC, SP, and registers
  - uses fewer transistors can a dual core CPU

### Multicore CPUs

- `core`: each core can be serial, pipeline, hyperthread design, etc.
- Separate L1 caches, shared L2 cache

### Symmetric vs Asymmetric Multiprocessing

- How the CPU cores are treated
  - Symmetric: all cores are equal
  - Asymmetric Multiprocessing (AMP)
    - Typically one "master" core runs the OS
    - Other cores do workloads distributed by the master

### Distributed Computing

- Idea: use multiple standalone computers, e.g., cloud computing

## Operating System Fundamentals

### OS kernel

#### Kernel Mode vs User Mode

- Kernel mode: The privileged mode that the kernel (and its device drives) runs in
- User mode softwares can only access low-level services through a kernel call
- Protection rings: levels of privilege, e.g., kernel -> drivers -> trusted apps -> user apps

#### Kernel Mode Privileges

- Access to all machine languages defined by the CPU's ISA
  - x86: `wrmsr` write to model-specific register, `cli` clear interrupt flag (i.e., disable interrupts)

### Interrupts

- Interrupt: a signal to the CPU to notify some low-level event
  - E.g., key press, a signal from peripheral device, the expiration of a timer, etc.
- Every event raises an interrupt request (IRQ)
- If the OS wishes to respond, it will pauses the current stream and calls a interrupt service routine (ISR) function
- Hardware interrupts: triggered by external devices
  - requested by placing a non-zero voltage on one of the CPU's pins
  - they may be raised by devices such as keyboard, mouse, or a periodic timer circuit
  - They can happen even when the CPU is in the middle of executing an instruction, so there may be delay to handle the request
- Software interrupts: triggered by software
  - can be triggered explicitly by executing an "interrupt" instruction
  - can also be triggered in response to an error detected by the CPU, which is what we call a "trap" or "exception"
  - often leads to core dump, or a breakpoint hit if a debugger is attached

### Kernel Calls

- User software issues a kernel call (system call) to perform a privileged operation
  - e.g., map a physical page into virtual memory (`mmap`), access a raw network socket (`socket`)
  - The call is accomplished by placing the arguments in a specific place, then triggering a software interrupt
  - The low-level details are wrapped by kernel API functions, which are actually called by the user program

### Preemptive Multitasking

- cooperative multitasking: each program occupies a certian period, called a `time slice`, and yield the CPU
  - formally, `time division multiplexing (TDM)` or `temporal multitasking (TMT)`
  - problem: what if a program fails to yield the CPU?
- preemptive multitasking: 
  - programs still share the CPU by time-slicing, but OS controls the scheduling
  - `quantum`: the time slice, by default 100ms on Linux

### Processes

- Process: a program in execution
  - i.e., the OS's way of managing a running instance of a program

#### Anatomy of a Process

- A process consists of:
  - pid
  - a set of permissions
  - a reference to the parent process
  - a virtual memory space
  - the values of environment variables
  - a set of open file handles
  - the current working directory
  - resources for managing synchronization and communication with other processes
    - message queues, pipes, semaphores
  - one or more threads
- `thread`: a running instance of a single stream of maching language instructions
  - **fundamental unit** of program execution
  - where as a process merely provides an environment for threads to run
    - the virtual memory space
    - a set of **resources** shared among threads

#### Virtul Memory Map of a Process

- protection
  - kernel has a reserved kernel space, which can only accessed by code running in kernel mode
- a process's `memory map` typically contains:
  - the text, data and BSS sections read from the program's executable file
  - a view of shared libraries, e.g., DLLs
  - a call stack for each thread
  - a heap
  - (optional) pages shared with other processes
  - a range of kernel space addresses, only accessible whenever a kernel call executes
- dynamic linking
  - when first time a DLL is needed, it is loaded into memory, and a view of it is mapped to the process's virtual memory
  - when another process needs the same DLL, the corresponding physical pages are mapped
  - avoid code copying, contrary to static linking; avoid relinking when the DLL is updated
  - problem: compatability between the program and DLL, when the DLL is updated
- kernel space
  - on 32-bit Windows, user space ranges from 0x00000000 to 0x7FFFFFFF, and kernel space ranges from 0x80000000 to 0xFFFFFFFF
  - the virtual page table for the kernel is shared among all processes
- Relocatable code
  - OS may load the program to different addresses each time (e.g., `PIC`: position independent code, `ASLR`: address space layout randomization), so the executable code should be relocatable, and OS will fix up the addresses within the code

### Threads

(Review) A `thread` encapsulates a running instance of a single stream of machine language instructions. It consists of:

- a thread id, unique within the process
- call stack, containing the stack frames of all currently-executing functions
- values of registers, e.g., program counter and stack pointer
- thread local storage (`TLS`), storing private data of the thread

#### Thread Libraries

Examples: *pthread* and *std::thread*

Universal APIs include:

- create
- terminate
- request another thread to exit
- sleep
- yield
- join

#### Thread Creation and Termination

- thread termination happens when:
  - the thread ends naturally by returning from the entry point function
    - returning from `main` ends the process as well
  - a exit function is called, e.g., `pthread_exit`
  - the thread is killed (cancelled) externally
  - the thread is focibly killed because the process has ended

#### Polling, Blocking and Bielding

A thread may want to wait for some event to happen. There are three ways to do this:

- polling
  - busy waiting (spin waiting) by repeatedly checking the status
  - wastes CPU resources
  - only suitable for very short waits
- blocking
  - calls a `blocking function`: if condition not met, add to some queue and sleep
  - e.g., `fopen`, `std::this_thread::sleep_until()`, `pthread_join()`, `pthread_mutex_wait()`
- yielding
  - like polling, but instead of busy waiting, the thread yields the CPU, thus wasting less resources

#### Context Switching

- Three states:
  - running
  - runnable (ready)
  - blocked
- Context switch
  - def.: the kernel causes a thread to transition from any state to another
  - must happen in privileged mode
  - the kernel needs to load/save the thread's context when transitioning to/from the running state
    - note that the thread's call stack need not to move, since it's already in the memory, and we refer to it by `sp` and `bp`
  - if the thread is from a different process, the kernel has to save and set up the virtual memory map and flush translation lookaside buffer (`TLB`)

#### Thread Priorities and Affinity

- These are two ways for programmer's to schedule threads.
- Priority: no lower-priority thread can run as long as some higher-priority runnable thread exists
  - problem: starvation. Can be mitigated by introducing some exceptions to the simple rule.
- Affinity: request to lock a thread to a particular core, or prefer some core when scheduling it

#### TLS: thread local storage

- TLS: a way to store private data for each thread

### Fibers

- cooperative multitasking mechanism
- Fibers are like threads, but they run within the context of a thread, and are scheduled cooperatively by each other

#### Fiber Creation and Destruction