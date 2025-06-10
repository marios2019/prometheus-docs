---
title: "VS Code Remote Development"
linkTitle: "VS Code"
weight: 40
description: >
  Set up Visual Studio Code for remote development on the Prometheus cluster
tags: ["vscode", "remote-development", "ssh", "jupyter", "development", "ide", "setup"]
categories: ["environment-setup", "development-tools"]
---

## Overview

Visual Studio Code provides excellent remote development capabilities for the Prometheus cluster. You can edit code, run Jupyter notebooks, and debug applications directly on the cluster while using your local VS Code interface.

## Prerequisites

- **VS Code Desktop** installed on your local machine
- **Remote-SSH extension** for VS Code
- **SSH access** to Prometheus cluster (see [Getting Started](../getting-started/))
- **Valid cluster account** with SSH keys configured

## Install Required Extensions

Install these essential VS Code extensions:

1. **Remote - SSH** (`ms-vscode-remote.remote-ssh`)
2. **Python** (`ms-python.python`) 
3. **Jupyter** (`ms-toolsai.jupyter`)
4. **Git** integration (built-in)

Optional but recommended:
- **Remote - SSH: Editing Configuration Files** (`ms-vscode-remote.remote-ssh-edit`)
- **GitLens** (`eamodio.gitlens`)
- **Thunder Client** for API testing (`rangav.vscode-thunder-client`)

## SSH Configuration

### Basic SSH Setup

First, ensure your `~/.ssh/config` file is properly configured:

```bash
# ~/.ssh/config
Host prometheus
  Hostname prometheus.cyens.org.cy
  User <your-username>
  IdentityFile ~/.ssh/id_rsa

Host *.cluster
  User <your-username>
  IdentityFile ~/.ssh/prometheus_user_sshd
  ProxyJump prometheus
```

Replace `<your-username>` with your actual cluster username.

## User SSHD Process Setup

To connect VS Code directly to compute nodes, you need to set up a user SSHD process through SLURM.

### Step 1: Generate SSH Keys for User SSHD

Connect to Prometheus and create SSH keys for the user SSHD process:

```bash
ssh prometheus
ssh-keygen -t rsa -f ~/.ssh/prometheus_user_sshd
```

This creates:
- **Private key**: `~/.ssh/prometheus_user_sshd`
- **Public key**: `~/.ssh/prometheus_user_sshd.pub`

### Step 2: Create SSHD Job Script

Create `~/sshd.sh` script for launching the user SSHD process:

```bash
#!/bin/bash
#SBATCH -o res_%j.txt      # output file
#SBATCH -e res_%j.err      # error file
#SBATCH -J sshd            # job name
#SBATCH --partition=defq   # partition
#SBATCH --qos=normal       # priority queue
#SBATCH --ntasks=1         # number of tasks
#SBATCH --cpus-per-task=2  # CPU cores
#SBATCH --gres=gpu:1       # number of GPUs
#SBATCH --mem=1000         # memory in MB
#SBATCH --time=0-04:00     # 4 hours maximum

# Find an available port
PORT=$(python -c 'import socket; s=socket.socket(); s.bind(("", 0)); print(s.getsockname()[1]); s.close()')

echo "********************************************************************"
echo "Starting sshd in Slurm as user"
echo "Environment information:"
echo "Date:" $(date)
echo "Allocated node:" $(hostname)
echo "Path:" $(pwd)
echo "Listening on:" $PORT
echo "********************************************************************"

# Start user SSHD process
/usr/sbin/sshd -D -p ${PORT} -f /dev/null -h ${HOME}/.ssh/prometheus_user_sshd
```

Make the script executable:

```bash
chmod +x ~/sshd.sh
```

### Step 3: Submit SSHD Job

Submit the SSHD job to get a compute node:

```bash
sbatch sshd.sh
```

Check the job status and get the allocated node and port:

```bash
# Check job status
squeue -u $USER

# View the output file to get connection details
cat res_<job_id>.txt
```

The output will show something like:
```
Starting sshd in Slurm as user
Date: Thu Jun 5 10:30:00 UTC 2025
Allocated node: gpu02
Listening on: 45672
```

## Connecting VS Code

### Method 1: Direct Connection

1. **Open VS Code** on your local machine
2. **Press F1** or Ctrl/Cmd+Shift+P to open command palette
3. **Type**: "Remote-SSH: Connect to Host"
4. **Enter**: `ssh -p 45672 gpu02.cluster` (use your actual port and node)

VS Code will automatically update your SSH config file.

### Method 2: Manual SSH Config

Add the connection details to your `~/.ssh/config`:

```bash
Host gpu02.cluster
    HostName gpu02.cluster
    Port 45672
    User <your-username>
    IdentityFile ~/.ssh/prometheus_user_sshd
    ProxyJump prometheus
```

Then connect using "Remote-SSH: Connect to Host" → `gpu02.cluster`

## Development Workflow

### 1. Connect to Compute Node

```bash
# Submit SSHD job
sbatch sshd.sh

# Wait for job to start (check with squeue)
squeue -u $USER

# Get connection details
cat res_<job_id>.txt

# Connect VS Code to the allocated node
```

### 2. Open Your Project

Once connected to the compute node:

1. **File → Open Folder**
2. **Navigate to**: `/lustreFS/data/mygroup/experiments/myproject`
3. **Open the folder**

### 3. Set Up Python Environment

