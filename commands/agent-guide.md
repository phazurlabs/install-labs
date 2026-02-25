---
description: "Orientation and routing for AI agent packaging — assess what you built, see the full journey, get a personalized roadmap"
phase: "0"
phase_step: "0.1"
phase_name: "ASSESS"
step_label: "Orientation"
---

# /agent-guide — Orientation & Routing

## Purpose

The entry point for turning any AI agent or automation into an installable package. Assess what the user built, show them the full journey, and route them to the right next step.

---

## Protocol

When the user invokes `/agent-guide`, follow this process exactly:

### Step 1: Assess What the User Built

Determine the agent's type, framework, target audience, and infrastructure requirements. Ask the user directly or infer from context they have already provided.

**Gather these five dimensions:**

| Dimension | Question | Examples |
|-----------|----------|----------|
| **Framework / Platform** | What did you build it with? | LangChain, CrewAI, AutoGen, LangGraph, DSPy, Haystack, custom Python, custom Node.js/TypeScript, n8n, Flowise, Dify, ComfyUI, Make/Zapier, browser automation (Playwright/Puppeteer), shell scripts |
| **Agent Behavior** | What does the agent do? | Conversational assistant, tool-using agent, multi-agent system, workflow automation, RAG pipeline, image/video generation, data extraction, code generation, research agent, monitoring/alerting |
| **Target Audience** | Who will install and use it? | Developers comfortable with CLI, non-technical business users, enterprise IT teams, other AI agents (agent-to-agent), open source community |
| **Runtime Model** | How should it run? | Locally on the user's machine, cloud-hosted (SaaS), hybrid (local agent + cloud APIs), edge device, ephemeral (runs once and exits) |
| **Dependencies** | What does it need to function? | API keys only, local model weights (specify size), database (Postgres, SQLite), vector store (ChromaDB, Pinecone, Qdrant), GPU (CUDA/Metal), system tools (ffmpeg, Chrome, etc.) |

**Inference rules:**
- If the user says "I built a CrewAI agent that researches companies," infer: Framework = CrewAI (Python), Behavior = research agent (tool-using, multi-agent), Dependencies = likely API keys + possibly a search API. Ask only what you cannot infer.
- If the user points to a GitHub repo or local directory, scan the codebase to identify the framework (look for `requirements.txt`, `package.json`, `pyproject.toml`, `Dockerfile`, `crew.yaml`, `langgraph.json`, `n8n` workflow exports, etc.).
- If the user says "I have an MCP server," they may already know their distribution target. Confirm and skip to `/pick-target` or `/gen-package`.

**Do not ask all five questions in a numbered list.** Ask conversationally. If the user provides a rich description, acknowledge what you inferred and ask only about gaps.

### Step 2: Present the Install Labs Agent Packaging Journey

After gathering context, show the full 10-command journey so the user understands the scope. Present this map exactly:

```
┌──────────────────────────────────────────────────────────────────────┐
│                    INSTALL LABS: AGENT → INSTALL                    │
│                    10 Commands · 3 Phases                           │
│                                                                     │
│  ── PHASE 0: ASSESS ──────────────────────────────────────────────  │
│     Figure out what you have and where it should go                 │
│                                                                     │
│     /agent-guide  (0.1)  Orientation & routing      [You are here] │
│     /agent-audit  (0.2)  Packaging readiness audit                 │
│                                                                     │
│  ── PHASE 1: PACKAGE ─────────────────────────────────────────────  │
│     Choose a distribution target and generate packaging files       │
│                                                                     │
│     /pick-target    (1.1)  Choose: MCP, plugin, GPT, Docker, etc. │
│     /pick-framework (1.2)  Framework-specific packaging guide      │
│     /gen-package    (1.3)  Generate all packaging files            │
│                                                                     │
│  ── PHASE 2: SHIP ────────────────────────────────────────────────  │
│     Validate, secure, and release                                   │
│                                                                     │
│     /gen-install  (2.1)  Generate install script/instructions      │
│     /pre-ship     (2.2)  Agent-specific pre-ship checklist         │
│     /secure       (2.3)  Agent security audit                      │
│                                                                     │
│  ── STANDALONE ────────────────────────────────────────────────────  │
│     /agent-roast         Critique any agent's install experience   │
│     /glossary            Agent packaging terminology               │
│                                                                     │
│  ── FLOW ──────────────────────────────────────────────────────────  │
│     ASSESS ──────► PACKAGE ──────► SHIP                            │
│     What do you    Turn it into    Validate &                      │
│     have?          an installable  release                         │
│     (2 commands)   (3 commands)    (3 commands) + 2 standalone     │
└──────────────────────────────────────────────────────────────────────┘
```

### Step 3: Generate Personalized Roadmap

Based on the five dimensions gathered in Step 1, generate a tailored roadmap. The roadmap has three sections:

**A. Recommended Distribution Target**

