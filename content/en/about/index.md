---
title: About Prometheus Cluster
linkTitle: About
menu: {main: {weight: 10}}
tags: ["about", "prometheus", "cluster", "hardware", "architecture", "cyens", "hpc", "nvidia", "amd"]
categories: ["cluster-computing", "hardware"]
---

{{% blocks/cover title="" image_anchor="bottom" height="auto" %}}
<div class="text-center mb-4">
<div class="ascii-container">
<pre class="ascii-art" style="color: #ffffff; font-weight: bold; text-shadow: 2px 2px 8px rgba(0,0,0,0.9); background: linear-gradient(135deg, rgba(0, 123, 255, 0.6), rgba(0, 86, 179, 0.7)); border-radius: 10px; border: 2px solid rgba(255,255,255,0.4); display: inline-block; margin: 0; font-family: 'Courier New', monospace;">
   ________  ________  ________  _____ ______   _______  _________  ___  ___  _______   ___  ___  ________        
  |\   __  \|\   __  \|\   __  \|\   _ \  _   \|\  ___ \|\___   ___\\  \|\  \|\  ___ \ |\  \|\  \|\   ____\       
  \ \  \|\  \ \  \|\  \ \  \|\  \ \  \\\__\ \  \ \   __/\|___ \  \_\ \  \\\  \ \   __/|\ \  \\\  \ \  \___|_      
   \ \   ____\ \   _  _\ \  \\\  \ \  \\|__| \  \ \  \_|/__  \ \  \ \ \   __  \ \  \_|/_\ \  \\\  \ \_____  \     
    \ \  \___|\ \  \\  \\ \  \\\  \ \  \    \ \  \ \  \_|\ \  \ \  \ \ \  \ \  \ \  \_|\ \ \  \\\  \|____|\  \    
     \ \__\    \ \__\\ _\\ \_______\ \__\    \ \__\ \_______\  \ \__\ \ \__\ \__\ \_______\ \_______\____\_\  \   
      \|__|     \|__|\|__|\|_______|\|__|     \|__|\|_______|   \|__|  \|__|\|__|\|_______|\|_______|\_________\  
                                                                                                   \|_________|  
</pre>
</div>
<div style="margin-top: 20px;">
<p class="lead ascii-description" style="color: #ffffff; font-weight: bold; text-shadow: 2px 2px 8px rgba(0,0,0,0.9); background: linear-gradient(135deg, rgba(0, 123, 255, 0.8), rgba(0, 86, 179, 0.9)); padding: 20px 30px; border-radius: 10px; border: 2px solid rgba(255,255,255,0.4); display: inline-block;">
High-Performance GPU Computing Cluster at CYENS Centre of Excellence
</p>
</div>
</div>

{{% /blocks/cover %}}

{{% blocks/lead %}}

Prometheus is the flagship high-performance computing cluster at CYENS Centre of Excellence, specifically designed 
for deep learning and artificial intelligence research. This state-of-the-art facility combines powerful NVIDIA GPUs, 
AMD EPYC processors, and enterprise-grade storage to deliver exceptional performance for the most demanding computational workloads.

Running Rocky Linux 8.5 with professional-grade job scheduling and environment management, Prometheus provides 
researchers with a robust, scalable platform for breakthrough AI research and development.

{{% /blocks/lead %}}

{{% blocks/section %}}

## Hardware Architecture
{.text-center}

### Head Node
- **Chassis:** GIGABYTE R182-Z90-00
- **Motherboard:** GIGABYTE MZ92-FS0-00  
- **CPU:** 2x AMD EPYC 7313 (16C/32T each)
- **RAM:** 16x 32GB Samsung modules (512GB total)
- **Storage:** 2x 1.92TB Intel SSDs

### Compute Nodes (gpu01-08)
- **Chassis:** Supermicro AS-4124GS-TNR
- **Motherboard:** Supermicro H12DSG-O-CPU
- **CPU:** 2x AMD EPYC 7313 (16C/32T each)
- **GPU:** 8x NVIDIA A5000 (24GB VRAM, 8192 CUDA cores, 256 Tensor cores)
- **Performance:** 27.8 TFLOPS FP32 per GPU
- **RAM:** 16x 32GB SK Hynix modules (512GB total)
- **Storage:** 1x 1TB Samsung SSD 980

