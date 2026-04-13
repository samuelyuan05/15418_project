# 3D FLIP Fluid Simulation with CPU/GPU Parallelization

## Authors
Samuel Yuan, Daniel Stankiewicz

## URL
https://samuelyuan05.github.io/15418_project/

---
## Summary

We plan to implement a 3D FLIP fluid solver, and explore optimizations using CUDA and OpenMP to improve runtime performance. Throughout the process, we will analyze performance tradeoffs between running on different hardware (CPU/GPU) because of bottlenecks such as communication amongst other things.

---


## Background
FLIP (Fluid-Implicit Particle) simulation is a hybrid simulation technique that combines both Lagrangian particles and a grid. While particles carry fluid information like mass or velocity, a background grid is used to more efficiently compute forces and enforce incompressibility. Compared to something like SPH which is reliant on particle interactions, FLIP avoids this scaled complexity by leveraging structured grid computations, while also reducing numerical dissipation by preserving particle velocities. For a FLIP simulation, during each timestep, the algorithm breaks down into a few highly parallel parts:

1. Particle to grid conversion: Each particle transfers its velocity and mass to nearby grid cells using interpolation. 
2. Grid update and application: With velocities in the grid, we apply external forces (such as gravity) and compute quantities such as divergence. Because we are now working on a regular grid, we have quite strong spatial locality.
3. Pressure solver: To enforce incompressability, we need to solve a poisson equation over the grid. This is typically one of the most expensive parts of the simulation and would require us to use some sort of iterative solver.
4. Velocity projection: The grid velocities are then updated by subtracting the pressure gradient in order to ensure our velocity field is free of divergence
5. Grid to particle: Finally, we transfer our new velocities to our particles.

Our main data structures here include particles (with position, velocity, and mass), a 3d grid storing velocities and pressure, and other buffers for interpolation of weights and divergence. The most computationally expensive parts are the particle grid transfers and the pressure solver, which both are good candidates for parallelization. 


---

## The Challenge

Workload:

The workload here consists of both particle based and grid-based computations. While particle operations are quite parallelizable, the grid computations have structured dependencies, and require synchronization between iterations. Along with this, memory access patterns may be mixed. Particle simulations have scattered irregular memory accesses, while grid operations have very strong spatial locality comparatively. On GPUs we may run into divergence issues due to irregular particle distributions

Constraints:

Mapping this workload to hardware presents a unique challenge due to the combination of irregular particle accesses and structured grid computation. The pressure solver is likely to be the hardest to solve, requiring iterative algorithms which introduce a lot of synchronization and limit scaling. On the other hand, particle to grid transfers may require atomics, or other data access patterns to prevent data races.


---

## Resources

Our goal is to implement the solver in C++. We plan on using OpenMP for CPU based implementations and experimentation, and CUDA for GPU implementations (We also hope to experiment with OpenGL compute shaders if we have time!). For visualization, we will use OpenGL with GLFW, and for basic vector and matrix operations, we will likely use GLM but we will consider writing our own if we find GLM slow.

For our implementation, we plan on referencing the SIGGRAPH 2007 Fluid Simulation Course notes, as well as the original paper, FLIP: A Low-Dissipation, Particle-in-Cell Method for Fluid Flow. We also plan on referencing other people’s implementation details which are available online.


---

## Goals and Deliverables

**Plan to achieve:**
  - Working 3D FLIP based fluid simulator
  - A parallel CPU implementation using OpenMP
  - A GPU implementation using CUDA
  - A combined implementation
  - Evaluation comparing CPU and GPU workloads
  - Analysis of bottlenecks
 
**Hope to achieve:**
  - Real time simulation for moderately sized grids
  - Optimized pressure solver
  - Improved parallel memory locality via layout optimizations
  - A demo of fluid behavior


---

## Platform Choice

We will use both CPU (OpenMP) and GPU (CUDA) platforms to explore different parallelization strategies. GPU’s are generally suited for large data-parallel workloads like the ones we have in our particle updates and grid computations, while the CPU may be better because of irregular memory access patterns and complex computations. Since FLIP combines these two aspects, it is not immediately clear which implementation will perform better. We will test on different hardware ranging from a pc with a RTX 3070 to laptops with integrated graphics.

---

## Schedule

During the first week, we finalize the system design, set up the codebase, and implement core data structures for particles and grids.

In the second week, we implement a complete serial version of the FLIP pipeline, including particle-to-grid transfer, grid updates, and a basic pressure solver.

By the third week, we begin implementing CUDA kernels for the GPU version, focusing on particle-grid transfers and grid updates. This stage corresponds to the milestone report.

In the fourth week, we parallelize the CPU implementation using OpenMP and profile performance to identify bottlenecks. 

During the fifth week, we optimize both implementations by improving memory access patterns and reducing synchronization overhead. We also perform detailed performance comparisons.

In the final week, we complete benchmarking, refine visualization, and prepare the final report and poster.


---

# Milestone Report (April 14, 2025)

## Work Completed

