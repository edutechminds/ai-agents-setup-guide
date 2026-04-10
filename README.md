# AI Coding Agents Setup Guide

## Run Pi + Hermes Agent for Free on Ubuntu

A complete, beginner-friendly guide to setting up two powerful open-source AI coding agents side-by-side in a Multipass VM — with **zero cost, no API keys required**.

Built for students and new developers. Every command is copy-pasteable. No prior experience required.

---

### What's Inside

| File | Description |
|------|------------|
| [AI-Coding-Agents-Setup-Guide.md](AI-Coding-Agents-Setup-Guide.md) | **Start here.** Full step-by-step install guide (16 parts) — Multipass VM, Pi, Hermes, free provider options, tmux workflow, glossary |
| [agent-vm-init.yaml](agent-vm-init.yaml) | Cloud-init config file — auto-installs Node.js, Pi, Python, and Hermes inside the VM |
| [EXTRAS-Free-Tools-Tips-Resources.md](EXTRAS-Free-Tools-Tips-Resources.md) | More free tools (LM Studio, VS Code + Continue/Cline, OpenRouter, Aider, Gemini CLI), cheat sheets, learning path, privacy tips |
| [VM-Guide-Files-Security-Best-Practices.md](VM-Guide-Files-Security-Best-Practices.md) | How to move files in/out of VMs, security considerations, snapshots & backups, networking, resource management |
| [PROMPTING-Guide-How-To-Talk-To-Agents.md](PROMPTING-Guide-How-To-Talk-To-Agents.md) | How to write good prompts, worked example building a project in Pi, Hermes skills workflow, prompt templates, handling mistakes |
| [README.md](README.md) | You're reading it |

### Suggested Reading Order

1. **Setup Guide** — get everything installed and running
2. **Prompting Guide** — learn how to actually use the agents effectively
3. **Extras** — explore more free tools and plan your learning path
4. **VM Guide** — understand files, security, and VM management

---

### The Two Agents

**[Pi Coding Agent](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent)** (v0.65.x) — Minimal, fast, terminal-based coding agent with session branching, extensions, and skills. Four core tools: read, write, edit, bash. Best for hands-on coding.

**[Hermes Agent](https://github.com/NousResearch/hermes-agent)** (v0.8.0) — Self-improving autonomous agent with persistent memory, messaging gateway (Telegram/Discord/Slack), 40+ built-in tools, skill creation, and cron scheduling. Best for research, long-running tasks, and cross-session knowledge.

### Free Options Covered

| Option | Type | Account Needed? |
|--------|------|----------------|
| **Ollama** | Local (runs on your computer) | No |
| **LM Studio** | Local (GUI app) | No |
| **Canonical Inference Snaps** | Local (auto-detects hardware) | No |
| **Google Gemini CLI** | Cloud (free tier) | Google account |
| **Google AI Studio API** | Cloud (free tier) | Google account |
| **Nous Portal** | Cloud (free tier) | Nous account |
| **LLM Checker** | Hardware scanner + model recommender | No |

### Quick Start

```bash
# 1. Install Multipass
sudo snap install multipass

# 2. Create the cloud-init file (or download agent-vm-init.yaml from this repo)
# 3. Launch the VM
multipass launch 24.04 --name agents --cpus 4 --memory 8G --disk 40G --cloud-init ~/agent-vm-init.yaml

# 4. Wait 10-15 min for install, then enter the VM
multipass shell agents

# 5. Run the agents
pi       # coding agent (left tmux pane)
hermes   # autonomous agent (right tmux pane)
```

See the [full setup guide](AI-Coding-Agents-Setup-Guide.md) for provider configuration, tmux workflow, and troubleshooting.

---

### Who This Is For

- **Students** learning to code with AI assistance
- **New developers** exploring AI coding agents for the first time
- **Teachers** who want a ready-made resource for their classes
- **Anyone** curious about local AI tools without spending money

### Contributing

Found an error? Have a suggestion? Contributions welcome via [Issues](../../issues) or Pull Requests.

---

*Created April 2026 by [edutechminds](https://github.com/edutechminds)*
*Pi: https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent*
*Hermes: https://github.com/NousResearch/hermes-agent*

