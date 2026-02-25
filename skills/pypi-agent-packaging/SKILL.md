---
name: pypi-agent-packaging
description: "Packaging Python AI agents for distribution via PyPI, pip, uvx, and uv. Use when the user mentions: pypi agent, pip install agent, python package agent, uvx agent, uv publish, pyproject.toml agent, python cli agent, langchain package, crewai package, autogen package, mcp server python, python distribution, publish to pypi, python agent packaging, pip packaging, python entry point, python build, trusted publisher"
---

# PyPI Agent Packaging

## When to Use PyPI

PyPI is the right distribution channel when:

| Situation | Use PyPI? | Why |
|---|---|---|
| Agent built with Python | Yes | Users expect `pip install` for Python tools |
| LangChain / CrewAI / AutoGen agent | Yes | Entire ecosystem is Python-native |
| Python CLI tool (ask questions, get answers) | Yes | `uvx my-agent "query"` is zero-install |
| MCP server written in Python | Yes | `uvx my-mcp-server` is the standard MCP pattern |
| Agent needs GPU inference (PyTorch, transformers) | Yes (with extras) | `pip install my-agent[gpu]` keeps base install light |
| Multi-service system (agent + DB + vector store) | No | Use Docker Compose instead |
| Agent written in Node.js / TypeScript | No | Use npm instead |
| Users are non-technical | No | Use a desktop app, web app, or Docker one-click deploy |

**The key advantage of PyPI:** `uvx my-agent` gives users a zero-install experience. No cloning, no virtual environment setup, no dependency management. It just works.

---

## pyproject.toml Anatomy for AI Agents

`pyproject.toml` is the single source of truth for your Python package. Every field below is annotated with why it matters for agents specifically.

```toml
[project]
name = "my-agent"                          # PyPI package name (globally unique)
version = "0.1.0"                          # Semver — bump on every publish
description = "An AI agent that researches topics and writes summaries"
readme = "README.md"
license = { text = "MIT" }
requires-python = ">=3.11"                 # Pin minimum Python — agents use modern features
authors = [
    { name = "Your Name", email = "you@example.com" },
]
keywords = ["ai", "agent", "langchain", "cli"]
classifiers = [
    "Development Status :: 4 - Beta",
    "Environment :: Console",
    "Intended Audience :: Developers",
    "Programming Language :: Python :: 3.11",
    "Programming Language :: Python :: 3.12",
    "Programming Language :: Python :: 3.13",
    "Topic :: Scientific/Engineering :: Artificial Intelligence",
]

# Core dependencies — keep this list SMALL
# Users who run `pip install my-agent` get only these
dependencies = [
    "anthropic>=0.40.0",                   # API client (lightweight)
    "click>=8.0",                          # CLI framework
    "python-dotenv>=1.0",                  # .env file support
    "rich>=13.0",                          # Terminal formatting
    "httpx>=0.27",                         # Async HTTP client
]

[project.optional-dependencies]
# Heavy ML deps are opt-in, not forced on every user
gpu = [
    "torch>=2.0",
    "transformers>=4.40",
    "sentence-transformers>=3.0",
]
# LangChain ecosystem (large dependency tree)
langchain = [
    "langchain>=0.3.0",
    "langchain-anthropic>=0.3.0",
    "langchain-community>=0.3.0",
]
# Development tools
dev = [
    "pytest>=8.0",
    "pytest-asyncio>=0.24",
    "ruff>=0.8.0",
    "mypy>=1.13",
]

# THIS IS CRITICAL — without this, your agent has no CLI command
[project.scripts]
my-agent = "my_agent.cli:main"            # `my-agent` command → calls main() in cli.py

# For MCP servers, also add:
# my-agent-mcp = "my_agent.mcp_server:main"

[project.urls]
Homepage = "https://github.com/your-org/my-agent"
Repository = "https://github.com/your-org/my-agent"
Issues = "https://github.com/your-org/my-agent/issues"

# =========================================================================
# Build system — Hatchling is modern, fast, and requires minimal config
# =========================================================================
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/my_agent"]                # Tells hatch where to find your package

# =========================================================================
# Tool configurations
# =========================================================================
[tool.ruff]
target-version = "py311"
line-length = 100

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "B", "SIM"]

[tool.pytest.ini_options]
testpaths = ["tests"]
asyncio_mode = "auto"

[tool.mypy]
python_version = "3.11"
strict = true
```