In the VS Code terminal on the remote machine:

```bash
# Load required modules
module load CUDA/11.3.1 Python/3.9.5

# Activate your conda environment
conda activate myenv

# Verify GPU access
python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}')"
```

### 4. Configure Python Interpreter

1. **Press Ctrl/Cmd+Shift+P**
2. **Type**: "Python: Select Interpreter"
3. **Choose**: Your conda environment interpreter
   - Usually: `~/miniconda3/envs/myenv/bin/python`

## Working with Jupyter Notebooks

### Start Jupyter Server

In the VS Code terminal on the remote machine:

```bash
# Load modules and activate environment
module load Python/3.9.5
conda activate myenv

# Start Jupyter (no browser needed)
jupyter notebook --no-browser --port=8888 --ip=0.0.0.0
```

### Connect VS Code to Jupyter

1. **Open a `.ipynb` file** in VS Code
2. **Click "Select Kernel"** in the top-right
3. **Choose "Existing Jupyter Server"**
4. **Enter**: `http://localhost:8888`
5. **Enter the token** from the Jupyter output

## Development Best Practices

### Resource Management

1. **Request appropriate resources** for development:
   ```bash
   #SBATCH --cpus-per-task=4  # Not 32 for development
   #SBATCH --gres=gpu:1       # Usually sufficient for development
   #SBATCH --mem=8000         # 8GB for most development tasks
   #SBATCH --time=0-04:00     # 4 hours for development session
   ```

2. **Use longer sessions for intensive work**:
   ```bash
   #SBATCH --time=0-08:00     # 8 hours for longer development
   #SBATCH --qos=long         # If you need more than 1 day
   ```

### File Organization

Set up a consistent workspace structure:

```bash
# Recommended project structure
/lustreFS/data/mygroup/experiments/myproject/
├── data/                   # Datasets and data files
├── notebooks/              # Jupyter notebooks
├── src/                    # Source code
├── configs/                # Configuration files
├── scripts/                # Training and utility scripts
├── results/                # Experiment results
└── README.md               # Project documentation
```

### Environment Configuration

Create a workspace settings file (`.vscode/settings.json`):

```json
{
    "python.defaultInterpreterPath": "~/miniconda3/envs/myenv/bin/python",
    "python.terminal.activateEnvironment": true,
    "jupyter.jupyterServerType": "local",
    "files.watcherExclude": {
        "**/data/**": true,
        "**/results/**": true,
        "**/.git/**": true
    }
}
```

### Git Integration

Configure Git for your project:

```bash
# Set up Git credentials
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Initialize repository (if new project)
cd /lustreFS/data/mygroup/experiments/myproject
git init
git add .
git commit -m "Initial commit"
```

## Troubleshooting

### Connection Issues

**"Could not establish connection"**:
1. Check if SSHD job is running: `squeue -u $USER`
2. Verify node name and port from job output
3. Ensure SSH keys are properly configured

**"Permission denied"**:
1. Check SSH key permissions: `chmod 600 ~/.ssh/prometheus_user_sshd`
2. Verify ProxyJump configuration in SSH config
3. Test SSH connection manually: `ssh -p <port> <node>.cluster`

### Performance Issues

**Slow file operations**:
1. Exclude large directories from VS Code watcher
2. Use local storage for frequently accessed files
3. Consider using VS Code on the head node for file browsing only

**High memory usage**:
1. Close unused notebooks and files
2. Restart VS Code Python extension if needed
3. Request more memory in SSHD job if necessary

### SSHD Job Management

**Job terminated unexpectedly**:
1. Check job logs: `cat res_<job_id>.err`
2. Resubmit SSHD job: `sbatch sshd.sh`
3. Update VS Code connection with new port/node

**Need longer development time**:
```bash
# Modify sshd.sh for longer sessions
#SBATCH --time=0-08:00     # 8 hours
#SBATCH --qos=long         # For >24 hours
```

## Cleanup and Best Practices

### End Development Session

When finishing your work:

1. **Save all files** in VS Code
2. **Close VS Code connection**
3. **Cancel the SSHD job**:
   ```bash
   scancel <job_id>
   ```
4. **Remove SSH config entries** added by VS Code (optional)

### Security Considerations

1. **Don't leave SSHD jobs running** when not needed
2. **Use strong passphrases** for SSH keys
3. **Regularly rotate SSH keys** if required by policy
4. **Monitor your running jobs**: `squeue -u $USER`

## Advanced Configuration

### Multiple Concurrent Sessions

You can run multiple SSHD jobs for different projects:

```bash
# Submit multiple jobs
sbatch sshd.sh
sbatch sshd.sh

# Connect VS Code to different nodes
# Node 1: gpu01.cluster:45672
# Node 2: gpu03.cluster:45673
```

### Custom SSHD Configuration

Create specialized SSHD scripts for different use cases:

```bash
# sshd_gpu4.sh - For multi-GPU development
#SBATCH --gres=gpu:4
#SBATCH --cpus-per-task=16
#SBATCH --mem=64000

# sshd_a6000.sh - For high-memory development
#SBATCH --partition=a6000
#SBATCH --qos=normal-a6000
#SBATCH --gres=gpu:1
```

## Next Steps

- **Learn about software installation**: [Software Installation](../software/)
- **Explore advanced job submission**: [Job Submission](../job-submission/)
- **Understand storage optimization**: [Storage Systems](../storage/)