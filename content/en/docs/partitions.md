---
title: "Partitions & Queues"
linkTitle: "Partitions"
weight: 25
description: >
  Understanding SLURM partitions and priority queues on Prometheus
tags: ["partitions", "slurm", "queues", "resource-management", "scheduling", "defq", "a6000"]
categories: ["job-submission", "cluster-computing"]
---

## Overview

The Prometheus cluster has **two partitions** with different priority queues (QoS) that control resource limits and scheduling priority. All limits are applied **per group**, and the default time limit is **4 hours** for all partitions.

## Partition Architecture

### `defq` Partition (Default)
- **Nodes**: 8 compute nodes (`gpu[01-08]`)
- **GPU Type**: NVIDIA A5000 (24GB VRAM each)
- **Total GPUs**: 64 (8 GPUs per node)
- **Default partition**: Jobs submitted without specifying partition go here

### `a6000` Partition
- **Nodes**: 1 compute node (`gpu09`)
- **GPU Type**: NVIDIA RTX A6000 Ada Generation (48GB VRAM each)
- **Total GPUs**: 4
- **Use case**: High-memory GPU workloads

## Priority Queues (QoS)

{{% alert title="Resource Limits" color="info" %}}
All resource limits are applied **per group**, not per user. Coordinate with your group members to avoid conflicts.
{{% /alert %}}

### `defq` Partition Queues

<table class="table table-striped">
<thead>
<tr>
<th>Priority Queue</th>
<th>Time Limit</th>
<th>Max CPUs</th>
<th>Max GPUs</th>
<th>Max RAM</th>
<th>Max Jobs</th>
<th>Priority</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>normal</code></td>
<td>1 day</td>
<td>384</td>
<td>48</td>
<td>3TB</td>
<td>30</td>
<td>High</td>
</tr>
<tr>
<td><code>long</code></td>
<td>7 days</td>
<td>384</td>
<td>48</td>
<td>3TB</td>
<td>20</td>
<td>Medium</td>
</tr>
<tr>
<td><code>preemptive</code></td>
<td>Infinite</td>
<td>All*</td>
<td>All*</td>
<td>All*</td>
<td>10</td>
<td>Low</td>
</tr>
</tbody>
</table>

### `a6000` Partition Queues

<table class="table table-striped">
<thead>
<tr>
<th>Priority Queue</th>
<th>Time Limit</th>
<th>Max CPUs</th>
<th>Max GPUs</th>
<th>Max RAM</th>
<th>Max Jobs</th>
<th>Priority</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>normal-a6000</code></td>
<td>1 day</td>
<td>48</td>
<td>3</td>
<td>384GB</td>
<td>6</td>
<td>High</td>
</tr>
<tr>
<td><code>long-a6000</code></td>
<td>7 days</td>
<td>48</td>
<td>3</td>
<td>384GB</td>
<td>4</td>
<td>Medium</td>
</tr>
<tr>
<td><code>preemptive-a6000</code></td>
<td>Infinite</td>
<td>All*</td>
<td>All*</td>
<td>All*</td>
<td>2</td>
<td>Low</td>
</tr>
</tbody>
</table>

\* **Preemptive queues** can use all available resources but jobs may be automatically terminated when higher-priority jobs need resources.

## Queue Selection Guidelines

### Use `normal` or `normal-a6000` for:
- **Interactive development** and testing
- **Short training runs** (< 24 hours)
- **Production jobs** that need guaranteed completion
- **Debugging** and experimentation

```bash
#SBATCH --partition=defq
#SBATCH --qos=normal
#SBATCH --time=12:00:00
```

### Use `long` or `long-a6000` for:
- **Extended training** (1-7 days)
- **Large model training** requiring multiple days
- **Parameter sweeps** with many iterations
- **Production workloads** with longer time requirements

```bash
#SBATCH --partition=defq
#SBATCH --qos=long
#SBATCH --time=3-00:00:00  # 3 days
```

### Use `preemptive` queues sparingly for:
- **Low-priority background jobs**
- **Opportunistic computing** when cluster is idle
- **Jobs that can handle interruption** (with checkpointing)
- **Testing with unlimited time**

{{% alert title="Warning" color="warning" %}}
Preemptive jobs can be **automatically terminated** at any time! Use the `requeue` option and implement checkpointing.
{{% /alert %}}

```bash
#SBATCH --partition=defq
#SBATCH --qos=preemptive
#SBATCH --requeue          # Automatically resubmit if preempted
#SBATCH --time=7-00:00:00
```

## Choosing the Right Partition

### Use `defq` partition when:
- Your models fit in **24GB GPU memory**
- You need **multiple GPUs** (up to 8 per node)
- Running **distributed training** across nodes
- Working with **standard deep learning models**

### Use `a6000` partition when:
- Your models require **> 24GB GPU memory**
- Training **large language models** (70B+ parameters)
- Working with **high-resolution images** or **long sequences**
- Need **maximum GPU memory** per device

## Example Job Submissions

### Standard Training Job (defq/normal)

