---
name: agent-install-templates
description: "Production-ready templates for packaging AI agents as installable software. Use when the user mentions: template, scaffold, boilerplate, starter, skeleton, package.json, pyproject.toml, Dockerfile, docker-compose, MCP server template, Claude Code plugin template, LangGraph deploy, install script, release action, GitHub Actions, agent packaging template, generate template, gen-template, cookiecutter, project structure"
---

# AI Agent Install Templates

Production-ready templates for packaging AI agents and automations as installable software. Every template is complete, annotated, and ready to copy into a project.

---

## 1. MCP Server (Node.js / TypeScript)

### package.json

```json
{
  "name": "@my-org/my-agent-mcp",
  "version": "0.1.0",
  "description": "MCP server that exposes my-agent capabilities as tools",
  "license": "MIT",
  "author": "my-org",
  "type": "module",
  "bin": {
    "my-agent-mcp": "./dist/index.js"
  },
  "main": "./dist/index.js",
  "files": [
    "dist"
  ],
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "start": "node dist/index.js",
    "lint": "eslint src/",
    "prepublishOnly": "npm run build"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.12.0",
    "zod": "^3.23.0"
  },
  "devDependencies": {
    "@types/node": "^22.0.0",
    "typescript": "^5.7.0",
    "eslint": "^9.0.0"
  },
  "engines": {
    "node": ">=18.0.0"
  },
  "keywords": [
    "mcp",
    "model-context-protocol",
    "ai-agent"
  ]
}
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "declaration": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### src/index.ts

```typescript
#!/usr/bin/env node

// MCP Server skeleton — registers tools that an LLM client can invoke.
// The MCP SDK handles stdio transport, JSON-RPC framing, and capability negotiation.

import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

// --- Server instance ---------------------------------------------------
const server = new McpServer({
  name: "my-agent-mcp",       // Appears in client UI (Claude Desktop, etc.)
  version: "0.1.0",           // Keep in sync with package.json
});

// --- Tool: greet -------------------------------------------------------
// Replace this with your agent's real capability.
// Each tool gets a name, description (shown to the LLM), input schema, and handler.
server.tool(
  "greet",                                        // Tool name (lowercase, hyphen-separated)
  "Generate a greeting for the given name",       // LLM-facing description
  {
    name: z.string().describe("Name to greet"),   // Zod schema = JSON Schema for the LLM
  },
  async ({ name }) => {
    // Your agent logic goes here.
    // Return content as an array of text/image/resource blocks.
    return {
      content: [
        { type: "text", text: `Hello, ${name}! Welcome to my-agent.` },
      ],
    };
  }
);

// --- Tool: analyze (example with structured output) --------------------
server.tool(
  "analyze",
  "Analyze the provided text and return key insights",
  {
    text: z.string().describe("Text to analyze"),
    depth: z.enum(["quick", "thorough"]).default("quick").describe("Analysis depth"),
  },
  async ({ text, depth }) => {
    // Replace with your real analysis logic.
    const wordCount = text.split(/\s+/).length;
    const result = {
      wordCount,
      depth,
      summary: `Analyzed ${wordCount} words at ${depth} depth.`,
    };
    return {
      content: [
        { type: "text", text: JSON.stringify(result, null, 2) },
      ],
    };
  }
);

// --- Start server ------------------------------------------------------
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  // Server is now listening on stdin/stdout. The MCP client drives the conversation.
}

main().catch((error) => {
  console.error("Fatal:", error);
  process.exit(1);
});
```

### .github/workflows/publish.yml

```yaml
# Publishes to npm on every GitHub Release (tag v*)
name: Publish to npm

on:
  release:
    types: [published]

permissions:
  contents: read
  id-token: write          # Required for npm provenance

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "22"
          registry-url: "https://registry.npmjs.org"

      - run: npm ci
      - run: npm run build
      - run: npm run lint

      # --provenance attaches a build attestation so users can verify origin
      - run: npm publish --provenance --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

**Customize:** Change `@my-org/my-agent-mcp` to your package name. Replace the `greet` and `analyze` tools with your agent's real capabilities. Update `version`, `description`, `author`, and `keywords`.

---

## 2. MCP Server (Python)

