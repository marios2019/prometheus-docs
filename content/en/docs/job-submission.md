---
title: "Job Submission"
linkTitle: "Job Submission"
weight: 20
description: >
  Submit and manage jobs using SLURM on the Prometheus cluster
tags: ["slurm", "job-submission", "batch-jobs", "interactive-jobs", "gpu-computing", "cluster"]
categories: ["job-submission", "cluster-computing"]
---

## SLURM Job Scheduler

The Prometheus cluster uses **SLURM** (Simple Linux Utility for Resource Management) for job scheduling and resource allocation. SLURM ensures fair resource sharing and efficient cluster utilization.

{{% alert title="Important" color="warning" %}}
**NEVER run compute jobs directly on the head node!** Always use SLURM to submit jobs to compute nodes.
{{% /alert %}}

## Interactive Jobs

Interactive jobs are perfect for development, testing, and debugging. Use `srun` to request resources immediately.

### Basic Interactive Session

```bash
# Request 1 GPU, 2 CPUs, 1GB RAM for 1 hour
srun -c 1 -n 1 -p defq --qos=normal --mem=100 --gres=gpu:1 -t 01:00 --pty /bin/bash
```

### Interactive Session Parameters

```bash
# Request specific resources
srun --partition=defq \
     --qos=normal \
     --cpus-per-task=4 \
     --gres=gpu:2 \
     --mem=20000 \
     --time=04:00:00 \
     --pty /bin/bash
```

### A6000 Interactive Session

```bash
# Request A6000 GPU with high memory
srun --partition=a6000 \
     --qos=normal-a6000 \
     --cpus-per-task=8 \
     --gres=gpu:1 \
     --mem=50000 \
     --time=02:00:00 \
     --pty /bin/bash
```

## Batch Jobs

Batch jobs run unattended and are ideal for training models, parameter sweeps, and long-running experiments.

### Basic Batch Script

Create a file `train_model.slurm`:

```bash
#!/bin/bash
#SBATCH -o res_%j.txt      # output file
#SBATCH -e res_%j.err      # error file
#SBATCH -J my_training     # job name
#SBATCH --partition=defq   # partition
#SBATCH --qos=normal       # priority queue
#SBATCH --ntasks=1         # number of tasks
#SBATCH --cpus-per-task=4  # CPU cores per task
#SBATCH --gres=gpu:2       # number of GPUs
#SBATCH --mem=32000        # memory in MB
#SBATCH --time=1-12:00     # time limit (1 day, 12 hours)

# Load required modules
module load CUDA/11.3.1
module load Python/3.9.5

# Activate your environment
source ~/anaconda3/bin/activate
conda activate myenv

# Set CUDA devices
export CUDA_VISIBLE_DEVICES=0,1

# Run your training script
cd /lustreFS/data/mygroup/myproject
python train_model.py --epochs 100 --batch-size 64
```

### Submit the Batch Job

```bash
sbatch train_model.slurm
```

## SLURM Parameters Reference

### Common SBATCH Directives

| Parameter | Description | Example |
|-----------|-------------|---------|
| `-J, --job-name` | Job name | `#SBATCH -J training_job` |
| `-o, --output` | Output file | `#SBATCH -o output_%j.txt` |
| `-e, --error` | Error file | `#SBATCH -e error_%j.err` |
| `-p, --partition` | Partition | `#SBATCH --partition=defq` |
| `--qos` | Priority queue | `#SBATCH --qos=normal` |
| `-n, --ntasks` | Number of tasks | `#SBATCH --ntasks=1` |
| `-c, --cpus-per-task` | CPUs per task | `#SBATCH --cpus-per-task=8` |
| `--gres` | Generic resources | `#SBATCH --gres=gpu:4` |
| `--mem` | Memory (MB) | `#SBATCH --mem=64000` |
| `-t, --time` | Time limit | `#SBATCH --time=2-00:00` |

### Time Format Examples

```bash
# 30 minutes
#SBATCH --time=00:30:00

# 4 hours
#SBATCH --time=04:00:00

# 1 day, 12 hours
#SBATCH --time=1-12:00:00

# 7 days (maximum for long queue)
#SBATCH --time=7-00:00:00
```

### GPU Allocation Examples

```bash
# Request any available GPU
#SBATCH --gres=gpu:1

# Request multiple GPUs
#SBATCH --gres=gpu:4

# All GPUs on A5000 node (8 GPUs)
#SBATCH --gres=gpu:8

# A6000 GPU specifically
#SBATCH --partition=a6000 --gres=gpu:1
```

## Advanced Job Examples

### Multi-GPU Training Script

```bash
#!/bin/bash
#SBATCH -J multi_gpu_training
#SBATCH --partition=defq
#SBATCH --qos=long
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --gres=gpu:4
#SBATCH --mem=128000
#SBATCH --time=2-00:00:00
#SBATCH -o logs/multi_gpu_%j.out
#SBATCH -e logs/multi_gpu_%j.err

# Ensure logs directory exists
mkdir -p logs

# Load modules
module load CUDA/11.3.1 Python/3.9.5

# Activate environment
conda activate pytorch-env

# Set environment variables
export CUDA_VISIBLE_DEVICES=0,1,2,3
export OMP_NUM_THREADS=4

# Run distributed training
python -m torch.distributed.launch \
    --nproc_per_node=4 \
    --master_port=12355 \
    train_distributed.py \
    --config config.yaml \
    --output-dir /lustreFS/data/mygroup/results
```

### Parameter Sweep with Job Arrays

