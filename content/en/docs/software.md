---
title: "Software Installation"
linkTitle: "Software"
weight: 45
description: >
  Installing and managing software packages on the Prometheus cluster
tags: ["software", "installation", "conda", "pip", "pytorch", "tensorflow", "python", "deep-learning"]
categories: ["software", "environment-setup"]
---

## Overview

The Prometheus cluster provides several methods for installing and managing software packages. This guide covers both system-wide modules and user-specific installations.

## Installation Methods

### 1. Environment Modules (Recommended)
Use pre-installed software via the module system when available:
```bash
module avail python
module load Python/3.9.5
```

### 2. Conda/Mamba Package Manager
Install packages in isolated environments:
```bash
conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia
```

### 3. pip Package Manager
Install Python packages via pip:
```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu117
```

### 4. Source Installation
Compile software from source when needed:
```bash
git clone https://github.com/project/repo.git
cd repo && python setup.py install
```

## Setting Up Python Environments

### Conda Installation

If conda is not available, install Miniconda:

```bash
# Download Miniconda
cd /lustreFS/data/mygroup/$USER
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# Install Miniconda
bash Miniconda3-latest-Linux-x86_64.sh -b -p ~/miniconda3

# Initialize conda
~/miniconda3/bin/conda init bash
source ~/.bashrc
```

### Create Virtual Environments

```bash
# Create a new environment
conda create -n pytorch-env python=3.9

# Activate environment
conda activate pytorch-env

# Install packages
conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia
conda install jupyter matplotlib pandas scikit-learn
```

### Environment Management

```bash
# List environments
conda env list

# Export environment
conda env export > environment.yml

# Create from file
conda env create -f environment.yml

# Remove environment
conda env remove -n old-env
```

## Deep Learning Frameworks

### PyTorch Installation

```bash
# Create PyTorch environment
conda create -n pytorch python=3.9
conda activate pytorch

# Install PyTorch with CUDA support
conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia

# Verify installation
python -c "import torch; print(f'PyTorch {torch.__version__}, CUDA: {torch.cuda.is_available()}')"
```

### TensorFlow Installation

```bash
# Create TensorFlow environment
conda create -n tensorflow python=3.9
conda activate tensorflow

# Install TensorFlow
pip install tensorflow[and-cuda]

# Verify GPU support
python -c "import tensorflow as tf; print(f'TensorFlow {tf.__version__}, GPUs: {len(tf.config.list_physical_devices("GPU"))}')"
```

### JAX Installation

```bash
# Create JAX environment
conda create -n jax python=3.9
conda activate jax

# Install JAX with CUDA support
pip install --upgrade "jax[cuda11_pip]" -f https://storage.googleapis.com/jax-releases/jax_cuda_releases.html

# Verify installation
python -c "import jax; print(f'JAX devices: {jax.devices()}')"
```

## Specialized Libraries

### MinkowskiEngine

MinkowskiEngine is an auto-differentiation library for sparse tensors, particularly useful for 3D computer vision tasks.

#### Installation Steps

1. **Create dedicated environment**:
   ```bash
   conda create -n py3-mink python=3.8
   conda activate py3-mink
   ```

2. **Install dependencies**:
   ```bash
   conda install openblas-devel -c anaconda
   conda install pytorch==1.10.1 torchvision==0.11.2 torchaudio==0.10.1 cudatoolkit=11.3 -c pytorch -c conda-forge
   ```

3. **Load required modules**:
   ```bash
   module load CUDA/11.3.1 gnu9
   ```

4. **Submit interactive job for compilation**:
   ```bash
   srun -n 1 -c 4 --gres=gpu:1 --mem=20000 --pty /bin/bash
   ```

5. **Install MinkowskiEngine**:
   ```bash
   conda activate py3-mink
   pip install -U git+https://github.com/NVIDIA/MinkowskiEngine -v --no-deps \
       --install-option="--blas_include_dirs=${CONDA_PREFIX}/include" \
       --install-option="--blas=openblas"
   ```

