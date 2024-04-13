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

### Dask vs Data Parallelism and Flynn's Taxonomy

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

### Stall

At some clock cycle the CPU cannot issue a new instruction, causing a "bubble" which passes in the pipeline.

### Data Dependencies

Stalls are typically caused by dependencies:

- data dependency
- control dependency (branch dependency)
- structural dependency (resource dependency)

Here is some ways to avoid data dependencies.

#### Instruction Reordering

