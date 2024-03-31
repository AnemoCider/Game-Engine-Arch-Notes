# Tools of the Trade

## Verion Control

- Distributed, e.g., git. Can work without the internet.
- Centralized, e.g., SVN (subversion). Must connect to the internet.
  - Basic workflow: Checkout & Commit. Relies less on branching & merging.

## Profilers

- Statistical profilers: don't largely affect performance. They periodically sample program counter to know which functions is running, and finally estimates the distribution of running time. E.g., Intel VTune.
- Instrumenting profilers: inserts prologue and epilogue into every function, inspecing call stack and recording details.