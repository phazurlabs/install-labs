# Install Labs v2.0.0

**Turn your AI agents and automations into installable plugins and packages.**

You built an agent. Now make it so other people can install and use it.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-Plugin-blueviolet)](https://docs.anthropic.com/en/docs/claude-code)
[![Version](https://img.shields.io/badge/version-2.0.0-green.svg)]()

---

## Table of Contents

- [What This Plugin Does](#what-this-plugin-does)
- [The Problem](#the-problem)
- [The Journey](#the-journey)
- [Commands (10)](#commands-10)
- [Skills (12)](#skills-12)
- [Frameworks Covered (17)](#frameworks-covered-17)
- [Distribution Targets (10)](#distribution-targets-10)
- [Academic Foundations](#academic-foundations)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Architecture](#architecture)
- [References](#references)
- [License](#license)

---

## What This Plugin Does

Install Labs is a Claude Code plugin that guides you through packaging any AI agent or automation -- built with any framework -- into an installable product. It covers every major distribution target: MCP servers, Claude Code plugins, Custom GPTs, Docker containers, PyPI packages, npm packages, and cloud deployments.

**One mission:** "I built an AI agent. Help me turn it into something other people can install and use."

---

## The Problem

AI agents are harder to package than traditional software. Traditional software has a single runtime, a known dependency tree, and a build artifact. AI agents break every assumption:

| Challenge | Traditional Software | AI Agents |
|---|---|---|
| **Secrets** | Maybe one DB connection string | 3-8 API keys across providers |
| **Model weights** | No large binary blobs | 500MB-70GB model files |
| **Runtime environment** | One language runtime | Python + Node.js + system libs + CUDA/Metal |
| **Config sprawl** | One config file | `.env`, `config.yaml`, `plugin.json`, `mcp.json`, framework configs |
| **Framework churn** | Stable APIs | LangChain, CrewAI, AutoGen -- breaking changes monthly |
| **Hardware variance** | CPU is CPU | GPU type, VRAM, quantization all affect behavior |
| **State** | Database handles it | Vector stores, conversation memory, tool caches |

The result: an agent that works on the author's machine silently depends on 15 things the author forgot they installed.

Research in cognitive load theory (Sweller, 1988) shows that every additional install step increases the probability of user abandonment. The Fogg Behavior Model (Fogg, 2009) predicts that install completion requires the intersection of motivation (wanting the agent), ability (low friction install), and trigger (clear call to action). When any of these three factors is insufficient, the user drops off. Install Labs applies these principles systematically to every distribution target.

---

## The Journey

```
+---------------------------------------------------------------------------+
|                    INSTALL LABS: AGENT -> INSTALL                          |
|                    10 Commands . 3 Phases . 12 Skills                     |
|                                                                           |
|  -- PHASE 0: ASSESS ---------------------------------------------------- |
|     Figure out what you have and where it should go                       |
|                                                                           |
|     /agent-guide  (0.1)  Orientation & routing                           |
|     /agent-audit  (0.2)  Packaging readiness audit                       |
|                                                                           |
|  -- PHASE 1: PACKAGE --------------------------------------------------- |
|     Choose a distribution target and generate packaging files             |
|                                                                           |
|     /pick-target    (1.1)  Choose: MCP, plugin, GPT, Docker, etc.       |
|     /pick-framework (1.2)  Framework-specific packaging guide            |
|     /gen-package    (1.3)  Generate all packaging files                  |
|                                                                           |
|  -- PHASE 2: SHIP ------------------------------------------------------ |
|     Validate, secure, and release                                         |
|                                                                           |
|     /gen-install  (2.1)  Generate install script/instructions            |
|     /pre-ship     (2.2)  Agent-specific pre-ship checklist               |
|     /secure       (2.3)  Agent security audit                            |
|                                                                           |
|  -- STANDALONE ---------------------------------------------------------- |
|     /agent-roast         Critique any agent's install experience         |
|     /glossary            Agent packaging terminology                     |
|                                                                           |
|  -- FLOW ---------------------------------------------------------------- |
|     ASSESS ---------> PACKAGE ---------> SHIP                            |
|     What do you       Turn it into       Validate &                      |
|     have?             an installable     release                         |
|     (2 commands)      (3 commands)       (3 commands) + 2 standalone     |
+---------------------------------------------------------------------------+
```

---

## Commands (10)

### Phase 0: ASSESS
| Command | Step | What It Does |
|---------|------|-------------|
| `/agent-guide` | 0.1 | Orientation -- identify your agent, get routed to the right path |
| `/agent-audit` | 0.2 | Audit your agent's packaging readiness across 6 dimensions (30-point scoring) |

### Phase 1: PACKAGE
| Command | Step | What It Does |
|---------|------|-------------|
| `/pick-target` | 1.1 | Interactive decision tree for choosing distribution target: MCP, plugin, GPT, Docker, PyPI, npm, cloud |
| `/pick-framework` | 1.2 | Framework-specific packaging guide for your stack (17 frameworks x 8 targets) |
| `/gen-package` | 1.3 | Generate all packaging files -- pyproject.toml, package.json, Dockerfile, plugin.json, etc. |

### Phase 2: SHIP
| Command | Step | What It Does |
|---------|------|-------------|
| `/gen-install` | 2.1 | Generate production install.sh/install.ps1 scripts and README install sections |
| `/pre-ship` | 2.2 | Run 39-item pre-ship quality gate across 7 sections with pass/fail verdicts |
| `/secure` | 2.3 | 10-point agent security audit (API keys, supply chain, permissions, model integrity) |

### Standalone
| Command | What It Does |
|---------|-------------|
| `/agent-roast` | Critique any agent's install experience across 8 dimensions with letter grade (A-F) |
| `/glossary` | Plain-language definitions of 40+ agent packaging terms with code examples |

---

## Skills (12)

Skills are auto-invoked when the conversation matches their domain. Each skill contains production-ready code examples, decision frameworks, and citations to official documentation.

| Skill | Lines | Coverage |
|-------|-------|----------|
| **agent-packaging-foundations** | 229 | Core packaging principles, dependency taxonomy, install funnel, 8 anti-patterns |
| **mcp-server-packaging** | 640 | MCP specification, 6 distribution methods, config injection, registries, testing |
| **claude-code-plugin-packaging** | 590 | Plugin architecture, skills, commands, CLAUDE.md, distribution methods |
| **custom-gpt-packaging** | 428 | GPT Builder, Actions (OpenAPI 3.1), knowledge files, GPT Store publishing |
| **docker-agent-packaging** | 804 | Multi-stage builds, GPU support, one-click deploy, 5 cloud platforms |
| **pypi-agent-packaging** | 837 | pyproject.toml, pip/uvx, lazy imports, Trusted Publishers, CI/CD |
| **npm-agent-packaging** | 614 | package.json, npx zero-install, TypeScript, npm provenance, bundling |
| **framework-packaging-guides** | 598 | 17 frameworks with project structure, key configs, pitfalls, decision matrix |
| **agent-security** | 349 | API key management, model integrity, MCP permissions, 7 agent-specific threats |
| **agent-update-distribution** | 309 | Version management, migration patterns, rollback, auto-update |
| **agent-install-templates** | 1,145 | 10 production-ready templates (MCP, plugin, Docker, PyPI, npm, install script) |
| **agent-pre-ship-checklist** | 292 | 39-item quality gate, 7 sections, CRITICAL/RECOMMENDED severity levels |

**Total: ~6,835 lines of skill content.**

---

## Frameworks Covered (17)

| Category | Frameworks |
|----------|-----------|
| **Python Agent Frameworks** | LangChain / LangGraph, CrewAI, AutoGen / AG2, Pydantic AI, LlamaIndex, Smolagents |
| **Multi-Language SDKs** | Semantic Kernel (Python + C#), Vercel AI SDK (TypeScript), OpenAI Assistants API, Anthropic Tool Use |
| **No-Code / Low-Code** | n8n, Flowise, Dify, ComfyUI |
| **Browser Automation** | Playwright, Puppeteer |
| **Custom** | Custom Python, Custom Node.js/TypeScript |

Each framework includes: project structure, key config files, best distribution target, install command, and framework-specific pitfalls.

---

## Distribution Targets (10)

| Target | Install Command | Best For |
|--------|----------------|----------|
| **MCP Server (npm)** | `npx -y @org/server` | Tool-using agents for Claude/Cursor/Windsurf |
| **MCP Server (PyPI)** | `uvx server-name` | Python tool agents |
| **Claude Code Plugin** | `git clone ... ~/.claude/plugins/` | Domain knowledge + workflow protocols |
| **Custom GPT** | Zero install (GPT Store) | Non-technical users, conversational agents |
| **Docker** | `docker compose up` | Complex deps, GPU, multi-agent systems |
| **PyPI** | `pip install agent` / `uvx agent` | Python CLI agents |
| **npm** | `npm install -g agent` / `npx agent` | Node.js CLI agents |
| **HuggingFace Space** | Zero install (web) | ML demos, Smolagents |
| **Replicate** | `replicate.run("org/model")` | GPU inference APIs |
| **Cloud (Railway/Render/Cloud Run)** | Deploy button | SaaS agents |

---

## Academic Foundations

Install Labs is grounded in established research from cognitive science, human-computer interaction, and software engineering:

### Cognitive Science

- **Cognitive Load Theory** (Sweller, 1988) -- Every additional install step imposes extraneous cognitive load. Install Labs minimizes steps-to-working-agent across all distribution targets by applying Sweller's three-load model: intrinsic (task complexity), extraneous (bad UX), germane (schema building).

- **Hick's Law** (Hick, 1952) -- Decision time increases logarithmically with the number of choices. The `/pick-target` decision tree reduces choice overload by filtering distribution targets based on agent characteristics rather than presenting all options simultaneously.

- **Fogg Behavior Model** (Fogg, 2009) -- Behavior = Motivation x Ability x Trigger. Install completion requires all three: motivation (wanting the agent), ability (low-friction install), and trigger (clear "install now" call to action). Install Labs optimizes the ability dimension.

- **Peak-End Rule** (Kahneman et al., 1993) -- Users judge experiences by their peak moment and final moment. Install Labs designs the "first successful run" as the peak and the verification command as a satisfying end.

### Software Engineering

- **The Twelve-Factor App** (Wiggins, 2011) -- Factor III (Config) and Factor V (Build/Release/Run) directly inform agent packaging. Config via environment variables, strict separation of build and run stages.

- **Semantic Versioning** (Preston-Werner, 2013) -- SemVer 2.0.0 governs version management across all distribution targets, with agent-specific definitions for major/minor/patch boundaries.

- **OWASP Top 10** (OWASP Foundation) -- Security audit protocols are grounded in OWASP's risk categories, adapted for AI agent-specific threat vectors (prompt injection via config, malicious model files, tool abuse escalation).

- **SLSA Framework** (OpenSSF) -- Supply chain security levels inform the `/secure` command's dependency pinning, provenance, and artifact signing recommendations.

### Human-Computer Interaction

- **Nielsen's 10 Usability Heuristics** (Nielsen, 1994) -- Error prevention (validate before shipping), recognition over recall (health check commands), and help users recognize/recover from errors (actionable error messages).

- **Don't Make Me Think** (Krug, 2014) -- The simplicity principle applied to install flows: if the user has to think about how to install, the install is broken.

- **The Design of Everyday Things** (Norman, 2013) -- Mental models and affordances in software installation: the install command should do what the user expects, not what the developer assumes.

Full bibliography available in [REFERENCES.md](REFERENCES.md).

---

## Quick Start

```bash
# Get oriented
/agent-guide

# Choose your distribution target
/pick-target

# Generate all packaging files
/gen-package

# Validate before shipping
/pre-ship
```

Or jump straight to what you need:

- **"I built a CrewAI agent, how do I distribute it?"** -> `/pick-framework`
- **"Help me package my MCP server"** -> `/pick-target`
- **"Roast my agent's install experience"** -> `/agent-roast`
- **"What's an MCP server?"** -> `/glossary`

---

## Installation

```bash
git clone https://github.com/phazurlabs/install-labs.git ~/.claude/plugins/install-labs
```

Then restart Claude Code.

### Verify Installation

After restarting Claude Code, type `/agent-guide` in a conversation. If the plugin loaded correctly, you will see the orientation protocol activate.

---

## Architecture

```
~/.claude/plugins/install-labs/
|-- .claude-plugin/
|   |-- plugin.json          # Plugin metadata (name, version, description)
|   +-- marketplace.json     # Marketplace listing metadata
|-- commands/                 # 10 slash commands (user-invocable)
|   |-- agent-guide.md       # Phase 0.1: Orientation
|   |-- agent-audit.md       # Phase 0.2: Readiness audit
|   |-- pick-target.md       # Phase 1.1: Distribution target
|   |-- pick-framework.md    # Phase 1.2: Framework guide
|   |-- gen-package.md       # Phase 1.3: File generation
|   |-- gen-install.md       # Phase 2.1: Install script
|   |-- pre-ship.md          # Phase 2.2: Quality gate
|   |-- secure.md            # Phase 2.3: Security audit
|   |-- agent-roast.md       # Standalone: Critique
|   +-- glossary.md          # Standalone: Terminology
|-- skills/                   # 12 auto-invoked knowledge skills
|   |-- agent-packaging-foundations/SKILL.md
|   |-- mcp-server-packaging/SKILL.md
|   |-- claude-code-plugin-packaging/SKILL.md
|   |-- custom-gpt-packaging/SKILL.md
|   |-- docker-agent-packaging/SKILL.md
|   |-- pypi-agent-packaging/SKILL.md
|   |-- npm-agent-packaging/SKILL.md
|   |-- framework-packaging-guides/SKILL.md
|   |-- agent-security/SKILL.md
|   |-- agent-update-distribution/SKILL.md
|   |-- agent-install-templates/SKILL.md
|   +-- agent-pre-ship-checklist/SKILL.md
|-- REFERENCES.md             # Comprehensive bibliography
|-- LICENSE                   # MIT License
+-- README.md                 # This file
```

### How It Works

**Skills** are loaded automatically by Claude Code's plugin system. When a conversation mentions keywords like "MCP server," "package my agent," or "Docker container," the relevant skill is auto-invoked, providing Claude with domain-specific knowledge for that topic.

**Commands** are structured protocols triggered by the user typing a slash command (e.g., `/pre-ship`). Each command follows a step-by-step protocol that guides the user through a specific task -- assessing readiness, choosing a target, generating files, or validating before release.

The plugin contains no executable code. It is pure structured knowledge -- 12 SKILL.md files and 10 command .md files totaling ~10,000 lines of production-ready guidance, templates, and decision frameworks.

---

## Uninstall

```bash
rm -rf ~/.claude/plugins/install-labs
```

Then restart Claude Code.

---

## References

See [REFERENCES.md](REFERENCES.md) for the complete annotated bibliography covering:

- Model Context Protocol specification and SDKs
- Python and Node.js packaging standards (PEP 517/518/621, npm docs)
- Docker best practices and cloud deployment platforms
- AI agent framework documentation (17 frameworks)
- Security standards (OWASP, SLSA, NIST, Sigstore)
- Cognitive science research (Sweller, Hick, Fogg, Kahneman, Nielsen, Norman, Krug)
- Software engineering principles (Twelve-Factor App, Semantic Versioning)

---

## License

[MIT](LICENSE) -- phazurlabs

---

**phazurlabs** | v2.0.0