### Specialized Compute Node (gpu09)
- **Chassis:** ASUS RS720A-E11-RS12
- **Motherboard:** ASUS KMPP-D32
- **CPU:** 2x AMD EPYC 7313 (16C/32T each)
- **GPU:** 4x NVIDIA A6000 Ada (48GB VRAM, 18176 CUDA cores, 568 Tensor cores)
- **Performance:** 91.06 TFLOPS FP32 per GPU
- **RAM:** 16x 32GB SK Hynix modules (512GB total)
- **Storage:** 1x 1TB Samsung SSD 980

### Storage Infrastructure
- **Storage Nodes:** 2x Supermicro servers
- **CPU:** AMD EPYC 7302P (16C/32T each)
- **RAM:** 8x 16GB Samsung modules (256GB total)
- **Storage System:** 24x 7.68TB Samsung SSDs
- **Total Capacity:** 305TB Lustre parallel filesystem

### Networking Infrastructure
- **Management Switch:** Netgear M4300-52G - 48x1G Stackable Managed Switch with 2x10GBASE-T and 2xSFP+
- **Interconnection Switch:** Mellanox HDR InfiniBand QM8700 Switch, 40x QSFP56 ports, 2 Power Supplies (AC), x86 dual core, standard depth, P2C airflow
- **Interconnection Cables:** 11x Mellanox passive copper hybrid cable, IB HDR 200Gb/s to 2x100Gb/s, QSFP28 to 2xQSFP28, colored pulltabs, 2m, 26AWG

{{% /blocks/section %}}

{{% blocks/section color="dark" %}}

## Operating Environment
{.text-center}

### System Software
- **Operating System:** Rocky Linux 8.5 (Green Obsidian)
- **Kernel:** Linux 4.18.0-348.23.1.el8_5.x86_64
- **Job Scheduler:** Slurm Workload Manager
- **Module System:** Lmod Environment Modules

### Partitions and Scheduling
- **defq Partition:** 8 nodes (gpu01-08) with NVIDIA A5000 GPUs
- **a6000 Partition:** 1 node (gpu09) with NVIDIA A6000 Ada GPUs
- **Priority Queues:** normal, long, preemptive with different time limits and priorities
- **Resource Limits:** Per-group quotas for fair resource allocation

### Storage Quotas
- **Home Directories:** 20GB per user (/trinity/home)
- **Work Directories:** 30TB per group (/lustreFS/data)
- **Performance:** High-throughput Lustre filesystem optimized for parallel I/O

{{% /blocks/section %}}

{{% blocks/section %}}

## Access and Development
{.text-center}

### Connection Methods
- **SSH Access:** Secure shell connections with RSA key authentication
- **VS Code Remote:** Direct IDE integration via Remote-SSH extension
- **Interactive Jobs:** Real-time development with `srun` command
- **Batch Processing:** Automated job submission with `sbatch`

### Software Environment
- **Environment Modules:** Easy software stack management with Lmod
- **Pre-configured Libraries:** CUDA, deep learning frameworks, scientific computing tools
- **Container Support:** Singularity/Apptainer for reproducible environments
- **Development Tools:** Git, compilers, debuggers, and profiling tools

### Supported Frameworks
- **Deep Learning:** PyTorch, TensorFlow, JAX, Hugging Face Transformers
- **Scientific Computing:** NumPy, SciPy, scikit-learn, OpenCV
- **Specialized Libraries:** MinkowskiEngine, PointGPT, and domain-specific tools
- **Data Processing:** Pandas, Dask, and high-performance data analytics

{{% /blocks/section %}}

{{% blocks/section color="white" %}}

## Research Capabilities
{.text-center}

### AI and Machine Learning
Prometheus excels at training large neural networks, from computer vision models to natural language processing 
systems. The combination of NVIDIA A5000 and A6000 Ada GPUs provides the computational power needed for 
state-of-the-art research in artificial intelligence.

### Scientific Computing
Beyond AI, the cluster supports a wide range of scientific applications including computational biology, 
physics simulations, climate modeling, and data-intensive research across multiple disciplines.

### Collaborative Research
With group-based resource allocation and shared storage, Prometheus facilitates collaborative research 
projects while maintaining security and fair resource distribution among research teams.

### Performance Optimization
Professional-grade job scheduling ensures optimal resource utilization, while the high-performance Lustre 
filesystem minimizes I/O bottlenecks for data-intensive applications.

{{% /blocks/section %}}