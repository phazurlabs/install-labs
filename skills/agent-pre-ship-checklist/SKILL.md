---
name: agent-pre-ship-checklist
description: "Pre-ship quality gate for AI agents and automations. Use when the user mentions: pre-ship, checklist, ship check, ready to ship, ready to publish, launch checklist, release checklist, pre-publish, quality gate, go/no-go, agent packaging review, pre-ship check, before release, before publish, packaging audit"
---

# AI Agent Pre-Ship Checklist

A quality gate specifically for AI agents and automations. Every item is about the experience of installing, configuring, running, and maintaining an agent -- not generic software quality. Work through each section before publishing.

Severity levels:
- **CRITICAL** -- Must fix. Shipping without this will cause install failures, security issues, or broken first runs.
- **RECOMMENDED** -- Should fix. Skipping this creates friction that drives users away.
- **NICE** -- Polish. Improves the experience for power users and edge cases.

---

## 1. Agent Install Works

The install is the first impression. If it fails, nothing else matters.

- [ ] **[CRITICAL]** Agent tested on a clean machine (not your dev machine)
  - Use a fresh VM, container, or a colleague's laptop. Your dev machine has tools, paths, and env vars that mask broken installs.
  - Quick test: spin up a Docker container (`docker run -it python:3.12-slim bash`) and try your install steps from scratch.
- [ ] **[CRITICAL]** Install works without prerequisites not listed in docs
  - If it needs Python 3.11+, say so. If it needs `ffmpeg`, say so. Do not assume anything is pre-installed.
  - Audit your implicit dependencies: `ldd` on Linux, `otool -L` on macOS, or check import errors in a bare environment.
- [ ] **[CRITICAL]** First run produces a response (not a crash or silent failure)
  - Run the agent immediately after install with no additional setup beyond what the install docs describe. It must produce output.
  - Test the exact commands from your README. Copy-paste them. Do not type from memory.
- [ ] **[CRITICAL]** API key handling: clear error if missing, guided setup if needed
  - If the agent needs an API key and one is not set, the error must say: what key is missing, where to get it, and how to set it. Never print a raw traceback or generic "unauthorized" error.
  - Test this: unset the key (`unset ANTHROPIC_API_KEY`) and run the agent. Read the error message as a new user would.
- [ ] **[RECOMMENDED]** Install is 1-2 steps (not 10+)
  - Ideal: `pipx install my-agent` then `my-agent --help`. Anything beyond 3 steps needs strong justification.
  - Count your steps honestly. "Clone the repo" is step 1. "cd into it" is step 2. "Create a venv" is step 3. You are already at 3 and the agent is not installed yet.
- [ ] **[RECOMMENDED]** Time from install command to first successful agent response: under 2 minutes
  - Measure this with a stopwatch. Start when the user types the install command. Stop when the agent produces its first useful output. Under 2 minutes is the target.
  - If dependency installation alone takes 90 seconds, you need to reduce the dependency footprint or pre-build wheels.
- [ ] **[NICE]** Works behind corporate firewalls and proxies
  - Respect `HTTPS_PROXY` and `HTTP_PROXY` environment variables. Do not hard-fail on certificate verification without a clear message.
  - Test with: `HTTPS_PROXY=http://localhost:9999 my-agent ask "test"` -- it should fail with a clear proxy/network error, not a cryptic SSL exception.

---

## 2. Configuration

AI agents have unique configuration needs: API keys, model selection, token budgets, and provider fallbacks.

- [ ] **[CRITICAL]** API keys via environment variables (not hardcoded)
  - Never commit API keys to source, bake them into Docker images, or include them in published packages. Environment variables are the universal standard.
  - Verify: `grep -rn "sk-ant-\|sk-proj-\|AKIA" src/` should return zero results.
- [ ] **[CRITICAL]** `.env.example` file exists with every required variable documented
  - Each variable gets a comment explaining what it is, where to get it, and an example value. Include both required and optional variables, clearly labeled.
  - Template: `ANTHROPIC_API_KEY=sk-ant-...  # Required. Get at https://console.anthropic.com/`
