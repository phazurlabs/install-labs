---
name: agent-packaging-foundations
description: "Core principles for packaging AI agents and automations as installable software. Use when the user mentions: agent packaging, packaging principles, install funnel, agent dependencies, packaging anti-patterns, why agents are hard to install, agent distribution basics, package an agent, make agent installable, agent install UX"
---

# Agent Packaging Foundations

## Why Agents Are Harder to Package Than Traditional Software

Traditional software has a single runtime, a known dependency tree, and a build artifact. AI agents break every assumption:

| Challenge | Traditional Software | AI Agents |
|---|---|---|
| **Secrets** | Maybe one DB connection string | 3-8 API keys across providers (OpenAI, Anthropic, Pinecone, etc.) |
| **Model weights** | No large binary blobs | 500MB-70GB model files that can't ship in a package |
| **Runtime environment** | One language runtime | Python + Node.js + system libs + CUDA/Metal |
| **Config sprawl** | One config file | `.env`, `config.yaml`, `plugin.json`, `mcp.json`, framework configs, all in different directories |
| **Framework churn** | Stable APIs | LangChain, CrewAI, AutoGen — breaking changes monthly |
| **Hardware variance** | CPU is CPU | GPU type, VRAM, quantization level all affect behavior |
| **State** | Database handles it | Vector stores, conversation memory, tool caches across restarts |

The result: an agent that works on the author's machine silently depends on 15 things the author forgot they installed.

---

## 8 Core Packaging Principles for Agents

### 1. One Command to Install, One Command to Uninstall

```bash
# GOOD: single entry point
npx @your-org/my-agent install
npx @your-org/my-agent uninstall

# GOOD: platform package manager
brew install my-agent && brew uninstall my-agent

# BAD: multi-step scavenger hunt
git clone ... && cd ... && pip install -r ... && cp config.example.yaml ...
```

If your install instructions have more than one step, wrap them in a script.

### 2. Secrets via Environment Variables, Never Hardcoded

```bash
# GOOD: read at runtime
OPENAI_API_KEY=sk-... my-agent start

# GOOD: .env file (gitignored)
my-agent init  # creates .env template with comments

# BAD: baked into config
# config.yaml: api_key: "sk-proj-abc123..."
```

Ship a `.env.example` with every key listed, commented, and explained. Never write a real key to disk in plaintext unless the user explicitly opts in.

### 3. Test Install on a Clean Machine

The author's machine has everything. Test on:
- A fresh macOS user account (no Homebrew, no Python)
- A clean Ubuntu container (`docker run -it ubuntu:24.04 bash`)
- A Windows machine with nothing but the OS

If you skip this, your first 50 users will file the same bug.

### 4. Progressive Disclosure

Show the simplest install method first. Collapse alternatives.

```markdown
## Install
npm install -g my-agent

<details>
<summary>Alternative: install from source</summary>
git clone ...
</details>

<details>
<summary>Alternative: Docker</summary>
docker run ...
</details>
```

### 5. Health Checks and Actionable Error Messages

Every agent should have a `doctor` or `health` command:

```bash
$ my-agent doctor
[PASS] Node.js 20.11.0
[PASS] Python 3.12.1
[FAIL] ANTHROPIC_API_KEY not set
       Fix: export ANTHROPIC_API_KEY=your-key-here
       Get a key: https://console.anthropic.com/keys
[FAIL] chromadb not reachable at localhost:8000
       Fix: docker run -d -p 8000:8000 chromadb/chroma
```

Never print a stack trace without a human-readable sentence above it.

### 6. Lazy Authentication

Don't ask for API keys during install. Ask on first use.

```
$ my-agent install        # no keys needed
$ my-agent run "do thing"
> This task requires an Anthropic API key.
> Get one at: https://console.anthropic.com/keys
> Enter your key (or set ANTHROPIC_API_KEY): _
```

Why: users abandon installs that demand credentials upfront. Let them see the tool first.

### 7. Convention Over Configuration

Ship sensible defaults. Only ask the user to configure what they must.

```yaml
# BAD: 40-line config.yaml the user must edit before first run

# GOOD: zero config needed, everything has defaults
# config.yaml (optional overrides)
# model: claude-sonnet-4-20250514  # default: claude-sonnet-4-20250514
# max_tokens: 4096          # default: 4096
# log_level: info            # default: info
```

### 8. Detect, Never Ask

Auto-detect everything you can:

```bash
# Detect OS and architecture
OS=$(uname -s)    # Darwin, Linux, MINGW...
ARCH=$(uname -m)  # x86_64, arm64...

# Detect shell
SHELL_NAME=$(basename "$SHELL")  # bash, zsh, fish

# Detect package manager
if command -v brew &>/dev/null; then ...
elif command -v apt &>/dev/null; then ...

# Detect GPU
if command -v nvidia-smi &>/dev/null; then ...  # NVIDIA
if system_profiler SPDisplaysDataType | grep -q "Metal"; then ...  # Apple Silicon
```

