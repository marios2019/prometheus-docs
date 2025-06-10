---
title: "Prometheus Cluster Documentation"
linkTitle: "Docs"
weight: 20
menu:
  main:
    weight: 20
description: >
  Complete guide to using the Prometheus deep learning cluster at CYENS
tags: ["cluster", "documentation", "deep-learning", "hpc", "prometheus", "gpu-computing"]
categories: ["documentation", "cluster-computing"]
---

{{% pageinfo %}}
This section contains comprehensive documentation for the Prometheus cluster - a high-performance computing environment for deep learning research at CYENS.
{{% /pageinfo %}}

## Overview

The Prometheus cluster is a state-of-the-art deep learning computing facility featuring:

- **64 NVIDIA A5000 GPUs** (24GB each) across 8 compute nodes
- **4 NVIDIA A6000 Ada GPUs** (48GB each) on a dedicated node
- **4.6TB total GPU memory** for large-scale model training
- **High-performance Lustre storage** with 305TB capacity
- **SLURM job scheduler** for efficient resource management

This documentation will guide you through:

- **[Getting Started](getting-started/)** - SSH access and account setup
- **[Hardware Specifications](hardware/)** - Detailed cluster architecture
- **[Job Submission](job-submission/)** - SLURM batch and interactive jobs
- **[Partitions & Queues](partitions/)** - Resource allocation policies
- **[Storage Systems](storage/)** - File systems and quotas
- **[Environment Modules](modules/)** - Software stack management
- **[VS Code Setup](vscode/)** - Remote development environment
- **[Software Installation](software/)** - Third-party libraries and tools

## Quick Start

1. **Generate SSH keys** and request cluster access
2. **Connect via SSH** to `prometheus.cyens.org.cy`
3. **Submit your first job** using SLURM
4. **Set up development environment** with modules or containers

## Cluster Specifications

### Compute Resources
- **9 compute nodes** total
- **GPU nodes `gpu[01-08]`**: 8×A5000 GPUs each (64 total GPUs)
- **GPU node `gpu09`**: 4×A6000 Ada GPUs (48GB VRAM each)
- **512GB RAM** per compute node
- **32 CPU cores** per node (AMD EPYC 7313)

### Storage
- **Home directories**: 20GB SSD per user
- **Shared storage**: 30TB Lustre filesystem per group
- **Local storage**: 1TB NVMe SSD per compute node

### Networking Infrastructure
- **Management Network**: Netgear M4300-52G switch with 48×1G ports plus 2×10GBASE-T and 2×SFP+
- **High-Performance Interconnect**: Mellanox HDR InfiniBand switch with 40×QSFP56 ports
- **InfiniBand Speed**: 200Gb/s HDR connectivity with hybrid copper cables
- **Low Latency**: Sub-microsecond messaging for distributed computing workloads

### Software Environment
- **Rocky Linux 8.5** operating system
- **SLURM** workload manager
- **Lmod** environment modules
- **CUDA 11.3+** with deep learning frameworks

## Support & Resources

- **System administrators**: Contact your MRG leader
- **Documentation**: This site and `/opt/cluster/docs/`
- **Cluster status**: Monitor with `sinfo` and `squeue`

For detailed instructions, start with the [Getting Started](getting-started/) guide.