- [ ] **[CRITICAL]** Missing config produces helpful error message
  - Bad: `KeyError: 'ANTHROPIC_API_KEY'`
  - Bad: `Error: unauthorized`
  - Good: `ANTHROPIC_API_KEY is not set. Get one at https://console.anthropic.com/ then run: export ANTHROPIC_API_KEY=sk-ant-...`
  - Test every required variable: unset each one individually and confirm the error message is helpful.
- [ ] **[RECOMMENDED]** Sensible defaults for all non-secret configuration
  - Model name, temperature, max tokens, timeout, retry count -- all should have defaults that work. The user should only need to set secrets, not tune parameters, for a first run.
  - Verify: delete your config file and run the agent with only API keys set. It should still work.
- [ ] **[RECOMMENDED]** Config file location follows platform conventions
  - macOS: `~/Library/Application Support/my-agent/` or `~/.config/my-agent/`
  - Linux: `~/.config/my-agent/` (XDG spec)
  - Windows: `%APPDATA%\my-agent\`
  - Respect `XDG_CONFIG_HOME` if set. Never scatter dotfiles in `$HOME` without reason.
- [ ] **[NICE]** Supports multiple AI providers with fallback
  - If Anthropic is unavailable, can the agent fall back to OpenAI or a local model? This is not always needed but is valuable for reliability-critical agents.
  - At minimum, make the model name configurable so users can swap providers without editing source code.

---

## 3. Documentation

The README is the install experience. Everything above the fold must get the user to a working agent.

- [ ] **[CRITICAL]** Install instructions in README, above the fold
  - The first thing a user sees when they open the repo must be how to install it. Not the architecture diagram, not the feature list, not the badge wall. Install instructions first.
  - Test: open your GitHub repo in an incognito browser window. Can you see install instructions without scrolling?
- [ ] **[CRITICAL]** One primary install method (not 5 alternatives with no guidance)
  - Pick the best method for your audience and lead with it. Alternatives go in a collapsible section or a separate doc. Decision fatigue kills installs.
  - If you support pip, pipx, uvx, conda, and Docker, pick ONE as primary. The rest go under "Alternative installation methods."
- [ ] **[CRITICAL]** Prerequisites listed with version numbers and links
  - Bad: "Requires Python"
  - Good: "Requires [Python 3.11+](https://python.org/downloads/). Check with: `python3 --version`"
  - Include the verification command so users can self-diagnose before filing an issue.
- [ ] **[RECOMMENDED]** Quick start gets to first successful response in under 5 minutes
  - A "Quick Start" section with numbered steps: install, configure API key, run first command, see output. Four steps maximum.
  - Show the expected output so users know what success looks like.
- [ ] **[RECOMMENDED]** Uninstall instructions documented
  - Users need to know they can cleanly remove your agent. This builds trust. Include what files/dirs to remove and what config to clean up.
  - List everything: the binary, config directory, cache files, log files, any database files.
- [ ] **[NICE]** Troubleshooting section for top 3 install failures
  - Check your GitHub Issues. What do people actually fail on? Document the fix for the top 3.
  - Common ones for agents: wrong Python version, missing API key, firewall blocking API calls, conflicting package versions.

---

## 4. Error Handling

AI agents have failure modes that traditional software does not: rate limits, model deprecation, token budget exhaustion, and API outages.

- [ ] **[CRITICAL]** Missing API key: clear message with link to get one
  - This is the single most common first-run failure for AI agents. The error message must be perfect.
  - Pattern: `Error: ANTHROPIC_API_KEY not set. Get one at https://console.anthropic.com/ then run: export ANTHROPIC_API_KEY=sk-ant-...`
- [ ] **[CRITICAL]** Network failure: human-readable message
  - Bad: `ConnectionError: HTTPSConnectionPool(host='api.anthropic.com', port=443)`
  - Good: `Could not reach the Anthropic API. Check your internet connection and try again.`
- [ ] **[CRITICAL]** Model not found or deprecated: suggest alternative
  - Models get renamed and retired. If the configured model is unavailable, say which model to use instead.
  - Pattern: `Model "claude-2" is no longer available. Update to "claude-sonnet-4-20250514" in your config.`
- [ ] **[RECOMMENDED]** Rate limit: automatic retry with user feedback
  - Pattern: `Rate limited by the API. Retrying in 15 seconds... (attempt 2 of 3)`
  - Do not silently hang. Do not crash. Retry with backoff and tell the user what is happening.
