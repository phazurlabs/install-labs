---
name: claude-code-plugin-packaging
description: "Complete guide to packaging AI agents and domain knowledge as Claude Code plugins. Use when the user mentions: Claude Code plugin, claude plugin, plugin packaging, plugin structure, plugin.json, CLAUDE.md plugin, skill file, command file, slash command packaging, MCP plugin, plugin distribution, plugin marketplace, plugin install, plugin directory, plugin testing, plugin versioning, extend Claude Code, Claude Code extension, bundle MCP server, domain expertise plugin"
---

# Claude Code Plugin Packaging

Complete reference for building, structuring, distributing, and maintaining Claude Code plugins — the native extension format for Claude Code.

---

## What Is a Claude Code Plugin?

A Claude Code plugin is a directory installed at `~/.claude/plugins/<name>/` that extends Claude Code with three capabilities:

| Component | Purpose | Invocation |
|-----------|---------|------------|
| **Skills** | Domain knowledge auto-injected into conversations | Automatic — keyword match in the skill's `description` field triggers inclusion |
| **Commands** | Structured protocols the user triggers explicitly | Manual — user types `/command-name` in the conversation |
| **MCP Servers** | Tool servers providing actions Claude can call | Automatic — declared in plugin.json, started when the plugin loads |

A plugin can contain any combination of these. A pure-knowledge plugin might have only skills. A workflow plugin might have only commands. A tool plugin might bundle an MCP server with contextual skills that teach Claude how to use it effectively.

**Key mental model:** Skills are what Claude *knows*. Commands are what Claude *does on request*. MCP servers are what Claude *can interact with*.

---

## When to Build a Claude Code Plugin

Build a plugin when you want to:

- **Bundle domain expertise** — Give Claude deep knowledge in a specific area (medical coding, legal research, design systems) that activates automatically when relevant
- **Create repeatable workflows** — Define multi-step protocols (code review checklists, deployment procedures, audit processes) invokable via slash commands
- **Package MCP tools with context** — An MCP server alone gives Claude tools; a plugin wraps those tools with skills that teach Claude *when and how* to use them
- **Distribute to a team** — Share a standardized Claude Code configuration across developers, designers, or analysts
- **Sell on a marketplace** — Package expertise as a product others install with one command

**Do NOT build a plugin when:**
- A single CLAUDE.md file in your project root is sufficient
- The knowledge is project-specific and won't be reused
- You need a standalone application (use a CLI tool or web app instead)

---

## Canonical Directory Structure

```
~/.claude/plugins/my-plugin/
├── .claude-plugin/
│   ├── plugin.json            # Required: metadata
│   └── marketplace.json       # Optional: marketplace listing
├── skills/
│   └── my-skill/
│       └── SKILL.md           # Auto-invoked knowledge file
├── commands/
│   └── my-command.md          # Slash-command protocol
├── mcp-config.json            # Optional: MCP server declarations
├── CLAUDE.md                  # Required: plugin manifest loaded every session
└── README.md                  # Documentation for humans
```

Every plugin MUST have:
1. `.claude-plugin/plugin.json` — so Claude Code recognizes it as a plugin
2. `CLAUDE.md` — so Claude knows the plugin exists and what it offers

Everything else is optional depending on what the plugin provides.

---

## plugin.json Format

Located at `.claude-plugin/plugin.json`. This is the plugin's identity card.

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "One-sentence description of what this plugin does",
  "author": {
    "name": "Your Name or Org",
    "url": "https://yoursite.com",
    "email": "you@example.com"
  },
  "mcpServers": {}
}
```

**Field rules:**
- `name` — Lowercase, hyphenated. Must match the directory name under `~/.claude/plugins/`.
- `version` — Semantic versioning (MAJOR.MINOR.PATCH). Increment MAJOR for breaking changes, MINOR for new skills/commands, PATCH for content fixes.
- `description` — One sentence, under 120 characters. Appears in plugin listings.
- `author.name` — Required. Person or organization.
- `author.url` — Optional. Link to homepage or repo.
- `author.email` — Optional. Contact email.
- `mcpServers` — Optional object. Declares MCP servers bundled with the plugin (see MCP section below).

---

## marketplace.json Format

Located at `.claude-plugin/marketplace.json`. Required only if listing on a plugin marketplace.

```json
{
  "name": "My Plugin Collection",
  "owner": "your-username",
  "plugins": [
    {
      "name": "my-plugin",
      "path": ".",
      "description": "One-sentence description",
      "categories": ["development", "ai-agents"],
      "tags": ["packaging", "distribution", "mcp"]
    }
  ]
}
```

The `plugins` array allows a single repository to contain multiple plugins (monorepo pattern). For single-plugin repos, keep one entry with `"path": "."`.

---

## Skills: Auto-Invoked Knowledge Files

### File Location and Naming

```
skills/
└── my-skill-name/
    └── SKILL.md
