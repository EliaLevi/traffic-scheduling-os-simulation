# Concurrent Traffic Simulation & OS Scheduling Engine

A concurrent system-level simulation in C implementing core Operating Systems principles: multi-process management (`fork`), Inter-Process Communication (IPC via non-blocking UNIX pipes), and synchronization via POSIX semaphores mapped in shared memory (`mmap`). The application models dynamic traffic routing across a weighted topological graph, scheduled using CPU scheduling policies (**FCFS** vs. **SJF**) with real-time graphical visualization powered by **Raylib**.

---

## Live Simulation Demonstration



https://github.com/user-attachments/assets/9ad6dac5-f4fe-47be-b95e-ebc7ddcdf115



---

## Architectural Overview

```
                  +----------------------------------+
                  |     Parent Process (Scheduler)   |
                  |   - Non-blocking Polling (IPC)   |
                  |   - Raylib GUI Rendering Loop    |
                  |   - Central Junction Queues      |
                  +----------------------------------+
                     ^             |             |
    Non-blocking     |             |             | POSIX Semaphores
    UNIX Pipes       |             |             | (mmap Shared Memory)
    (IPCMessage)     |             |             | [Wake-up Grants]
                     |             v             v
            +-----------------------------------------------+
            |   Child Traveler Processes (fork() Agents)    |
            |   - Independent Execution State & PID         |
            |   - Dijkstra Shortest Path Planning           |
            |   - Autonomous Node Traversal Requests        |
            +-----------------------------------------------+
```

### Core Systems Concepts Demonstrated
* **Process Lifecycle & Concurrency:** Child traveler processes spawned via `fork()`, supervised, and reaped deterministically with `waitpid()` and graceful signal handling (`SIGINT` / `SIGTERM`).
* **Inter-Process Communication (IPC):** Communication handled using non-blocking UNIX pipes (`fcntl` with `O_NONBLOCK`) to prevent rendering stall while reading agent status packets (`IPCMessage`).
* **Shared Memory & Synchronization:** Dynamic POSIX semaphores (`sem_t`) mapped across memory boundaries via `mmap` (`MAP_SHARED | MAP_ANONYMOUS`) to prevent data races and ensure strict mutual exclusion at intersections.
* **Graph Algorithms & Real-Time Pathfinding:** Autonomous route resolution using Dijkstra's shortest path algorithm across dynamic weighted adjacency lists.

---

## Milestone Development History

* **Milestone 1:** Core graph data structures, adjacency lists, dynamic memory layout, and custom file parsing subsystem.
* **Milestone 2:** Extended graph engine supporting dynamic directional edge weights and node index tracking.
* **Milestone 3:** Integrated Raylib rendering pipeline for real-time graphical representation of nodes, paths, and entities.
* **Milestone 4:** Integrated Dijkstra’s shortest-path algorithm enabling passenger route computation.
* **Milestone 5:** Transformed simulation into a distributed multi-process architecture utilizing `fork()` for individual travelers.
* **Milestone 6:** Implemented IPC messaging protocol over non-blocking pipes and synchronization via dedicated intersection semaphores.
* **Milestone 7 (Final Architecture):** Centralized scheduling engine replacing local races with managed wait-queues supporting **FCFS** and **SJF / Priority** scheduling based on evaluated process burst times.

---

## Scheduling Policies & Performance Analysis

A comparative evaluation was conducted using identical graph topologies and traveler profiles:

### 1. First-Come, First-Served (FCFS)
* **Mechanics:** Processes requesting clearance to a specific junction node are queued in strict chronological arrival order (`arrivalOrder`).
* **Observation:** Ensures strict entry fairness, but causes the **Convoy Effect**. Long-burst processes holding an intersection unnecessarily delay short-burst processes queued immediately behind them.

### 2. Shortest Job First (SJF / Priority)
* **Mechanics:** The central scheduler prioritizes waiting processes with the lowest evaluated Burst Time. Ties are broken using arrival order (FCFS fallback).
* **Observation:** Significantly minimizes average turnaround and node wait times across the graph. Shorter tasks quickly traverse contested intersections, maximizing overall system throughput.

---

## Compilation & Build Targets

A multi-target `Makefile` manages compilation and library linkage:

```bash
# Build specific milestone targets
make milestone1
make milestone2
make milestone3
make milestone4
make milestone5
make milestone6
make milestone7    # Final submission build

# Clean binaries and object files
make clean
```

---

## Execution Guide

Run the compiled executable with the target scheduling policy:

```bash
# Execute with First-Come, First-Served (FCFS)
./sim -schd fcfs

# Execute with Shortest Job First (SJF)
./sim -schd sjf
```

> **Interactive Runtime Control:**  
> Press **`S`** at any point during execution to toggle dynamically between **FCFS** and **SJF / Priority** scheduling modes.
