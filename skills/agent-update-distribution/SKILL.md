---
name: agent-update-distribution
description: "Update mechanisms and distribution strategies for AI agents and automations. Use when the user mentions: update agent, upgrade agent, agent versioning, semantic versioning agents, auto-update, rollback agent, migration, breaking changes, distribute updates, agent release, MCP update, plugin update, docker update, npm update agent, pip upgrade agent, agent rollback, version pinning"
---

# Agent Update & Distribution

## MCP Server Updates

### npx Always-Latest Pattern

```json
{ "mcpServers": { "my-agent": { "command": "npx", "args": ["-y", "@your-org/my-agent-mcp"] } } }
```

**Pros:** Users always get the latest. No update command needed.
**Cons:** Breaking changes hit immediately. No offline support. Slower startup.

### Pinned Install

```bash
npm install -g @your-org/my-agent-mcp@2.1.0   # exact version
npm update -g @your-org/my-agent-mcp           # update explicitly
```

### Version Pinning in MCP Config

- `@2.1.0` -- exact version, never auto-updates
- `@^2.0.0` -- allows minor/patch within major 2
- `@latest` or omitted -- always fetches newest

---

## Claude Code Plugin Updates

### Git-Based Plugins

```bash
cd ~/.claude/plugins/my-agent-plugin
git pull origin main
./install.sh                   # if plugin has install script
npm install                    # if it has JS dependencies
```

### Versioning in plugin.json

```json
{ "name": "my-agent-plugin", "version": "2.3.1", "minClaudeCodeVersion": "1.5.0" }
```

Bump `version` on every release. Set `minClaudeCodeVersion` to block incompatible hosts. Tag in git: `git tag v2.3.1 && git push --tags`.

### Self-Updating Plugin Pattern

```bash
#!/bin/bash
# commands/update/run.sh
PLUGIN_DIR="$(dirname "$(dirname "$(realpath "$0")")")"
cd "$PLUGIN_DIR"
echo "Current: $(jq -r .version plugin.json)"
git fetch origin main
LOCAL=$(git rev-parse HEAD); REMOTE=$(git rev-parse origin/main)
if [ "$LOCAL" = "$REMOTE" ]; then echo "Up to date."
else git pull origin main; echo "Updated to: $(jq -r .version plugin.json)"; fi
```

---

## Docker Agent Updates

### Image Tagging Strategy

```bash
docker build -t my-agent:2.3.1 -t my-agent:2.3 -t my-agent:2 -t my-agent:latest .
```

- `2.3.1` -- immutable, never changes after push
- `2.3` / `2` -- mutable, points to latest in that range
- `latest` -- newest release

### Docker Compose Update Flow

```bash
docker compose pull && docker compose up -d
docker compose logs --tail 20 my-agent   # verify
```

### Migration Scripts in Docker

```bash
#!/bin/bash
# migrate.sh — runs before agent starts
CURRENT=$(cat /app/data/.version 2>/dev/null || echo "0.0.0")
TARGET=$(jq -r .version /app/package.json)
if [ "$CURRENT" != "$TARGET" ]; then
  python /app/migrations/run.py "$CURRENT" "$TARGET"
  echo "$TARGET" > /app/data/.version
fi
```

```dockerfile
COPY migrations/ /app/migrations/
CMD ["sh", "-c", "./migrate.sh && python agent.py"]
```

---

## PyPI Agent Updates

```bash
pip install --upgrade my-agent          # latest
pip install my-agent==2.3.1             # specific version
uv tool upgrade my-agent                # if installed via uv
```

### Version Constraints

```txt
my-agent==2.3.1        # exact pin (production)
my-agent~=2.3.0        # >=2.3.0, <2.4.0 (allow patches)
my-agent>=2.3,<3       # any 2.x (development)
```

---

## npm Agent Updates

```bash
npm update -g @your-org/my-agent       # update global install
npm install -g @your-org/my-agent@2.3.1  # specific version
npx --yes @your-org/my-agent@latest    # force fresh download
```

`npx` caches packages. Use `--yes` with `@latest` to guarantee a fresh fetch.

---

## Custom GPT Updates

No package manager -- updates go through the OpenAI builder interface.

### Versioning Instructions

Embed version in the GPT's system instructions:
```
You are MyAgent v2.3.1 (2026-02-15).
CHANGELOG:
- v2.3.1: Fixed date parsing
- v2.3.0: Added CSV export
```

### Knowledge File Refresh

```bash
python build_knowledge.py --output knowledge_v2.3.1.json
# Upload via https://chatgpt.com/gpts/editor/g-xxx
# Update instructions version number, save, publish
```

Automate knowledge file generation. Keep the manual upload step as small as possible.

---

## Migration Patterns

### Config Schema Migration

```python
def migrate_v1_to_v2(config_path="~/.my-agent/config.yaml"):
    """v1 used 'api_key', v2 uses 'credentials.anthropic_key'"""
    config = load_yaml(config_path)
    shutil.copy(config_path, config_path + ".v1.backup")   # backup first
    if "api_key" in config:
        config.setdefault("credentials", {})["anthropic_key"] = config.pop("api_key")
    save_yaml(config_path, config)
```

