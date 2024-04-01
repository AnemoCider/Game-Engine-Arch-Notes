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

```Cpp

global objects + initialization (no singleton)

```Cpp
void BigInit() {
    g_textDb.Init();
}
```

## Memory Management

