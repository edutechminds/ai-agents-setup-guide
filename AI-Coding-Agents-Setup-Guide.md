# AI Coding Agents Setup Guide (Free — No API Key Required)

## Pi Coding Agent + Hermes Agent on Ubuntu using Multipass VM

**What you'll end up with:** Two AI coding agents running side-by-side in a clean Ubuntu virtual machine. Multiple free options to power them — no credit card, no API key, no cost.

**Time needed:** About 20-30 minutes (mostly waiting for downloads).

**What you need:**
- An Ubuntu Desktop computer (any recent version)
- An internet connection
- A Google account (easiest free cloud option) OR 16GB+ RAM (local Ollama option)

---

## TABLE OF CONTENTS

1. Free options overview
2. Auto-detect your best local model (NEW)
3. Install Multipass
4. (Optional) Install Ollama for local models
5. Create the VM config file
6. Launch the VM
7. Enter the VM
8. Configure Pi (Option A: Google / Option B: Ollama)
9. Configure Hermes (Option A: Google / Option B: Ollama)
10. Split-screen workflow (tmux)
11. Share project files with the VM
12. Daily usage
13. Upgrading to paid API keys later
14. When to use which agent
15. Glossary for students
16. Troubleshooting

---

## PART 1: Free Options to Power Your Agents (Pick One)

### Option A: Google Gemini CLI / Antigravity (Easiest — Cloud, Free)
- **What:** Google Gemini models, free with a Google account
- **Pi:** Built-in — select `google-gemini-cli` or `google-antigravity` as provider
- **Hermes:** Use "Custom endpoint" with a free Gemini API key from AI Studio
- **Pros:** No hardware requirements, powerful Gemini 3 Flash models, zero setup
- **Cons:** Rate limited (60 req/min), requires internet, Google sees your prompts
- **Best for:** Getting started fast, lighter machines, students

### Option B: Ollama (Most Private — Local, Free)
- **What:** Runs open-source AI models entirely on your computer
- **Pi:** Built-in via custom provider config
- **Hermes:** Built-in — select "Custom endpoint" during setup
- **Pros:** Completely private, works offline, no rate limits, no account needed
- **Cons:** Needs 16GB+ RAM and ideally a GPU, slower than cloud
- **Best for:** Privacy, offline work, learning how models run

### Option C: Canonical Inference Snaps (Ubuntu-Native — Local, Free)
- **What:** Canonical's own hardware-optimized AI model packages for Ubuntu
- **Pi/Hermes:** Exposes an OpenAI-compatible API, works as custom endpoint
- **Pros:** Auto-detects your hardware (GPU/CPU/NPU) and picks the best engine
- **Cons:** Limited model selection (currently DeepSeek R1, Qwen VL, Gemma 3)
- **Best for:** Ubuntu users who want one-command setup with hardware optimization

### Option D: Nous Portal Free Tier (Hermes-native)
- **What:** Nous Research's inference service with a free tier
- **Hermes:** Built-in first-class provider
- **Pros:** Optimized for Hermes, free MiMo v2 Pro for auxiliary tasks
- **Cons:** Limited quota, mainly for Hermes
- **Best for:** Getting Hermes running quickly

### Recommended Mix
- Pi → Google Gemini CLI (free, just Google login)
- Hermes → Nous Portal free tier or Ollama
- Later → Add Anthropic/OpenAI API keys when you want more power

---

## PART 2: Auto-Detect Your Best Local Model

Before committing to a specific model, let your system tell you what it can handle. Two tools help with this.

### Tool 1: Canonical Inference Snaps (One-Command Auto-Detection)

This is Canonical's official solution. When you install a model snap, it automatically detects your hardware (NVIDIA GPU, Intel GPU/NPU, Intel CPU, etc.) and installs the best-optimized engine. No manual configuration needed.

**On your host machine (not inside the VM):**

```
# Install Gemma 3 — auto-detects your hardware
sudo snap install gemma3

# Check which engine was selected for your hardware
gemma3 status
```

If the status shows `intel-cpu`, `nvidia-gpu`, or similar, the snap correctly detected your hardware and chose the optimal runtime. The snap exposes an OpenAI-compatible API that both Pi and Hermes can connect to.

**Available inference snaps (as of April 2026):**

| Snap | Install Command | What It Does |
|------|----------------|-------------|
| Gemma 3 | `sudo snap install gemma3` | Google's latest open model |
| Qwen VL | `sudo snap install qwen-vl --channel 2.5/beta` | Vision + language model |
| DeepSeek R1 | `sudo snap install deepseek-r1` | Strong reasoning model |

**Note:** These snaps require Docker to be installed. Install it first if you don't have it:

```
sudo apt install docker.io
sudo usermod -aG docker $USER
# Log out and back in for group change to take effect
```