### pyproject.toml

```toml
[project]
name = "my-agent-mcp"
version = "0.1.0"
description = "MCP server exposing my-agent capabilities as tools"
readme = "README.md"
license = { text = "MIT" }
requires-python = ">=3.11"
authors = [{ name = "my-org" }]
keywords = ["mcp", "model-context-protocol", "ai-agent"]

dependencies = [
    "mcp[cli]>=1.9.0",
    "httpx>=0.27.0",
]

[project.scripts]
# Entry point for `uvx my-agent-mcp` or `pip install && my-agent-mcp`
my-agent-mcp = "my_agent_mcp.server:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/my_agent_mcp"]
```

### src/my_agent_mcp/server.py

```python
"""
MCP Server for my-agent.

Run directly:   python -m my_agent_mcp.server
Run via uvx:    uvx my-agent-mcp
Run via mcp:    mcp run my-agent-mcp
"""

from mcp.server.fastmcp import FastMCP

# --- Server instance ---------------------------------------------------
# The name appears in client UI (Claude Desktop, etc.)
server = FastMCP("my-agent-mcp")


# --- Tool: greet -------------------------------------------------------
# Replace with your agent's real capability.
# The docstring becomes the LLM-facing description.
# Type hints become the JSON Schema for the LLM.
@server.tool()
def greet(name: str) -> str:
    """Generate a greeting for the given name."""
    return f"Hello, {name}! Welcome to my-agent."


# --- Tool: analyze -----------------------------------------------------
@server.tool()
def analyze(text: str, depth: str = "quick") -> str:
    """Analyze the provided text and return key insights.

    Args:
        text: Text to analyze.
        depth: Analysis depth — "quick" or "thorough".
    """
    word_count = len(text.split())
    return f"Analyzed {word_count} words at {depth} depth."


# --- Entrypoint --------------------------------------------------------
def main():
    server.run(transport="stdio")


if __name__ == "__main__":
    main()
```

### src/my_agent_mcp/__init__.py

```python
"""my-agent MCP server package."""
```

**Customize:** Change `my-agent-mcp` and `my_agent_mcp` to your package/module name. Replace `greet` and `analyze` with your tools. Add dependencies to `pyproject.toml`.

---

## 3. Claude Code Plugin

### plugin.json

```json
{
  "name": "My Agent Plugin",
  "version": "1.0.0",
  "description": "One-sentence description of what this plugin does for the user",
  "author": "my-org",
  "skills": [
    "skills/core-capability/SKILL.md"
  ],
  "commands": [
    "commands/run-agent/command.md"
  ]
}
```

### marketplace.json

```json
{
  "slug": "my-agent-plugin",
  "display_name": "My Agent Plugin",
  "tagline": "One-liner shown in marketplace search results",
  "category": "ai-agents",
  "tags": ["agent", "automation"],
  "icon": "icon.png",
  "repository": "https://github.com/my-org/my-agent-plugin",
  "install_command": "git clone https://github.com/my-org/my-agent-plugin.git ~/.claude/plugins/my-agent-plugin"
}
```

### skills/core-capability/SKILL.md

```markdown
---
name: core-capability
description: "Description of when this skill activates. Use when the user mentions: keyword1, keyword2, keyword3"
---

# Core Capability

## Context
Describe the domain knowledge this skill provides.

## Guidance
- Bullet point instructions for Claude when this skill is active.
- Reference specific frameworks, patterns, or rules.
- Keep guidance actionable — every line should change Claude's behavior.

## Reference
Key facts, tables, or data Claude needs to apply this skill.
```

### commands/run-agent/command.md

```markdown
---
name: run-agent
description: "Run the agent with the user's input"
---

# /run-agent

## Steps
1. Collect the user's input or goal.
2. Validate any required configuration (API keys, model selection).
3. Execute the agent's core capability.
4. Return structured results.

## Output Format
- Summary of what the agent did.
- Key results or artifacts produced.
- Suggested next steps.
```

### CLAUDE.md (plugin root)

