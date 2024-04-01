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

We need to align the memory allocated.

Elegant implementation:

```Cpp
// align an address to a align-byte boundary
// adds align - 1, then striping off the last log2(align) bits
uintptr_t AlignAddress(uintptr_t addr, size_t align) {
    const size_t mask = align - 1;
    assert ((align & mask) == 0); // ensure align is a power of 2
    return (addr + mask) & ~mask;
}
```

Freeing aligned blocks: 