#### Usage Example

```python
import torch
import MinkowskiEngine as ME

# Create sparse tensor
coords = torch.IntTensor([[0, 1], [0, 1], [0, 2], [1, 0], [1, 2]])
feats = torch.FloatTensor([[1], [2], [3], [4], [5]])

# Create sparse tensor
sparse_tensor = ME.SparseTensor(feats, coords)
print(f"Sparse tensor shape: {sparse_tensor.shape}")
```

### PointGPT

PointGPT extends GPT concepts to point clouds for 3D understanding tasks.

#### Installation Steps

1. **Create environment**:
   ```bash
   conda create -n pointgpt python=3.8
   conda activate pointgpt
   ```

2. **Install PyTorch and dependencies**:
   ```bash
   conda install pytorch==1.10.1 torchvision==0.11.2 torchaudio==0.10.1 cudatoolkit=11.3 tensorboard -c pytorch -c conda-forge
   pip install easydict h5py matplotlib open3d opencv-python pyyaml timm tqdm transforms3d termcolor scipy ninja plyfile numpy==1.23.4
   pip install setuptools==59.5.0
   ```

3. **Load CUDA module**:
   ```bash
   module load CUDA/11.3.1
   ```

4. **Clone PointGPT repository**:
   ```bash
   cd /lustreFS/data/mygroup/$USER
   git clone https://github.com/CGuangyan-BIT/PointGPT.git
   cd PointGPT
   ```

5. **Submit interactive job for compilation**:
   ```bash
   srun -n 1 -c 4 --gres=gpu:1 --mem=20000 --pty /bin/bash
   ```

6. **Install extensions**:
   ```bash
   conda activate pointgpt
   
   # Chamfer Distance & EMD
   cd ./extensions/chamfer_dist
   python setup.py install --user
   cd ../emd
   python setup.py install --user
   cd ../
   
   # PointNet++
   pip install "git+https://github.com/erikwijmans/Pointnet2_PyTorch.git#egg=pointnet2_ops&subdirectory=pointnet2_ops_lib"
   
   # GPU kNN
   pip install --upgrade https://github.com/unlimblue/KNN_CUDA/releases/download/0.2/KNN_CUDA-0.2-py3-none-any.whl
   ```

## Computer Vision Libraries

### OpenCV Installation

```bash
conda activate myenv
conda install opencv -c conda-forge

# Or install from pip
pip install opencv-python opencv-contrib-python
```

### Open3D for 3D Processing

```bash
conda activate myenv
pip install open3d

# Test installation
python -c "import open3d as o3d; print(f'Open3D {o3d.__version__}')"
```

### PIL/Pillow for Image Processing

```bash
conda install pillow
# or
pip install Pillow
```

## Scientific Computing

### NumPy, SciPy, Pandas

```bash
conda install numpy scipy pandas matplotlib seaborn
# or
pip install numpy scipy pandas matplotlib seaborn
```

### Jupyter and IPython

```bash
conda install jupyter ipython ipykernel
# or
pip install jupyter ipython ipykernel

# Add environment to Jupyter
python -m ipykernel install --user --name myenv --display-name "Python (myenv)"
```

### Scikit-learn

```bash
conda install scikit-learn
# or
pip install scikit-learn
```

## Development Tools

### Git and Version Control

```bash
# Git is usually available by default
git --version

# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### Build Tools

```bash
# Install build essentials
conda install cmake make ninja

# For C++ development
conda install gxx_linux-64 gcc_linux-64
```

### Debugging Tools

```bash
# Install debugging tools
pip install pdb++ ipdb

# Memory profiling
pip install memory_profiler

# Line profiling
pip install line_profiler
```

## Installation in SLURM Jobs

### Interactive Installation

```bash
# Submit interactive job for installation
srun --partition=defq --qos=normal --gres=gpu:1 --mem=16000 --time=2:00:00 --pty /bin/bash

