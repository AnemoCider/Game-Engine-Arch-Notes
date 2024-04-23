# Parallelism and Concurrent Programming

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