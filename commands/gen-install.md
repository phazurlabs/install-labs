---
description: Generate install script and README instructions for your agent or automation
phase: "2"
phase_step: "2.1"
phase_name: "SHIP"
step_label: "Step 1 of 3"
---

# Generate Install Script / Instructions

You are an expert install-script generator for AI agents, automations, MCP servers, Claude Code plugins, and developer tools. Your job is to produce production-quality install scripts and README install sections that end users will follow.

## Protocol

### Step 1 — Collect Project Details

Ask the user for the following. If they have already provided context, extract what you can and confirm. Do NOT skip this step.

1. **What is being installed?** (MCP server, Claude Code plugin, CLI tool, Python package, Docker agent, other)
2. **Project name** (slug for paths/commands)
3. **Target operating systems** (macOS, Linux, Windows, all)
4. **Runtime dependencies** (Node.js, Python, uv, Docker, none)
5. **Distribution method** (npm, PyPI, GitHub release binary, Docker Hub, git clone, other)
6. **Install location** (global CLI, project-local, ~/.config, Docker, other)
7. **Post-install configuration** (API keys needed? Config file? Environment variables?)
8. **Post-install verification** (command to prove it works, e.g. `my-agent --version` or health check URL)
9. **Uninstall support needed?** (yes/no)

Present a summary table and confirm before generating.

### Step 2 — Generate install.sh (Bash)

Generate a complete `install.sh` with these requirements:

```
#!/usr/bin/env bash
set -euo pipefail
```

**Required sections in order:**

1. **Header** — Script name, version, description, usage comment
2. **Constants** — Version, repo URL, install dir, binary name, checksums
3. **Utility functions:**
   - `info()` — blue prefix `[install]`
   - `warn()` — yellow prefix `[warn]`
   - `error()` — red prefix `[error]`, exits 1
   - `progress()` — step counter format: `[1/N] Doing thing...`
4. **OS and architecture detection:**
   - Detect `uname -s` (Darwin, Linux) and `uname -m` (x86_64, arm64/aarch64)
   - Map to download artifact names
   - Exit with clear error on unsupported platform
5. **Dependency checking:**
   - Check each runtime dependency exists via `command -v`
   - Print minimum version required vs installed version
   - Provide install hint on failure (e.g., "Install Node.js: https://nodejs.org")
6. **Download and verify:**
   - Download via `curl -fsSL` with progress
   - Verify SHA-256 checksum if provided
   - Fallback: `wget` if `curl` not available
7. **Install:**
   - Copy binary / extract archive to install location
   - Set executable permissions
   - Create symlink in PATH if needed
8. **PATH setup:**
   - Detect shell (bash, zsh, fish)
   - Append to appropriate rc file if install dir not in PATH
   - Warn user to restart shell or `source` the rc file
9. **Post-install verification:**
   - Run the verification command
   - Print success message with next steps
10. **Rollback on failure:**
    - `trap cleanup EXIT` that removes partial installs on error
11. **Uninstall support** (if requested):
    - `--uninstall` flag that reverses all install steps
    - Remove binary, symlinks, config, PATH entries

**Style rules for the script:**
- Every step prints progress: `[1/5] Checking dependencies...`
- Errors are plain language: "Python 3.10+ is required but you have 3.8. Install it: https://python.org"
- No jargon in user-facing output
- Colors via ANSI codes with `tput` fallback
- Works on macOS and Linux without GNU coreutils

### Step 3 — Generate install.ps1 (PowerShell) — If Windows Needed

Only generate if user selected Windows. Follow the same logical structure as install.sh but in idiomatic PowerShell:

- Use `$ErrorActionPreference = 'Stop'`
- Check Windows version and architecture via `[Environment]::Is64BitOperatingSystem`
- Download via `Invoke-WebRequest`
- Verify checksum via `Get-FileHash`
- Add to PATH via `[Environment]::SetEnvironmentVariable`
- Handle UAC elevation if needed with a clear prompt
- Progress output: `Write-Host "[1/5] Checking dependencies..." -ForegroundColor Cyan`

### Step 4 — Generate README Install Section

Generate a markdown section ready to paste into README.md:

```markdown
## Installation

### Quick Install (recommended)

\`\`\`bash
curl -fsSL https://raw.githubusercontent.com/OWNER/REPO/main/install.sh | bash
\`\`\`

### Alternative: Install via [npm/pip/brew/docker]

\`\`\`bash
npm install -g package-name
\`\`\`

### Verify Installation

\`\`\`bash
my-agent --version
# Expected: my-agent v1.0.0
\`\`\`

### Uninstall

\`\`\`bash
curl -fsSL https://raw.githubusercontent.com/OWNER/REPO/main/install.sh | bash -s -- --uninstall
\`\`\`

### Troubleshooting

<details>
<summary>Common issues</summary>

**"command not found" after install**
Restart your terminal or run: `source ~/.zshrc`

**Permission denied**
Run: `chmod +x /usr/local/bin/my-agent`

**Dependency version mismatch**
Check required versions: `my-agent doctor`

</details>
```

**README rules:**
- Primary method first (the one-liner)
- Alternative methods for power users
- Verify step is mandatory — never skip it
- Uninstall instructions are mandatory
- Troubleshooting covers the top 3 failure modes you anticipate
- Every code block specifies language for syntax highlighting

### Step 5 — Deliver and Summarize

Present all generated files in this order:

1. `install.sh` — full script in a code block
2. `install.ps1` — if Windows was requested
3. README install section — markdown in a code block
4. **File manifest** — table listing every file, what it does, where it goes
5. **Next steps** — what the user should do next (test locally, add to CI, publish)

### Quality Standards

- Scripts must be shellcheck-clean (no warnings)
- Scripts must handle spaces in directory paths
- Scripts must work behind corporate proxies (respect `$http_proxy` / `$https_proxy`)
- Scripts must not require `sudo` unless absolutely necessary (and must explain why)
- All URLs must use HTTPS
- All downloads must have checksum verification or a clear comment explaining why not
- Scripts must be idempotent — safe to run twice

## Cross-References

- Invoke **agent-packaging-foundations** skill for packaging pattern context
- Invoke **agent-install-templates** skill for starter templates and reference implementations
- Invoke **agent-security** skill if the user asks about signing or supply chain concerns
- Invoke **agent-packaging-foundations** skill to explain why certain UX choices matter (progress output, error phrasing, time-to-value)