```markdown
# My Agent Plugin (v1.0.0)

Installed as a Claude Code plugin at `~/.claude/plugins/my-agent-plugin/`.
Skills are auto-invoked by the plugin system based on context.
Commands are user-invocable via `/command-name`.

## Available Skills
- **core-capability** — Description of what it does

## Available Commands
`/run-agent` — Description of what it does
```

**Customize:** Replace all instances of `my-agent-plugin`, `my-org`, `core-capability`, and `run-agent`. Add more skills and commands as needed. The CLAUDE.md is what appears in the host project's instructions.

---

## 4. Docker AI Agent (Python)

### Dockerfile

```dockerfile
# --- Stage 1: Build dependencies ---
FROM python:3.12-slim AS builder

WORKDIR /app

# Install build tools only in the builder stage
RUN pip install --no-cache-dir uv

COPY pyproject.toml uv.lock ./
RUN uv pip install --system --no-cache -r pyproject.toml

# --- Stage 2: Runtime ---
FROM python:3.12-slim AS runtime

WORKDIR /app

# Copy installed packages from builder
COPY --from=builder /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin

# Copy application code last (cache-friendly layer ordering)
COPY src/ ./src/

# Non-root user for security
RUN useradd --create-home agent
USER agent

# Health check endpoint (see src/health.py)
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8080/health')"

EXPOSE 8080

CMD ["python", "-m", "src.main"]
```

### docker-compose.yml

```yaml
services:
  agent:
    build: .
    ports:
      - "8080:8080"
    env_file: .env                  # Load secrets from .env (never committed)
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: agent_db
      POSTGRES_USER: agent
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-changeme}   # Override in .env
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U agent"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  pgdata:
```

### .env.example

```bash
# --- Required: AI Provider ---
ANTHROPIC_API_KEY=sk-ant-...          # Get one at https://console.anthropic.com/
# OPENAI_API_KEY=sk-...               # Optional fallback provider

# --- Required: Database ---
POSTGRES_PASSWORD=changeme            # CHANGE THIS in production
DATABASE_URL=postgresql://agent:${POSTGRES_PASSWORD}@postgres:5432/agent_db

# --- Optional ---
REDIS_URL=redis://redis:6379/0
LOG_LEVEL=info                        # debug | info | warning | error
AGENT_MODEL=claude-sonnet-4-20250514  # Model to use for agent reasoning
```

**Customize:** Change the Python version if needed. Replace `src.main` with your actual module. Add agent-specific environment variables to `.env.example`. Adjust Postgres/Redis config for your data model.

---

## 5. Docker AI Agent (Node.js)

### Dockerfile

```dockerfile
# --- Stage 1: Install dependencies ---
FROM node:22-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --omit=dev

# --- Stage 2: Runtime ---
FROM node:22-alpine AS runtime

WORKDIR /app

# Copy node_modules from builder
COPY --from=builder /app/node_modules ./node_modules

# Copy application code
COPY src/ ./src/
COPY package.json ./

# Non-root user
RUN adduser -D agent
USER agent

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost:8080/health || exit 1

EXPOSE 8080

CMD ["node", "src/index.js"]
```

### docker-compose.yml

```yaml
services:
  agent:
    build: .
    ports:
      - "8080:8080"
    env_file: .env
    depends_on:
      redis:
        condition: service_healthy
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
```

### .env.example

```bash
# --- Required: AI Provider ---
ANTHROPIC_API_KEY=sk-ant-...          # Get one at https://console.anthropic.com/

# --- Optional ---
REDIS_URL=redis://redis:6379/0
LOG_LEVEL=info
PORT=8080
AGENT_MODEL=claude-sonnet-4-20250514
```

**Customize:** Change Node.js version if needed. Replace `src/index.js` with your entry point. Add a Postgres service if your agent needs persistent storage.

---

## 6. Python CLI Agent

### pyproject.toml

```toml
[project]
name = "my-agent-cli"
version = "0.1.0"
description = "CLI agent that does X"
readme = "README.md"
license = { text = "MIT" }
requires-python = ">=3.11"
authors = [{ name = "my-org" }]

dependencies = [
    "click>=8.1.0",
    "anthropic>=0.42.0",
    "rich>=13.0.0",           # Pretty terminal output
    "python-dotenv>=1.0.0",   # .env file support
]

[project.scripts]
# This creates the CLI command when installed via pip/pipx
my-agent = "my_agent_cli.cli:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/my_agent_cli"]
```

