# 3D FLIP Fluid Simulation with CPU/GPU Parallelization

## Authors
Samuel Yuan, Daniel Stankiewicz

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
  - 
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
