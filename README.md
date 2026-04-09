# AI Coding Agents Setup Guide

## Run Pi + Hermes Agent for Free on Ubuntu

A complete, beginner-friendly guide to setting up two powerful open-source AI coding agents side-by-side in a Multipass VM — with **zero cost, no API keys required**.

### What's Inside

| File | Description |
|------|------------|
| `AI-Coding-Agents-Setup-Guide.md` | Full step-by-step guide (16 parts) |
| `agent-vm-init.yaml` | Cloud-init config — auto-installs everything in the VM |

### The Two Agents

**[Pi Coding Agent](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent)** — Minimal, fast, terminal-based coding agent with session branching, extensions, and skills. Best for hands-on coding.

**[Hermes Agent](https://github.com/NousResearch/hermes-agent)** — Self-improving autonomous agent with persistent memory, messaging gateway (Telegram/Discord/Slack), and 40+ built-in tools. Best for research, long-running tasks, and cross-session knowledge.

### Free Options Covered

- **Google Gemini CLI** — Free with any Google account, no API key
- **Ollama** — Run open-source models locally, completely private
- **Canonical Inference Snaps** — Auto-detects your hardware, installs optimized models
- **LLM Checker** — Scans your system, recommends the best model for your hardware
- **Nous Portal** — Free tier for Hermes

### Quick Start

```bash
# 1. Install Multipass
sudo snap install multipass

# 2. Download agent-vm-init.yaml from this repo, then:
multipass launch 24.04 --name agents --cpus 4 --memory 8G --disk 40G --cloud-init ~/agent-vm-init.yaml

# 3. Wait 10-15 min, then enter the VM
multipass shell agents

# 4. Run the agents
pi       # coding agent
hermes   # autonomous agent
```

See the full guide for provider setup, tmux split-screen workflow, and troubleshooting.

### Who This Is For

Students and developers who want to explore AI coding agents without spending money. Every command is copy-pasteable. No prior experience with VMs, AI models, or terminal tools required.

### Versions

- Pi coding-agent v0.65.x
- Hermes Agent v0.8.0
- Guide created April 2026

---

*Contributions and corrections welcome via Issues or PRs.*