### src/my_agent_cli/cli.py

```python
"""
CLI entry point for my-agent.

Install:  pipx install my-agent-cli
Run:      my-agent ask "What is the capital of France?"
"""

import os
import sys

import click
from dotenv import load_dotenv

load_dotenv()  # Load .env file if present


@click.group()
@click.version_option()
def main():
    """My Agent CLI — one-line description of what it does."""
    pass


@main.command()
@click.argument("prompt")
@click.option("--model", default="claude-sonnet-4-20250514", help="Model to use.")
def ask(prompt: str, model: str):
    """Send a prompt to the agent and print the response."""
    api_key = os.environ.get("ANTHROPIC_API_KEY")
    if not api_key:
        click.echo(
            "Error: ANTHROPIC_API_KEY not set.\n"
            "Get one at: https://console.anthropic.com/\n"
            "Then run:   export ANTHROPIC_API_KEY=sk-ant-...",
            err=True,
        )
        sys.exit(1)

    # Lazy import: heavy deps load only when the command actually runs.
    # This keeps `my-agent --help` and `my-agent --version` fast.
    from anthropic import Anthropic

    client = Anthropic(api_key=api_key)
    response = client.messages.create(
        model=model,
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}],
    )

    # Rich output for terminals, plain text for pipes
    if sys.stdout.isatty():
        from rich.console import Console
        from rich.markdown import Markdown

        console = Console()
        console.print(Markdown(response.content[0].text))
    else:
        click.echo(response.content[0].text)


@main.command()
def config():
    """Show current configuration and check connectivity."""
    api_key = os.environ.get("ANTHROPIC_API_KEY")
    status = "set" if api_key else "MISSING"
    click.echo(f"ANTHROPIC_API_KEY: {status}")
    click.echo(f"Config file:      ~/.my-agent/config.toml")
```

### src/my_agent_cli/__init__.py

```python
"""my-agent CLI package."""
```

**Customize:** Change `my-agent-cli`, `my_agent_cli`, and `my-agent` to your names. Replace the `ask` command with your agent's capabilities. Add more commands under `@main.command()`.

---

## 7. LangGraph Deployment

### langgraph.json

```json
{
  "dependencies": ["."],
  "graphs": {
    "my_agent": "./src/my_agent/graph.py:graph"
  },
  "env": ".env"
}
```

### pyproject.toml

```toml
[project]
name = "my-agent-langgraph"
version = "0.1.0"
description = "LangGraph agent that does X"
requires-python = ">=3.11"

dependencies = [
    "langgraph>=0.3.0",
    "langchain-anthropic>=0.3.0",
    "langchain-core>=0.3.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/my_agent"]
```

### src/my_agent/graph.py

```python
"""
LangGraph agent graph definition.

This file defines the agent as a state machine. Each node is a function,
edges define transitions, and the state schema flows through the graph.
"""

from typing import Annotated, TypedDict

from langchain_anthropic import ChatAnthropic
from langchain_core.messages import BaseMessage
from langgraph.graph import StateGraph, END
from langgraph.graph.message import add_messages


# --- State schema -------------------------------------------------------
class AgentState(TypedDict):
    """State that flows through the graph. `messages` accumulates the conversation."""
    messages: Annotated[list[BaseMessage], add_messages]


# --- Nodes --------------------------------------------------------------
model = ChatAnthropic(model="claude-sonnet-4-20250514")


def call_model(state: AgentState) -> dict:
    """Invoke the LLM with the current conversation history."""
    response = model.invoke(state["messages"])
    return {"messages": [response]}


def should_continue(state: AgentState) -> str:
    """Route to 'end' or back to the model based on the last message."""
    last = state["messages"][-1]
    # If the model made tool calls, continue the loop; otherwise finish.
    if hasattr(last, "tool_calls") and last.tool_calls:
        return "continue"
    return "end"


# --- Graph assembly -----------------------------------------------------
workflow = StateGraph(AgentState)

workflow.add_node("agent", call_model)
workflow.set_entry_point("agent")
workflow.add_conditional_edges("agent", should_continue, {
    "continue": "agent",
    "end": END,
})

# Compile the graph — this is the object LangGraph Platform serves.
graph = workflow.compile()
```

