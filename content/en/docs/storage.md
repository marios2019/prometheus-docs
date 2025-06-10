---
title: "Storage Systems"
linkTitle: "Storage"
weight: 30
description: >
  File systems, quotas, and data management on the Prometheus cluster
tags: ["storage", "lustre", "file-systems", "quotas", "data-management", "backup"]
categories: ["storage", "cluster-computing"]
---

## Storage Overview

The Prometheus cluster provides multiple storage systems optimized for different use cases, from personal files to high-performance parallel computing workloads.

## Storage Architecture

### Home Directories (`/trinity/home/`)
- **Type**: SSD-backed storage
- **Mount point**: `/trinity/home/<username>`
- **Quota**: 20GB per user
- **Purpose**: Personal configuration files, small scripts
- **Backup**: Regular backups maintained
- **Performance**: Fast random I/O, moderate capacity

### Shared Group Storage (`/lustreFS/data/`)
- **Type**: Lustre parallel file system
- **Mount point**: `/lustreFS/data/<group-name>`
- **Quota**: 30TB per group (or 20,971,520 files)
- **Purpose**: Primary workspace for research data and results
- **Performance**: High-throughput parallel I/O
- **Shared**: All group members have access

### Local Node Storage
- **Type**: NVMe SSD (1TB per compute node)
- **Purpose**: Temporary files during job execution
- **Access**: Only available during allocated jobs
- **Performance**: Highest IOPS for temporary data

## File System Details

### Home Directory Usage

{{% alert title="Quota Limit" color="warning" %}}
Home directories have a strict **20GB limit**. Use them only for configuration files, not data or models.
{{% /alert %}}

```bash
# Check your home directory quota
quota -us

# View home directory contents
ls -la /trinity/home/$USER

# Typical home directory structure
/trinity/home/username/
├── .bashrc                 # Shell configuration
├── .ssh/                   # SSH keys and config
├── .jupyter/               # Jupyter configuration
├── .conda/                 # Conda configuration
├── scripts/                # Small utility scripts
└── .local/                 # Local Python packages
```

**Best practices for home directories**:
- Store only configuration files and small scripts
- Link to shared storage for data access
- Use symbolic links to avoid quota issues

### Shared Group Storage

The `/lustreFS/data/` directory provides high-performance storage for your research work:

```bash
# Access your group's shared storage
cd /lustreFS/data/<group-name>

# Check group quota
lfs quota -gh <group-name> /lustreFS/

# Example group directory structure
/lustreFS/data/mygroup/
├── datasets/               # Shared datasets
├── models/                 # Pre-trained and trained models
├── experiments/            # Individual user experiments
│   ├── user1/
│   ├── user2/
│   └── shared/
├── code/                   # Shared code repositories
├── results/                # Experiment results
└── tmp/                    # Temporary files
```

**Quota information**:
- **Space limit**: 30TB per group
- **File limit**: 20,971,520 files per group
- **Shared**: All group members can read/write

### Local Node Storage

Each compute node has local NVMe storage for temporary files:

```bash
# During a SLURM job, use local storage for temporary files
export TMPDIR=/tmp/$SLURM_JOB_ID
mkdir -p $TMPDIR

# Example usage in job script
#SBATCH --job-name=training
#SBATCH --gres=gpu:1

# Create temporary directory
export TMPDIR=/tmp/$SLURM_JOB_ID
mkdir -p $TMPDIR

# Copy data to local storage for faster I/O
cp /lustreFS/data/mygroup/dataset.tar.gz $TMPDIR/
cd $TMPDIR
tar -xzf dataset.tar.gz

# Run training with local data
python train.py --data-dir $TMPDIR/dataset

# Copy results back to shared storage
cp -r $TMPDIR/results /lustreFS/data/mygroup/experiments/
```

## Quota Management

### Checking Quotas

```bash
# Check your home directory quota
quota -us

# Check group quota on Lustre filesystem
lfs quota -gh <group-name> /lustreFS/

# Example quota output:
# Disk quotas for group mygroup (gid 1001):
#      Filesystem    used   quota   limit   grace   files   quota   limit   grace
#        /lustreFS   15.2T     30T     30T       -  1234567  20971520 20971520   -
```

### Understanding Quota Output

- **used**: Current usage
- **quota**: Soft limit (warning threshold)
- **limit**: Hard limit (cannot exceed)
- **grace**: Time allowed to exceed soft quota
- **files**: Number of files/inodes used

### Managing Quota Issues

When approaching quota limits:

1. **Clean up temporary files**:
   ```bash
   # Find large files
   find /lustreFS/data/mygroup -type f -size +1G -ls
   
   # Find old temporary files
   find /lustreFS/data/mygroup -name "*.tmp" -mtime +7 -delete
   ```

2. **Archive old data**:
   ```bash
   # Compress old experiments
   tar -czf old_experiments.tar.gz experiments/2023/
   rm -rf experiments/2023/
   ```

3. **Use efficient storage**:
   ```bash
   # Use compressed formats for datasets
   # Store checkpoints selectively
   # Remove duplicate files
   ```

## Data Management Best Practices

### Directory Organization

Organize your group's shared storage efficiently:

