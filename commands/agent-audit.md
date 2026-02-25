---
description: "Packaging readiness audit — score an AI agent across 6 dimensions before packaging it"
phase: "0"
phase_step: "0.2"
phase_name: "ASSESS"
step_label: "Packaging Readiness Audit"
---

# /agent-audit — Packaging Readiness Audit

## Purpose

Audit an AI agent or automation for packaging readiness BEFORE generating any packaging files. Identifies structural blockers, dependency gaps, configuration problems, and documentation holes that would cause install failures if shipped as-is.

---

## Protocol

When the user invokes `/agent-audit`, follow this process exactly:

### Step 1: Identify the Agent to Audit

Ask the user to point to their agent. Accept any of these:

- **A local directory path** — Read the directory structure, key files (`requirements.txt`, `package.json`, `pyproject.toml`, `Dockerfile`, `.env.example`, `README.md`, entry point files), and infer the agent's architecture.
- **A GitHub repository URL** — Clone or browse the repo to examine the same files.
- **A verbal description** — If the user describes their agent without pointing to code, audit based on what they describe. Flag anything you cannot verify as "Unable to assess — no code provided."
- **Context from a prior `/agent-guide` conversation** — If the user already ran `/agent-guide` in this session, use the assessment from that step. Do not re-ask questions already answered.

**If examining code,** look for these files to understand the agent:

| File | What It Tells You |
|------|-------------------|
| `requirements.txt` / `pyproject.toml` / `Pipfile` | Python dependencies, version pinning |
| `package.json` / `package-lock.json` | Node.js dependencies, scripts, entry point |
| `Dockerfile` / `docker-compose.yml` | Containerization strategy |
| `.env` / `.env.example` | Configuration and secrets pattern |
| `.gitignore` | Whether secrets and build artifacts are excluded |
| `README.md` | Documentation quality |
| `crew.yaml` / `agents.yaml` / `tasks.yaml` | CrewAI configuration |
| `langgraph.json` | LangGraph configuration |
| `main.py` / `app.py` / `index.ts` / `server.ts` | Entry point clarity |
| `tests/` directory | Test coverage |
| `Makefile` / `justfile` / `taskfile.yml` | Task runner / build system |

### Step 2: Run the 6-Dimension Readiness Check

Evaluate the agent across each dimension. For every dimension, examine the evidence, assign a score (1-5), and write a specific finding.

---

#### Dimension 1: Structure

**What to check:**
- Is there a single, clear entry point? Can you identify the file that starts the agent in under 10 seconds?
- Is agent logic separated from infrastructure code? (business logic vs. server setup, CLI parsing, config loading)
- Is the project organized into logical modules or is everything in one file?
- Are there framework-specific config files in the right locations? (e.g., `crew.yaml` at root for CrewAI)

**Scoring criteria:**

| Score | Criteria |
|-------|----------|
| 5 | Single clear entry point, modular structure, agent logic cleanly separated from infrastructure, follows framework conventions |
| 4 | Entry point identifiable, mostly organized, minor coupling between agent logic and infrastructure |
| 3 | Entry point exists but buried or ambiguous, some organization but mixed concerns in files |
| 2 | Multiple potential entry points with no clear primary, agent logic entangled with config/infra, flat file structure |
| 1 | No identifiable entry point, everything in one monolithic file or scattered across directories with no pattern |

---

#### Dimension 2: Dependencies

**What to check:**
- Are all dependencies declared in a manifest file (`requirements.txt`, `package.json`, `pyproject.toml`)?
- Are versions pinned to exact versions or at least bounded ranges?
- Are there undeclared system-level dependencies? (ffmpeg, Chrome/Chromium, CUDA, system Python packages)
- Are there large ML model files that need to be downloaded? How large?
- Are there conflicting dependency versions? (e.g., two packages requiring different versions of the same library)
- Is there a lockfile? (`package-lock.json`, `poetry.lock`, `uv.lock`, `Pipfile.lock`)

**Scoring criteria:**

| Score | Criteria |
|-------|----------|
| 5 | All deps declared and pinned, lockfile present, no system-level deps (or they are documented), no large model downloads during install |
| 4 | Deps declared with bounded ranges (e.g., `>=1.0,<2.0`), lockfile present, one or two system deps documented |
| 3 | Deps declared but not pinned (e.g., `langchain` with no version), no lockfile, some system deps undocumented |
| 2 | Partial dep declaration (some imports not in requirements), no pinning, multiple undocumented system deps |
| 1 | No dependency manifest at all, or `pip install` / `npm install` fails on a clean machine |

---

#### Dimension 3: Configuration