### Key decisions explained

1. **`requires-python = ">=3.11"`** -- Agents use `match` statements, `asyncio.TaskGroup`, modern typing syntax. Do not target 3.9 or 3.10 unless you have a specific reason.
2. **Core deps are minimal** -- `anthropic` + `click` + `rich` is ~20MB installed. Adding `torch` makes it 2GB+. Keep the mandatory install small.
3. **Optional dependencies for heavy extras** -- Users who need GPU inference opt in with `pip install my-agent[gpu]`. Everyone else gets a fast install.
4. **`[project.scripts]`** -- This is the most commonly forgotten field. Without it, `pip install my-agent` installs the package but creates no command. Users cannot run your agent from the terminal.
5. **Hatchling over setuptools** -- Hatchling is faster, requires less configuration, and is the default for modern Python projects. setuptools works fine but requires more boilerplate.

---

## Project Structure for Python Agents

```
my-agent/
├── pyproject.toml              # Package metadata, dependencies, entry points
├── src/
│   └── my_agent/               # Package directory (underscore, not hyphen)
│       ├── __init__.py          # Version: __version__ = "0.1.0"
│       ├── cli.py               # CLI entry point (Click or Typer)
│       ├── agent.py             # Core agent logic
│       ├── tools.py             # Tool definitions the agent can call
│       ├── config.py            # Configuration management (env vars, defaults)
│       └── prompts.py           # System prompts and templates
├── tests/
│   ├── __init__.py
│   ├── test_agent.py
│   └── test_tools.py
├── .env.example                 # Template with every required env var
├── .gitignore
├── LICENSE
└── README.md
```

### Why `src/` layout?

The `src/` layout prevents a subtle bug: without it, `import my_agent` in tests resolves to the local directory (not the installed package). This means tests pass locally but the published package is broken. The `src/` layout forces Python to use the installed version.

### cli.py: the entry point

```python
"""CLI entry point for my-agent."""
import click
from rich.console import Console

console = Console()


@click.group()
@click.version_option()
def main():
    """An AI agent that researches topics and writes summaries."""
    pass


@main.command()
@click.argument("query")
@click.option("--model", default="claude-sonnet-4-20250514", help="Model to use")
@click.option("--verbose", is_flag=True, help="Show agent reasoning")
def run(query: str, model: str, verbose: bool):
    """Run the agent with a query."""
    from my_agent.config import get_config   # Lazy import
    from my_agent.agent import Agent         # Lazy import

    config = get_config()

    if not config.api_key:
        console.print(
            "[red]Error:[/] ANTHROPIC_API_KEY not set.\n"
            "Get a key at: https://console.anthropic.com/keys\n"
            "Then: export ANTHROPIC_API_KEY=your-key-here"
        )
        raise SystemExit(1)

    agent = Agent(config=config, model=model)
    result = agent.run(query, verbose=verbose)
    console.print(result)


@main.command()
def doctor():
    """Check that all dependencies and credentials are configured."""
    from my_agent.config import get_config

    config = get_config()
    checks = [
        ("ANTHROPIC_API_KEY", bool(config.api_key), "https://console.anthropic.com/keys"),
        ("Python >= 3.11", _check_python(), "https://python.org"),
    ]

    all_ok = True
    for name, ok, fix_url in checks:
        if ok:
            console.print(f"  [green]PASS[/] {name}")
        else:
            console.print(f"  [red]FAIL[/] {name}")
            console.print(f"         Fix: {fix_url}")
            all_ok = False

    raise SystemExit(0 if all_ok else 1)


def _check_python() -> bool:
    import sys
    return sys.version_info >= (3, 11)
```

### config.py: environment variable management