```bash
# Recommended structure
/lustreFS/data/mygroup/
├── datasets/
│   ├── imagenet/           # Large shared datasets
│   ├── coco/
│   └── custom/
├── models/
│   ├── pretrained/         # Downloaded pre-trained models
│   └── checkpoints/        # Training checkpoints
├── experiments/
│   ├── user1/
│   │   ├── project_a/
│   │   └── project_b/
│   └── user2/
├── code/
│   ├── shared_utils/       # Shared code libraries
│   └── experiments/        # Experiment code
└── results/
    ├── papers/             # Results for publications
    └── ongoing/            # Current experiment results
```

### File Permissions

Set appropriate permissions for shared access:

```bash
# Make directories group-writable
chmod g+w /lustreFS/data/mygroup/datasets/

# Set default permissions for new files
umask 002

# Change group ownership if needed
chgrp -R mygroup /lustreFS/data/mygroup/shared/
```

### Data Transfer

#### Small Files (< 1GB)
```bash
# Copy from local machine using scp
scp large_dataset.tar.gz prometheus:/lustreFS/data/mygroup/datasets/

# Copy between directories on cluster
cp -r /lustreFS/data/mygroup/datasets/source /lustreFS/data/mygroup/experiments/
```

#### Large Files (> 1GB)
```bash
# Use rsync for large transfers with progress
rsync -avP large_dataset/ prometheus:/lustreFS/data/mygroup/datasets/

# Parallel compression for large datasets
tar -cf - dataset/ | pigz > dataset.tar.gz
```

#### Download Datasets
```bash
# Download directly to shared storage
cd /lustreFS/data/mygroup/datasets/
wget https://example.com/large_dataset.tar.gz

# Use aria2 for faster parallel downloads
aria2c -x 8 -s 8 https://example.com/dataset.tar.gz
```

### Backup Strategies

While the cluster provides reliable storage, implement your own backup strategy:

1. **Important results**: Copy to external storage
2. **Code**: Use git repositories
3. **Large datasets**: Document download sources for re-acquisition
4. **Models**: Keep important checkpoints on external storage

## Performance Optimization

### Lustre File System Tips

1. **Use parallel I/O** for large files:
   ```python
   # PyTorch DataLoader with multiple workers
   dataloader = DataLoader(dataset, batch_size=64, num_workers=8)
   ```

2. **Avoid small random writes**:
   ```bash
   # Bad: Many small writes
   for i in {1..1000}; do echo $i >> file.txt; done
   
   # Good: Batch writes
   seq 1 1000 > file.txt
   ```

3. **Use appropriate stripe settings** for large files:
   ```bash
   # Set stripe count for large files (> 1GB)
   lfs setstripe -c 4 /lustreFS/data/mygroup/large_dataset/
   ```

### Local Storage Performance

1. **Copy frequently accessed data** to local storage during jobs
2. **Use local storage for temporary files** and intermediate results
3. **Copy final results back** to shared storage

```bash
# Example job with local storage optimization
#!/bin/bash
#SBATCH --gres=gpu:1
#SBATCH --time=4:00:00

# Set up local temporary directory
export TMPDIR=/tmp/$SLURM_JOB_ID
mkdir -p $TMPDIR

# Copy dataset to local storage
echo "Copying dataset to local storage..."
cp /lustreFS/data/mygroup/dataset.tar.gz $TMPDIR/
cd $TMPDIR
tar -xzf dataset.tar.gz

# Run training with local data (much faster I/O)
python train.py --data-dir $TMPDIR/dataset --output-dir $TMPDIR/results

# Copy results back to shared storage
echo "Copying results back..."
cp -r $TMPDIR/results /lustreFS/data/mygroup/experiments/
```

## Environment Variables

Set up useful environment variables for data management:

```bash
# Add to your ~/.bashrc
export GROUP_DATA="/lustreFS/data/mygroup"
export DATASETS="$GROUP_DATA/datasets"
export MODELS="$GROUP_DATA/models"
export EXPERIMENTS="$GROUP_DATA/experiments/$USER"
export RESULTS="$GROUP_DATA/results"

# Create your experiment directory
mkdir -p $EXPERIMENTS
```

## Common Storage Issues

### Quota Exceeded
```bash
# Error: "Disk quota exceeded"
# Solution: Check and clean up usage
lfs quota -gh mygroup /lustreFS/
find $GROUP_DATA -type f -size +1G -ls
```

### Permission Denied
```bash
# Error: "Permission denied"
# Solution: Check file permissions and group membership
ls -la /lustreFS/data/mygroup/
groups  # Check your group membership
```

### Slow I/O Performance
```bash
# Solutions:
# 1. Use local storage for temporary files
# 2. Reduce number of small files
# 3. Use parallel I/O libraries
# 4. Check stripe settings for large files
lfs getstripe /lustreFS/data/mygroup/large_file
```

### File System Full
```bash
# Check available space
df -h /lustreFS

# If file system is full, clean up:
# 1. Remove temporary files
# 2. Compress old data
# 3. Archive completed experiments
```

## Next Steps

- **Set up your development environment**: [Environment Modules](../modules/)
- **Configure VS Code for remote development**: [VS Code Setup](../vscode/)
- **Learn about software installation**: [Software Installation](../software/)