### Dockerfile (for self-hosted LangGraph deployment)

```dockerfile
FROM python:3.12-slim

WORKDIR /app

RUN pip install --no-cache-dir uv
COPY pyproject.toml ./
RUN uv pip install --system --no-cache -r pyproject.toml

COPY src/ ./src/
COPY langgraph.json ./

EXPOSE 8000

CMD ["langgraph", "up", "--host", "0.0.0.0", "--port", "8000"]
```

**Customize:** Rename `my_agent` and `my-agent-langgraph`. Add tool nodes between `call_model` and `should_continue` for agentic tool use. Extend `AgentState` with agent-specific fields.

---

## 8. Multi-Agent Docker Compose

### docker-compose.yml

```yaml
# Multi-agent system: coordinator dispatches tasks to specialized workers.
# Shared state via Redis (ephemeral) and Postgres (persistent).

services:
  # --- Coordinator: routes tasks to the right worker ---
  coordinator:
    build:
      context: .
      dockerfile: agents/coordinator/Dockerfile
    ports:
      - "8080:8080"
    env_file: .env
    depends_on:
      redis:
        condition: service_healthy
      postgres:
        condition: service_healthy
    environment:
      AGENT_ROLE: coordinator
      WORKER_URLS: "http://researcher:8081,http://writer:8082"
    restart: unless-stopped

  # --- Worker: research agent ---
  researcher:
    build:
      context: .
      dockerfile: agents/researcher/Dockerfile
    env_file: .env
    environment:
      AGENT_ROLE: researcher
      PORT: 8081
    depends_on:
      redis:
        condition: service_healthy
    restart: unless-stopped

  # --- Worker: writing agent ---
  writer:
    build:
      context: .
      dockerfile: agents/writer/Dockerfile
    env_file: .env
    environment:
      AGENT_ROLE: writer
      PORT: 8082
    depends_on:
      redis:
        condition: service_healthy
    restart: unless-stopped

  # --- Shared state: Redis (task queue, ephemeral cache) ---
  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    volumes:
      - redis_data:/data

  # --- Shared state: Postgres (persistent results, audit log) ---
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: agents_db
      POSTGRES_USER: agents
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-changeme}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U agents"]
      interval: 5s
      timeout: 3s
      retries: 5
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  redis_data:
  pgdata:
```

### .env.example

```bash
# --- Required: AI Provider (shared by all agents) ---
ANTHROPIC_API_KEY=sk-ant-...

# --- Required: Database ---
POSTGRES_PASSWORD=changeme

# --- Optional ---
LOG_LEVEL=info
COORDINATOR_MODEL=claude-sonnet-4-20250514
WORKER_MODEL=claude-haiku-4-20250514    # Cheaper model for workers
```

**Customize:** Add or remove worker services. Change `WORKER_URLS` in the coordinator to match. Each agent gets its own Dockerfile under `agents/<name>/Dockerfile` sharing the same base image pattern from Template 4.

---

## 9. GitHub Release Action for Agent Binaries

### .github/workflows/release.yml

```yaml
# Build and publish agent binaries for macOS, Linux, and Windows.
# Triggered by pushing a tag like v1.0.0.

name: Release Agent Binaries

on:
  push:
    tags: ["v*"]

permissions:
  contents: write    # Required to create GitHub Releases

jobs:
  build:
    strategy:
      matrix:
        include:
          - os: ubuntu-latest
            target: x86_64-linux
            artifact: my-agent-linux-amd64
          - os: macos-latest
            target: aarch64-darwin
            artifact: my-agent-darwin-arm64
          - os: windows-latest
            target: x86_64-windows
            artifact: my-agent-windows-amd64.exe

    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      # Build a self-contained binary with PyInstaller
      - run: pip install pyinstaller
      - run: pip install -r requirements.txt
      - run: pyinstaller --onefile --name ${{ matrix.artifact }} src/main.py

      - uses: actions/upload-artifact@v4
        with:
          name: ${{ matrix.artifact }}
          path: dist/${{ matrix.artifact }}

  release:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          path: artifacts/
          merge-multiple: true

      # Generate checksums so users can verify integrity
      - name: Generate SHA-256 checksums
        run: |
          cd artifacts
          sha256sum * > SHA256SUMS.txt

      - uses: softprops/action-gh-release@v2
        with:
          files: |
            artifacts/*
          generate_release_notes: true
```