- [ ] **[RECOMMENDED]** All errors include what happened AND what to do next
  - Every error message is two parts: the problem ("Could not connect to Redis") and the action ("Make sure Redis is running: docker compose up redis").
- [ ] **[NICE]** Errors include searchable error codes
  - Pattern: `[MY-AGENT-E004] Model not found.` Users can search your docs or Issues for the code.

---

## 5. Security

AI agents handle API keys, user data, and often have broad system access. Security is not optional.

- [ ] **[CRITICAL]** No API keys in source code, Docker images, or published packages
  - Search your repo: `grep -rn "sk-ant-\|sk-proj-\|AKIA\|password\s*=" src/`
  - Search your Docker image: `docker history --no-trunc <image>` and look for ARG/ENV lines with secrets.
  - Search your published package: `pip download my-agent && unzip *.whl && grep -r "sk-" .` or `npm pack && tar xzf *.tgz && grep -r "sk-" package/`
- [ ] **[CRITICAL]** No secrets logged or displayed in output
  - API keys must never appear in logs, error messages, or debug output. Mask them: `sk-ant-...xxxx`.
  - Test: set `LOG_LEVEL=debug` and run the agent. Read every line of output. If you see a key, fix it.
- [ ] **[CRITICAL]** Dependencies pinned to exact or minimum versions (no `*` or `latest`)
  - `"anthropic": "*"` in package.json or `anthropic` with no version in requirements.txt means any version can be installed, including compromised ones.
  - Use `pip freeze > requirements.txt` or `npm shrinkwrap` to lock exact versions for reproducible builds.
- [ ] **[CRITICAL]** `.gitignore` covers secrets and build artifacts
  - Must include: `.env`, `*.key`, `*.pem`, `credentials.json`, `dist/`, `node_modules/`, `__pycache__/`, `.venv/`
  - Verify: `git status` should never show secret files as untracked.
- [ ] **[RECOMMENDED]** Model files and weights from trusted sources with checksum verification
  - If your agent downloads model files at install or runtime, verify SHA-256 checksums. Do not download arbitrary files from URLs without verification.
  - Use Hugging Face Hub, Ollama, or other registries that provide integrity checks.
- [ ] **[RECOMMENDED]** Minimum permissions: agent accesses only what it needs
  - If the agent only needs to read files in one directory, do not give it access to the entire filesystem. If it only needs one API scope, do not request all scopes.
  - For MCP servers: declare only the tools and resources the agent actually provides.
- [ ] **[NICE]** SBOM or dependency manifest published with the release
  - A Software Bill of Materials lets security teams audit your supply chain. Generate one with `syft`, `cdxgen`, or `npm sbom`.
  - Attach the SBOM as a release artifact alongside your binaries.

---

## 6. Updates and Uninstall

Agents evolve fast. Models get deprecated, APIs change, and new capabilities ship weekly. Users need to update easily and leave cleanly.

- [ ] **[CRITICAL]** Version number accessible via `--version` or equivalent
  - Every agent must respond to a version query. This is the first thing support will ask for. It should match the published version exactly.
  - For MCP servers: include version in the server metadata. For Docker: use image tags. For plugins: version in plugin.json.
- [ ] **[RECOMMENDED]** Update path documented
  - Tell users exactly how to update. `pipx upgrade my-agent`, `npm update -g my-agent`, `docker pull my-org/my-agent:latest`. One line in the README.
  - If the update requires migration steps (new env vars, config format changes), document those alongside the update command.
- [ ] **[RECOMMENDED]** Uninstall is one command and removes everything
  - `pipx uninstall my-agent`, `npm uninstall -g my-agent`, `docker compose down -v`. If there are config files or caches to clean up, list them.
  - Test it: install, run, uninstall, then search for leftover files. Nothing should remain unless the user explicitly chose to keep config.
- [ ] **[RECOMMENDED]** Breaking changes in model or API dependencies are communicated proactively
  - If you update the default model from claude-sonnet to claude-opus, users need to know -- their costs and latency will change.
  - Use semantic versioning: model changes that affect cost or behavior are breaking changes (major version bump).
