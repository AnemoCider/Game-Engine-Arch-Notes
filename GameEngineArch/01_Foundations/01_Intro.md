# Introduction

## What is a game engine

Different levels of "game", by customizability:

- Cannot be used to build more than one game
- Can be customized to make very similar games
- Can be "modded" to build any game in a specific genre
- Can be used to build any game imaginable

Modern game engines lie between the last two levels.

Tradeoff: general purpose vs. optimal for specific genre / platform

## Runtime Engine Architecture Overview

From the bottom to the top...

- Hardware - Drivers - OS
- 3rd party SDKs & Middleware: Vulkan, Boost, etc.
- Platform Independence Layer: File System, Networking, Graphics, etc.
- Core Systems
  - Assertions, for error checking
  - Memory management
  - Math lib
  - Custom DS & Algo
- Resource Manager (and resources)
- Functions: Rendering Engine, Profiling and Debugging, Physics, Gameplay foundation systems, etc.
  - Static & Dynamic world objects, event system, etc.
  - scripting system
- Game Specific Subsystems: game-specific rendering, cameras, AI, etc.