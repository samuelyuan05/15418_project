# 3D FLIP Fluid Simulation with CPU/GPU Parallelization

## Authors
Samuel Yuan, [Partner Name]

---

## Overview

This project implements a three-dimensional FLIP (Fluid-Implicit Particle) fluid simulator and explores parallelization strategies across CPU and GPU architectures. The system combines particle-based and grid-based computation, making it an ideal workload for studying performance tradeoffs between OpenMP and CUDA.

Our primary goal is to design efficient parallel implementations of the most computationally intensive stages of the FLIP pipeline and evaluate how hardware characteristics such as memory bandwidth, synchronization overhead, and execution divergence impact performance.

---

## Summary

We implement a 3D FLIP fluid solver in C++ and optimize it using OpenMP for CPU parallelism and CUDA for GPU acceleration. The project focuses on parallelizing particle-grid transfers and the pressure solver, while analyzing performance bottlenecks and architectural tradeoffs between CPUs and GPUs.

---

## Background

FLIP (Fluid-Implicit Particle) is a hybrid fluid simulation technique that combines Lagrangian particles with an Eulerian grid. Particles store physical quantities such as velocity and mass, while a background grid is used to efficiently compute forces and enforce incompressibility.

Compared to purely particle-based methods like SPH, FLIP reduces computational complexity by leveraging structured grid operations. It also minimizes numerical dissipation by preserving particle velocities during simulation updates.

Each timestep of the simulation consists of the following stages:

1. Particle-to-grid transfer, where particle data is interpolated onto nearby grid cells.
2. Grid update, where forces such as gravity are applied and intermediate quantities are computed.
3. Pressure solve, where a Poisson equation is solved to enforce incompressibility.
4. Velocity projection, which removes divergence from the velocity field.
5. Grid-to-particle transfer, where updated velocities are mapped back to particles.

The most computationally expensive components are the particle-grid transfers and the pressure solver, both of which present opportunities for parallelization.

---

## The Challenge

The FLIP simulation presents a hybrid workload that combines highly parallel particle operations with structured grid computations that involve dependencies and synchronization.

Particle operations are well-suited for parallel execution, but they require irregular memory access patterns due to scattered particle positions. In contrast, grid-based computations benefit from strong spatial locality but introduce dependencies, especially during iterative solvers such as the pressure solve.

One of the key challenges is the particle-to-grid transfer stage, which requires multiple particles to update shared grid cells. This creates potential race conditions and necessitates the use of atomic operations or reduction strategies, particularly on the GPU.

The pressure solver is another major challenge because it requires iterative computation over the entire grid. Each iteration depends on the results of the previous one, introducing synchronization overhead that limits scalability.

On GPU architectures, additional challenges arise from thread divergence caused by non-uniform particle distributions and load imbalance across threads. On CPUs, the main limitation is lower throughput compared to GPUs, although CPUs may handle irregular memory access patterns more efficiently.

This project aims to explore how these competing factors influence performance and determine when each hardware platform is most effective.

---

## Resources

The implementation is written in C++.

We use OpenMP to parallelize CPU execution and CUDA to implement GPU kernels. For visualization, we use OpenGL with GLFW, and GLM for vector and matrix operations.

We reference the following materials:
- *FLIP: A Low-Dissipation, Particle-in-Cell Method for Fluid Flow*
- SIGGRAPH 2007 Fluid Simulation Course Notes
- Public implementations and technical blogs on FLIP simulation

We begin implementation from scratch to fully control data structures and parallelization strategies.

---

## Goals and Deliverables

The primary goal of this project is to build a working 3D FLIP simulator and evaluate parallel performance across CPU and GPU implementations.

We plan to deliver a complete simulation pipeline with both OpenMP and CUDA implementations. We will benchmark performance across different stages of the algorithm and analyze bottlenecks such as synchronization, memory access patterns, and communication overhead.

We aim to achieve at least a 5× speedup on the GPU compared to a single-threaded CPU baseline. Our target simulation size is a grid of at least 64³ cells with approximately 100,000 particles.

In addition to core functionality, we hope to achieve near real-time simulation performance for moderate grid sizes and explore optimizations such as improved memory layouts and more efficient pressure solvers.

If time permits, we will include a real-time visualization of the simulation and a side-by-side comparison of CPU and GPU performance.

---

## Platform Choice

We use both CPU and GPU platforms to explore different parallelization strategies.

GPUs are well-suited for high-throughput data-parallel workloads such as particle updates and grid computations. CPUs, on the other hand, provide more flexibility for handling irregular memory access patterns and complex control flow.

Because FLIP combines both structured and unstructured computation, it provides an excellent opportunity to evaluate the strengths and weaknesses of each platform.

---

## Schedule

During the first week, we finalize the system design, set up the codebase, and implement core data structures for particles and grids.

In the second week, we implement a complete serial version of the FLIP pipeline, including particle-to-grid transfer, grid updates, and a basic pressure solver.

By the third week, we parallelize the CPU implementation using OpenMP and profile performance to identify bottlenecks. This stage corresponds to the milestone report.

In the fourth week, we begin implementing CUDA kernels for the GPU version, focusing on particle-grid transfers and grid updates.

During the fifth week, we optimize both implementations by improving memory access patterns and reducing synchronization overhead. We also perform detailed performance comparisons.

In the final week, we complete benchmarking, refine visualization, and prepare the final report and poster.

---

## Expected Results

We expect to observe significant speedups for data-parallel portions of the simulation on the GPU, particularly during particle updates and grid operations.

However, we anticipate that the pressure solver and synchronization-heavy stages will limit scalability. The results of this project will provide insight into how hybrid workloads behave across different parallel architectures.

---