```python
"""Configuration management — reads from environment and .env files."""
import os
from dataclasses import dataclass, field
from dotenv import load_dotenv


@dataclass
class Config:
    api_key: str = ""
    model: str = "claude-sonnet-4-20250514"
    max_tokens: int = 4096
    log_level: str = "info"


def get_config() -> Config:
    """Load configuration from environment variables and .env file."""
    # Load .env file if present (does not override existing env vars)
    load_dotenv()

    return Config(
        api_key=os.environ.get("ANTHROPIC_API_KEY", ""),
        model=os.environ.get("MODEL", "claude-sonnet-4-20250514"),
        max_tokens=int(os.environ.get("MAX_TOKENS", "4096")),
        log_level=os.environ.get("LOG_LEVEL", "info"),
    )
```

---

## Handling Large ML Dependencies

The biggest mistake in Python agent packaging is making `torch` a core dependency. A user who just wants to call the Anthropic API should not download 2GB of PyTorch.

### Strategy 1: Lazy imports

```python
# BAD — torch loads on every import, even if user just wants --help
import torch
from transformers import AutoModel

def embed(text: str) -> list[float]:
    model = AutoModel.from_pretrained("...")
    return model.encode(text)

# GOOD — torch only loads when this function is actually called
def embed(text: str) -> list[float]:
    try:
        import torch
        from transformers import AutoModel
    except ImportError:
        raise ImportError(
            "Local embedding requires GPU dependencies.\n"
            "Install them with: pip install my-agent[gpu]"
        )
    model = AutoModel.from_pretrained("...")
    return model.encode(text)
```

### Strategy 2: API-first, local-optional

```python
def get_embeddings(text: str, mode: str = "api") -> list[float]:
    """Get embeddings via API (default) or local model."""
    if mode == "api":
        # Light dependency: just an HTTP call
        from anthropic import Anthropic
        client = Anthropic()
        # ... use Anthropic or OpenAI embedding API
    elif mode == "local":
        # Heavy dependency: only imported if user explicitly requests local mode
        try:
            from sentence_transformers import SentenceTransformer
        except ImportError:
            raise ImportError(
                "Local embeddings require: pip install my-agent[gpu]"
            )
        model = SentenceTransformer("all-MiniLM-L6-v2")
        return model.encode(text).tolist()
```

### Strategy 3: Dependency groups via optional extras

```toml
# pyproject.toml
[project]
dependencies = ["anthropic>=0.40.0", "click>=8.0"]  # Core: ~20MB

[project.optional-dependencies]
gpu = ["torch>=2.0", "transformers>=4.40"]           # GPU: ~2GB
local = ["sentence-transformers>=3.0", "chromadb>=0.5"]  # Local RAG: ~500MB
all = ["my-agent[gpu,local]"]                        # Everything
```

This keeps the default `pip install my-agent` fast (seconds, not minutes) while giving power users access to local inference when they need it.

---

## Distribution Methods

Python agents have four distribution paths, ordered from best to worst user experience.

### 1. uvx (zero-install)

```bash
# User runs this — no install step, no virtualenv, nothing persisted
uvx my-agent run "summarize this topic"

# With a specific Python version
uvx --python 3.12 my-agent run "summarize this topic"
```

How it works: `uvx` creates a temporary virtual environment, installs the package, runs the command, and cleans up. The user never manages dependencies. This is the best distribution method for CLI agents.

**Requirement:** Your `pyproject.toml` must have `[project.scripts]` defined. Without an entry point, `uvx` has nothing to execute.

### 2. uv tool install (persistent global install)

```bash
# User installs globally in an isolated environment
uv tool install my-agent

# Now available as a command
my-agent run "summarize this topic"

# Upgrade later
uv tool upgrade my-agent

# Uninstall
uv tool uninstall my-agent
```

How it works: `uv tool install` creates a persistent isolated virtual environment in `~/.local/share/uv/tools/` and symlinks the CLI command to `~/.local/bin/`. No conflicts with other packages.

### 3. pip install (traditional)

```bash
# In a virtual environment (recommended)
python -m venv .venv && source .venv/bin/activate
pip install my-agent

# Global (not recommended — can conflict with system Python)
pip install my-agent
```

This is the most familiar method but has the most footguns: system Python conflicts, no isolation, `pip install --user` path issues on some systems.