```bash
#!/bin/bash
#SBATCH -J standard_training
#SBATCH --partition=defq
#SBATCH --qos=normal
#SBATCH --cpus-per-task=8
#SBATCH --gres=gpu:2
#SBATCH --mem=64000
#SBATCH --time=18:00:00

# Your training code here
python train_model.py --gpus 2
```

### Long Training Job (defq/long)

```bash
#!/bin/bash
#SBATCH -J long_training
#SBATCH --partition=defq
#SBATCH --qos=long
#SBATCH --cpus-per-task=16
#SBATCH --gres=gpu:4
#SBATCH --mem=128000
#SBATCH --time=5-00:00:00  # 5 days

# Long-running training with checkpointing
python train_model.py --gpus 4 --checkpoint-freq 1000
```

### Large Model Training (a6000/normal-a6000)

```bash
#!/bin/bash
#SBATCH -J large_model
#SBATCH --partition=a6000
#SBATCH --qos=normal-a6000
#SBATCH --cpus-per-task=16
#SBATCH --gres=gpu:2
#SBATCH --mem=256000
#SBATCH --time=20:00:00

# Large model requiring high GPU memory
python train_llm.py --model-size 70B --gpus 2
```

### Preemptive Job with Checkpointing

```bash
#!/bin/bash
#SBATCH -J preemptive_job
#SBATCH --partition=defq
#SBATCH --qos=preemptive
#SBATCH --cpus-per-task=4
#SBATCH --gres=gpu:1
#SBATCH --mem=32000
#SBATCH --time=7-00:00:00
#SBATCH --requeue
#SBATCH --signal=SIGUSR1@90  # Signal 90 seconds before termination

# Handle preemption gracefully
trap 'echo "Job preempted, saving checkpoint..."; python save_checkpoint.py' SIGUSR1

python train_model.py --resume-from-checkpoint
```

## Monitoring Queue Status

### Check Partition Information

```bash
# View all partitions and their status
sinfo

# Example output:
# PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
# defq*        up   infinite      6  idle~ gpu[03-08]
# defq*        up   infinite      1    mix gpu02
# defq*        up   infinite      1   idle gpu01
# a6000        up   infinite      1    mix gpu09
```

### Check Queue Status

```bash
# View current job queue
squeue

# View jobs by partition
squeue -p defq
squeue -p a6000

# View jobs by QoS
squeue --qos=normal
squeue --qos=long
```

### Check Your Resource Usage

```bash
# View your running jobs
squeue -u $USER

# Check job priorities
sprio -u $USER

# View resource limits
sacctmgr show qos format=Name,MaxWall,MaxTRES,MaxJobs
```

## Resource Planning

### Calculate Resource Needs

Before submitting jobs, consider:

1. **GPU Memory Requirements**:
   - Small models (< 1B params): A5000 (24GB)
   - Large models (> 10B params): A6000 (48GB)

2. **Training Time Estimates**:
   - Quick experiments: `normal` queue (< 1 day)
   - Full training: `long` queue (1-7 days)

3. **Number of GPUs**:
   - Single GPU: Any node
   - Multi-GPU: Consider node topology
   - Distributed: Multiple nodes in `defq`

### Group Coordination

Since limits are per group:

1. **Communicate with group members**
2. **Check current group usage**:
   ```bash
   squeue -A your-group-name
   ```
3. **Plan resource allocation** to avoid conflicts

## Best Practices

### Queue Selection Strategy

1. **Start with `normal` queues** for development
2. **Use `long` queues** only when necessary
3. **Avoid preemptive queues** unless jobs can handle interruption
4. **Test on smaller resources** before scaling up

### Resource Efficiency

1. **Don't over-allocate resources**:
   ```bash
   # Bad: Requesting 8 GPUs for single-GPU code
   #SBATCH --gres=gpu:8
   
   # Good: Request what you actually use
   #SBATCH --gres=gpu:1
   ```

2. **Use appropriate memory**:
   ```bash
   # Calculate actual memory needs
   #SBATCH --mem=32000  # 32GB, not 500GB
   ```

3. **Estimate time accurately**:
   ```bash
   # Add buffer but don't overestimate
   #SBATCH --time=18:00:00  # 18 hours, not 7 days
   ```

## Troubleshooting

### Job Stuck in Queue

```bash
# Check why job is pending
scontrol show job <job_id> | grep Reason

# Common reasons:
# - Resources: Requesting more than available
# - Priority: Lower priority than other jobs
# - QoSMaxJobsPerUser: Too many jobs running
```

### Resource Limit Exceeded

```bash
# Check current group usage
squeue -A your-group

# Reduce resource requests or wait for jobs to complete
```

### Wrong Partition Choice

```bash
# Cancel and resubmit with correct partition
scancel <job_id>
# Edit script and resubmit
sbatch corrected_script.slurm
```

## Next Steps

- **Learn about storage systems**: [Storage Guide](../storage/)
- **Set up your environment**: [Environment Modules](../modules/)
- **Configure VS Code**: [VS Code Setup](../vscode/)