Never ask "What OS are you on?" or "Do you have a GPU?"

---

## Agent Dependency Taxonomy

Classify every dependency your agent needs:

| Type | Examples | Install Strategy |
|---|---|---|
| **API-only** | OpenAI, Anthropic, Tavily | No install; just need keys at runtime |
| **Model weights** | GGUF files, LoRA adapters, embeddings | Download on first use, verify checksum, cache locally |
| **Runtime** | Python 3.11+, Node.js 20+, Rust | Check version, link to installer, or bundle |
| **Framework** | LangChain, CrewAI, Haystack, DSPy | Pin exact version in requirements; these break often |
| **Infrastructure** | Redis, Postgres, ChromaDB, Qdrant | Offer Docker one-liner or cloud-hosted alternative |

**Rule of thumb:** API-only deps are free (just keys). Every other category adds install friction. Minimize the non-API categories ruthlessly.

---

## The Agent Install Funnel

```
Discovery          100 users find your agent
    |
Install attempt     60 users try to install (40% bounce from bad README)
    |
Configuration       35 users finish install (25 hit dependency errors)
    |
First successful    20 users get it working (15 stuck on config/keys)
    run
    |
Retention           12 users keep using it (8 hit bugs on second use)
```

Where agents lose people:
1. **Discovery to attempt** — README doesn't show what the agent does in 10 seconds
2. **Attempt to configured** — Missing system dependency with cryptic error
3. **Configured to first run** — API key rejected, no useful error message
4. **First run to retention** — Agent works once but state is lost on restart

Optimize the narrowest part of your funnel first.

---

## Common Packaging Anti-Patterns for Agents

### 1. The "Works on My Machine" README
```markdown
# BAD
1. Clone the repo
2. Install dependencies
3. Set up your environment
4. Run the agent
```
The author has Python 3.11, six API keys exported, CUDA 12.1, and a running ChromaDB. None of this is mentioned.

### 2. The Config File Avalanche
Requiring the user to create or edit 3+ config files before first run. Merge them or generate them.

### 3. Secrets in the Repo
Shipping a `config.yaml` with `api_key: "REPLACE_ME"`. Users will commit their real key. Use `.env` + `.gitignore` instead.

### 4. The Eager Downloader
Downloading 4GB of model weights during `pip install` before the user knows if the agent even works. Download on first use, not on install.

### 5. The Framework Lock-In
Requiring a specific LangChain version that conflicts with the user's other agents. Use virtual environments or containers.

### 6. The Silent Failure
Agent installs successfully, runs without error, but silently does nothing because a vector store isn't connected. Always validate the full chain on startup.

### 7. No Uninstall Path
User can't figure out what was installed or where. Always provide `uninstall` or document exactly what files/dirs were created.

### 8. Root/Sudo Install
Requiring `sudo pip install` or `sudo npm install -g`. Install to user space. If you need system access, explain exactly why.

---

## Sources & References

1. **[The Twelve-Factor App]** — Adam Wiggins / Heroku. https://12factor.net/. Methodology for building SaaS apps; Factor III (Config) and Factor V (Build, Release, Run) directly inform agent packaging principles around environment variables and reproducible builds.
2. **[Cognitive Load Theory]** — Sweller, J. (1988). "Cognitive Load During Problem Solving: Effects on Learning." *Cognitive Science*, 12(2), 257-285. Foundational research explaining why simpler install processes succeed: reducing extraneous cognitive load increases completion rates.
3. **[Fogg Behavior Model]** — BJ Fogg, Stanford Behavior Design Lab. https://behaviormodel.org/. The B=MAP framework (Behavior = Motivation x Ability x Prompt) explains why install funnels fail when any factor is missing — high motivation cannot compensate for low ability (complex installs).
4. **[Hick's Law]** — Hick, W.E. (1952). "On the Rate of Gain of Information." *Quarterly Journal of Experimental Psychology*, 4(1), 11-26. Decision time increases logarithmically with the number of choices — directly applicable to why progressive disclosure of install options (one primary, alternatives collapsed) improves conversion.
5. **[10 Usability Heuristics for User Interface Design]** — Jakob Nielsen, Nielsen Norman Group. https://www.nngroup.com/articles/ten-usability-heuristics/. Heuristic #5 (Error Prevention) and #6 (Recognition Rather Than Recall) apply directly to health check commands and convention-over-configuration principles.
6. **[Don't Make Me Think, Revisited]** — Steve Krug (2014). New Riders Publishing. The simplicity principle — "every question the user has to answer adds friction" — is the foundation for one-command installs, lazy authentication, and detect-never-ask patterns.