### Tool 2: LLM Checker (Detailed Hardware Analysis + Recommendations)

This tool scans your entire system and recommends the best models across quality, speed, fit, and context dimensions. It works with Ollama.

**On your host machine:**

```
# Install LLM Checker
npm install -g llm-checker

# Step 1: Scan your hardware
llm-checker hw-detect
```

This prints a detailed report of your system — CPU, GPU, VRAM, RAM, memory bandwidth, and a tier rating (LOW / MEDIUM / HIGH / ULTRA).

```
# Step 2: Get model recommendations for coding
llm-checker recommend --category coding
```

This outputs a ranked list of models that will actually run well on YOUR specific hardware, with the exact `ollama pull` commands to install them.

```
# Other useful categories
llm-checker recommend --category general
llm-checker recommend --optimize speed      # fastest option
llm-checker recommend --optimize quality    # best output quality
llm-checker recommend --optimize balanced   # best of both worlds
```

**What LLM Checker tells you that Ollama doesn't:** Ollama will happily let you download a model that's too large for your RAM and then run painfully slowly. LLM Checker prevents this by only recommending models that genuinely fit your hardware.

### Quick Hardware Reference (If You Skip the Tools)

| Your RAM | Your GPU | Best Model | Ollama Command |
|----------|----------|-----------|----------------|
| 8GB | None | Phi-4-mini (3.8B) | `ollama pull phi4-mini` |
| 16GB | None | Qwen 2.5 Coder 7B | `ollama pull qwen2.5-coder:7b` |
| 16GB | 8GB VRAM | Qwen3 Coder | `ollama pull qwen3-coder` |
| 32GB | 12-16GB VRAM | Qwen 2.5 Coder 14B | `ollama pull qwen2.5-coder:14b` |
| 32GB+ | 24GB VRAM | Qwen 2.5 Coder 32B | `ollama pull qwen2.5-coder:32b` |

---

## PART 3: Install Multipass (the VM Manager)

Multipass is made by Canonical. It creates Ubuntu virtual machines with one command.

### Step 3.1: Open a terminal

Press **Ctrl + Alt + T** on your keyboard.

### Step 3.2: Install Multipass

```
sudo snap install multipass
```

Type your password (you won't see characters — that's normal). Wait for "multipass installed".

### Step 3.3: Verify

```
multipass version
```

---

## PART 4: (Option B Only) Install Ollama on Your Host

**Skip this if using Option A (Google Gemini) or Option C (Inference Snaps).** Jump to Part 5.

### Step 4.1: Install Ollama

```
curl -fsSL https://ollama.com/install.sh | sh
```

### Step 4.2: Pull a model

Use the recommendation from Part 2, or pick from the quick reference table. Example:

```
ollama pull qwen3-coder
```

### Step 4.3: Make Ollama accessible from the VM

```
sudo systemctl edit ollama.service
```

Paste between the comment blocks:

```
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
```

Save (Ctrl+O, Enter, Ctrl+X) and restart:

```
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

Verify:

```
curl http://localhost:11434/v1/models
```

---

## PART 5: Create the VM Configuration File

Copy-paste this **entire block** into your terminal:

```
cat > ~/agent-vm-init.yaml << 'ENDOFFILE'
#cloud-config
package_update: true
package_upgrade: true

packages:
  - git
  - curl
  - build-essential
  - tmux
  - ripgrep
  - jq
  - unzip
  - htop

write_files:
  - path: /home/ubuntu/setup-agents.sh
    permissions: '0755'
    owner: ubuntu:ubuntu
    content: |
      #!/bin/bash
      set -e

      echo ""
      echo "=========================================="
      echo "  STEP 1 of 4: Installing Node.js 22"
      echo "=========================================="
      curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
      sudo apt install -y nodejs
      echo "  Node version: $(node --version)"

      echo ""
      echo "=========================================="
      echo "  STEP 2 of 4: Installing Pi Coding Agent"
      echo "=========================================="
      npm install -g @mariozechner/pi-coding-agent
      echo "  Pi installed!"

      echo ""
      echo "=========================================="
      echo "  STEP 3 of 4: Installing uv + Python"
      echo "=========================================="
      curl -LsSf https://astral.sh/uv/install.sh | sh
      export PATH="$HOME/.local/bin:$PATH"
      echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc

      echo ""
      echo "=========================================="
      echo "  STEP 4 of 4: Installing Hermes Agent"
      echo "=========================================="
      curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

      cat > ~/.tmux.conf << 'TMUX'
      set -g mouse on
      set -g base-index 1
      setw -g pane-base-index 1
      set -g status-style 'bg=#333333 fg=#ffffff'
      set -g history-limit 50000
      TMUX

      echo ""
      echo "============================================"
      echo "  ALL DONE!"
      echo "  Run 'pi' for Pi coding agent"
      echo "  Run 'hermes' for Hermes Agent"
      echo "============================================"