Match the user's context to the best-fit distribution format:

| User Context | Recommended Target | Rationale |
|---|---|---|
| Tool-using agent, developer audience | **MCP server** | Native integration with Claude Desktop, Claude Code, Cursor, Windsurf. Widest AI-host reach. |
| Domain knowledge / workflow protocols | **Claude Code plugin** | Skills for auto-invoked knowledge, commands for structured workflows. |
| Conversational agent, non-technical users | **Desktop app** (Electron/Tauri) or **web app** | GUI required for non-technical users. |
| Complex deps, GPU, multi-container | **Docker / Docker Compose** | Isolates dependencies, reproducible environment. |
| Automation workflow (n8n, Make, Zapier) | **Template / marketplace listing** | Distribute as an importable workflow template on the platform's marketplace. |
| General-purpose CLI tool | **npm/PyPI package** | Standard package manager distribution. |
| Multi-agent system, orchestration | **Docker Compose + MCP** | Container per agent, MCP for host integration. |
| Custom GPT action | **OpenAPI endpoint** | GPTs consume REST APIs via OpenAPI spec. |

If multiple targets apply, list them ranked with the primary recommendation first.

**B. Required Steps**

List the commands from the journey that apply to this user. For each, write one sentence explaining why it is relevant to their specific agent.

Example:
```
Your roadmap (CrewAI research agent → MCP server):

  1. /agent-audit  (0.2) — Check if your CrewAI project structure is ready for packaging
  2. /pick-target  (1.1) — Confirm MCP server as your distribution target
  3. /pick-framework (1.2) — Get CrewAI-specific packaging guidance (crew.yaml → MCP bridge)
  4. /gen-package  (1.3) — Generate package.json, tsconfig, MCP server entry point
  5. /gen-install  (2.1) — Generate the npx one-liner and README install block
  6. /pre-ship     (2.2) — Run the agent-specific pre-ship checklist
  7. /secure       (2.3) — Audit API key handling and dependency pinning
```

**C. Skip List**

Identify commands that do not apply and explain why. This saves the user time.

Example:
```
You can skip:
  - /pick-target — You already know you want an MCP server
  - /glossary — You are a developer comfortable with packaging terminology
```

### Step 4: Route to the Right Next Command

Based on the user's readiness, recommend exactly one next step:

| User State | Recommended Next Command | Reason |
|---|---|---|
| Has a working agent, unsure if code is ready | `/agent-audit` | Check packaging readiness before committing to a target |
| Has a working agent, knows the target | `/pick-target` | Confirm target and get distribution-specific guidance |
| Knows the target, needs framework-specific help | `/pick-framework` | Framework-to-target bridge (e.g., CrewAI → MCP) |
| Agent is clean, deps are declared, ready to go | `/gen-package` | Skip assessment, jump straight to file generation |
| Already packaged, wants a critique | `/agent-roast` | External review of install experience |
| Just exploring, not ready to commit | Share the journey map and let the user choose | No pressure routing |

End with a clear call to action:

> "Your agent looks ready for a packaging audit. Run `/agent-audit` next and I will check your project across 6 dimensions before we start generating files."

Or:

> "You already know you want an MCP server and your code is well-structured. Skip the audit and go straight to `/pick-target` to lock in your distribution format."

---

## Output Format

Structure the response as:

1. **Assessment Summary** — 3-5 sentence summary of what the user built, using their own terminology
2. **Journey Map** — The ASCII box from Step 2
3. **Your Roadmap** — Numbered list of applicable commands with one-line rationale each
4. **Skip List** — Commands that do not apply (if any)
5. **Next Step** — Bold recommendation of exactly one command to run next, with a sentence explaining why

---

## Cross-References

- **Skills:** `agent-packaging-foundations` (core packaging principles), `mcp-server-packaging` (if MCP is the target), `claude-code-plugin-packaging` (if plugin is the target)
- **Commands:** `/agent-audit` (readiness check), `/pick-target` (distribution target selection), `/pick-framework` (framework-specific guidance)

---

## Edge Cases

- **User has no working agent yet** — Acknowledge they are early. Suggest building a minimal working version first (one tool, one happy path) before packaging. Point them to framework documentation rather than packaging commands.
- **User has multiple agents** — Ask which one to package first. Recommend starting with the simplest one to learn the packaging workflow, then applying it to others.
- **User's agent is a no-code workflow** (n8n, Make, Zapier) — These have their own export/template formats. Redirect to the platform's native distribution mechanism. Install Labs can still help with the install script and README around the exported template.
- **User already packaged and is iterating** — Skip the orientation. Ask what specifically they want to improve and route directly to the relevant command.
- **User says "just do it"** — If enough context exists from the conversation, infer all five dimensions, state your assumptions, and proceed to the roadmap without asking more questions. Confirm assumptions at the end: "I assumed X, Y, Z. Correct?"
