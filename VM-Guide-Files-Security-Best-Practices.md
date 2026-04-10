# Ubuntu VM Guide: Files, Security & Best Practices

## A Practical Reference for Working with Multipass VMs (and VMs in General)

**Companion to the [AI Coding Agents Setup Guide](AI-Coding-Agents-Setup-Guide.md)**

This guide covers the things that trip people up when working with VMs: getting files in and out, keeping things secure, managing resources, networking, backups, and knowing what lives where.

---

## TABLE OF CONTENTS

1. How files work between host and VM
2. Moving files in and out
3. Security: what to know
4. Snapshots and backups
5. Networking: how VMs talk to the world
6. Resource management
7. Multiple VMs
8. When things go wrong
9. Other VM options beyond Multipass
10. Quick reference

---

## PART 1: How Files Work Between Host and VM

Your VM is a separate computer. Files on your host (your real desktop) and files inside the VM are **completely separate** by default. Nothing is shared unless you explicitly set it up.

There are three ways to bridge the gap:

| Method | How It Works | Best For |
|--------|-------------|----------|
| **Mount** | Host folder appears live inside the VM. Changes sync both ways instantly. | Active development — edit on host, run in VM |
| **Transfer** | Copy files one at a time between host and VM. | Moving finished files, one-off copies |
| **Shared clipboard** | Not available in Multipass (it's terminal-only, no GUI). | N/A — use transfer or mount instead |

### Important: What Lives Where

```
YOUR HOST (real computer)
├── ~/projects/           ← Your code, edited with VS Code etc.
│
└── Multipass VM: "agents"
    ├── ~/                ← VM's home directory (separate filesystem)
    ├── ~/projects/       ← ONLY exists if you mounted it from host
    ├── ~/.pi/            ← Pi config and sessions (lives IN the VM)
    └── ~/.hermes/        ← Hermes config and memories (lives IN the VM)
```

**Key insight:** Pi and Hermes configs live inside the VM. If you delete the VM, you lose those configs, sessions, and Hermes memories. Your mounted project files are safe because they live on the host.

---

## PART 2: Moving Files In and Out

### Method 1: Mounting (Recommended for Development)

Mount makes a host folder appear inside the VM. Edits on either side appear instantly on the other.

```bash
# Mount your projects folder
multipass mount ~/projects agents:/home/ubuntu/projects

# Mount with a specific VM path
multipass mount ~/Documents/research agents:/home/ubuntu/research

# Mount at launch time (persists automatically)
multipass launch 24.04 --name myvm --mount ~/projects:/home/ubuntu/projects

# See what's mounted
multipass info agents   # Look for "Mounts:" section

# Unmount a specific folder
multipass umount agents:/home/ubuntu/projects

# Unmount everything
multipass umount agents
```

**Performance tips for mounts:**

- Mount your **source code** (few, larger files) — this works great
- Do NOT mount `node_modules`, `venv`, `.git` internals, or database files from the host — these are thousands of tiny files and will be very slow over SSHFS
- Instead, run `npm install` or `pip install` **inside the VM** so those folders live on the VM's native disk

```bash
# GOOD pattern: mount source, install dependencies in VM
multipass mount ~/projects/myapp agents:/home/ubuntu/myapp
multipass exec agents -- bash -c "cd /home/ubuntu/myapp && npm install"

# BAD pattern: mounting node_modules from host (painfully slow)
```

### Method 2: Transfer (Copy Files)

For one-off file moves when you don't need a live connection.

```bash
# Copy a file FROM your host TO the VM
multipass transfer ./my-script.sh agents:/home/ubuntu/

# Copy a file FROM the VM TO your host
multipass transfer agents:/home/ubuntu/results.txt ./

# Copy multiple files
multipass transfer agents:/home/ubuntu/file1.txt agents:/home/ubuntu/file2.txt ./

# Copy an entire folder (transfer doesn't support folders directly — use tar)
# From host to VM:
tar czf - ~/projects/myapp | multipass exec agents -- tar xzf - -C /home/ubuntu/

# From VM to host:
multipass exec agents -- tar czf - /home/ubuntu/myapp > myapp-backup.tar.gz
```

### Method 3: SCP / SSH (Advanced)

If you want traditional SSH access:

```bash
# Find the VM's IP
multipass info agents | grep IPv4

# SSH in (Multipass uses key-based auth automatically)
ssh ubuntu@<VM_IP>

# SCP files
scp ./file.txt ubuntu@<VM_IP>:/home/ubuntu/
scp ubuntu@<VM_IP>:/home/ubuntu/file.txt ./
```

---

## PART 3: Security — What to Know

### The Basics

Multipass VMs are **isolated from your host** by default — the VM can't see your host's files, processes, or network interfaces unless you explicitly allow it (via mounts or network config).

However, Multipass is designed for **development, not production**. Here are the key things to understand:

### What's Isolated

- **Filesystem:** The VM has its own disk. It cannot see host files unless you mount them.
- **Processes:** Programs in the VM can't see or affect host processes.
- **Users:** The VM has its own user accounts. `ubuntu` inside the VM is not your host user.

### What's NOT Isolated

- **Mounted folders:** If you mount `~/projects`, the VM has full read/write access to that folder. An AI agent running inside the VM could modify or delete those files.
- **Network:** The VM can access the internet and your local network. An AI agent could theoretically make network requests.
- **Multipass daemon:** Runs as root on your host. Anyone with access to the Multipass socket has full control.

### Practical Security Tips

1. **Only mount what you need.** Don't mount your entire home directory. Mount specific project folders.

```bash
# GOOD — mount only what's needed
multipass mount ~/projects/myapp agents:/home/ubuntu/myapp

# RISKY — mounting your entire home gives VM access to everything
multipass mount ~ agents:/home/ubuntu/host-home   # Don't do this
```

2. **Keep secrets off mounts.** Don't mount folders containing SSH keys, API keys, password files, or `.env` files with secrets. If an agent needs an API key, set it as an environment variable inside the VM.

3. **Back up before experiments.** Before letting an AI agent make big changes, back up your project:

```bash
# Quick backup from host
cp -r ~/projects/myapp ~/projects/myapp-backup-$(date +%Y%m%d)
```

4. **Use git for safety.** If your project is in a git repo, the agent's changes can always be reviewed and reverted:

```bash
# Inside the VM, in your project
git diff          # See what changed
git checkout .    # Undo all changes
git stash         # Save changes for later without committing
```

5. **The VM is disposable.** If something goes wrong inside the VM, you can always delete it and start fresh. Your mounted host files are unaffected:

```bash
multipass delete agents && multipass purge
# Re-run the launch command from the setup guide
```

6. **Hermes file safety.** Hermes has a built-in rollback feature — it takes filesystem checkpoints before destructive operations. Type `/rollback` inside Hermes to restore.

---

## PART 4: Snapshots and Backups

### Backing Up Agent Configs

Pi and Hermes configs live inside the VM. If you want to preserve them:

```bash
# Back up Pi config to your host
multipass transfer agents:/home/ubuntu/.pi/agent/settings.json ./pi-settings-backup.json
multipass transfer agents:/home/ubuntu/.pi/agent/models.json ./pi-models-backup.json

# Back up Hermes config
multipass exec agents -- tar czf /tmp/hermes-backup.tar.gz -C /home/ubuntu .hermes/
multipass transfer agents:/tmp/hermes-backup.tar.gz ./hermes-backup.tar.gz

# Back up everything (entire VM home directory)
multipass exec agents -- tar czf /tmp/full-backup.tar.gz -C /home/ubuntu .pi .hermes
multipass transfer agents:/tmp/full-backup.tar.gz ./vm-configs-backup.tar.gz
```

### Multipass Snapshots

Multipass supports VM snapshots — freeze the entire VM state and restore later:

```bash
# Take a snapshot
multipass snapshot agents --name before-experiment

# List snapshots
multipass snapshot agents --list

# Restore a snapshot (VM must be stopped first)
multipass stop agents
multipass restore agents --snapshot before-experiment
multipass start agents
```

**When to snapshot:**
- Before major config changes
- Before letting an agent run a big automated task
- Before updating Pi or Hermes to a new version
- Before installing unfamiliar software in the VM

### Cloning a VM

Want a second copy of your setup (for a student, or to experiment without risk)?

```bash
# There's no direct clone command, but you can export and re-import
# Or simply save your cloud-init YAML and re-launch:
multipass launch 24.04 --name agents2 --cpus 4 --memory 8G --disk 40G --cloud-init ~/agent-vm-init.yaml
```

---

## PART 5: Networking

### How VMs Connect to the Network

By default, Multipass VMs get their own private IP address on a virtual network. They can:
- Access the internet (through your host's connection)
- Access services running on your host (like Ollama on port 11434)
- Be accessed from your host (via their IP)

### Find Your VM's IP

```bash
multipass info agents | grep IPv4
# Output: IPv4:  10.140.26.17
```

### Access a Service Running in the VM

If you run a web server inside the VM (say on port 3000):

```bash
# Find the VM's IP
multipass info agents | grep IPv4
# Then open in your browser: http://10.140.26.17:3000
```

### Access Host Services from Inside the VM

To reach Ollama (or anything else) running on your host from inside the VM:

```bash
# Inside the VM, find the gateway IP (your host):
ip route show default | awk '{print $3}'
# Use that IP to connect: curl http://10.140.26.1:11434/v1/models
```

### Port Forwarding (Access VM from Outside)

If you want other devices on your network to reach a service in the VM, you'd need to set up port forwarding. This is an advanced topic — for most student use, accessing via the VM's IP from the host is sufficient.

---

## PART 6: Resource Management

### Check Current Usage

```bash
# VM resource usage
multipass info agents

# Inside the VM — detailed system info
htop          # Interactive process viewer (press q to quit)
df -h         # Disk usage
free -h       # Memory usage
```

### Resize a Running VM

You can change CPU, memory, and disk **while the VM is stopped**:

```bash
multipass stop agents

# Give it more CPUs
multipass set local.agents.cpus=6

# Give it more memory
multipass set local.agents.memory=12G

# Give it more disk (can only increase, not decrease)
multipass set local.agents.disk=60G

multipass start agents
```

### When to Resize

| Symptom | Fix |
|---------|-----|
| Agents respond slowly | Increase CPUs or memory |
| "No space left on device" | Increase disk |
| Everything is swapping | Increase memory |
| Host becomes unresponsive | Decrease VM resources — you gave it too much |

**Rule of thumb:** Don't give the VM more than half your host's total RAM.

---

## PART 7: Multiple VMs

You can run multiple VMs for different purposes:

```bash
# A VM for AI agents
multipass launch 24.04 --name agents --cpus 4 --memory 8G --disk 40G --cloud-init ~/agent-vm-init.yaml

# A VM for web development
multipass launch 24.04 --name webdev --cpus 2 --memory 4G --disk 20G

# A VM for testing something risky
multipass launch 24.04 --name sandbox --cpus 2 --memory 2G --disk 10G

# List all VMs
multipass list

# They're independent — deleting one doesn't affect the others
multipass delete sandbox && multipass purge
```

**Tip:** Stop VMs you're not using to free up resources: `multipass stop webdev`

---

## PART 8: When Things Go Wrong

### VM Won't Start

```bash
# Check status
multipass list

# If stuck in "Starting" or "Unknown":
multipass stop agents --force
multipass start agents

# Nuclear option:
multipass delete agents && multipass purge
# Re-launch from your cloud-init YAML
```

### Ran Out of Disk Space in VM

```bash
# Check from outside
multipass info agents | grep Disk

# Increase disk (VM must be stopped)
multipass stop agents
multipass set local.agents.disk=60G
multipass start agents
```

### VM is Using Too Much Host RAM

```bash
# Check what's running
multipass info agents

# Reduce VM memory
multipass stop agents
multipass set local.agents.memory=4G
multipass start agents
```

### Mount Stopped Working

```bash
# Remount
multipass umount agents:/home/ubuntu/projects
multipass mount ~/projects agents:/home/ubuntu/projects

# If mount fails, SSHFS might need installing
multipass exec agents -- sudo apt install -y sshfs
```

### Lost My Pi/Hermes Config

If you deleted the VM without backing up:
- Pi config: Must be recreated (follow setup guide Part 8)
- Hermes config: Must be recreated (`hermes setup`)
- Hermes memories: Gone (this is why backups matter)
- Your project files: Safe if they were on a mounted host folder

**Prevention:** Back up regularly (see Part 4) or keep project files on the host.

---

## PART 9: Other VM Options Beyond Multipass

Multipass is the easiest option for Ubuntu, but it's not the only one.

| Tool | Best For | Complexity |
|------|----------|------------|
| **Multipass** | Quick Ubuntu VMs, development | Easy |
| **VirtualBox** | Multi-OS VMs with GUI, classroom labs | Medium |
| **virt-manager / KVM** | Full Linux virtualization with GUI management | Medium |
| **LXD / Incus** | Lightweight system containers (faster than VMs) | Medium |
| **Docker** | Application containers (not full VMs) | Medium |
| **GNOME Boxes** | Simple VM GUI on GNOME desktops | Easy |

### When Multipass Isn't Enough

- **Need a GUI inside the VM** → Use VirtualBox or virt-manager
- **Need Windows or macOS VMs** → Use VirtualBox
- **Need dozens of lightweight instances** → Use LXD/Incus containers
- **Need to run just one app in isolation** → Use Docker
- **Need GPU passthrough** → Use virt-manager with KVM

### Quick VirtualBox Setup (for GUI VMs)

```bash
sudo apt install virtualbox
# Then use the VirtualBox GUI to create VMs with a full desktop
```

### Quick LXD Setup (for fast lightweight containers)

```bash
sudo snap install lxd
lxd init --auto
lxc launch ubuntu:24.04 mycontainer
lxc exec mycontainer -- bash
```

LXD containers share the host kernel and start in seconds. They're faster than VMs but slightly less isolated.

---

## PART 10: Quick Reference

### Essential Multipass Commands

```bash
# Lifecycle
multipass launch 24.04 --name NAME --cpus N --memory NG --disk NG
multipass start NAME
multipass stop NAME
multipass delete NAME && multipass purge
multipass restart NAME

# Access
multipass shell NAME                    # Interactive shell
multipass exec NAME -- COMMAND          # Run a single command

# Files
multipass mount ~/local NAME:~/remote   # Live folder sharing
multipass umount NAME:~/remote          # Stop sharing
multipass transfer ./file NAME:~/       # Copy to VM
multipass transfer NAME:~/file ./       # Copy from VM

# Info
multipass list                          # All VMs
multipass info NAME                     # Details for one VM
multipass find                          # Available images

# Snapshots
multipass snapshot NAME --name SNAP     # Take snapshot
multipass snapshot NAME --list          # List snapshots
multipass restore NAME --snapshot SNAP  # Restore (VM must be stopped)

# Resize (VM must be stopped)
multipass set local.NAME.cpus=N
multipass set local.NAME.memory=NG
multipass set local.NAME.disk=NG
```

### File Location Cheat Sheet

| What | Where (Host) | Where (VM) |
|------|-------------|------------|
| Your project code | `~/projects/` | `~/projects/` (if mounted) |
| Pi config | N/A | `~/.pi/agent/` |
| Pi sessions | N/A | `~/.pi/agent/sessions/` |
| Hermes config | N/A | `~/.hermes/config.yaml` |
| Hermes memories | N/A | `~/.hermes/memories/` |
| Hermes skills | N/A | `~/.hermes/skills/` |
| Hermes sessions | N/A | `~/.hermes/sessions/` |
| Ollama models | `~/.ollama/models/` | N/A (runs on host) |
| LM Studio models | `~/.cache/lm-studio/` | N/A (runs on host) |
| Cloud-init YAML | `~/agent-vm-init.yaml` | N/A |
| Multipass data | `/var/snap/multipass/` | N/A |

---

*Last updated April 2026. Additions and corrections welcome via Issues or PRs.*
