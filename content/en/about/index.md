---
title: About Prometheus
linkTitle: About
menu: {main: {weight: 10}}
---

{{% blocks/cover title="About the Prometheus Cluster" image_anchor="bottom" height="auto" %}}

A high-performance computing cluster for AI and machine learning research at CYENS.
{.mt-5}

{{% /blocks/cover %}}

{{% blocks/lead %}}

Goldydocs is a sample site using the [Docsy](https://github.com/google/docsy)
Hugo theme that shows what it can do and provides you with a template site
structure. It’s designed for you to clone and edit as much as you like. See the
different sections of the documentation and site for more ideas.

{{% /blocks/lead %}}

{{% blocks/section %}}

## Hardware Specifications
{.text-center}

**Compute Power:** 9 compute nodes featuring powerful AMD EPYC processors and NVIDIA GPUs  
**GPU Resources:** NVIDIA A5000 (24GB) and A6000 Ada (48GB) graphics cards  
**Memory:** Up to 512GB RAM per node for memory-intensive applications  
**Storage:** High-speed NVMe SSDs and scalable Lustre filesystem (305TB capacity)  
**Networking:** HDR InfiniBand for ultra-low latency inter-node communication

{{% /blocks/section %}}

{{% blocks/section %}}

## Software Environment
{.text-center}

**Operating System:** Rocky Linux 8.5 for enterprise-grade stability  
**Job Management:** Slurm workload manager for efficient resource allocation  
**Module System:** Lmod environment modules for flexible software management  
**Development Tools:** Pre-installed libraries and frameworks for AI/ML development

The cluster supports popular frameworks including PyTorch, TensorFlow, and specialized libraries 
like MinkowskiEngine for sparse tensor operations and PointGPT for point cloud processing.

{{% /blocks/section %}}