### Migration Registry

```python
MIGRATIONS = {
    "1.0.0 -> 2.0.0": migrate_v1_to_v2,
    "2.0.0 -> 3.0.0": migrate_v2_to_v3,
}

def migrate(current, target):
    for step in find_migration_path(current, target):
        MIGRATIONS[step]()
    save_version(target)
```

**Always:** Back up before migrating. Print each step. Provide `my-agent doctor` to verify success. Document in release notes.

---

## Semantic Versioning for Agents

### MAJOR (Breaking)
- Config format changes incompatibly
- Required env vars renamed or removed
- Tool interface changes (different inputs/outputs)
- Minimum runtime version increases
- Data/memory format changes incompatibly

### MINOR (Feature)
- New tools or capabilities added
- New optional config fields (with defaults)
- Support for additional model providers

### PATCH (Fix)
- Bug fixes, prompt improvements (same interface)
- Non-breaking dependency bumps
- Performance improvements

### Pre-release Tags
```
2.3.1-alpha.1   # Incomplete, early feedback
2.3.1-beta.1    # Feature-complete, not fully tested
2.3.1-rc.1      # Believed ready, final testing
```

---

## Auto-Update Patterns

### Check on Startup

```python
import requests
from packaging import version

def check_for_updates():
    try:
        resp = requests.get(UPDATE_CHECK_URL, timeout=3)
        latest = resp.json()["tag_name"].lstrip("v")
        if version.parse(latest) > version.parse(CURRENT_VERSION):
            print(f"Update available: v{latest}. Run: pip install --upgrade my-agent")
    except Exception:
        pass  # Never block startup for update checks
```

**Rules:** Timeout at 2-3s. Check at most once per day. Never auto-install without consent. Allow disabling: `MY_AGENT_NO_UPDATE_CHECK=1`.

### Update Strategy Matrix

| Strategy | When to Use |
|---|---|
| Silent notice | Default for CLI tools and plugins |
| Prompt to update | Critical security fix available |
| Block until updated | Only if old version risks others' security |
| Auto-update background | Docker/container deployments you control |

---

## Rollback Strategies

### Package Manager Rollback

```bash
pip install my-agent==2.2.0                    # pip
npm install -g @your-org/my-agent@2.2.0        # npm
uv tool install my-agent==2.2.0                # uv
```

### Config Backup Before Update

```bash
#!/bin/bash
BACKUP_DIR="$HOME/.my-agent/backups/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"
cp ~/.my-agent/config.yaml "$BACKUP_DIR/"
cp ~/.my-agent/.env "$BACKUP_DIR/" 2>/dev/null
echo "Backup: $BACKUP_DIR"
```

### Docker Rollback

```bash
docker tag my-agent:latest my-agent:known-good   # save current
docker pull my-agent:latest && docker compose up -d  # update
# If broken:
docker tag my-agent:known-good my-agent:latest && docker compose up -d
```

### Git Plugin Rollback

```bash
cd ~/.claude/plugins/my-agent-plugin
git log --oneline --tags     # see versions
git checkout v2.2.0          # rollback
git checkout main            # return to latest
```

### Automated Rollback on Failure

```bash
#!/bin/bash
CURRENT=$(my-agent --version)
pip install --upgrade my-agent
if my-agent doctor --quiet; then
  echo "Updated to $(my-agent --version)"
else
  echo "Health check failed. Rolling back..."
  pip install "my-agent==$CURRENT"
  exit 1
fi
```

**Golden rule:** Every update path must have a corresponding rollback path. If you can't roll back, you can't safely update.

---

## Sources & References

1. **[Semantic Versioning 2.0.0]** — Tom Preston-Werner. https://semver.org/. The versioning standard defining MAJOR.MINOR.PATCH semantics, pre-release tags, and build metadata. The foundation for all agent version numbering and compatibility signaling.
2. **[npm-update]** — npm, Inc. https://docs.npmjs.com/cli/commands/npm-update. Official documentation for the `npm update` command, including behavior with global installs (`-g`), semver range resolution, and interaction with `package-lock.json`.
3. **[pip install — Upgrade]** — Python Packaging Authority (PyPA). https://pip.pypa.io/en/stable/cli/pip_install/. Reference for `pip install --upgrade`, version constraint syntax (`==`, `~=`, `>=`), and dependency resolution behavior during agent upgrades.
4. **[Docker Tagging Best Practices]** — Docker, Inc. https://docs.docker.com/build/building/tags/. Guidelines for image tagging strategies including immutable tags (exact version), mutable tags (major/minor), and the `latest` tag convention used in agent container distribution.
5. **[The Twelve-Factor App — Factor V: Build, Release, Run]** — Adam Wiggins / Heroku. https://12factor.net/build-release-run. Strictly separating the build, release, and run stages enables clean rollbacks and reproducible deployments — directly applicable to agent update and migration workflows.
