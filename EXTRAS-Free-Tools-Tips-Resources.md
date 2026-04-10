# Extras: More Free AI Tools, Tips & Resources for Students

**Companion to the [AI Coding Agents Setup Guide](AI-Coding-Agents-Setup-Guide.md)**

This document covers additional free tools, IDE options, learning tips, and resources that pair well with your Pi + Hermes setup.

---

## LM Studio — The Visual Alternative to Ollama

If you're not comfortable with the terminal yet, **LM Studio** is the best place to start with local AI. It's a free desktop app that lets you browse, download, and chat with AI models through a graphical interface — no command line needed.

### Why Students Should Know About It

- **Visual model browser** — search and download models from Hugging Face with one click
- **Built-in chat UI** — talk to models like you would with ChatGPT, but running on your computer
- **Hardware auto-detection** — it detects your GPU (NVIDIA CUDA, AMD ROCm, Apple Metal) automatically
- **Parameter sliders** — adjust temperature, context length, etc. visually
- **OpenAI-compatible API** — runs a local server on port 1234 that Pi and Hermes can connect to
- **Privacy** — no data collection, no telemetry, everything stays on your machine

### Install

Download from https://lmstudio.ai — available for Linux, Windows, and macOS. No account needed.

### Using LM Studio with Pi and Hermes

LM Studio exposes an API at `http://localhost:1234/v1` that works the same way as Ollama. Configure Pi and Hermes exactly like the Ollama setup in the main guide, but use port `1234` instead of `11434`:

**Pi config** (`~/.pi/agent/models.json`):
```json
{
  "providers": {
    "lmstudio": {
      "baseUrl": "http://YOUR_IP:1234/v1",
      "api": "openai-completions",
      "apiKey": "lm-studio",
      "compat": {
        "supportsDeveloperRole": false,
        "supportsReasoningEffort": false
      },
      "models": [
        {
          "id": "YOUR_MODEL_NAME",
          "name": "Local Model (LM Studio)",
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
```

**Hermes:** Choose "Custom OpenAI-compatible endpoint" → URL: `http://YOUR_IP:1234/v1`

### Ollama vs LM Studio — When to Use Which

| | Ollama | LM Studio |
|---|--------|-----------|
| Interface | Terminal / command line | Visual GUI |
| Best for | Automation, scripting, always-on server | Exploring models, comparing outputs, learning |
| Runs as | Background service (survives reboots) | Desktop app (stops when you close it) |
| Model format | GGUF (auto-managed) | GGUF (browse from Hugging Face) |
| API port | localhost:11434 | localhost:1234 |
| Docker support | Yes (with GPU passthrough) | Preview only (CPU-only) |
| Multi-model | Switches automatically | One model at a time via server |

**Tip:** Many people use both — LM Studio to discover and test models visually, then Ollama to run the ones they like as a background service.

---

## VS Code — Adding AI to Your Editor

VS Code (Visual Studio Code) is the most popular free code editor. You can add AI capabilities directly inside it with these free extensions:

### Continue.dev (Recommended for Local Models)

The best free VS Code extension for connecting to Ollama or LM Studio.

**Install:** Open VS Code → Extensions (Ctrl+Shift+X) → Search "Continue" → Install

**Setup with Ollama:**
1. Install Continue
2. Open Continue sidebar (Ctrl+L)
3. Click the gear icon → config
4. Add Ollama as a provider:

```json
{
  "models": [
    {
      "title": "Local Ollama",
      "provider": "ollama",
      "model": "qwen3-coder"
    }
  ]
}
```

Now you have AI autocomplete, chat, and code explanations — all powered by your local model, completely free.

### GitHub Copilot Free

If you have a `.edu` email address, GitHub Copilot is **completely free** with unlimited usage.

1. Go to https://education.github.com
2. Verify your student status with your .edu email
3. Install the GitHub Copilot extension in VS Code
4. Sign in with your GitHub account

Even without a student email, the free tier gives you 2,000 completions and 50 chat messages per month.

### Cline (Open Source Agent in VS Code)

Cline turns VS Code into a full AI agent — it can create files, edit code, run terminal commands, and browse websites.

**Install:** Extensions → Search "Cline" → Install

**Free setup:** Point Cline at your local Ollama endpoint. In Cline settings, choose "Ollama" as provider and enter your model name. Zero cost.

### How VS Code Fits With Pi and Hermes

Think of it as three layers:

| Layer | Tool | What It Does |
|-------|------|-------------|
| **Editor** | VS Code + Continue/Cline | Autocomplete, inline suggestions, quick edits |
| **Coding agent** | Pi | Multi-file changes, debugging, architecture decisions |
| **Autonomous agent** | Hermes | Research, memory, long tasks, messaging |

You don't have to choose one — use all three. VS Code for typing code, Pi for complex coding tasks, Hermes for everything else.

---

## OpenRouter — The Model Buffet