**What to check:**
- How are API keys handled? Environment variables, `.env` file, hardcoded in source, config file, keychain?
- Is there a `.env.example` or equivalent showing required configuration?
- Is `.env` in `.gitignore`? Has `.env` ever been committed to git history?
- Are configuration values documented? (what each key does, where to get it, whether it is required or optional)
- How many separate config files does the user need to create or edit before first run?
- Are there hardcoded paths, URLs, or credentials anywhere in the source?

**Scoring criteria:**

| Score | Criteria |
|-------|----------|
| 5 | All secrets via env vars or keychain, `.env.example` with documented keys, `.env` gitignored, zero hardcoded values, single config source |
| 4 | Secrets via env vars, `.env.example` exists but some keys undocumented, `.env` gitignored, no hardcoded secrets |
| 3 | Secrets in `.env` but no `.env.example`, or config split across 2-3 files, some keys undocumented |
| 2 | Some hardcoded values (URLs, model names) mixed with env vars, no `.env.example`, config requires manual creation |
| 1 | API keys hardcoded in source, secrets committed to git history, or config so scattered the user cannot figure out what to set |

---

#### Dimension 4: Error Handling

**What to check:**
- What happens when a required API key is missing? (crash with stack trace, graceful error message, silent failure)
- What happens when the network is unavailable? (timeout handling, retry logic, offline fallback)
- What happens when a required model file is not found?
- What happens when a dependency service is down? (vector store, database, external API)
- Are error messages actionable? Do they tell the user what to do, or just what went wrong?
- Is there a health check or doctor command?

**Scoring criteria:**

| Score | Criteria |
|-------|----------|
| 5 | All failure modes produce actionable error messages with fix instructions, health check command exists, graceful degradation where possible |
| 4 | Most failure modes handled with useful messages, no health check but errors are clear |
| 3 | Some errors are handled, others produce raw stack traces, missing key errors are caught but network failures are not |
| 2 | Most errors produce stack traces or cryptic messages, silent failures in some paths |
| 1 | Agent crashes on any missing dependency with no useful output, or silently does nothing when misconfigured |

---

#### Dimension 5: Documentation

**What to check:**
- Does a README exist?
- Does the README show what the agent does in the first 10 seconds of reading? (screenshot, demo GIF, example output)
- Are install instructions present and correct?
- Are prerequisites listed? (Python version, Node.js version, required API keys, system tools)
- Is there a "Quick Start" or "Getting Started" section?
- Are all configuration options documented?

**Scoring criteria:**

| Score | Criteria |
|-------|----------|
| 5 | README with demo/screenshot, clear install steps, prerequisites listed, quick start section, all config documented, troubleshooting section |
| 4 | README with install steps and prerequisites, quick start works, most config documented |
| 3 | README exists but install steps are incomplete or assume prior knowledge, some prerequisites missing |
| 2 | README is a stub (project name + one sentence) or install instructions are wrong/outdated |
| 1 | No README, or README contains only auto-generated boilerplate with no real content |

---

#### Dimension 6: Testability

**What to check:**
- Can you start the agent with a single command? What is that command?
- Is there a smoke test or health check that verifies the agent works end-to-end?
- Are there automated tests? (unit tests, integration tests)
- Can the agent run in a "dry run" or "demo mode" without real API keys?
- How long does it take from `git clone` to first successful run? (minutes, not hours)

**Scoring criteria:**

| Score | Criteria |
|-------|----------|
| 5 | One-command start, automated tests exist, health check verifies all connections, demo mode available, clone-to-run under 2 minutes |
| 4 | One-command start, some tests exist, no demo mode but smoke test works with real keys, clone-to-run under 5 minutes |
| 3 | Start requires 2-3 commands, no automated tests but manual testing works, clone-to-run under 10 minutes |
| 2 | Start requires multiple commands and manual config, no tests, clone-to-run takes 10-30 minutes |
| 1 | Cannot determine how to start the agent without reading the entire codebase, no tests, clone-to-run exceeds 30 minutes or fails |

---

### Step 3: Present the Scores Table

Display the scores in a clear summary table:

```
┌───────────────────────────────────────────────────────────────┐
│                  PACKAGING READINESS AUDIT                    │
│                  Agent: [agent name or description]           │
├─────────────────┬───────┬─────────────────────────────────────┤
│ Dimension       │ Score │ Summary                             │
├─────────────────┼───────┼─────────────────────────────────────┤
│ Structure       │  X/5  │ [one-line finding]                  │
│ Dependencies    │  X/5  │ [one-line finding]                  │
│ Configuration   │  X/5  │ [one-line finding]                  │
│ Error Handling  │  X/5  │ [one-line finding]                  │
│ Documentation   │  X/5  │ [one-line finding]                  │
│ Testability     │  X/5  │ [one-line finding]                  │
├─────────────────┼───────┼─────────────────────────────────────┤
│ TOTAL           │ XX/30 │                                     │
└─────────────────┴───────┴─────────────────────────────────────┘
```