**Customize:** Swap PyInstaller for your actual build tool (Go: `go build`, Rust: `cargo build --release`, Node.js: `pkg`). Add/remove OS targets from the matrix. Add code signing steps for macOS (`codesign`) and Windows (`signtool`).

---

## 10. curl Install Script for Agent

### install.sh

```bash
#!/usr/bin/env bash
# install.sh — One-liner installer for my-agent.
#
# Usage:
#   curl -fsSL https://raw.githubusercontent.com/my-org/my-agent/main/install.sh | bash
#
# What it does:
#   1. Detects OS and architecture
#   2. Checks prerequisites (Python >= 3.11 or Node.js >= 18)
#   3. Downloads the latest release binary
#   4. Installs to ~/.local/bin (or /usr/local/bin with sudo)
#   5. Verifies the checksum
#   6. Prints next steps (API key setup)

set -euo pipefail

# --- Configuration (change these) ---
REPO="my-org/my-agent"
BINARY_NAME="my-agent"
MIN_PYTHON="3.11"
MIN_NODE="18"

# --- Colors (disabled if not a terminal) ---
if [ -t 1 ]; then
  RED='\033[0;31m'; GREEN='\033[0;32m'; YELLOW='\033[1;33m'; NC='\033[0m'
else
  RED=''; GREEN=''; YELLOW=''; NC=''
fi

info()  { echo -e "${GREEN}[info]${NC} $*"; }
warn()  { echo -e "${YELLOW}[warn]${NC} $*"; }
error() { echo -e "${RED}[error]${NC} $*" >&2; exit 1; }

# --- OS and architecture detection ---
detect_platform() {
  OS="$(uname -s)"
  ARCH="$(uname -m)"

  case "$OS" in
    Linux)  PLATFORM="linux" ;;
    Darwin) PLATFORM="darwin" ;;
    *)      error "Unsupported OS: $OS. This installer supports Linux and macOS." ;;
  esac

  case "$ARCH" in
    x86_64)  ARCH="amd64" ;;
    aarch64|arm64) ARCH="arm64" ;;
    *)       error "Unsupported architecture: $ARCH" ;;
  esac

  info "Detected platform: ${PLATFORM}-${ARCH}"
}

# --- Prerequisite checks ---
check_python() {
  if command -v python3 &>/dev/null; then
    PY_VER="$(python3 -c 'import sys; print(f"{sys.version_info.major}.{sys.version_info.minor}")')"
    if python3 -c "import sys; exit(0 if sys.version_info >= (3, 11) else 1)" 2>/dev/null; then
      info "Python ${PY_VER} found (>= ${MIN_PYTHON} required)"
      return 0
    else
      warn "Python ${PY_VER} found but >= ${MIN_PYTHON} required"
      return 1
    fi
  fi
  warn "Python not found"
  return 1
}

check_node() {
  if command -v node &>/dev/null; then
    NODE_VER="$(node -v | sed 's/v//' | cut -d. -f1)"
    if [ "$NODE_VER" -ge "$MIN_NODE" ] 2>/dev/null; then
      info "Node.js v${NODE_VER} found (>= ${MIN_NODE} required)"
      return 0
    else
      warn "Node.js v${NODE_VER} found but >= ${MIN_NODE} required"
      return 1
    fi
  fi
  warn "Node.js not found"
  return 1
}

check_prerequisites() {
  # Adjust this: require Python, Node, or either
  if ! check_python; then
    error "Python >= ${MIN_PYTHON} is required. Install it from https://python.org"
  fi
}

# --- Download and install ---
install_binary() {
  LATEST_TAG="$(curl -fsSL "https://api.github.com/repos/${REPO}/releases/latest" | grep '"tag_name"' | cut -d'"' -f4)"
  info "Latest release: ${LATEST_TAG}"

  DOWNLOAD_URL="https://github.com/${REPO}/releases/download/${LATEST_TAG}/${BINARY_NAME}-${PLATFORM}-${ARCH}"
  CHECKSUM_URL="https://github.com/${REPO}/releases/download/${LATEST_TAG}/SHA256SUMS.txt"

  INSTALL_DIR="${HOME}/.local/bin"
  mkdir -p "$INSTALL_DIR"

  info "Downloading ${BINARY_NAME}..."
  curl -fsSL "$DOWNLOAD_URL" -o "${INSTALL_DIR}/${BINARY_NAME}"
  chmod +x "${INSTALL_DIR}/${BINARY_NAME}"

  # Verify checksum
  info "Verifying checksum..."
  EXPECTED="$(curl -fsSL "$CHECKSUM_URL" | grep "${BINARY_NAME}-${PLATFORM}-${ARCH}" | awk '{print $1}')"
  if command -v sha256sum &>/dev/null; then
    ACTUAL="$(sha256sum "${INSTALL_DIR}/${BINARY_NAME}" | awk '{print $1}')"
  else
    ACTUAL="$(shasum -a 256 "${INSTALL_DIR}/${BINARY_NAME}" | awk '{print $1}')"
  fi

  if [ "$EXPECTED" = "$ACTUAL" ]; then
    info "Checksum verified"
  else
    error "Checksum mismatch. Expected: ${EXPECTED}, Got: ${ACTUAL}. The download may be corrupted."
  fi
}

# --- PATH check ---
check_path() {
  if [[ ":$PATH:" != *":${INSTALL_DIR}:"* ]]; then
    warn "${INSTALL_DIR} is not in your PATH."
    echo ""
    echo "  Add it by running:"
    echo "    echo 'export PATH=\"\$HOME/.local/bin:\$PATH\"' >> ~/.bashrc"
    echo "    source ~/.bashrc"
    echo ""
  fi
}

# --- Post-install guidance ---
print_next_steps() {
  echo ""
  info "Installation complete! ${BINARY_NAME} ${LATEST_TAG}"
  echo ""
  echo "  Next steps:"
  echo ""
  echo "  1. Set your API key:"
  echo "     export ANTHROPIC_API_KEY=sk-ant-..."
  echo "     (Get one at https://console.anthropic.com/)"
  echo ""
  echo "  2. Run your first command:"
  echo "     ${BINARY_NAME} --help"
  echo ""
}

# --- Main ---
main() {
  info "Installing ${BINARY_NAME}..."
  detect_platform
  check_prerequisites
  install_binary
  check_path
  print_next_steps
}

main "$@"
```

