# Engine Support Systems

## Table of Contents

- [Engine Support Systems](#engine-support-systems)
  - [Table of Contents](#table-of-contents)
  - [Start-up and shut-down](#start-up-and-shut-down)
    - [C++ static initialization](#c-static-initialization)
      - [Attempt: Construct on Demand](#attempt-construct-on-demand)
    - [Working approach: Actually construct in other functions](#working-approach-actually-construct-in-other-functions)
      - [Real examples](#real-examples)
        - [OGRE startUp](#ogre-startup)
        - [Naughty Dog's startUp](#naughty-dogs-startup)
  - [Memory Management](#memory-management)
    - [Optimizing dynamic memory allocation](#optimizing-dynamic-memory-allocation)
      - [Stack-based allocators](#stack-based-allocators)
        - [Example: Frame-based Memory Allocation](#example-frame-based-memory-allocation)
      - [Pool Allocators](#pool-allocators)
      - [Aligned Allocations](#aligned-allocations)
      - [Single-Frame and Double-buffered memory allocators](#single-frame-and-double-buffered-memory-allocators)
    - [Memory Fragmentation](#memory-fragmentation)
      - [Virtual memory](#virtual-memory)
      - [Avoiding Fragmentation with Stack and Pool allocators](#avoiding-fragmentation-with-stack-and-pool-allocators)
      - [Defragmentation and Relocation](#defragmentation-and-relocation)
  - [Containers](#containers)
    - [String](#string)
  - [Engine Configuration](#engine-configuration)
    - [Loading and Saving options](#loading-and-saving-options)
    - [Per-user options](#per-user-options)
    - [Configuration Management in Real Engines](#configuration-management-in-real-engines)
      - [Example: Quake's Cvars](#example-quakes-cvars)
      - [Example: OGRE's configuration](#example-ogres-configuration)
      - [Example: The Uncharted and The Last of Us Configurations](#example-the-uncharted-and-the-last-of-us-configurations)

## Start-up and shut-down

### C++ static initialization

Global & static objects are constructed and destructed in unpredictable order.

Therefore, a trivial singleton like this won't work:

```Cpp
class RenderManager {
public:
    static RenderManager gRenderManager;
};
```

#### Attempt: Construct on Demand

```Cpp
class RenderManager {
public:
    static RenderManager& get() {
        // s for static
        static RenderManager sSingleton;
        return sSingleton;
    }
    RenderManager() {
        VideoManageer::get();
        TextureMnageer::get();
    }
};
```

problem: we have no simple way to control the destruction order

### Working approach: Actually construct in other functions

```Cpp
class RenderManager {
public:
    RenderManager() {
        // empty
    }
    void startUp() {
        // Do the construction here!
    }
};

RenderManager g_renderManager;

int main() {
    g_renderManager.startUp();
    // ...
    g_renderManager.shutDown();
}
```

#### Real examples

##### OGRE startUp

global pointers + singleton

```Cpp
class Root {
    LogManager* mLogManager;
    Root() {
        if (LogManager::getSingletonPtr() == nullptr){
            mLogManager = new LogManager();
        }
    }
};
```

##### Naughty Dog's startUp

global objects + initialization (no singleton)

```Cpp
void BigInit() {
    g_textDb.Init();
}
```

## Memory Management

Focus on efficient use of RAM.

- malloc, or global `operator new` is very slow
- on modern CPUs, memory access patterns dominates performance

### Optimizing dynamic memory allocation

Reasons why heap allocation is slow:

- default allocator is general-purpose, thus requires great overhead
- call to `malloc` or `delete` will switch into kernel mode to process the request, and switch back, which is slow.

Reasons why custom allocators are faster:

- they can request from a preallocated memory block, which is completed in user mode, avoiding context switch
- they can be optimized for specific use cases

#### Stack-based allocators

Allocate memory like a stack (LIFO), in terms of levels.

Modification: double-ended stack allocator

- uses two allcators that collaborate on a memory chunk
- one from the bottom and the other from the top

##### Example: Frame-based Memory Allocation

(Game Programming Gems 1, 1.9)

- problems of conventional memory allocation
  - memory can be fragmented
- frame-based memory
  - suitable for the level-loading pattern and level-initialization modules
  - pre-allocates a large chunk from OS;
  - Uses 4 pointers: const Base ptr: lowest, const Cap ptr: highest; Lower and higher heap frame pointers
- Advantage
  - we can simultaneously allocate 2 types of data (one from the top and the other from the bottom), and free them frequently, without causing fragmentation
  - If we need more types of memory, just use multiple chunks.

#### Pool Allocators

Suitable for allocating lots of small blocks of the same size, e.g., a pool for 4x4 matrices. It manages the free memory blocks in a linked list.

Trick: store the `next` pointer in the block itself, as long as the block size is no smaller than that of a pointer. If the block size is small, we use indices to indicate the next free memory position.

#### Aligned Allocations

We need to align the memory allocated. This is done by simply allocating more memory, at most L - 1 bytes, and returned the aligned address within.

Problem: how to free the memory, given the address after alignment?

Solution: store the alignment offset in the extra memory.

Problem: what if the address is naturally aligned, and there is no place to store the offset?

Solution: explicitly align all address. Not aligned -> align by L bytes; then L - 1 to 1 bytes. This way, there will be at least 1 byte to store the offset.

#### Single-Frame and Double-buffered memory allocators

- Single Frame
  - For objects that are only used in the current frame
  - Using a stack, and clears it by moving the top pointer to the bottom
  - Drawback: need to be very careful don't use pointer to the memory across frames
- Double-buffered
  - For objects that are used in the next frame
  - Using 2 single-frame stacks, swap them every frame
  - Able to use objects allocated in the previous frame

### Memory Fragmentation

Lots of small free chunks ("holes") in the memory. Problem: allocation may fail even with enough free memory

#### Virtual memory

VM largely solves this problem, because the OS maps discontinuous physical memory to continuous virtual memory. Now, theoretically, fragmentation can only happen in the virtual memory. In addition, applications have the illusion owing the entire memory space, so it is unlikely to suffer from fragmentation.

However, most console games cannot afford the overhead of VM.

#### Avoiding Fragmentation with Stack and Pool allocators

They naturally avoid fragmentation.

#### Defragmentation and Relocation

Suppose we have to allocate and free differently sized objects in random order.

- Defragmentation: move objects on the heap together
  - Problem: invalidate pointers, so we need...
- (Pointer) Relocation: when moving a certain chunk, find all pointers pointing to it, and update them
  - The extra cost to maintain all pointers is high
  - Smart pointer: Use a wrapper class around a pointer
    - Store the objects in a linked list, and update the pointers in the list
  - Handle: use a map from some index to the actual address. Let the application use index, and the engine updates the mapping.
    - (Reader's Note: This is exactly virtual memory...)
  - Problem: There are always non-relocatable memory, e.g., those from the third party.
    - Address them manually!
  - Amortizing cost: shift only a few blocks per frame

## Containers

### String

Key points to notice:

- `std::string` may incur overhead. Use `char[]`.
- `strcmp`, `strcpy`, and `hash(string)` are too slow.

Optimization:

- with 64-bit hash function, we can assume that no string collision will happen
- hash all strings before runtime, and store them in a global table. Manipulate the hashed values at runtime.

    An example:

    ```Cpp
    void processString(uint32_t id) {
        if (id == g_sid_foo) {
            // handle "foo"
        } else if (id == g_sid_bar) {
            // handle "bar"
        }
        // ...
    }
    ```

- store the global stuff in the Debug memory, because we don't need to access the actual strings at runtime

## Engine Configuration

### Loading and Saving options

- Text configuration files
  - INI, Json, XML, etc.
- Compressed binary files, for space limited platforms'
- Online user profiles, stored in the central server

### Per-user options

Most game engines differentiate global and per-user options.

In Windows, they are typically saved under C:\Users\username\AppData\

### Configuration Management in Real Engines

#### Example: Quake's Cvars

- cvar = console variable, a global floating-point or string
- stored in a global linked list during runtime

#### Example: OGRE's configuration

- stored in Windows INI
  - plugins.cfg, etc.

#### Example: The Uncharted and The Last of Us Configurations

- In-game menu settings
  - bool, int, float
  - represented by global variables, or members of a singleton class
  - saved in INI text files
- Command Line Arguments
- Scheme Data Definition
