# Foundementals

## Table of Contents

- [Foundementals](#foundementals)
  - [Table of Contents](#table-of-contents)
  - [Data, Code and Memory Layout](#data-code-and-memory-layout)
    - [Numeric Representations](#numeric-representations)
      - [Signed vs Unsigned](#signed-vs-unsigned)
      - [Fixed-point](#fixed-point)
      - [Floating-point](#floating-point)
        - [Special cases](#special-cases)
        - [Machine epsilon](#machine-epsilon)
      - [kB vs KiB](#kb-vs-kib)
  - [Hardware Fundamentals](#hardware-fundamentals)
    - [CPU](#cpu)
      - [Interesting take aways from CPU arch](#interesting-take-aways-from-cpu-arch)
    - [Memory](#memory)
    - [Buses](#buses)
  - [Memory Architectures](#memory-architectures)
    - [Display](#display)
    - [Memory Gap](#memory-gap)
      - [Memory Cache Hierarchies](#memory-cache-hierarchies)
        - [Cache Lines](#cache-lines)
      - [Instruction Cache and Data cache](#instruction-cache-and-data-cache)
      - [Cache Coherency](#cache-coherency)
      - [Avoid cache misses](#avoid-cache-misses)


## Data, Code and Memory Layout

### Numeric Representations

#### Signed vs Unsigned

- Unsigned: `0x00000000` = 0, `0xFFFFFFFF` = 4294967295
- Signed bit: `0x00000000` = 0, `0xFFFFFFFF` = -1; with the most significant bit being the sign bit
  - problem: two representations for 0: `0x00000000` and `0x80000000`
- 2's complement: `0x00000000` = 0, `0xFFFFFFFF` = -1, `0x80000000` = -2147483648
  - Positive as usual, negatives are calculated by taking the complement then plus 1

#### Fixed-point

Simply concatenate the signed bit, the binary whole and binary fractional part.

Problem: too narrow

#### Floating-point

IEEE 754:

- 1 bit sign
- 8 bits exp
  - subtract 127 as bias
- 23 bits significand (mantissa)

$n = (-1)^{sign} \times (1 + sig) \times 2^{exp - 127}$

##### Special cases

- the smallest normal number: `0x00000001` would normally be of $2^{-127}$, large gap (compared to step size) from 0
  - subnormal numbers: exp = 0, sig != 0; use -126 as exp, drop leading 1 for significand.
    - `0x00000001` = $2^{-149}$, `0x00000002` = $2^{-148}$, etc. 
- zero: `0x00000000` or `0x80000000`
  - exp = 0, sig = 0
- infinity: `0x7F800000` or `0xFF800000`
  - exp = 255, sig = 0
- NaN: exp = 255, sig != 0

##### Machine epsilon

The smallest 32-bit floating-point $\epsilon$, s.t., $1 + \epsilon \neq 1$

For IEEE 754, 1 = `0x3F800000`, 1 + $\epsilon$ = `0x3F800001`, thus $\epsilon = 2^{-23}$

*Units in the Last Place* (ULP) $ = 2^{power} * \epsilon$

#### kB vs KiB

- kB: SI unit, kilobyte, 1000 bytes
  - MB: megabytes, 1000 kB
- KiB: IEC unit, kibibyte, 1024 bytes

## Hardware Fundamentals

### CPU

- ALU: for integer arithmetic and bit-wise operations
- FPU: floating processing unit; for floating-point arithmetic. Modern CPU directly uses VPU instead of FPU.
- VPU: vector processing, or, SIMD; able to apply arithmetic operations to vectors of data
- MC/MMU: memory controller/memory management unit; for interfacing with memory
- registers. Implemented by (S)RAM.
  - PC: program counter; for address of the next instruction
  - SP: stack pointer; for address of the top of the stack
  - BP: base pointer; for address of the base of the stack
  - status register; reflecting the results of most recent ALU operation
  - GPR: general purpose registers; for storing data
- CU: control unit; for decoding and dispatching instructions, and routing data
  - this is where branch prediction and out-of-order execution happens

All above are driven by a clock, which also determines the speed of the CPU.

#### Interesting take aways from CPU arch

- Conversions betweein `int` and `float` were expensive when using FPU, because the data had to be moved between the two units, in addition to convert the bits.

### Memory

- ROM: read-only memory; preserves data even when powered off
- RAM: random access memory; loses data when powered off
  - SRAM: static RAM;
  - DRAM: dynamic RAM;

### Buses

Data is transferred between the CPU and memory over buses.
A bus is just a bundle of parallel digital wires, i.e., lines.

- CPUs typically have 3 buses: address, data, and control
  - Load data from memory: sends an address through address bus to MMU, who then responds by sending the data through data bus
- Width of the address bus determines the addressable memory in the machine
- n-bit machine: has an n-bit data bus and/or registers

## Memory Architectures

### Display

VRAM: video RAM; for storing the display data

It may resides:

- on the motherboard, thus sharing the main memory (unified memory)
- or, on the video card so that GPU can quickly access it.
  - in this case, it communicates with the main memory through a bus protocol, e.g., PCIe

### Memory Gap

In early days, memory access latency was rougly equal to instruction execution latency. However, CPUs improve much faster than memory, thus a gap was formed.

Workarounds:

- reducing average memory latency, e.g., cache
- hiding memory access lantency by arraging CPU to do other work
- minimizing access to main memory by arraging the program's data

#### Memory Cache Hierarchies

- L1
- L2
- L3 (or even L4)

##### Cache Lines

Based on the pattern called locality of reference:

- Spatial: nearby addresses will also likely be accessed
- Temporal: recently accessed addresses will likely be accessed again

Review CS61C for more details about cache.

#### Instruction Cache and Data cache

L1 separates instruction cache and data cache, while L2 and L3 are unified.

#### Cache Coherency

L1 caches are private to each core, while starting from L2 or L3, caches are shared.
We need a protocol to ensure cache coherency, e.g., MESI.

#### Avoid cache misses

Avoid Instruction cache misses: in terms of code size, keep loops small, and avoid calling functions in the inner most loop. Keep functions small as well.

`inline` tradeoff: 

- pros: avoid function call overhead
- cons: code size increases, thus increasing I-cache misses