```

Each skill lives in its own subdirectory. The directory name should be descriptive and hyphenated.

### SKILL.md Anatomy

```markdown
---
name: my-skill-name
description: "What this skill covers. Use when the user mentions: keyword1, keyword2, keyword3, keyword4"
---

# Skill Title

## Section One
Content here...

## Section Two
More content...
```

### Frontmatter Fields

| Field | Required | Purpose |
|-------|----------|---------|
| `name` | Yes | Identifier, must match directory name |
| `description` | Yes | Trigger string — Claude matches conversation context against these keywords to decide whether to load the skill |

### The Description Field Is Critical

The `description` field controls when your skill activates. It must contain:
1. A brief statement of the skill's domain
2. Explicit trigger keywords after "Use when the user mentions:"

**Good description:**
```yaml
description: "Guide to Docker containerization for AI agents. Use when the user mentions: docker, container, dockerfile, docker-compose, containerize, image build, multi-stage build"
```

**Bad description:**
```yaml
description: "Docker stuff"
```

The bad version will rarely trigger because it lacks keyword diversity.

### Skill Design Principles

1. **Focused** — One domain per skill. Do not combine "React components" and "database design" in one skill file. Split them.
2. **Actionable** — Include code examples, shell commands, configuration templates, and checklists. Knowledge without action is a reference manual, not a skill.
3. **Structured** — Use headers (##, ###), tables, numbered lists, and code blocks. Claude parses structured content more reliably than prose paragraphs.
4. **Size-conscious** — Keep each skill under 10,000 tokens (~7,500 words). If a skill exceeds this, split it into two related skills.
5. **Self-contained** — A skill should make sense on its own. Do not require the user to have read another skill first.

### Multiple Skills Example

```
skills/
├── react-components/
│   └── SKILL.md          # React component patterns
├── react-state/
│   └── SKILL.md          # State management patterns
└── react-testing/
    └── SKILL.md          # Testing React applications
```

Each skill activates independently based on conversation context.

---

## Commands: User-Invoked Slash Protocols

### File Location and Naming

```
commands/
└── my-command.md
```

The filename (minus `.md`) becomes the slash command: `/my-command`.

### Command File Anatomy

```markdown
---
description: "What this command does — shown in command listings"
phase: 1
phase_step: 1.2
phase_name: "DISCOVER"
step_label: "Research Competitors"
---

# /my-command — Research Competitors

## Purpose
One sentence explaining what this command produces.

## Protocol

When the user invokes /my-command, follow this process:

### Step 1: Gather Context
Ask the user for:
- Input A
- Input B

### Step 2: Analyze
Using the gathered context:
1. Do analysis step one
2. Do analysis step two
3. Do analysis step three

### Step 3: Deliver Output
Present results in this format:

**Section Title**
- Finding 1
- Finding 2
- Finding 3

## Output Format
[Define the exact structure of the response]
```

### Frontmatter Fields

| Field | Required | Purpose |
|-------|----------|---------|
| `description` | Yes | Shown when listing available commands |
| `phase` | No | Groups commands into workflow phases (0, 1, 2...) |
| `phase_step` | No | Position within a phase (1.1, 1.2, 1.3...) |
| `phase_name` | No | Human label for the phase (DISCOVER, BUILD, etc.) |
| `step_label` | No | Human label for this specific step |

Phase fields are optional but recommended for plugins that define a multi-step workflow. They help users understand the intended order.

### Command Design Tips

- Write the protocol as instructions TO Claude, not documentation about Claude
- Be explicit: "Ask the user for X" not "The user might provide X"
- Define the output format precisely — Claude follows structure better than vague instructions
- Include conditional logic: "If the user provides a URL, do X. Otherwise, ask for it."

---

## CLAUDE.md: The Plugin Manifest

The CLAUDE.md file at the plugin root is loaded into every Claude Code conversation. This is how Claude discovers your plugin.

### What to Include

```markdown
# My Plugin Name (v1.0.0)

Installed as a Claude Code plugin at `~/.claude/plugins/my-plugin/`.
Skills are auto-invoked by the plugin system based on context.
Commands are user-invocable via `/command-name`.

## Available Skills (N)
- **skill-name-1** -- Description
- **skill-name-2** -- Description

## Available Commands (N)

### Phase 0: SETUP
`/command-a` (0.1), `/command-b` (0.2)

