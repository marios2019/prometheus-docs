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

Before accessing the Prometheus cluster, you need:

- A valid cluster account (contact your MRG leader)
- SSH client installed on your local machine
- Basic familiarity with Linux command line

## Generate SSH Keys

The Prometheus cluster uses **RSA key authentication** for secure access. You need to generate a public/private key pair:

### Step 1: Create SSH Key Pair

Open your terminal and run:

```bash
ssh-keygen
```

Follow the on-screen instructions. This creates:
- **Private key**: `~/.ssh/id_rsa` (keep this secure!)
- **Public key**: `~/.ssh/id_rsa.pub` (share this with administrators)

### Step 2: Secure Your Private Key

For Linux/Mac users, set proper permissions:

```bash
chmod 600 ~/.ssh/id_rsa
```

### Step 3: Request Cluster Access

1. **Send your public key** to your MRG leader
2. **Request a Prometheus account** 
3. **Wait for account confirmation**

### Step 4: Add Passphrase (Optional but Recommended)

For additional security, add a passphrase to your key:

```bash
ssh-keygen -p -f ~/.ssh/id_rsa
```

## Connect to Prometheus

### Configure SSH Client

Create or edit `~/.ssh/config` file with the following content:

```bash
Host prometheus
  Hostname prometheus.cyens.org.cy
  User <your-username>
  IdentityFile ~/.ssh/id_rsa
```

Replace `<your-username>` with your actual cluster username.

### Connect via SSH

Once your account is activated, connect using:

```bash
ssh prometheus
```

You should now be logged into the Prometheus head node!

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
- **Documentation**: Check `/opt/cluster/docs/` on the cluster
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