### 4. pipx (predecessor to uv tool)

```bash
pipx install my-agent
```

Works identically to `uv tool install` but is slower. `uv` has largely replaced `pipx` in practice, but some users still have `pipx` and not `uv`.

### README pattern for install instructions

Lead with the best method. Collapse alternatives.

```markdown
## Install

### Recommended: uvx (zero install)
\`\`\`bash
uvx my-agent run "your query"
\`\`\`

### Persistent install
\`\`\`bash
uv tool install my-agent
my-agent run "your query"
\`\`\`

<details>
<summary>Alternative: pip</summary>

\`\`\`bash
pip install my-agent
my-agent run "your query"
\`\`\`
</details>

<details>
<summary>With GPU support (local inference)</summary>

\`\`\`bash
pip install my-agent[gpu]
\`\`\`
</details>
```

---

## Publishing to PyPI

### Build the package

```bash
# Build source distribution (.tar.gz) and wheel (.whl)
uv build

# Output:
#   dist/my_agent-0.1.0.tar.gz
#   dist/my_agent-0.1.0-py3-none-any.whl
```

### Test on TestPyPI first

Never publish directly to PyPI without testing. TestPyPI is a separate instance for dry runs.

```bash
# Upload to TestPyPI
uv publish --publish-url https://test.pypi.org/legacy/

# Test the install from TestPyPI
uvx --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ my-agent --version
```

The `--extra-index-url` fallback to real PyPI is needed because your agent's dependencies (anthropic, click, etc.) are only on real PyPI, not TestPyPI.

### Publish to PyPI

```bash
# Requires a PyPI API token (or Trusted Publisher — see below)
uv publish
```

### Trusted Publishers: the right way to publish

Trusted Publishers use OpenID Connect (OIDC) to let GitHub Actions publish to PyPI without long-lived API tokens. No secrets to manage, no tokens to rotate.

Setup:
1. Go to pypi.org > Your Project > Settings > Publishing
2. Add a "Trusted Publisher" with your GitHub repo, workflow filename, and environment name
3. Use the GitHub Actions workflow below

```yaml
# .github/workflows/pypi-publish.yml
name: Publish to PyPI

on:
  push:
    tags: ["v*"]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      id-token: write    # Required for Trusted Publishers OIDC
    environment:
      name: pypi         # Must match the environment name on PyPI

    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Build package
        run: uv build

      - name: Publish to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        # No token needed — OIDC handles authentication
```

**Why Trusted Publishers matter:** API tokens can leak (committed to git, stolen from CI logs, compromised developer machine). OIDC tokens are ephemeral — they are minted for a single workflow run and expire immediately. This is the most secure way to publish Python packages.

---

## API Key Handling in Python Agents

### Priority order for finding keys

```python
import os
from dotenv import load_dotenv


def get_api_key() -> str:
    """Resolve API key using standard priority order."""

    # 1. Explicit environment variable (highest priority)
    key = os.environ.get("ANTHROPIC_API_KEY")
    if key:
        return key

    # 2. .env file in current directory
    load_dotenv()
    key = os.environ.get("ANTHROPIC_API_KEY")
    if key:
        return key

    # 3. System keyring (persistent, encrypted storage)
    try:
        import keyring
        key = keyring.get_password("my-agent", "anthropic_api_key")
        if key:
            return key
    except ImportError:
        pass

    # 4. Not found — return empty, let caller handle
    return ""
```

### First-use prompt when key is missing

```python
import click

def ensure_api_key() -> str:
    """Get API key, prompting user if not found."""
    key = get_api_key()
    if key:
        return key

    click.echo("This agent requires an Anthropic API key.")
    click.echo("Get one at: https://console.anthropic.com/keys\n")

    key = click.prompt("Enter your API key", hide_input=True)

    # Offer to save for next time
    if click.confirm("Save to .env for future use?", default=True):
        with open(".env", "a") as f:
            f.write(f"\nANTHROPIC_API_KEY={key}\n")
        click.echo("Saved to .env (add .env to .gitignore!)")

    return key
```

### .env.example template