- [ ] **[NICE]** Changelog linked from update notifications
  - When the agent detects a new version is available, link to the changelog so the user can decide whether to update.

---

## 7. CI/CD

AI agents must be built and published by automation, not by a developer running `npm publish` on their laptop.

- [ ] **[CRITICAL]** Package built by CI (not on a developer's laptop)
  - Laptop builds are unreproducible and may leak secrets or include dev-only files. Use GitHub Actions, GitLab CI, or equivalent.
  - Verify: check your latest release. Was it built by a CI bot or by a human account?
- [ ] **[RECOMMENDED]** Install smoke test in CI
  - After building the package, install it in a clean environment and run a basic command. Verify it produces output. This catches packaging errors before users hit them.
  - Python pattern: `pip install dist/*.whl && my-agent --version && my-agent ask "test" --dry-run`
  - Node pattern: `npm install -g ./my-agent-*.tgz && my-agent --version`
  - Docker pattern: `docker build -t test . && docker run --rm test --version`
- [ ] **[RECOMMENDED]** Automated publishing on tag
  - Push a version tag (`v1.2.3`), CI builds and publishes. No manual steps. This eliminates "forgot to publish" and "published the wrong version" errors.
  - Use GitHub Actions `on: push: tags: ["v*"]` trigger with npm provenance or PyPI trusted publishers.
- [ ] **[RECOMMENDED]** CI tests with the actual AI provider mocked or in dry-run mode
  - You cannot call the Anthropic API in every CI run (cost, rate limits, flakiness). Use `--dry-run` flags, mock responses, or recorded cassettes.
  - But do run a live integration test on release tags, not on every PR.
- [ ] **[NICE]** Post-publish verification
  - After publishing to the registry, install from the registry (not from local build) and verify it works. This catches registry-specific issues like missing files in the published package.
  - Pattern: add a delayed CI job that waits 60 seconds after publish, then runs `pip install my-agent==X.Y.Z && my-agent --version`.

---

## Quick Pass/Fail

Five questions that determine if your agent is ready to ship. If any answer is "no," stop and fix it.

| # | Question | Target |
|---|----------|--------|
| 1 | Can someone install your agent in under 2 steps? | Yes |
| 2 | Does the first run produce a useful response (not a crash)? | Yes |
| 3 | Do errors say what went wrong AND what to do about it? | Yes |
| 4 | Are all secrets in environment variables (not in code or images)? | Yes |
| 5 | Have you tested on a machine that is not yours? | Yes |

If all five are "yes," your agent is ready for early users. The full checklist above is for production readiness.

---

## How to Use This Checklist

**Before first publish:** Work through every CRITICAL item. Fix all of them. No exceptions. There are 16 CRITICAL items across 7 sections. Expect this to take 2-4 hours for a well-structured agent, or 1-2 days if you need to restructure packaging.

**Before each release:** Re-run the Quick Pass/Fail (5 questions, 2 minutes). Spot-check 2-3 RECOMMENDED items, rotating through different sections each release.

**Quarterly:** Full audit of all items including NICE-to-have. These accumulate into the difference between a tool people tolerate and one they recommend.

**When onboarding a new team member:** Have them install the agent using only the README on their machine. Watch them do it. Every hesitation, confusion, or failure is a checklist item that needs attention.

**Scoring:**
- All CRITICAL items passing = **Ship it** (with known gaps documented)
- All CRITICAL + RECOMMENDED items passing = **Confident release**
- All items passing = **Best-in-class agent packaging**

**Item counts by severity:**
- CRITICAL: 16 items (the floor -- all must pass)
- RECOMMENDED: 15 items (the standard -- most should pass)
- NICE: 8 items (the ceiling -- tackle over time)

---

## Common Agent-Specific Failures

These are the packaging failures unique to AI agents that this checklist is designed to catch. Each one maps back to specific checklist items.

1. **The "works on my machine" API key** -- Developer has `ANTHROPIC_API_KEY` in their shell profile. Agent works for them, crashes for everyone else with a raw `AuthenticationError`.
   - Caught by: Section 1 (clean machine test) + Section 2 (missing config error) + Section 4 (API key error message)

2. **The silent model change** -- Agent is pinned to a model that gets deprecated. No error, just a 404 from the API that surfaces as an inscrutable stack trace.
   - Caught by: Section 4 (model not found error) + Section 6 (breaking changes communication)

3. **The 20-step install** -- Clone repo, install Python, create venv, install deps, install Postgres, create database, run migrations, set 8 env vars, download model weights, configure CUDA drivers. User gives up at step 4.
   - Caught by: Section 1 (1-2 step install, 2-minute target) + Section 3 (one primary install method)

4. **The leaking Docker image** -- API key passed as a build arg and baked into a layer. Anyone who pulls the image can extract the key with `docker history`.
   - Caught by: Section 5 (no keys in Docker images) + Section 2 (keys via environment variables)

5. **The phantom dependency** -- Agent imports a package that is installed globally on the developer's machine but is not listed in `requirements.txt`. Passes CI (which has the same global installs). Fails on every user machine.
   - Caught by: Section 1 (clean machine test) + Section 7 (install smoke test in CI)

6. **The hanging agent** -- API call times out but there is no timeout configured. Agent hangs forever with no output. User has no idea if it is working, stuck, or dead.
   - Caught by: Section 4 (network failure message) + Section 2 (sensible defaults for timeout)

7. **The cost surprise** -- Agent defaults to the most expensive model. User runs 50 test queries and gets a $40 API bill. No cost warning, no token budget, no dry-run mode.
   - Caught by: Section 2 (sensible defaults) + Section 6 (breaking changes for model swaps)

8. **The abandoned README** -- Install instructions reference a deprecated flag, a renamed package, or a removed endpoint. Nobody has tested the README in 6 months.
   - Caught by: Section 3 (install instructions) + Section 7 (install smoke test in CI that follows README steps)

---

## Verification Commands Cheat Sheet

Quick commands to verify specific checklist items without reading through each section:

```bash
# --- Secrets scan ---
grep -rn "sk-ant-\|sk-proj-\|AKIA\|password\s*=\|secret\s*=" src/ --include="*.py" --include="*.ts" --include="*.js"

# --- .env.example completeness ---
# Compare required vars in code vs documented vars in .env.example
grep -roh "os.environ\[.*?\]\|process.env\.\w+" src/ | sort -u

# --- Dependency pinning check (Python) ---
grep -E "^[a-zA-Z].*[^=]$" requirements.txt  # Lines without version pins

# --- Dependency pinning check (Node) ---
grep '"*"' package.json  # Star versions
grep '"latest"' package.json  # Latest versions

# --- Docker secret leak check ---
docker history --no-trunc my-agent:latest | grep -i "key\|secret\|password\|token"

# --- Version accessible ---
my-agent --version  # Should print a semver string

# --- Clean install test (Docker) ---
docker run --rm -it python:3.12-slim bash -c "pip install my-agent && my-agent --version"
```

---

## Sources & References

1. **[OWASP Dependency-Check]** — OWASP Foundation. https://owasp.org/www-project-dependency-check/ Open-source software composition analysis tool that identifies known vulnerabilities in project dependencies.
2. **[The Checklist Manifesto: How to Get Things Right]** — Atul Gawande (2009). Metropolitan Books. Foundational work on checklist methodology applied to high-stakes processes; informs the structure and severity-based approach of this pre-ship checklist.
3. **[Release It! Design and Deploy Production-Ready Software]** — Michael T. Nygard (2018). Pragmatic Bookshelf. Production readiness patterns including circuit breakers, timeouts, health checks, and failure mode analysis applied to agent packaging.
4. **[The Twelve-Factor App]** — Adam Wiggins. https://12factor.net/ Methodology for building software-as-a-service applications, informing the configuration (env vars), dependency isolation, and build/release/run separation in this checklist.
5. **[pip-audit]** — Python Packaging Authority (PyPA). https://github.com/pypa/pip-audit Tool for scanning Python environments and packages for known security vulnerabilities using the OSV database.
6. **[npm audit]** — npm, Inc. https://docs.npmjs.com/cli/commands/npm-audit Built-in npm command for scanning Node.js dependencies for known security vulnerabilities, referenced in the dependency pinning and security sections.
7. **[Semantic Versioning 2.0.0]** — Tom Preston-Werner. https://semver.org/ The versioning specification referenced throughout the checklist for version accessibility, breaking change communication, and release tagging.
