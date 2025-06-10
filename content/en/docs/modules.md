---
title: "Environment Modules"
linkTitle: "Modules"
weight: 35
description: >
  Using Lmod environment modules to manage software on Prometheus
tags: ["modules", "lmod", "environment", "software", "setup", "cuda", "python"]
categories: ["environment-setup", "software"]
---

## Overview

The Prometheus cluster uses **Lmod** (Lua-based Environment Modules) to manage software packages and their dependencies. This system allows you to easily load and unload different software versions without conflicts.

## Module System Basics

Environment modules modify your shell environment to provide access to specific software packages. When you load a module, it typically:

- Adds software to your `PATH`
- Sets environment variables
- Loads required dependencies
- Configures library paths

## Basic Module Commands

### List Available Modules

```bash
# Show all available modules
module available
module avail
module av
ml av

# Search for specific modules
module avail gcc
module avail python
module avail cuda
```

### Load Modules

```bash
# Load a specific module
module load GCC/10.3.0
module load CUDA/11.3.1
module load Python/3.9.5

# Short form using 'ml'
ml GCC/10.3.0 CUDA/11.3.1 Python/3.9.5
```

### Check Loaded Modules

```bash
# List currently loaded modules
module list
ml list
```

### Unload Modules

```bash
# Unload a specific module
module unload GCC/10.3.0

# Unload all modules
module purge
ml purge
```

## Common Software Modules

### Compilers

```bash
# GNU Compiler Collection
module load GCC/10.3.0
module load GCC/11.2.0

# Intel Compilers (if available)
module load intel/2021.4.0
```

### CUDA and GPU Development

```bash
# CUDA Toolkit
module load CUDA/11.3.1
module load CUDA/11.7.0
module load CUDA/12.0.0

# Check CUDA after loading
nvcc --version
nvidia-smi
```

### Python Environments

```bash
# Python interpreter
module load Python/3.9.5
module load Python/3.10.8

# Python with scientific libraries
module load Python/3.9.5-GCCcore-10.3.0
```

### Deep Learning Frameworks

```bash
# PyTorch (if pre-installed as module)
module load PyTorch/1.12.1-foss-2022a-CUDA-11.7.0

# TensorFlow (if pre-installed as module)
module load TensorFlow/2.11.0-foss-2022a-CUDA-11.7.0
```

### Development Tools

```bash
# Git (usually available by default)
module load git/2.36.0

# CMake
module load CMake/3.24.3

# HDF5 for data storage
module load HDF5/1.12.2
```

## Module Dependencies

Lmod automatically handles dependencies. When you load a module, it loads required dependencies:

```bash
# Loading Python might automatically load GCC
module load Python/3.9.5

# Check what was loaded
module list
# Might show:
# GCCcore/10.3.0
# Python/3.9.5-GCCcore-10.3.0
```

## Setting Up Your Environment

### Create a Module Loading Script

Create `~/load_modules.sh` for consistent environment setup:

```bash
#!/bin/bash
# ~/load_modules.sh - Load standard development environment

# Clear any existing modules
module purge

# Load core development tools
module load GCC/10.3.0
module load CUDA/11.3.1
module load Python/3.9.5

# Optional: Load additional tools
# module load git/2.36.0
# module load CMake/3.24.3

echo "Development environment loaded:"
module list
```

Make it executable and use it:

```bash
chmod +x ~/load_modules.sh
source ~/load_modules.sh
```

### Add to Your Shell Configuration

Add common modules to your `~/.bashrc`:

```bash
# Add to ~/.bashrc
# Load standard modules at login
if [ -f ~/load_modules.sh ]; then
    source ~/load_modules.sh
fi
```

## SLURM Job Scripts with Modules

### Basic Job with Modules

```bash
#!/bin/bash
#SBATCH -J module_job
#SBATCH --partition=defq
#SBATCH --qos=normal
#SBATCH --gres=gpu:1
#SBATCH --time=2:00:00

# Load required modules
module purge
module load CUDA/11.3.1
module load Python/3.9.5

# Verify modules are loaded
echo "Loaded modules:"
module list

# Check CUDA availability
echo "CUDA version:"
nvcc --version

# Activate your conda environment
source ~/anaconda3/bin/activate
conda activate myenv

# Run your script
python train.py
```

### Multiple GPU Job with Modules

