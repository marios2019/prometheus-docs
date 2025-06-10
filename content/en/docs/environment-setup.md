---
title: "Environment Setup"
linkTitle: "Environment Setup"
weight: 20
description: >
  Configure your development environment on the Prometheus cluster
tags: ["environment-setup", "containers", "singularity", "conda", "python", "jupyter", "gpu", "development"]
categories: ["environment-setup", "getting-started"]
---

## Development Environment Options

The Prometheus cluster supports multiple development environments:

1. **Container-based** (Recommended)
2. **Module-based** (Traditional HPC)
3. **Custom Python environments**

## Container-Based Setup

### Using Pre-built Containers

The cluster provides optimized containers for common deep learning frameworks:

```bash
# List available containers
ls /shared/containers/

# Use PyTorch container
singularity shell --nv /shared/containers/pytorch-gpu.sif

# Use TensorFlow container
singularity shell --nv /shared/containers/tensorflow-gpu.sif
```

### Building Custom Containers

Create a definition file (`pytorch-custom.def`):

```singularity
Bootstrap: docker
From: pytorch/pytorch:2.0.1-cuda11.7-cudnn8-devel

%post
    apt-get update && apt-get install -y \
        git \
        vim \
        htop \
        tmux
    
    pip install \
        transformers \
        datasets \
        wandb \
        jupyter \
        matplotlib \
        seaborn

%environment
    export CUDA_VISIBLE_DEVICES=0,1,2,3
    export PYTHONPATH=/opt/code:$PYTHONPATH

%runscript
    exec "$@"
```

Build the container:

```bash
singularity build pytorch-custom.sif pytorch-custom.def
```

## Python Environment Setup

### Using Conda

```bash
# Load conda module
module load conda

# Create environment
conda create -n myenv python=3.9

# Activate environment
conda activate myenv

# Install packages
conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia
conda install -c conda-forge jupyter matplotlib pandas
```

### Using pip with virtual environments

```bash
# Load Python module
module load python/3.9

# Create virtual environment
python -m venv ~/venvs/deeplearning
source ~/venvs/deeplearning/bin/activate

# Install packages
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
pip install jupyter notebook jupyterlab
pip install transformers datasets wandb
```

## GPU Environment Configuration

### Checking GPU Availability

```bash
# Check available GPUs
nvidia-smi

# Check CUDA version
nvcc --version

# Test PyTorch GPU access
python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}')"
```

### Setting GPU Visibility

```bash
# Use specific GPUs
export CUDA_VISIBLE_DEVICES=0,1

# Use all available GPUs
export CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7
```

## Jupyter Notebook Setup

### Local Jupyter on Compute Node

1. **Request an interactive session**:
   ```bash
   srun --partition=gpu --gres=gpu:1 --time=4:00:00 --pty bash
   ```

2. **Start Jupyter**:
   ```bash
   module load python/3.9
   source ~/venvs/deeplearning/bin/activate
   jupyter notebook --no-browser --port=8888 --ip=0.0.0.0
   ```

3. **Set up SSH tunnel** (from your local machine):
   ```bash
   ssh -L 8888:compute-node:8888 username@prometheus-cluster.example.com
   ```

### JupyterHub Access

If available, access JupyterHub directly:
```
https://jupyter.prometheus-cluster.example.com
```

## Development Tools

### VS Code Remote Development

1. **Install VS Code** with Remote-SSH extension
2. **Configure SSH connection** in VS Code
3. **Connect to cluster** and open your project folder

### tmux for Session Management

```bash
# Start new session
tmux new-session -s training

# Detach session (Ctrl+b, then d)
# Reattach session
tmux attach-session -t training

# List sessions
tmux list-sessions
```

## Storage and Data Access

### Home Directory Setup

```bash
# Create project structure
mkdir -p ~/projects/{experiments,datasets,models,scripts}
mkdir -p ~/logs
```

### Using Shared Storage

```bash
# Link shared datasets
ln -s /shared/datasets ~/datasets

# Copy models to your space
cp -r /shared/models/pretrained ~/models/

# Use scratch space for temporary files
export TMPDIR=/scratch/$USER
mkdir -p $TMPDIR
```

## Environment Variables

Create `~/.cluster_env`:

```bash
# CUDA settings
export CUDA_VISIBLE_DEVICES=0,1,2,3
export CUDA_CACHE_PATH=/scratch/$USER/cuda_cache

# Python settings
export PYTHONPATH=$HOME/projects:$PYTHONPATH
export JUPYTER_CONFIG_DIR=$HOME/.jupyter

# Weights & Biases
export WANDB_DIR=$HOME/logs/wandb
export WANDB_CACHE_DIR=/scratch/$USER/wandb_cache

# Hugging Face
export HF_DATASETS_CACHE=/scratch/$USER/hf_cache
export TRANSFORMERS_CACHE=/scratch/$USER/transformers_cache
```

Source it in your `.bashrc`:
```bash
echo 'source ~/.cluster_env' >> ~/.bashrc
```

## Troubleshooting

### Common Issues

**CUDA out of memory**:
```bash
# Clear GPU memory
nvidia-smi --gpu-reset

# Monitor GPU usage
watch -n 1 nvidia-smi
```

**Module not found**:
```bash
# Check loaded modules
module list

# Reload environment
source ~/.bashrc
```

**Permission denied**:
```bash
# Check file permissions
ls -la

# Fix permissions
chmod 755 script.py
```

## Next Steps

- **Submit your first job**: [Job Submission Guide](../job-submission/)
- **Monitor your work**: [Monitoring Guide](../monitoring/)
- **Manage data**: [Storage Guide](../storage/)