```bash
# .env.example — copy to .env and fill in your values
# Required
ANTHROPIC_API_KEY=               # https://console.anthropic.com/keys

# Optional
MODEL=claude-sonnet-4-20250514        # claude-sonnet-4-20250514, claude-opus-4-20250514, etc.
MAX_TOKENS=4096                  # Max tokens per response
LOG_LEVEL=info                   # debug, info, warning, error
```

---

## MCP Server via Python

MCP (Model Context Protocol) servers are a special case of Python agent packaging. They run as subprocesses of an MCP client (Claude Desktop, Claude Code, etc.).

### Minimal MCP server

```python
# src/my_agent/mcp_server.py
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-agent")


@mcp.tool()
async def search(query: str) -> str:
    """Search for information about a topic."""
    # Your agent logic here
    return f"Results for: {query}"


@mcp.resource("config://settings")
async def get_settings() -> str:
    """Expose agent configuration as a resource."""
    return "model: claude-sonnet-4-20250514\nmax_tokens: 4096"


def main():
    mcp.run(transport="stdio")
```

### pyproject.toml for MCP servers

```toml
[project]
name = "my-agent-mcp"
dependencies = [
    "mcp>=1.0.0",
    "httpx>=0.27",
]

[project.scripts]
my-agent-mcp = "my_agent.mcp_server:main"
```

### User installs with uvx

The user configures their MCP client to run your server via `uvx`:

```json
{
  "mcpServers": {
    "my-agent": {
      "command": "uvx",
      "args": ["my-agent-mcp"],
      "env": {
        "ANTHROPIC_API_KEY": "sk-ant-..."
      }
    }
  }
}
```

This is the standard pattern: the MCP client spawns `uvx my-agent-mcp` as a subprocess. `uvx` creates an ephemeral environment, installs the package, and runs it. No manual install step for the user.

---