### Phase 1: BUILD
`/command-c` (1.1), `/command-d` (1.2)
```

### CLAUDE.md Rules

- **Keep it under 200 lines.** This file loads into every conversation. Bloated CLAUDE.md files waste context window.
- **List all skills and commands.** Claude needs to know what's available to invoke them.
- **Include the version number.** Users (and Claude) should know which version is active.
- **State the install path.** Helps Claude locate plugin files if needed.
- **Do NOT put skill content here.** CLAUDE.md is an index, not a knowledge dump. Skill content belongs in SKILL.md files.

---

## MCP Server Integration

Plugins can bundle MCP (Model Context Protocol) servers that give Claude access to external tools.

### mcp-config.json

Place at the plugin root:

```json
{
  "mcpServers": {
    "my-tool-server": {
      "command": "node",
      "args": ["./mcp-servers/my-tool/index.js"],
      "env": {
        "API_KEY": "${MY_TOOL_API_KEY}"
      }
    }
  }
}
```

### Referencing in plugin.json

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Plugin with bundled MCP server",
  "author": { "name": "You" },
  "mcpServers": {
    "my-tool-server": {
      "command": "node",
      "args": ["./mcp-servers/my-tool/index.js"]
    }
  }
}
```

### Bundling Pattern

```
~/.claude/plugins/my-plugin/
├── mcp-servers/
│   └── my-tool/
│       ├── index.js           # MCP server entry point
│       ├── package.json
│       └── node_modules/      # Pre-installed dependencies
├── skills/
│   └── my-tool-guide/
│       └── SKILL.md           # Teaches Claude WHEN and HOW to use the tools
├── .claude-plugin/
│   └── plugin.json
└── CLAUDE.md
```

The skill alongside the MCP server is what makes a plugin more valuable than a standalone MCP server. The skill provides context: "Use the `search_documents` tool when the user asks about finding files. Always pass the `recursive: true` flag for directory searches."

---

## Distribution Methods

### 1. Git Clone (Simplest)

```bash
git clone https://github.com/you/my-plugin.git ~/.claude/plugins/my-plugin
```

Pros: One command, version-controlled, easy updates (`git pull`).
Cons: Requires git, user must know the exact path.

### 2. Install Script (Recommended for Public Plugins)

```bash
curl -fsSL https://raw.githubusercontent.com/you/my-plugin/main/install.sh | bash
```

The install script should:
1. Detect the OS (macOS, Linux, WSL)
2. Create `~/.claude/plugins/my-plugin/` if it doesn't exist
3. Download and extract plugin files
4. Verify the installation
5. Print a success message with next steps

Example install script:

```bash
#!/bin/bash
set -euo pipefail

PLUGIN_NAME="my-plugin"
PLUGIN_DIR="$HOME/.claude/plugins/$PLUGIN_NAME"
REPO_URL="https://github.com/you/my-plugin"

echo "Installing $PLUGIN_NAME..."

if [ -d "$PLUGIN_DIR" ]; then
  echo "Updating existing installation..."
  cd "$PLUGIN_DIR" && git pull
else
  echo "Fresh install..."
  git clone "$REPO_URL" "$PLUGIN_DIR"
fi

echo ""
echo "$PLUGIN_NAME installed successfully!"
echo "Restart Claude Code to activate."
```

### 3. npm Package with Postinstall

```json
{
  "name": "claude-plugin-my-plugin",
  "version": "1.0.0",
  "scripts": {
    "postinstall": "node install.js"
  }
}
```

Where `install.js` copies the plugin directory to `~/.claude/plugins/my-plugin/`.

### 4. Release Archive

Provide a `.tar.gz` or `.zip` on GitHub Releases:

```bash
curl -L https://github.com/you/my-plugin/releases/latest/download/my-plugin.tar.gz | tar xz -C ~/.claude/plugins/
```

---

## Uninstall

One command removes any plugin completely:

```bash
rm -rf ~/.claude/plugins/my-plugin
```

There is no registry to update, no config to clean, no dependencies to unwind. The plugin directory IS the installation.

---

## Testing Your Plugin

### Manual Testing Checklist

1. **Restart Claude Code** — Plugins are loaded at startup. After installing or modifying a plugin, restart.
2. **Verify CLAUDE.md loads** — Start a conversation and ask Claude "What plugins are installed?" Your plugin should appear.
3. **Test skill auto-invocation** — Mention keywords from a skill's description. Claude should demonstrate knowledge from that skill.
4. **Test slash commands** — Type `/your-command` and verify Claude follows the protocol defined in the command file.
5. **Test MCP servers** — If your plugin bundles an MCP server, verify the tools appear in Claude's tool list.

### Testing on a Clean Machine

Before distributing, test on a machine that has never seen your plugin:

```bash
# Simulate clean install
rm -rf ~/.claude/plugins/my-plugin
# Run your install method
git clone https://github.com/you/my-plugin.git ~/.claude/plugins/my-plugin
# Restart Claude Code and verify
```