OpenRouter (https://openrouter.ai) is a single API key that gives you access to 200+ models from every major provider. It's not free, but it's the best value option when you're ready to move beyond free tiers.

### Why Students Should Care

- **One key, all models** — switch between Claude, GPT, Gemini, Llama, Mistral, DeepSeek, and more
- **Budget controls** — set a hard spending limit so you never get a surprise bill
- **Some free models** — certain models on OpenRouter cost $0 (look for the "free" tag)
- **Pay-per-use** — no subscription, you only pay for what you use
- **Cheapest path to good models** — DeepSeek via OpenRouter costs ~$0.10-0.50/day with moderate use

### Setup

1. Sign up at https://openrouter.ai
2. Add $5 of credit (this lasts a surprisingly long time with cheaper models)
3. Copy your API key

**In Pi:** Select "OpenRouter" as provider during setup, paste your key.

**In Hermes:** Run `hermes model`, select OpenRouter, paste your key, pick a model.

### Smart Spending Tips

| Model | Approximate Cost | Good For |
|-------|-----------------|----------|
| DeepSeek V3 | ~$0.27 per 1M input tokens | Coding, general use (very cheap) |
| Gemini Flash | ~$0.075 per 1M input tokens | Fast tasks, high volume |
| Claude Sonnet | ~$3.00 per 1M input tokens | High-quality coding |
| GPT-5.4 | ~$2.50 per 1M input tokens | General intelligence |

**Translation:** 1 million tokens ≈ 750,000 words. A typical coding session uses 10,000-50,000 tokens. So DeepSeek at $0.27/M tokens costs roughly **$0.003 to $0.014 per session** — basically free.

---

## More Free Terminal Agents

These all work alongside Pi and Hermes.

| Tool | Free? | What It Does | Install |
|------|-------|-------------|---------|
| **Gemini CLI** | Yes (Google account) | Google's terminal agent. 60 req/min free. 1M token context. | `npm install -g @google/gemini-cli` |
| **Aider** | Yes (BYOK or Ollama) | Git-native coding agent. Every change auto-committed. | `pip install aider-chat` |
| **OpenAI Codex CLI** | Yes (OpenAI account) | OpenAI's open-source terminal agent with sandbox. | `npm install -g @openai/codex` |

---

## Free API Keys & Credits

| Provider | Free Tier | How to Get It |
|----------|-----------|---------------|
| **Google AI Studio** | Free Gemini Flash API calls (rate limited) | aistudio.google.com/apikey |
| **OpenAI** | Free credits for new accounts | platform.openai.com |
| **Anthropic** | Free credits for new accounts ($5) | console.anthropic.com |
| **OpenRouter** | Some $0 models available | openrouter.ai |
| **Nous Portal** | Free MiMo v2 Pro for auxiliary tasks | Built into Hermes setup |
| **GitHub Copilot** | Free for students (.edu email) | education.github.com |
| **Groq** | Free tier (very fast inference) | console.groq.com |

---

## Recommended Learning Path

### Week 1: Get Running
- Follow the main setup guide
- Use Google Gemini CLI (free, no hassle) to power both agents
- Learn basic tmux (split screen, switch panes)
- Ask Pi to explain code you find online
- Ask Hermes to research a topic you're studying

### Week 2: Go Local
- Install Ollama OR LM Studio on your host
- Run `llm-checker hw-detect` to see what fits
- Pull a recommended model and connect both agents
- Compare: does the local model feel different from Gemini?

### Week 3: Explore the IDE
- Install VS Code + Continue.dev, point at your local model
- Now you have autocomplete while typing + Pi for bigger tasks
- Install a few Hermes skills: `hermes skills search`
- Try Pi's session branching (`/tree`) on a real problem

### Week 4: Level Up
- Add an OpenRouter key ($5 credit) and try Claude or DeepSeek
- Set up Hermes messaging gateway (talk to it from your phone)
- Create a `.pi/` folder in a project with custom skills
- Have Hermes schedule a daily task with cron

---

## Useful Commands Cheat Sheet

### Pi Commands (inside Pi)

| Command | What It Does |
|---------|-------------|
| `/model` or Ctrl+L | Switch models/providers |
| `/tree` | Browse session history, branch, explore |
| `/login` | Authenticate with a cloud provider |
| `/new` | Start a fresh session |
| `/hotkeys` | Show all keyboard shortcuts |
| Ctrl+C | Cancel current response |
| Enter (while streaming) | Queue a steering message |
| Alt+Enter | Queue a follow-up |
| Escape | Abort current response |

### Hermes Commands (inside Hermes)

| Command | What It Does |
|---------|-------------|
| `/model` | Switch models/providers |
| `/skills` | Browse and manage skills |
| `/memory` | View agent's memories about you |
| `/reset` | Reset the conversation |
| `/stop` | Kill current agent run |
| `/exit` | Exit Hermes |
| `/browser connect` | Attach to live Chrome via CDP |
| `/rollback` | Restore filesystem checkpoint |

### Hermes CLI (from terminal)

```bash
hermes              # Start chatting
hermes -c           # Resume last session
hermes model        # Switch provider/model
hermes tools        # Configure tools
hermes skills search <query>  # Find skills
hermes gateway setup          # Set up messaging
hermes doctor       # Diagnose issues
hermes update       # Update to latest
hermes logs         # View agent logs
```

### Multipass (from host)

```bash
multipass shell agents        # Enter VM
multipass start agents        # Start VM
multipass stop agents         # Stop VM
multipass info agents         # Resource usage
multipass mount ~/dir agents:~/dir  # Share folder
multipass exec agents -- CMD  # Run command in VM
```

### tmux

```
Ctrl+B %       Split left/right
Ctrl+B "       Split top/bottom
Ctrl+B →←↑↓   Navigate panes
Ctrl+B z       Zoom/unzoom pane
Ctrl+B d       Detach (keeps running)
Ctrl+B [       Scroll mode (q to exit)
tmux attach -t work   Reattach
```

---

## Privacy & Safety Tips

1. **Local models = private.** Ollama and LM Studio keep everything on your machine. Use for sensitive projects.

2. **Free cloud tiers may train on your data.** Don't paste passwords, API keys, or proprietary code into free cloud models.

3. **Hermes and Pi sessions are local.** Stored in `~/.hermes/` and `~/.pi/agent/sessions/` inside the VM.

4. **API keys are secrets.** Never commit them to a public repo, share in screenshots, or paste in public chat.

5. **The VM is your sandbox.** If something goes wrong: `multipass delete agents && multipass purge` and start fresh.

6. **OpenRouter budget limits.** Always set a monthly spending cap at openrouter.ai/account to avoid surprises.

---

## Common "I'm Stuck" Scenarios

**"The model is bad at coding"** — Small local models (7B) are noticeably weaker than cloud models. Try a bigger model if your hardware allows, or switch to Gemini CLI for free cloud quality.

**"Pi shows a blank screen"** — No provider configured. Run `pi` and follow the setup wizard, or create `~/.pi/agent/models.json` for Ollama (see main guide Part 8B).

**"Hermes keeps using Claude instead of Ollama"** — Run `hermes model`, select your custom endpoint, confirm. Sometimes you need to do this after initial setup.

**"I hit my free Gemini limit"** — Resets daily. Switch to Ollama for unlimited local use while you wait.

**"tmux is confusing"** — Start simple: just `tmux` then run your agent. Add splits later when you're comfortable.

**"VS Code + Continue isn't giving suggestions"** — Make sure Ollama is running (`ollama list` should show your model). Check Continue config points to `localhost:11434`.

**"LM Studio won't load my model"** — Check available RAM. If the model needs 16GB and you only have 16GB total, your OS uses some of that. Try a smaller model or a more aggressive quantization (Q4 instead of Q8).

**"OpenRouter says insufficient credits"** — Add more credit at openrouter.ai/account, or switch to a free model.

---

## The Full Free Stack (Everything Together)

Here's how all the tools fit together for a student who wants maximum capability at zero cost:

```
YOUR COMPUTER
│
├── LM Studio (GUI)         ← Browse and test models visually
├── Ollama (background)     ← Serve models to all your tools
│   └── qwen3-coder         ← Your local coding model
│
├── VS Code                 ← Your code editor
│   ├── Continue.dev        ← AI autocomplete from Ollama
│   └── Cline              ← AI agent inside VS Code
│
└── Multipass VM: "agents"
    ├── Pi                  ← Terminal coding agent → connects to Ollama
    └── Hermes              ← Autonomous agent → connects to Ollama
                              (or Gemini CLI / Nous Portal for free cloud)
```

**Total cost: $0.** All tools are free. All models are free. Everything runs locally. No accounts needed (except Google for Gemini CLI option).

---

## Further Exploration

| Topic | Link |
|-------|------|
| LM Studio | lmstudio.ai |
| Continue.dev (VS Code + local AI) | continue.dev |
| Cline (VS Code agent) | github.com/cline/cline |
| OpenRouter | openrouter.ai |
| Aider (git-native agent) | aider.chat |
| Pi extensions & skills | github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/extensions.md |
| Hermes skills ecosystem | github.com/0xNyk/awesome-hermes-agent |
| Hermes messaging gateway | hermes-agent.nousresearch.com/docs |
| Ollama model library | ollama.com/library |
| Canonical Inference Snaps | documentation.ubuntu.com/inference-snaps/ |
| LLM Checker (hardware scanner) | github.com/Pavelevich/llm-checker |
| Google AI Studio (free API key) | aistudio.google.com |
| GitHub Education (free Copilot) | education.github.com |
| Groq (fast free inference) | console.groq.com |

---

*Last updated April 2026. Additions and corrections welcome via Issues or PRs.*
