---
title: "Getting Started"
linkTitle: "Getting Started"
weight: 10
description: >
  SSH access and initial setup for the Prometheus cluster
tags: ["ssh", "getting-started", "authentication", "cluster-access", "setup"]
categories: ["getting-started", "authentication"]
---

## Prerequisites

Before connecting, you need:

- A Prometheus account (request one through your MRG leader)
- An OpenSSH-compatible client
- Your registered SSH private key
- Basic familiarity with the Linux command line

## Generate SSH Keys

The Prometheus cluster uses **RSA key authentication** for secure access. You need to generate a public/private key pair. 
_Skip this section if you already have a registered Prometheus key._

### Create an SSH key

#### macOS or Linux

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_rsa
```

#### Windows

Open PowerShell and run:

```powershell
ssh-keygen -t rsa -b 4096 -f "$HOME\.ssh\id_rsa"
```

This creates:

- `id_rsa`: your private key; **never share it**
- `id_rsa.pub`: the public key to send to your MRG leader or cluster administrator

### Request Cluster Access

1. Send your public key to your MRG leader
2. Request a Prometheus account 
3. Wait for account confirmation

### Add Passphrase (Optional but Recommended)

For additional security, add a passphrase to your key:

```bash
ssh-keygen -p -f ~/.ssh/id_rsa
```

## Configure SSH

Prometheus is reached through the CyI HPC bastion. SSH handles the additional hop automatically. The same Prometheus 
username and private key are used for the bastion and the Prometheus login node. Your private key remains on your computer.

### macOS

```bash
touch ~/.ssh/config
chmod 600 ~/.ssh/config
open -e ~/.ssh/config
```

### Linux

```bash
touch ~/.ssh/config
chmod 600 ~/.ssh/config
nano ~/.ssh/config
```

### Windows

Open PowerShell:

```powershell
notepad "$HOME\.ssh\config"
```

Save the file as `config`, without a `.txt` extension. Native Windows OpenSSH reads it from `C:\Users\<Windows-user>\.ssh\config`.

{{% alert title="Windows and WSL" color="info" %}}
Windows and WSL have separate SSH configurations. Windows applications normally use the Windows configuration above; applications running inside WSL use `~/.ssh/config` in WSL.
{{% /alert %}}

### SSH configuration

Add the following configuration in your SSH config file, replacing `<your-username>` with your Prometheus username and 
`<path-to-your-private-key>` with the path to your private key (e.g., `~/.ssh/id_rsa`):

```bash
Host cyens-bastion
    HostName bastion.hpcf.cyi.ac.cy
    User <your-username>
    IdentityFile <path-to-your-private-key>
    IdentitiesOnly yes

Host prometheus
    HostName prometheus.cyens.org.cy
    User <your-username>
    IdentityFile <path-to-your-private-key>
    IdentitiesOnly yes
    ProxyJump cyens-bastion
```

## Connect to Prometheus

Once your account is activated and your SSH configuration is set up, connect using:

```bash
ssh prometheus
```

The connection path is:

```text
Your computer → CYENS bastion → Prometheus login node
```

After logging in, verify your session:

```bash
hostname
whoami
```

## First Login Setup

### Check Your Environment

```bash
# Check current directory
pwd

# List available partitions
sinfo

# Check your groups
groups

# View your home directory quota
quota -us
```

### Understand the File System

```bash
# Your home directory (20GB limit)
ls -la /trinity/home/$USER

# Shared group storage (30TB per group)
ls -la /lustreFS/data/

# Check group quota
lfs quota -gh <group-name> /lustreFS/
```

## Cluster Architecture Overview

The Prometheus cluster consists of:

### Head Node
- **Login and job submission** point
- **DO NOT run compute jobs here**
- Used for file management and job scheduling

### Compute Nodes
- **`gpu[01-08]`**: 8 nodes with A5000 GPUs (8 GPUs each)
- **`gpu09`**: 1 node with A6000 Ada GPUs (4 GPUs)
- **512GB RAM** and **32 CPU cores** per node

### Storage Systems
- **`/trinity/home/`**: Personal home directories (SSD, 20GB limit)
- **`/lustreFS/data/`**: Shared group storage (305TB Lustre filesystem)
- **Local storage**: 1TB NVMe on each compute node

## Important Usage Rules

{{% alert title="Critical" color="warning" %}}
**NEVER run compute jobs directly on the head node!** Always use SLURM to submit jobs to compute nodes.
{{% /alert %}}

### Checking Cluster Status

```bash
# View partition information
sinfo

# Check job queue
squeue

# Check your running jobs
squeue -u $USER

# View detailed node information
scontrol show nodes
```

Example `sinfo` output:
```
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
defq*        up   infinite      6  idle~ gpu[03-08]
defq*        up   infinite      1    mix gpu02
defq*        up   infinite      1   idle gpu01
a6000        up   infinite      1    mix gpu09
```

## Next Steps

Now that you're connected to Prometheus:

1. **Set up your development environment** - See [Environment Modules](../modules/)
2. **Learn about partitions and queues** - Read [Partitions & Queues](../partitions/)
3. **Submit your first job** - Follow [Job Submission](../job-submission/)
4. **Configure VS Code** (optional) - See [VS Code Setup](../vscode/)

## Getting Help

- **Cluster status**: Use `sinfo` and `squeue` commands
- **Support**: Contact your MRG leader
- **System issues**: Report to cluster administrators

## Common First-Time Issues

### "Permission denied (publickey)"
- Verify your public key was added to the cluster
- Check your SSH config file syntax
- Ensure private key permissions are correct (`chmod 600`)

### "Connection refused"
- Verify the hostname: `prometheus.cyens.org.cy`
- Check if you're connected to the internet
- Confirm your account is activated

### "Quota exceeded"
- Home directory has a 20GB limit
- Use group storage in `/lustreFS/data/` for large files
- Check usage with `quota -us`