runcmd:
  - sudo -u ubuntu bash /home/ubuntu/setup-agents.sh
ENDOFFILE
```

Verify:

```
ls -la ~/agent-vm-init.yaml
```

---

## PART 6: Launch the VM

```
multipass launch 24.04 --name agents --cpus 4 --memory 8G --disk 40G --cloud-init ~/agent-vm-init.yaml
```

**Smaller machine?** Use `--cpus 2 --memory 4G --disk 20G` instead.

Wait for `Launched: agents`, then wait 5-10 more minutes for the install script. Check progress:

```
multipass exec agents -- tail -5 /var/log/cloud-init-output.log
```

When you see "ALL DONE!" you're ready.

---

## PART 7: Enter the VM

```
multipass shell agents
```

Verify both agents:

```
which pi && echo "Pi: OK" || echo "Pi: NOT FOUND"
source ~/.bashrc
which hermes && echo "Hermes: OK" || echo "Hermes: NOT FOUND"
```

---

## PART 8A: Configure Pi — Google Gemini (Option A)

```
pi
```

1. Select **`google-gemini-cli`** as provider
2. A URL appears — open it in your browser, log in with Google, paste the code back
3. Pick **`gemini-3-flash`**
4. Test: `What files are in the current directory?`

---

## PART 8B: Configure Pi — Ollama (Option B)

### Find your host's IP (from inside the VM):

```
ip route show default | awk '{print $3}'
```

Write down this IP (e.g. `10.140.26.1`).

### Test the connection:

```
curl http://YOUR_IP:11434/v1/models
```

### Create Pi config (replace YOUR_IP and your model name):

```
mkdir -p ~/.pi/agent