We have successfully implemented a fully functional 3D FLIP fluid simulation with an OpenGL-based real-time renderer. The core simulation pipeline is complete, with particle-to-grid transfer (P2G), gravity application, pressure solving, grid-to-particle transfer (G2P), and particle advection all producing fluid behavior. The simulation uses a MAC (Marker-and-Cell) staggered grid with trilinear interpolation for particle-grid transfers, a Jacobi solver for the pressure Poisson equation, and RK2 integration with adaptive sub-stepping for particle advection.

The CPU backend is parallelized with OpenMP and working correctly, although we haven’t optimized nor made benchmarks for it. The rendering system uses OpenGL with GLFW, displaying particles as colored point sprites with velocity-mapped colors (blue for slow, red for fast). Camera controls support Blender-style orbit, pan, and zoom. The GPU (CUDA) backend has all kernels implemented, and we are in the process of testing it against the CPU reference output.

---

## Goals and Deliverables

Status relative to original plan-to-achieve goals:

| Goal | Status |
|------|--------|
| Working 3D FLIP-based fluid simulator | Complete |
| Parallel CPU implementation using OpenMP | Complete |
| GPU implementation using CUDA | All kernels implemented, validation in progress |
| Evaluation comparing CPU and GPU workloads | Pending (no timing instrumentation yet) |
| Analysis of bottlenecks | Pending |

We believe all plan-to-achieve goals are still reachable by the poster session. The real-time demo (a hope-to-achieve goal) is also achievable given the rendering pipeline is already in place. The optimized pressure solver and memory locality improvements remain stretch goals we will attempt after completing the GPU backend.

Updated goals for the poster session:

- Validate GPU (CUDA) solver output against CPU reference baseline  
- Timing instrumentation and benchmarking across CPU and GPU backends  
- Speedup graphs comparing CPU and GPU per simulation stage (P2G, pressure solve, G2P, advection) and end-to-end  
- Live real-time fluid simulation demo  
- (Stretch) Optimized pressure solver (e.g., Red-Black Gauss-Seidel)  
- (Stretch) Memory layout improvements for better GPU coalescing  

---

## Poster Session Plan

We plan to show both a live interactive demo and performance graphs. The demo will display the real-time 3D fluid simulation with the Blender-style camera so viewers can explore the fluid from different angles. The performance graphs will compare CPU (OpenMP) and GPU (CUDA) execution times across different grid resolutions and particle counts, broken down by simulation stage, and will highlight where GPU parallelism provides the most benefit and where bottlenecks remain.

---

## Preliminary Results

We do not yet have quantitative benchmark numbers, as timing instrumentation has not been added to the codebase. Qualitatively, the CPU simulation runs in real time at a 20×20×20 grid resolution with approximately 27,000 particles, which is encouraging for our real-time demo goal. Adding per-stage timers is a top priority this week to establish a CPU baseline before GPU comparison.

---

## Concerns and Remaining Issues

The largest challenges we have encountered so far:

Pressure solver correctness: Getting the Jacobi solver for the pressure Poisson equation correct was the most significant implementation challenge. Proper handling of air/water/solid cell boundaries, correct divergence computation on the staggered MAC grid, and stable velocity projection required debugging. Small errors in boundary handling caused visible instability in the fluid behavior.

FLIP algorithm correctness: Correctly blending PIC and FLIP velocity updates and getting trilinear interpolation consistent with the staggered MAC grid face offsets required careful attention to grid indexing conventions.

Remaining concerns going forward:

GPU P2G with atomics: Particle-to-grid transfer uses atomicAdd to prevent data races. We need to benchmark whether this limits GPU speedup enough to warrant a sort-based approach instead.

GPU pressure solver scaling: The Jacobi solver's iterative nature is a known bottleneck for GPU implementations.

Host to device transfer overhead: The current step() copies all particle positions and velocities to the GPU at the start of each frame and back at the end. At large particle counts this transfer cost could dominate.

No timing infrastructure yet: This needs to be added immediately to produce meaningful benchmark results for the poster.

---

## Revised Schedule

| Period | Tasks | Owner |
|--------|------|--------|
| Apr 14–17 (Mon–Thu) | Validate GPU solver output against CPU on matching test cases; add CUDA event timing around each kernel stage | Daniel |
| Apr 14–17 (Mon–Thu) | Add std::chrono timing to CPU path; investigate host↔device transfer overhead; assess whether persistent GPU buffers are needed | Samuel |
| Apr 18–21 (Fri–Mon) | Run baseline benchmarks: CPU vs GPU per stage at 20³, 30³, 40³ grid sizes; generate initial speedup data | Both |
| Apr 18–21 (Fri–Mon) | Profile GPU kernels; identify memory bottlenecks; attempt SoA particle layout if coalescing is poor | Samuel |
| Apr 22–25 (Tue–Fri) | (Stretch) Implement Red-Black Gauss-Seidel or tiled Jacobi for improved GPU pressure convergence; benchmark improvement | Daniel |
| Apr 22–25 (Tue–Fri) | Produce final graphs (speedup by stage, overall, scaling); finalize demo scene and rendering quality for poster | Samuel |
| Apr 26–29 (Sat–Tue) | Write final report; prepare poster; rehearse demo; final end-to-end testing on target hardware | Both |
