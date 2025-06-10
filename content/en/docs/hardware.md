---
title: "Hardware Specifications"
linkTitle: "Hardware"
weight: 15
description: >
  Detailed hardware specifications of the Prometheus cluster
tags: ["hardware", "gpu", "nvidia", "a5000", "a6000", "specs", "cluster-architecture"]
categories: ["hardware", "cluster-computing"]
---

## Cluster Overview

The Prometheus cluster features a modern architecture optimized for deep learning workloads with high-performance GPUs, abundant memory, and fast storage systems.

## Head Node

**Management and login node for the cluster**

### Hardware Configuration
- **Chassis**: GIGABYTE R182-Z90-00
- **Motherboard**: GIGABYTE MZ92-FS0-00
- **CPU**: 2× AMD EPYC 7313 (16 cores/32 threads each)
- **Total CPU Cores**: 32 cores / 64 threads
- **RAM**: 16× 32GB Samsung M393A4K40EB3-CWE
- **Total RAM**: 512GB DDR4
- **Storage**: 2× 1.92TB Intel SSDSC2KB019T8 SSD
- **File System**: `/trinity/home` (400GB allocated)

### Purpose
- SSH login and job submission
- File management and transfers
- SLURM job scheduling
- **Not for compute workloads**

## Compute Nodes

### GPU Nodes `gpu[01-08]` (8 nodes)

**Primary compute nodes with NVIDIA A5000 GPUs**

#### Hardware Configuration
- **Chassis**: Supermicro AS-4124GS-TNR
- **Motherboard**: Supermicro H12DSG-O-CPU
- **CPU**: 2× AMD EPYC 7313 (16 cores/32 threads each)
- **Total CPU Cores**: 32 cores / 64 threads per node
- **RAM**: 16× 32GB SK Hynix HMAA4GR7AJR8N-XN
- **Total RAM**: 512GB DDR4 per node
- **Local Storage**: 1× 1TB Samsung SSD 980 NVMe

#### GPU Specifications
- **GPU Model**: NVIDIA A5000
- **GPU Count**: 8 GPUs per node (64 total across all nodes)
- **GPU Memory**: 24GB GDDR6 per GPU
- **CUDA Cores**: 8,192 per GPU
- **Tensor Cores**: 256 RT Cores (2nd gen)
- **Peak Performance**: 27.8 TFLOPS FP32 per GPU
- **Memory Bandwidth**: 768 GB/s per GPU

#### Total Resources (gpu[01-08])
- **Total GPUs**: 64× NVIDIA A5000
- **Total GPU Memory**: 1,536GB (1.5TB)
- **Total CPU Cores**: 256 cores / 512 threads
- **Total System RAM**: 4TB DDR4

### GPU Node `gpu09` (1 node)

**High-memory GPU node with NVIDIA A6000 Ada**

#### Hardware Configuration
- **Chassis**: ASUS RS720A-E11-RS12
- **Motherboard**: ASUS KMPP-D32
- **CPU**: 2× AMD EPYC 7313 (16 cores/32 threads each)
- **Total CPU Cores**: 32 cores / 64 threads
- **RAM**: 16× 32GB SK Hynix HMAA4GR7AJR8N-XN
- **Total RAM**: 512GB DDR4
- **Local Storage**: 1× 1TB Samsung SSD 980 NVMe

#### GPU Specifications
- **GPU Model**: NVIDIA RTX A6000 Ada Generation
- **GPU Count**: 4 GPUs
- **GPU Memory**: 48GB GDDR6 per GPU
- **CUDA Cores**: 18,176 per GPU
- **Tensor Cores**: 568 (4th gen)
- **Peak Performance**: 91.06 TFLOPS FP32 per GPU
- **Memory Bandwidth**: 960 GB/s per GPU

#### Total Resources (gpu09)
- **Total GPUs**: 4× NVIDIA A6000 Ada
- **Total GPU Memory**: 192GB
- **Total CPU Cores**: 32 cores / 64 threads
- **Total System RAM**: 512GB DDR4

## Storage Nodes

### Storage Architecture

**High-performance parallel file system**