**Customize:** Change `REPO`, `BINARY_NAME`, `MIN_PYTHON`, and `MIN_NODE`. Adjust `check_prerequisites` to require the right runtime. Update the post-install guidance with your agent's actual first command.

---

## Sources & References

1. **[MCP TypeScript SDK]** — Model Context Protocol. https://github.com/modelcontextprotocol/typescript-sdk Official TypeScript SDK for building MCP servers and clients, providing the McpServer class, transport layers, and Zod-based tool schemas.
2. **[MCP Python SDK (FastMCP)]** — Model Context Protocol. Part of the MCP Python SDK (`mcp` package on PyPI). Provides the FastMCP class for building Python MCP servers with decorator-based tool and resource registration.
3. **[Docker Multi-Stage Builds]** — Docker, Inc. https://docs.docker.com/build/building/multi-stage/ Guide to separating build-time dependencies from runtime images, used in all Docker agent templates to minimize image size.
4. **[GitHub Actions Documentation]** — GitHub. https://docs.github.com/en/actions Reference for CI/CD workflows including npm provenance publishing, PyPI trusted publishers, binary release builds, and install smoke tests.
5. **[ShellCheck — Shell Script Analysis Tool]** — Vidar Holen. https://www.shellcheck.net/ Static analysis tool for bash/sh scripts, used to validate curl install scripts for correctness, portability, and security.
6. **[Semantic Versioning 2.0.0]** — Tom Preston-Werner. https://semver.org/ The versioning specification used across all agent templates for package.json version fields, pyproject.toml versions, and Git release tags.