```bash
#!/bin/bash
#SBATCH -J param_sweep
#SBATCH --partition=defq
#SBATCH --qos=normal
#SBATCH --array=1-20%5      # 20 jobs, max 5 running
#SBATCH --cpus-per-task=2
#SBATCH --gres=gpu:1
#SBATCH --mem=16000
#SBATCH --time=08:00:00
#SBATCH -o logs/sweep_%A_%a.out
#SBATCH -e logs/sweep_%A_%a.err

# Parameter arrays
learning_rates=(0.001 0.01 0.1 0.2)
batch_sizes=(16 32 64 128 256)

# Calculate parameters for this task
lr_idx=$(( ($SLURM_ARRAY_TASK_ID - 1) / ${#batch_sizes[@]} ))
bs_idx=$(( ($SLURM_ARRAY_TASK_ID - 1) % ${#batch_sizes[@]} ))

lr=${learning_rates[$lr_idx]}
bs=${batch_sizes[$bs_idx]}

# Load environment
module load Python/3.9.5
conda activate myenv

# Run experiment
python train.py \
    --learning-rate $lr \
    --batch-size $bs \
    --experiment-name "sweep_${SLURM_ARRAY_TASK_ID}" \
    --output-dir /lustreFS/data/mygroup/sweep_results
```

### A6000 High-Memory Job

```bash
#!/bin/bash
#SBATCH -J large_model_training
#SBATCH --partition=a6000
#SBATCH --qos=long-a6000
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --gres=gpu:3        # Use 3 of 4 A6000 GPUs
#SBATCH --mem=256000        # 256GB RAM
#SBATCH --time=5-00:00:00   # 5 days
#SBATCH -o logs/large_model_%j.out
#SBATCH -e logs/large_model_%j.err

# Load modules
module load CUDA/11.3.1

# Activate environment with large model libraries
conda activate large-models

# Set memory optimization
export CUDA_VISIBLE_DEVICES=0,1,2
export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:512

# Train large language model
python train_llm.py \
    --model-size 70B \
    --gradient-checkpointing \
    --fp16 \
    --data-dir /lustreFS/data/mygroup/datasets \
    --output-dir /lustreFS/data/mygroup/llm_checkpoints
```

## Job Management Commands

### Monitoring Jobs

```bash
# Check job queue
squeue

# Check your jobs only
squeue -u $USER

# Detailed job information
scontrol show job <job_id>

# Job history
sacct -u $USER --starttime=2024-01-01

# Job efficiency statistics
seff <job_id>
```

### Job Control

```bash
# Cancel a job
scancel <job_id>

# Cancel all your jobs
scancel -u $USER

# Cancel jobs by name
scancel --name=training_job

# Hold a job (prevent it from running)
scontrol hold <job_id>

# Release a held job
scontrol release <job_id>
```

### Job Information

```bash
# Show partition information
sinfo

# Show detailed node information
scontrol show nodes

# Show QoS information
sacctmgr show qos

# Show your job priorities
sprio -u $USER
```

## Resource Monitoring

### During Job Execution

```bash
# Monitor GPU usage on your job
srun --jobid=<job_id> nvidia-smi

# Check memory usage
srun --jobid=<job_id> free -h

# Monitor CPU usage
srun --jobid=<job_id> htop
```

### Job Performance Analysis

```bash
# Job efficiency report
seff <job_id>

# Detailed job accounting
sacct -j <job_id> --format=JobID,JobName,Partition,Account,AllocCPUS,State,ExitCode,Start,End,Elapsed,MaxRSS,MaxVMSize
```

## Best Practices

### Resource Allocation

1. **Request appropriate resources**:
   ```bash
   # Don't over-allocate
   #SBATCH --cpus-per-task=4   # Not 32 if you only use 4
   #SBATCH --mem=16000         # Not 500000 if you only need 16GB
   ```

2. **Use job arrays for parameter sweeps**:
   ```bash
   #SBATCH --array=1-100%10    # Limit concurrent jobs
   ```

3. **Choose appropriate partitions**:
   - Use `defq` for most workloads
   - Use `a6000` only when you need >24GB GPU memory

### Data Management

1. **Use appropriate storage**:
   ```bash
   # Large datasets and results
   cd /lustreFS/data/mygroup
   
   # Temporary files during job
   export TMPDIR=/tmp/$SLURM_JOB_ID
   mkdir -p $TMPDIR
   ```

2. **Clean up after jobs**:
   ```bash
   # Add to end of script
   rm -rf /tmp/$SLURM_JOB_ID
   ```

### Debugging

1. **Test interactively first**:
   ```bash
   srun -p defq --qos=normal --gres=gpu:1 --time=1:00:00 --pty /bin/bash
   ```

2. **Use smaller datasets for debugging**:
   ```bash
   python train.py --debug --max-samples 1000
   ```

3. **Check logs regularly**:
   ```bash
   tail -f logs/training_12345.out
   ```

## Troubleshooting

### Common Issues

**Job pending forever**:
```bash
# Check why job is pending
squeue -u $USER -t PENDING
scontrol show job <job_id> | grep Reason
```

**Out of memory errors**:
```bash
# Reduce batch size or request more memory
#SBATCH --mem=64000  # Increase memory
```

**CUDA out of memory**:
```bash
# In your script, add:
export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:128
```

**Job killed by time limit**:
```bash
# Request more time or use checkpointing
#SBATCH --time=3-00:00:00
```

**Cannot access files**:
```bash
# Check file permissions and paths
ls -la /lustreFS/data/mygroup/
```

## Next Steps

- **Learn about partitions and queues**: [Partitions & Queues](../partitions/)
- **Understand storage systems**: [Storage Guide](../storage/)
- **Set up development environment**: [Environment Modules](../modules/)