#### Hardware Configuration (2 storage nodes)
- **Chassis**: Supermicro Super Server
- **Motherboard**: Supermicro H12SSL-i
- **CPU**: 1× AMD EPYC 7302P (16 cores/32 threads each)
- **RAM**: 8× 16GB Samsung M393A2K40DB3-CWE
- **Total RAM**: 256GB DDR4 per node
- **OS Storage**: 2× 240GB Intel SSDSC2KB240G7 SSD
- **Data Storage**: 24× 7.68TB Samsung MZILT7T6HALA/007 NVMe SSD

#### Storage Specifications
- **File System**: Lustre parallel file system
- **Mount Point**: `/lustreFS`
- **Raw Capacity**: 368TB per storage node
- **Total Raw Capacity**: 736TB across both nodes
- **Usable Capacity**: ~305TB (after RAID and file system overhead)
- **Performance**: High-throughput parallel I/O

## Software Environment

### Operating System
- **Distribution**: Rocky Linux 8.5 (Green Obsidian)
- **Kernel Version**: 4.18.0-348.23.1.el8_5.x86_64
- **Architecture**: x86_64

### Management Software
- **Job Scheduler**: SLURM Workload Manager
- **Module System**: Lmod (Lua-based Environment Modules)
- **File System**: Lustre for parallel storage

### Development Tools
- **CUDA Toolkit**: 11.3+ with cuDNN
- **Compilers**: GCC, Intel, NVCC
- **MPI**: OpenMPI, MPICH
- **Python**: Multiple versions with conda/pip
- **Deep Learning**: PyTorch, TensorFlow, JAX
- **Containers**: Singularity/Apptainer support

## Network Architecture

### Interconnect
- **Compute Network**: High-speed Ethernet
- **Storage Network**: Dedicated Lustre network
- **Management Network**: Separate administrative network

### Bandwidth
- **Node-to-Node**: High-bandwidth for distributed training
- **Storage Access**: Optimized for parallel I/O workloads
- **External Access**: Internet connectivity for downloads

## Performance Characteristics

### Compute Performance
- **Total GPU Performance**: 
  - A5000 nodes: 1,779 TFLOPS FP32 (64 × 27.8)
  - A6000 node: 364 TFLOPS FP32 (4 × 91.06)
  - **Combined**: ~2,143 TFLOPS FP32
- **Memory Bandwidth**: 
  - A5000 total: 49,152 GB/s
  - A6000 total: 3,840 GB/s
  - **Combined**: ~53TB/s GPU memory bandwidth

### Storage Performance
- **Lustre File System**: High-throughput parallel I/O
- **Local NVMe**: High IOPS for temporary data
- **Home Directories**: SSD-backed for fast access

## Resource Allocation

### Per-Node Resources
- **CPU Cores**: 32 physical / 64 logical per node
- **System Memory**: 512GB DDR4 per node
- **GPU Memory**: 
  - A5000 nodes: 192GB per node (8 × 24GB)
  - A6000 node: 192GB (4 × 48GB)
- **Local Storage**: 1TB NVMe SSD per compute node

### Total Cluster Resources
- **Compute Nodes**: 9 total
- **CPU Cores**: 288 physical / 576 logical
- **System Memory**: 4.5TB DDR4
- **GPUs**: 68 total (64 A5000 + 4 A6000 Ada)
- **GPU Memory**: 1.728TB total
- **Shared Storage**: 305TB Lustre + local NVMe

## Use Cases and Workloads

### Optimized For
- **Large Language Models**: High GPU memory for transformer models
- **Computer Vision**: Parallel training on multiple GPUs
- **Distributed Training**: Multi-node deep learning
- **High-throughput Computing**: Batch processing workflows
- **Interactive Development**: Jupyter notebooks and VS Code

### Performance Considerations
- **Memory-bound workloads**: Benefit from A6000's 48GB VRAM
- **Compute-intensive tasks**: Leverage A5000's efficiency
- **Data-intensive jobs**: Utilize high-performance Lustre storage
- **Multi-GPU training**: Scale across nodes with SLURM