### Common Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| Plugin not recognized | Missing or malformed `plugin.json` | Validate JSON syntax, ensure `name` field is present |
| Skills never activate | Weak description keywords | Add more diverse trigger keywords to the description field |
| Commands not listed | File not in `commands/` directory | Verify path is `commands/my-command.md` (not nested deeper) |
| MCP server fails to start | Missing dependencies | Ensure `node_modules/` is included or run `npm install` in postinstall |
| CLAUDE.md not loading | File not at plugin root | Must be at `~/.claude/plugins/my-plugin/CLAUDE.md` exactly |

---

## Versioning

Follow semantic versioning in `plugin.json`:

| Change Type | Version Bump | Example |
|-------------|-------------|---------|
| Fix typo in a skill | PATCH (1.0.0 -> 1.0.1) | Content correction |
| Add a new skill or command | MINOR (1.0.0 -> 1.1.0) | New capability |
| Rename/remove commands, restructure | MAJOR (1.0.0 -> 2.0.0) | Breaking change |

When bumping the version:
1. Update `version` in `.claude-plugin/plugin.json`
2. Update the version reference in `CLAUDE.md` header
3. Tag the git commit: `git tag v1.1.0`

---

## Best Practices

### CLAUDE.md
- Keep under 200 lines — it loads every conversation
- List skills and commands as a table of contents, not full documentation
- Include the install path so Claude can reference files directly
- Update version number with every release

### Skill Organization
- One domain per skill — do not mix unrelated topics
- Name skill directories semantically: `react-state-management` not `skill-3`
- Front-load the most important content — Claude may truncate long skills
- Include code examples for every concept — abstract advice without examples is ignored

### Command Design
- Define explicit output formats — Claude follows structure better than vague instructions
- Include conditional branching — handle cases where the user provides partial input
- Test commands with minimal input to verify Claude asks for what it needs

### Cross-Platform Compatibility
- Test on macOS, Linux, and WSL (Windows Subsystem for Linux)
- Use `$HOME` not `~` in install scripts (some shells don't expand tilde in scripts)
- Avoid macOS-only commands (e.g., `open`) in install scripts — provide alternatives
- Use `/bin/bash` not `/bin/zsh` in shebangs for maximum compatibility

### Distribution
- Always include a README.md with install instructions
- Provide both git clone and curl install options
- Test the install process from a clean state before publishing
- Pin dependencies if bundling an MCP server — do not rely on global installs

---

## Quick Start Template

To scaffold a new Claude Code plugin:

```bash
PLUGIN="my-plugin"
mkdir -p ~/.claude/plugins/$PLUGIN/.claude-plugin
mkdir -p ~/.claude/plugins/$PLUGIN/skills/my-first-skill
mkdir -p ~/.claude/plugins/$PLUGIN/commands

# Create plugin.json
cat > ~/.claude/plugins/$PLUGIN/.claude-plugin/plugin.json << 'EOF'
{
  "name": "my-plugin",
  "version": "0.1.0",
  "description": "What this plugin does in one sentence",
  "author": {
    "name": "Your Name"
  }
}
EOF

# Create CLAUDE.md
cat > ~/.claude/plugins/$PLUGIN/CLAUDE.md << 'EOF'
# My Plugin (v0.1.0)

Installed as a Claude Code plugin at `~/.claude/plugins/my-plugin/`.

## Available Skills (1)
- **my-first-skill** -- Description of the skill

## Available Commands (0)
None yet.
EOF

# Create first skill
cat > ~/.claude/plugins/$PLUGIN/skills/my-first-skill/SKILL.md << 'EOF'
---
name: my-first-skill
description: "Description here. Use when the user mentions: keyword1, keyword2"
---

# My First Skill

Content goes here.
EOF

echo "Plugin scaffolded at ~/.claude/plugins/$PLUGIN/"
echo "Restart Claude Code to activate."
```

---

## Sources & References

1. **[Claude Code Documentation]** — Anthropic. https://docs.anthropic.com/en/docs/claude-code. Official documentation for Claude Code, including plugin system architecture, CLAUDE.md conventions, skill auto-invocation, and slash command registration.
2. **[Anthropic Documentation]** — Anthropic. https://docs.anthropic.com/. Root documentation portal covering Claude models, API reference, tool use, and the broader Anthropic developer ecosystem that Claude Code plugins operate within.
3. **[Semantic Versioning 2.0.0]** — Tom Preston-Werner. https://semver.org/. The versioning standard used in plugin.json: MAJOR for breaking changes (renamed/removed commands), MINOR for new skills or commands, PATCH for content fixes.
4. **[XDG Base Directory Specification]** — freedesktop.org. https://specifications.freedesktop.org/basedir-spec/latest/. Defines standard directory locations for user configuration (`$XDG_CONFIG_HOME`), data, and cache on Linux systems. Informs the cross-platform config path conventions used by Claude Code and plugin install scripts.