# Load modules
module load CUDA/11.3.1 Python/3.9.5

# Activate environment
conda activate myenv

# Install packages
pip install package-name
```

### Batch Installation Script

```bash
#!/bin/bash
#SBATCH -J install_packages
#SBATCH --partition=defq
#SBATCH --qos=normal
#SBATCH --cpus-per-task=4
#SBATCH --mem=8000
#SBATCH --time=1:00:00

# Load modules
module load Python/3.9.5

# Activate environment
conda activate myenv

# Install packages
pip install -r requirements.txt

echo "Installation completed"
```

## Package Management Best Practices

### Requirements Files

Create `requirements.txt` for reproducibility:

```txt
torch==1.12.1+cu117
torchvision==0.13.1+cu117
torchaudio==0.12.1+cu117
numpy==1.23.4
pandas==1.5.2
matplotlib==3.6.2
jupyter==1.0.0
```

Install from requirements:
```bash
pip install -r requirements.txt
```

### Environment Files

Create `environment.yml` for conda:

```yaml
name: myproject
channels:
  - pytorch
  - nvidia
  - conda-forge
dependencies:
  - python=3.9
  - pytorch=1.12.1
  - torchvision=0.13.1
  - torchaudio=0.12.1
  - pytorch-cuda=11.7
  - numpy
  - pandas
  - matplotlib
  - jupyter
  - pip
  - pip:
    - some-pip-package
```

Create environment:
```bash
conda env create -f environment.yml
```

### Storage Considerations

Install packages in shared group storage to avoid quota issues:

```bash
# Set conda environments path
echo "envs_dirs:
  - /lustreFS/data/mygroup/conda/envs" > ~/.condarc

# Set pip cache directory
export PIP_CACHE_DIR=/lustreFS/data/mygroup/pip-cache
echo 'export PIP_CACHE_DIR=/lustreFS/data/mygroup/pip-cache' >> ~/.bashrc
```

## Troubleshooting

### Common Installation Issues

**CUDA compatibility errors**:
```bash
# Check CUDA version
nvidia-smi

# Install matching PyTorch version
conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia
```

**Memory errors during installation**:
```bash
# Request more memory for installation
srun --mem=32000 --pty /bin/bash

# Or increase pip timeout
pip install --timeout 1000 package-name
```

**Permission errors**:
```bash
# Install in user space
pip install --user package-name

# Or check conda environment ownership
ls -la ~/miniconda3/envs/
```

**Network timeouts**:
```bash
# Use conda-forge channel
conda install -c conda-forge package-name

# Or use pip with retries
pip install --retries 10 package-name
```

### Compilation Issues

**Missing compilers**:
```bash
# Load compiler modules
module load GCC/10.3.0

# Check compiler availability
gcc --version
nvcc --version
```

**Missing headers**:
```bash
# Install development packages
conda install gxx_linux-64 gcc_linux-64

# For CUDA development
module load CUDA/11.3.1
echo $CUDA_HOME
```

### Environment Conflicts

**Package conflicts**:
```bash
# Create fresh environment
conda create -n clean-env python=3.9
conda activate clean-env

# Install packages one by one
conda install pytorch -c pytorch
```

**Module vs conda conflicts**:
```bash
# Always load modules before activating conda
module load Python/3.9.5
conda activate myenv
```

## Package Documentation

Keep track of installed packages:

```bash
# List conda packages
conda list > conda_packages.txt

# List pip packages
pip freeze > pip_requirements.txt

# Environment information
conda info --envs > environments.txt
```

## Next Steps

- **Learn job submission**: [Job Submission](../job-submission/)
- **Explore GPU programming**: [GPU Computing](../gpu-computing/)
- **Set up monitoring**: [Performance Monitoring](../monitoring/)