cat > ~/.pi/agent/models.json << 'ENDOFFILE'
{
  "providers": {
    "ollama": {
      "baseUrl": "http://YOUR_IP:11434/v1",
      "api": "openai-completions",
      "apiKey": "ollama",
      "compat": {
        "supportsDeveloperRole": false,
        "supportsReasoningEffort": false
      },
      "models": [
        {
          "id": "qwen3-coder",
          "name": "Qwen3 Coder (Local)",
          "reasoning": false,
          "input": ["text"],
          "contextWindow": 32768,
          "maxTokens": 8192,
          "cost": { "input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0 }
        }
      ]
    }
  }
}
ENDOFFILE
```

**Edit to fix YOUR_IP:**

```
nano ~/.pi/agent/models.json
```

Replace `YOUR_IP` with the actual IP. Save: Ctrl+O, Enter, Ctrl+X.

### Set as default:

```
cat > ~/.pi/agent/settings.json << 'ENDOFFILE'
{
  "defaultProvider": "ollama",
  "defaultModel": "qwen3-coder"
}
ENDOFFILE
```

### Test:

```
pi
```

Type: `What files are in the current directory?`

---

## PART 9A: Configure Hermes — Google (Option A)

1. Get a free API key at https://aistudio.google.com/apikey
2. Run `hermes`
3. Select **"Custom OpenAI-compatible endpoint"**
4. API base URL: `https://generativelanguage.googleapis.com/v1beta/openai`
5. API key: paste your Gemini key
6. Model: `gemini-2.5-flash`
7. Test: `Hello! What tools do you have available?`

---

## PART 9B: Configure Hermes — Ollama (Option B)

1. Run `hermes`
2. Select **"Custom OpenAI-compatible endpoint"**
3. API base URL: `http://YOUR_IP:11434/v1`
4. API key: leave blank or type anything
5. Model: `qwen3-coder`
6. Context length: `32768`
7. If status bar shows wrong model, run `/exit` then `hermes model` to fix
8. Test: `Hello! What tools do you have available?`

---

## PART 10: Split-Screen Workflow (tmux)

### Start tmux:

```
tmux new-session -s work
```

### Split and run both agents:

1. **Ctrl+B** then **%** — splits left/right
2. Left pane: type `pi`
3. **Ctrl+B** then **→** — move to right pane
4. Right pane: type `hermes`

### Navigation:

| Keys | Action |
|------|--------|
| Ctrl+B then ← | Go to Pi (left) |
| Ctrl+B then → | Go to Hermes (right) |
| Ctrl+B then % | Split left/right |
| Ctrl+B then " | Split top/bottom |
| Ctrl+B then z | Zoom/unzoom pane |
| Ctrl+B then d | Detach (keeps running) |
| `tmux attach -t work` | Reattach |
| Mouse click | Switch pane |

---

## PART 11: Share Project Files

From your **host terminal** (Ctrl+Alt+T):

```
mkdir -p ~/projects
multipass mount ~/projects agents:/home/ubuntu/projects
```

Inside the VM, your projects appear at `~/projects`.

---

## PART 12: Daily Usage

### Starting:

```
multipass start agents
multipass shell agents
tmux attach -t work
```

### Stopping:

Ctrl+B then d (detach tmux). Then:

```
exit
multipass stop agents
```

### Quick reference:

| Command | What it does |
|---------|-------------|
| `multipass shell agents` | Enter the VM |
| `multipass start agents` | Start the VM |
| `multipass stop agents` | Stop (saves everything) |
| `multipass info agents` | Resource usage |
| `multipass delete agents && multipass purge` | Delete permanently |

---

## PART 13: Upgrading to Paid API Keys Later

**Pi:** Press Ctrl+L to switch models, or `/login` for Anthropic/OpenAI.

**Hermes:** Run `hermes model` to switch providers.

Both preserve sessions and history when switching. Recommended paid options:
- **Anthropic Claude** — best for coding precision
- **OpenRouter** — 200+ models, budget controls, one API key

---

## PART 14: When to Use Which Agent

| I want to... | Use |
|--------------|-----|
| Write or debug code | **Pi** |
| Research something | **Hermes** |
| Remember things across sessions | **Hermes** |
| Try two approaches to same problem | **Pi** (`/tree` to branch) |
| Run a long background task | **Hermes** |
| Customize behavior per-project | **Pi** (`.pi/` folder in repo) |
| Talk to agent from my phone | **Hermes** (messaging gateway) |
| Generate training data / RL | **Hermes** (Atropos) |

---

## PART 15: Glossary for Students

| Term | What It Means |
|------|--------------|
| **VM** | Virtual Machine — a simulated computer running inside your real computer |
| **Multipass** | Canonical's tool for quickly creating Ubuntu VMs |
| **Ollama** | Software that runs AI models locally on your machine |
| **LLM** | Large Language Model — the AI that powers these agents (like ChatGPT) |
| **Provider** | The service that hosts the AI model (Google, Anthropic, Ollama, etc.) |
| **API key** | A password-like code that gives you access to a cloud AI service |
| **tmux** | A terminal multiplexer — lets you split your terminal into multiple panes |
| **Cloud-init** | A standard for automatically configuring VMs on first boot |
| **Inference** | The process of an AI model generating a response to your prompt |
| **Quantization** | Compressing a model to use less memory (4-bit, 8-bit, etc.) |
| **VRAM** | Video RAM on your GPU — the main bottleneck for local AI models |
| **Context window** | How much text the AI can "see" at once (measured in tokens) |
| **Token** | A chunk of text (roughly 3/4 of a word) — how AI models count text |
| **Session** | A conversation with history — both agents save these automatically |
| **Skill** | A reusable instruction set that teaches an agent how to do something |
| **Extension** | A plugin that adds new capabilities to an agent |
| **MCP** | Model Context Protocol — a standard for connecting AI tools together |
| **Inference Snap** | Canonical's packaged AI model that auto-detects your hardware |

---

## PART 16: Troubleshooting

**"multipass: command not found"**
```
sudo snap install multipass
```
Close and reopen terminal.

**"Instance 'agents' already exists"**
Use it: `multipass start agents && multipass shell agents`
Or delete: `multipass delete agents && multipass purge`

**"pi: command not found" inside VM**
Cloud-init may still be running:
```
tail -20 /var/log/cloud-init-output.log
```
If failed, install manually:
```
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
npm install -g @mariozechner/pi-coding-agent
```

**"hermes: command not found"**
```
source ~/.bashrc
```
If still missing:
```
curl -LsSf https://astral.sh/uv/install.sh | sh && source ~/.bashrc
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
source ~/.bashrc
```

**Ollama "connection refused" from VM**
Ensure you did Part 4 Step 4.3 (listen on 0.0.0.0). Check from host:
```
curl http://localhost:11434/v1/models
```

**Model runs very slowly**
Your model is probably too large for your RAM. Run `llm-checker recommend --category coding` on your host to find a better fit, or switch to a smaller model.

**Updating agents later**
```
npm update -g @mariozechner/pi-coding-agent
hermes update
```

---

## Further Reading

- **Pi docs:** https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent
- **Hermes docs:** https://hermes-agent.nousresearch.com/docs
- **Canonical Inference Snaps:** https://documentation.ubuntu.com/inference-snaps/
- **LLM Checker:** https://github.com/Pavelevich/llm-checker
- **Ollama model library:** https://ollama.com/library
- **Hermes skills ecosystem:** https://github.com/0xNyk/awesome-hermes-agent
- **Pi extensions guide:** https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/extensions.md

---

*Guide created April 2026. Pi v0.65.x, Hermes v0.8.0.*
*Pi: https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent*
*Hermes: https://github.com/NousResearch/hermes-agent*