```bash
#!/bin/bash
#SBATCH -J multi_gpu_training
#SBATCH --partition=defq
#SBATCH --qos=long
#SBATCH --gres=gpu:4
#SBATCH --time=1-00:00:00

# Load modules for CUDA development
module load GCC/10.3.0
module load CUDA/11.3.1
module load Python/3.9.5

# Load MPI for distributed computing (if available)
# module load OpenMPI/4.1.4-GCC-10.3.0

# Set CUDA environment
export CUDA_VISIBLE_DEVICES=0,1,2,3

# Activate environment
conda activate pytorch-gpu

# Run distributed training
python -m torch.distributed.launch \
    --nproc_per_node=4 \
    train_distributed.py
```

## Python Package Management

### Using Conda with Modules

```bash
# Load Python module
module load Python/3.9.5

# Install conda (if not already available)
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -b -p ~/miniconda3

# Add conda to path
echo 'export PATH="$HOME/miniconda3/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Create environment
conda create -n myenv python=3.9
conda activate myenv

# Install packages
conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia
```

### Using pip with Modules

```bash
# Load Python module
module load Python/3.9.5

# Create virtual environment
python -m venv ~/venvs/myproject
source ~/venvs/myproject/bin/activate

# Upgrade pip
pip install --upgrade pip

# Install packages
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu117
pip install jupyter numpy pandas matplotlib
```

## Module Collections

Save frequently used module combinations:

```bash
# Save current modules as a collection
module save my_collection

# List saved collections
module savelist

# Restore a collection
module restore my_collection
```

## Custom Module Paths

If your group has custom modules:

```bash
# Add custom module path
module use /lustreFS/data/mygroup/modules

# Check module paths
module show MODULEPATH
```

## Troubleshooting Modules

### Common Issues

**Module not found**:
```bash
# Check available modules
module avail | grep -i package_name

# Check if you have access to the module path
ls -la /opt/modules/
```

**Conflicting modules**:
```bash
# Clear all modules and start fresh
module purge
module load GCC/10.3.0 CUDA/11.3.1
```

**CUDA not found after loading**:
```bash
# Verify CUDA module is loaded
module list | grep -i cuda

# Check CUDA environment
echo $CUDA_HOME
echo $CUDA_PATH
which nvcc
```

**Python packages not found**:
```bash
# Ensure Python module is loaded before using pip/conda
module load Python/3.9.5
which python
python --version
```

### Module Information

```bash
# Show detailed module information
module show CUDA/11.3.1
module help CUDA/11.3.1

# See what a module does before loading
module display CUDA/11.3.1
```

## Best Practices

### For Interactive Development

1. **Create a standard environment script**
2. **Use module collections** for frequently used combinations
3. **Load modules before activating conda/venv**

### For SLURM Jobs

1. **Always start with `module purge`**
2. **Load modules explicitly in job scripts**
3. **Verify modules are loaded** with `module list`
4. **Document module requirements** in your scripts

### For Reproducibility

1. **Pin module versions** in scripts:
   ```bash
   module load CUDA/11.3.1  # Not just 'CUDA'
   ```

2. **Document module requirements**:
   ```bash
   # Required modules:
   # - GCC/10.3.0
   # - CUDA/11.3.1
   # - Python/3.9.5
   ```

3. **Use environment files** for complex setups

## Example Workflows

### Deep Learning Setup

```bash
# Standard deep learning environment
module purge
module load GCC/10.3.0
module load CUDA/11.3.1
module load Python/3.9.5

# Activate conda environment
conda activate pytorch-env

# Verify setup
python -c "import torch; print(f'PyTorch: {torch.__version__}, CUDA: {torch.cuda.is_available()}')"
```

### Development Environment

```bash
# Development tools
module purge
module load GCC/10.3.0
module load CUDA/11.3.1
module load Python/3.9.5
module load git/2.36.0
module load CMake/3.24.3

# Save as collection
module save development

# Later, restore quickly
module restore development
```

### Compilation Environment

```bash
# For compiling CUDA code
module purge
module load GCC/10.3.0
module load CUDA/11.3.1

# Compile CUDA program
nvcc -o program program.cu

# For C++ with GPU support
g++ -I$CUDA_HOME/include -L$CUDA_HOME/lib64 -lcudart program.cpp -o program
```

## Next Steps

- **Configure VS Code for remote development**: [VS Code Setup](../vscode/)
- **Learn about software installation**: [Software Installation](../software/)
- **Submit your first job**: [Job Submission](../job-submission/)