## CI/CD: GitHub Actions for Automated Publishing

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12", "3.13"]

    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Run tests
        run: |
          uv sync --dev
          uv run pytest

      - name: Lint
        run: |
          uv run ruff check .
          uv run mypy src/

  test-uvx:
    # Verify that `uvx my-agent` actually works from a built wheel
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Build and test with uvx
        run: |
          uv build
          uvx --from dist/*.whl my-agent --help
```

The `test-uvx` job is critical. It catches the most common PyPI publishing failure: the package installs but the CLI command does not work (missing `[project.scripts]`, broken imports, missing dependencies). Test what users will actually run.

---

## Common Pitfalls

### 1. Not pinning minimum Python version

```toml
# BAD — allows Python 3.8 where modern syntax fails
requires-python = ">=3.8"

# GOOD — agents use modern features, be honest about it
requires-python = ">=3.11"
```

### 2. Importing heavy deps at top level

```python
# BAD — torch loads on `my-agent --help` (3-second startup penalty)
import torch
from transformers import pipeline

# GOOD — import inside the function that needs it
def generate():
    import torch
    from transformers import pipeline
    # ...
```

This is the single most impactful optimization for agent CLI tools. Users should not wait for PyTorch to initialize just to see the help text.

### 3. Missing [project.scripts]

Without this, `pip install my-agent` installs the library but creates no CLI command. Users cannot run your agent. This is the most common publishing mistake.

```toml
# You MUST have this
[project.scripts]
my-agent = "my_agent.cli:main"
```

### 4. Not testing with uvx

Your agent works with `pip install -e .` in development but fails with `uvx` because:
- A dependency is missing from `pyproject.toml` (it was installed in your venv from a previous project)
- A file is not included in the wheel (data files, templates, prompts)
- An import path is wrong (relative vs absolute)

Always test with `uv build && uvx --from dist/*.whl my-agent --help` before publishing.

### 5. Pickle-based model files

```python
# SECURITY RISK — pickle can execute arbitrary code on load
import pickle
model = pickle.load(open("model.pkl", "rb"))  # Never do this

# SAFER — use safetensors, GGUF, or ONNX formats
from safetensors.torch import load_file
model = load_file("model.safetensors")
```

Pickle files can execute arbitrary Python code when loaded. If your agent loads model files from untrusted sources (user-provided paths, downloads), pickle deserialization is a remote code execution vulnerability. Use `safetensors`, ONNX, or GGUF formats instead.

### 6. Using setuptools.find_packages() with src layout

```toml
# BAD — finds nothing because packages are inside src/
[tool.setuptools]
packages = {find = {}}

# GOOD with setuptools — point to src
[tool.setuptools.packages.find]
where = ["src"]

# BETTER — use hatchling, no discovery needed
[tool.hatch.build.targets.wheel]
packages = ["src/my_agent"]
```

### 7. Version string only in one place

```python
# __init__.py
__version__ = "0.1.0"
```

```toml
# pyproject.toml
version = "0.1.0"
```

These will inevitably drift. Use dynamic versioning instead:

```toml
[project]
dynamic = ["version"]

[tool.hatch.version]
path = "src/my_agent/__init__.py"
```

Now `pyproject.toml` reads the version from `__init__.py` at build time. One source of truth.

### 8. No .env.example

Users install your agent, run it, and get `AuthenticationError: API key not set` with no guidance on which key, where to get it, or how to set it. Always ship a `.env.example` with every required variable documented.

### 9. Forgetting to add py.typed

If your agent exposes a Python API (not just a CLI), type-checker users expect a `py.typed` marker file:

```
src/my_agent/py.typed    # Empty file — signals that the package ships type stubs
```

Without it, `mypy` ignores your type annotations when other projects import your package.

### 10. README that does not start with the install command

The first thing in your README after the one-line description should be how to install and run the agent. Not the architecture diagram. Not the feature list. The install command.

```markdown
# my-agent

An AI agent that researches topics and writes summaries.

## Quick start

\`\`\`bash
uvx my-agent run "explain quantum computing"
\`\`\`

## Install (persistent)

\`\`\`bash
uv tool install my-agent
\`\`\`
```

Users scan for the code block. Make sure the first one they see is the one that gets them running.

---

## Sources & References

1. **[Python Packaging User Guide]** — Python Packaging Authority (PyPA). https://packaging.python.org/ The authoritative guide to packaging and distributing Python projects, including tutorials, specifications, and tool recommendations.
2. **[PEP 517 — A Build-System Independent Format for Source Trees]** — Python Enhancement Proposals. https://peps.python.org/pep-0517/ Specification for the build backend interface that enables tools like hatchling, flit, and setuptools to work interchangeably.
3. **[PEP 518 — Specifying Minimum Build System Requirements]** — Python Enhancement Proposals. https://peps.python.org/pep-0518/ Introduces the `[build-system]` table in `pyproject.toml`, establishing it as the standard build configuration file.
4. **[PEP 621 — Storing Project Metadata in pyproject.toml]** — Python Enhancement Proposals. https://peps.python.org/pep-0621/ Defines the `[project]` table in `pyproject.toml` for declaring package metadata, dependencies, and entry points.
5. **[PyPI — The Python Package Index]** — Python Software Foundation. https://pypi.org/ The official repository for publishing and discovering Python packages, used by pip and uv for package resolution.
6. **[uv — An Extremely Fast Python Package Manager]** — Astral. https://github.com/astral-sh/uv Rust-based Python package installer and resolver, providing `uvx` for zero-install execution and `uv tool install` for isolated global installs.
7. **[Hatchling Build Backend]** — Ofek Lev. https://hatch.pypa.io/ Modern, extensible Python build backend used as the default for new projects, with minimal configuration requirements.
8. **[pip — The Python Package Installer]** — Python Packaging Authority (PyPA). https://pip.pypa.io/ The standard tool for installing Python packages from PyPI and other indexes.
9. **[Trusted Publishers on PyPI]** — Python Packaging Authority (PyPA). https://docs.pypi.org/trusted-publishers/ Guide to configuring OIDC-based publishing from CI systems like GitHub Actions, eliminating the need for long-lived API tokens.
10. **[Click — Python CLI Framework]** — Pallets Projects. https://click.palletsprojects.com/ Composable command-line interface toolkit used for building agent CLI entry points with argument parsing and help generation.