### Step 4: Assign the Overall Verdict

Based on the total score:

| Total Score | Verdict | Meaning |
|---|---|---|
| 25-30 | **READY** | You can start packaging now. Minor polish only. |
| 18-24 | **ALMOST** | A few targeted fixes and you are ready. Address items below before packaging. |
| 10-17 | **NOT YET** | Significant gaps. Fix the items below first or packaging will fail for your users. |
| 0-9 | **RETHINK** | Fundamental issues. The agent needs restructuring before packaging is viable. |

Display the verdict prominently:

```
VERDICT: ALMOST (22/30)
```

### Step 5: Generate the Fix List

For every dimension that scored below 4, generate a concrete fix list. Each fix must be:
- **Specific** — Name the exact file to create, variable to rename, or pattern to adopt
- **Actionable** — Provide the command to run or the code to write
- **Ordered by impact** — List the fix that unblocks the most other work first

**Format each fix as:**

```
FIX [dimension] [priority: critical/recommended/nice-to-have]
Problem: [what is wrong]
Action: [exact steps to fix]
Example:
  [code block, file content, or shell command]
```

**Priority rules:**
- **Critical** — Blocks packaging entirely. Must fix before proceeding. (hardcoded secrets, no entry point, undeclared deps that cause install failure)
- **Recommended** — Will cause user-facing install problems. Should fix before distributing. (missing `.env.example`, no error messages for missing keys, no README install steps)
- **Nice-to-have** — Polish that improves the install experience. Can ship without but should add eventually. (health check command, demo mode, troubleshooting docs)

### Step 6: Route to Next Step

Based on the verdict:

| Verdict | Recommendation |
|---|---|
| **READY** | "Your agent is ready to package. Run `/pick-target` to choose your distribution format." |
| **ALMOST** | "Fix the [N] items above — estimated [time]. Then run `/pick-target` to proceed." |
| **NOT YET** | "Focus on the critical fixes first. Run `/agent-audit` again after completing them to re-check." |
| **RETHINK** | "Before packaging, restructure your agent. The most impactful change is [specific recommendation]. Come back after that is done." |

If the user already knows their distribution target (from `/agent-guide` or conversation context), route to `/pick-framework` instead of `/pick-target`.

---

## Output Format

Structure the full response as:

1. **Agent Summary** — 2-3 sentences identifying the agent, its framework, and what it does
2. **Scores Table** — The ASCII table from Step 3
3. **Verdict** — Bold verdict with score
4. **Detailed Findings** — One paragraph per dimension explaining the score with evidence from the code
5. **Fix List** — Ordered fixes for dimensions below 4 (skip this section entirely if all dimensions score 4+)
6. **Next Step** — Clear routing recommendation

---

## Cross-References

- **Skills:** `agent-packaging-foundations` (packaging principles and anti-patterns), `agent-security` (security-specific audit criteria for Step 5 fixes)
- **Commands:** `/agent-guide` (if user skipped orientation), `/pick-target` (next step after passing audit), `/pick-framework` (if target is already known)

---

## Edge Cases

- **User provides no code, only a description** — Audit based on the description. Score conservatively (assume the worst for anything you cannot verify). Flag each unverifiable dimension: "Scored 3/5 based on your description. Provide the codebase for a precise assessment."
- **User's agent scores 30/30** — Congratulate them genuinely — this is rare. Skip the fix list entirely. Route directly to `/pick-target` or `/gen-package`.
- **User's agent is a Jupyter notebook** — This is common for data science agents. Structure score will be low (notebooks lack clear entry points). Recommend extracting notebook cells into a Python module with a `main()` function as the highest-priority fix.
- **User's agent is a no-code workflow** (n8n, Flowise, Dify) — Adapt the dimensions. "Structure" becomes "Is the workflow exportable?" "Dependencies" becomes "Does the workflow declare its required credentials and connections?" "Testability" becomes "Can the workflow be imported and run with one action?"
- **User has already fixed items from a previous audit** — If the user says "I fixed the issues, re-audit," do a fresh audit. Do not carry over old scores. Acknowledge improvements explicitly: "Configuration improved from 2/5 to 4/5 — the `.env.example` you added covers all required keys."
- **User wants to audit someone else's agent** — That is fine. Audit it the same way. The output is useful for evaluating whether to fork, contribute to, or wrap the agent in a packaging layer.
