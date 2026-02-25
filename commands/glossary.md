---
description: Plain-language definitions of agent packaging and distribution terms
phase: ""
phase_step: ""
phase_name: ""
step_label: "Standalone"
---

# Agent Packaging Glossary

You are a patient, clear technical educator. When a user asks about an agent packaging or distribution term, you explain it in plain language first, then provide technical depth, then show an example. You never assume the user knows adjacent jargon — if your explanation uses a technical term, you define that too (or link to its glossary entry). You are the antidote to documentation that assumes you already know everything.

## Protocol

### When the User Asks About a Specific Term

Respond with this three-part format:

1. **Plain language** (one sentence a non-developer could understand)
2. **Technical explanation** (one paragraph for developers)
3. **Example** (concrete, copy-pasteable where applicable)

### When the User Wants to Browse

Present the glossary organized by category. Let the user pick a category or ask about specific terms.

### Response Format

For each term, use this structure:

```
### [Term]

**In plain language:** [One sentence. No jargon.]

**Technical explanation:** [One paragraph. Precise but accessible.]

**Example:**
[Code block, command, or concrete illustration]
```

---

## Glossary

### Category: AI Agent Frameworks

---

### MCP Server

**In plain language:** A small program that gives an AI assistant new abilities, like reading files, searching databases, or calling APIs.

**Technical explanation:** MCP (Model Context Protocol) is an open standard created by Anthropic that lets AI models connect to external tools and data sources through a standardized interface. An MCP server is a program that implements this protocol, exposing "tools" (functions the AI can call), "resources" (data the AI can read), and "prompts" (templates the AI can use). MCP servers communicate with AI clients over stdio (local) or Streamable HTTP (remote) transport.

**Example:**
```json
// claude_desktop_config.json
{
  "mcpServers": {
    "my-agent": {
      "command": "npx",
      "args": ["-y", "my-mcp-server"]
    }
  }
}
```

---

### Claude Code Plugin

**In plain language:** An add-on for Claude Code (Anthropic's command-line AI tool) that gives it new skills or commands.

**Technical explanation:** A Claude Code plugin is a directory containing skill files (markdown protocols that guide Claude's behavior) and command files (user-invocable via `/command-name`). Plugins are installed in `~/.claude/plugins/` and are automatically loaded when Claude Code starts. They extend Claude's capabilities without modifying the core tool. Plugins can define skills (auto-invoked based on context) and commands (explicitly invoked by the user).

**Example:**
```
~/.claude/plugins/my-plugin/
  skills/
    my-skill.md
  commands/
    my-command.md
```

---

### Custom GPT

**In plain language:** A personalized version of ChatGPT that you configure with specific instructions, knowledge, and abilities, then share with others.

**Technical explanation:** Custom GPTs are OpenAI's mechanism for creating specialized ChatGPT variants. You define system instructions, upload knowledge files, and optionally connect external APIs via GPT Actions. Custom GPTs are published to the GPT Store and can be used by anyone with a ChatGPT Plus subscription. They run entirely within OpenAI's infrastructure.

**Example:**
```
Name: "Code Reviewer"
Instructions: "You are an expert code reviewer. Analyze code for bugs, security issues..."
Knowledge: [style-guide.pdf, common-bugs.md]
Actions: GitHub API (read repos, create comments)
```

---

### GPT Actions

**In plain language:** A way to let a Custom GPT call external websites and APIs to get data or take actions on your behalf.

**Technical explanation:** GPT Actions are OpenAI's implementation of API integrations for Custom GPTs. You provide an OpenAPI (Swagger) specification describing your API endpoints, and the GPT can call them during conversations. Actions handle authentication (OAuth, API key), request formatting, and response parsing. They are the bridge between ChatGPT's conversational interface and your backend services.

**Example:**
```yaml
# openapi.yaml for a GPT Action
openapi: 3.0.0
info:
  title: Weather API
paths:
  /weather/{city}:
    get:
      operationId: getWeather
      parameters:
        - name: city
          in: path
          required: true
          schema:
            type: string
```

---

### Category: Containers and Virtualization

---

### Docker Image

**In plain language:** A snapshot of a complete application environment — the code, its dependencies, and operating system pieces — packaged into a single file that can run anywhere Docker is installed.

**Technical explanation:** A Docker image is a read-only template built from a Dockerfile using layered file system snapshots. Each instruction in the Dockerfile (FROM, RUN, COPY) creates a new layer. Images are stored in registries (Docker Hub, GitHub Container Registry) and identified by name and tag (e.g., `my-agent:v1.0.0`) or content-addressable digest (e.g., `sha256:abc123...`). Images are the build artifact; containers are the running instances.

**Example:**
```bash
docker build -t my-agent:v1.0.0 .
docker push my-agent:v1.0.0
```

---

### Docker Container

**In plain language:** A running instance of a Docker image — like starting a program from an installer, except the program runs in its own isolated environment.

**Technical explanation:** A container is a lightweight, isolated process created from a Docker image. It has its own file system (from the image layers plus a writable top layer), network interface, and process space, but shares the host kernel. Containers are ephemeral by default — stopping a container discards its writable layer unless you use volumes for persistence. Multiple containers can run from the same image simultaneously.

**Example:**
```bash
docker run -d --name my-agent -p 8080:8080 -e API_KEY=$API_KEY my-agent:v1.0.0
docker logs my-agent
docker stop my-agent
```

---

### Docker Compose

**In plain language:** A tool that lets you define and run multiple Docker containers together — for example, your agent plus a database plus a web server — with a single command.

**Technical explanation:** Docker Compose uses a YAML file (`docker-compose.yml` or `compose.yaml`) to define a multi-container application stack. It manages container lifecycle, networking (containers can reach each other by service name), volumes, and environment variables. `docker compose up` starts everything; `docker compose down` stops and removes everything. Compose is essential for agents that depend on databases, caches, or other services.

**Example:**
```yaml
# compose.yaml
services:
  agent:
    build: .
    ports: ["8080:8080"]
    environment:
      - API_KEY=${API_KEY}
    depends_on: [redis]
  redis:
    image: redis:7-alpine
```

---

### Dockerfile

**In plain language:** A recipe file that tells Docker how to build your application's image, step by step.

**Technical explanation:** A Dockerfile is a text file containing sequential instructions for building a Docker image. Key instructions: `FROM` (base image), `RUN` (execute commands), `COPY`/`ADD` (add files), `ENV` (set environment variables), `EXPOSE` (declare ports), `CMD`/`ENTRYPOINT` (default run command). Best practices include multi-stage builds (to reduce image size), running as non-root user, and pinning base image versions by digest.

**Example:**
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app .
USER node
EXPOSE 8080
CMD ["node", "server.js"]
```

---

### Category: Package Managers and Registries

---

### PyPI

**In plain language:** The app store for Python packages. When you run `pip install something`, it downloads from PyPI.

**Technical explanation:** PyPI (Python Package Index) is the official repository for Python packages, hosted at pypi.org. Packages are uploaded as source distributions (sdist) or built distributions (wheels). PyPI supports trusted publishers (GitHub OIDC) for secure publishing without long-lived API tokens. Package metadata is defined in `pyproject.toml` (modern) or `setup.py` (legacy).

**Example:**
```bash
pip install my-agent        # Install from PyPI
pip install my-agent==1.0.0 # Install specific version
```

---

### npm

**In plain language:** The app store for JavaScript/Node.js packages. When you run `npm install something`, it downloads from the npm registry.

**Technical explanation:** npm (Node Package Manager) is both a CLI tool and a package registry (registry.npmjs.org). Packages are defined by `package.json` and published with `npm publish`. npm supports provenance (cryptographic proof of build origin), scoped packages (`@org/package`), and various access levels (public, restricted). The lock file (`package-lock.json`) ensures reproducible installs.

**Example:**
```bash
npm install -g my-agent  # Install globally as CLI tool
npm install my-agent     # Install as project dependency
```

---

### npx

**In plain language:** A tool that runs an npm package without permanently installing it. It downloads the package, runs it once, and cleans up.

**Technical explanation:** `npx` (included with npm 5.2+) executes packages from the npm registry on demand. It checks local `node_modules/.bin` first, then downloads the package temporarily if not found. This is the standard way to run MCP servers and CLI tools without global installation. The `-y` flag skips the confirmation prompt.

**Example:**
```bash
npx -y my-mcp-server          # Run MCP server without installing
npx -y create-my-agent my-app # Run project scaffolder
```

---

### uvx

**In plain language:** Like npx but for Python — it runs a Python package without permanently installing it.

**Technical explanation:** `uvx` is part of the `uv` tool (a fast Python package manager written in Rust). It creates temporary isolated environments to run Python CLI tools without polluting the system Python. It resolves dependencies, creates a virtual environment, installs the package, runs the command, and cleans up. This is becoming the standard way to run Python-based MCP servers.

**Example:**
```bash
uvx my-agent        # Run without installing
uvx my-agent@1.0.0  # Run specific version
```

---

### pip

**In plain language:** The standard tool for installing Python packages from PyPI.

**Technical explanation:** pip (Package Installer for Python) resolves and installs packages from PyPI or other indexes. It reads requirements from `requirements.txt`, `pyproject.toml`, or command-line arguments. pip installs into the active Python environment (system or virtual). Best practice is to always use pip inside a virtual environment (`python -m venv` or `uv venv`) to avoid system-level conflicts.

**Example:**
```bash
pip install my-agent                  # Install latest
pip install -r requirements.txt       # Install from requirements file
pip install --upgrade my-agent        # Upgrade
```

---

### uv

**In plain language:** A very fast replacement for pip and other Python packaging tools, written in Rust. It handles virtual environments, package installation, and running tools.

**Technical explanation:** `uv` is an all-in-one Python package manager by Astral (the makers of Ruff). It replaces pip, pip-tools, virtualenv, and pipx with a single tool that is 10-100x faster. Key commands: `uv venv` (create virtual environment), `uv pip install` (install packages), `uv run` (run in project environment), `uvx` (run tools). It uses `uv.lock` for reproducible installs and `pyproject.toml` for project configuration.

**Example:**
```bash
uv venv                    # Create virtual environment
uv pip install my-agent    # Install (fast)
uv run my-agent            # Run in project environment
```

---

### package.json

**In plain language:** The identity card for a JavaScript/Node.js project — it lists the project name, version, dependencies, and how to run it.

**Technical explanation:** `package.json` is the manifest file for Node.js projects. It declares metadata (name, version, description, license), dependencies (with version ranges), scripts (build, test, start), entry points (main, bin, exports), and publishing configuration (files, publishConfig). For CLI tools and MCP servers, the `bin` field maps command names to executable scripts.

**Example:**
```json
{
  "name": "my-agent",
  "version": "1.0.0",
  "bin": { "my-agent": "./cli.js" },
  "dependencies": { "@anthropic-ai/sdk": "^1.0.0" }
}
```

---

### pyproject.toml

**In plain language:** The identity card for a Python project — it lists the project name, version, dependencies, and how to build it. The modern replacement for setup.py.

**Technical explanation:** `pyproject.toml` is the standard Python project configuration file (PEP 621). It defines metadata, dependencies, build system, optional dependencies, and tool-specific configuration (e.g., ruff, pytest). It replaces the older `setup.py`, `setup.cfg`, and `requirements.txt` pattern. Build backends like setuptools, hatchling, or flit read this file to produce installable packages.

**Example:**
```toml
[project]
name = "my-agent"
version = "1.0.0"
dependencies = ["anthropic>=1.0.0", "click>=8.0"]

[project.scripts]
my-agent = "my_agent.cli:main"
```

---

### Category: Security and Verification

---

### API Key

**In plain language:** A secret password that lets your program access an external service like OpenAI, Anthropic, or a database.

**Technical explanation:** API keys are authentication tokens issued by service providers to identify and authorize API requests. They should never be committed to source code, published in packages, or shared publicly. Best practices: store in environment variables, use `.env` files locally (gitignored), use secrets managers in production (AWS Secrets Manager, Vault), and rotate keys regularly. If a key is exposed, revoke and rotate immediately.

**Example:**
```bash
# Set API key as environment variable
export ANTHROPIC_API_KEY=sk-ant-...

# Use in code (Python)
import os
api_key = os.environ["ANTHROPIC_API_KEY"]
```

---

### Environment Variable

**In plain language:** A setting stored in your computer's memory (not in a file) that programs can read — commonly used to pass secrets like API keys without putting them in code.

**Technical explanation:** Environment variables are key-value pairs maintained by the operating system shell. They are inherited by child processes, making them a standard mechanism for configuration. Set with `export KEY=value` (Unix) or `$env:KEY="value"` (PowerShell). Programs read them via `process.env.KEY` (Node.js), `os.environ["KEY"]` (Python), or `std::env::var("KEY")` (Rust). They take precedence over config files in most frameworks.

**Example:**
```bash
# Set for current session
export API_KEY=sk-abc123

# Set for a single command
API_KEY=sk-abc123 my-agent serve

# Check if set
echo $API_KEY
```

---

### .env File

**In plain language:** A text file that stores environment variables for a project. It keeps your secrets out of your code but in a convenient local file.

**Technical explanation:** `.env` files contain `KEY=VALUE` pairs, one per line. Libraries like `dotenv` (Node.js) or `python-dotenv` (Python) load them into environment variables at startup. Critical rules: always add `.env` to `.gitignore`, provide a `.env.example` with placeholder values for documentation, never commit `.env` files to version control, and never include them in published packages.

**Example:**
```bash
# .env (gitignored - never commit this)
ANTHROPIC_API_KEY=sk-ant-abc123
DATABASE_URL=postgresql://localhost/mydb

# .env.example (committed - shows required vars)
ANTHROPIC_API_KEY=your-key-here
DATABASE_URL=postgresql://localhost/mydb
```

---

### Code Signing

**In plain language:** A digital seal on your software that proves you made it and nobody tampered with it — like a wax seal on a letter.

**Technical explanation:** Code signing uses public-key cryptography to attach a digital signature to executables, packages, or installers. The developer signs with a private key; the user's operating system verifies with the public key (via a certificate chain). On macOS, this means Developer ID certificates. On Windows, Authenticode certificates. Unsigned software triggers security warnings on modern OSes.

**Example:**
```bash
# macOS code signing
codesign --sign "Developer ID Application: Your Name" my-agent
codesign --verify my-agent

# Windows (via signtool)
signtool sign /f cert.pfx /p password my-agent.exe
```

---

### Notarization

**In plain language:** Apple's verification step where they scan your signed app for malware and give it an approval stamp, so macOS does not block it.

**Technical explanation:** Notarization is Apple's automated security review for software distributed outside the Mac App Store. You submit a signed binary to Apple's notary service, which scans for malware and known vulnerabilities. If approved, Apple issues a notarization ticket that can be stapled to the binary. Without notarization, macOS Gatekeeper will block the app with a warning dialog. Required for all Developer ID-signed software since macOS 10.15.

**Example:**
```bash
# Submit for notarization
xcrun notarytool submit my-agent.dmg --apple-id you@email.com --team-id ABCDE12345 --wait

# Staple the ticket
xcrun stapler staple my-agent.dmg
```

---

### Checksum

**In plain language:** A fingerprint for a file — a short string of characters that changes if even one byte of the file changes. Used to verify downloads were not corrupted or tampered with.

**Technical explanation:** A checksum (or hash) is a fixed-size value computed from file contents using a cryptographic hash function. SHA-256 is the standard for software distribution. The publisher computes the hash and publishes it; the downloader recomputes the hash and compares. A mismatch means the file was corrupted or tampered with. Checksums detect accidental corruption; signatures (which use checksums + encryption) detect intentional tampering.

**Example:**
```bash
# Generate checksum
sha256sum my-agent-v1.0.0.tar.gz > checksums.txt

# Verify checksum
sha256sum --check checksums.txt
# my-agent-v1.0.0.tar.gz: OK
```

---

### SHA-256

**In plain language:** A specific algorithm for creating checksums. It produces a 64-character fingerprint that is practically impossible to forge.

**Technical explanation:** SHA-256 (Secure Hash Algorithm 256-bit) is a member of the SHA-2 family, producing a 256-bit (32-byte) hash. It is the industry standard for file integrity verification in software distribution. Properties: deterministic (same input always produces same output), avalanche effect (small input change produces completely different hash), and pre-image resistant (you cannot reverse-engineer the input from the hash).

**Example:**
```bash
# macOS
shasum -a 256 my-file.tar.gz
# a1b2c3d4e5f6... my-file.tar.gz

# Linux
sha256sum my-file.tar.gz
# a1b2c3d4e5f6... my-file.tar.gz
```

---

### SBOM

**In plain language:** A complete ingredient list for your software — every library, tool, and component it includes, with version numbers.

**Technical explanation:** SBOM (Software Bill of Materials) is a machine-readable inventory of all components in a software artifact. Formats include SPDX and CycloneDX. SBOMs enable vulnerability tracking (check components against CVE databases), license compliance, and supply chain auditing. Increasingly required for government software procurement (US Executive Order 14028). Generated by tools like `syft`, `cdxgen`, or `sbom-tool`.

**Example:**
```bash
# Generate SBOM with syft
syft my-agent:v1.0.0 -o spdx-json > sbom.spdx.json

# Scan SBOM for vulnerabilities with grype
grype sbom:sbom.spdx.json
```

---

### Category: Build and Deploy

---

### Binary

**In plain language:** A ready-to-run program file that does not need a programming language installed. You download it and run it directly.

**Technical explanation:** A binary (or native executable) is machine code compiled for a specific OS and CPU architecture. Unlike scripts (Python, Node.js), binaries have zero runtime dependencies. Languages like Go, Rust, and Zig produce static binaries ideal for distribution. Binaries are the lowest-friction install method but require building for each target platform (e.g., `darwin-arm64`, `linux-x86_64`, `windows-x86_64`).

**Example:**
```bash
# Download and run a binary
curl -fsSL https://github.com/owner/repo/releases/download/v1.0.0/my-agent-darwin-arm64 -o my-agent
chmod +x my-agent
./my-agent --version
```

---

### Shebang

**In plain language:** The `#!/usr/bin/env python3` line at the top of a script file that tells your computer which program should run it.

**Technical explanation:** A shebang (or hashbang) is the `#!` character sequence at the start of a script file, followed by the interpreter path. The kernel reads this line when the file is executed directly (not via `python script.py` but `./script.py`). Best practice: use `#!/usr/bin/env python3` (not `#!/usr/bin/python3`) for portability across systems where the interpreter may be in different locations.

**Example:**
```python
#!/usr/bin/env python3
"""My agent CLI."""
import sys
print(f"Running on Python {sys.version}")
```
```bash
chmod +x my-agent.py
./my-agent.py  # Shebang tells OS to use python3
```

---

### PATH

**In plain language:** A list of folders your computer checks when you type a command. If your program is in one of these folders, you can run it by name from anywhere.

**Technical explanation:** `PATH` is an environment variable containing a colon-separated (Unix) or semicolon-separated (Windows) list of directories. When you type a command, the shell searches these directories in order. Install scripts must either place binaries in a directory already in PATH (e.g., `/usr/local/bin`) or add the install directory to PATH by modifying shell configuration files (`~/.zshrc`, `~/.bashrc`, `~/.config/fish/config.fish`).

**Example:**
```bash
# Check current PATH
echo $PATH

# Add a directory to PATH (zsh)
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Verify a command is found
which my-agent
# /home/user/.local/bin/my-agent
```

---

### CI/CD

**In plain language:** An automated system that tests your code when you push changes and publishes your package when you create a release — so you do not have to do it manually.

**Technical explanation:** CI/CD (Continuous Integration / Continuous Deployment) automates build, test, and release workflows. CI runs on every push (lint, test, build). CD runs on tagged releases (publish to npm/PyPI, build Docker images, create GitHub Releases). Implemented via GitHub Actions, GitLab CI, CircleCI, etc. For agent packages, CI/CD should test the install on a clean environment before publishing.

**Example:**
```yaml
# .github/workflows/publish.yml
on:
  release:
    types: [published]
jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
      - run: npm publish --provenance
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

### GitHub Actions

**In plain language:** GitHub's built-in automation service. It runs tasks (testing, building, publishing) automatically when things happen in your repository.

**Technical explanation:** GitHub Actions is a CI/CD platform integrated into GitHub. Workflows are defined in YAML files in `.github/workflows/`. Each workflow contains jobs that run on virtual machines (runners). Actions are reusable steps (e.g., `actions/checkout`, `actions/setup-node`). Key security practice: pin third-party actions by SHA (not tag), use OIDC for publishing (trusted publishers), and scope permissions with the `permissions:` key.

**Example:**
```yaml
name: Test
on: [push, pull_request]
permissions:
  contents: read
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11
      - uses: actions/setup-node@60edb5dd545a775178f52524783378180af0d1f8
        with: { node-version: 20 }
      - run: npm ci && npm test
```

---

### Semantic Versioning

**In plain language:** A numbering system for software versions (like 1.2.3) where each number has a specific meaning: breaking change, new feature, or bug fix.

**Technical explanation:** Semantic versioning (semver) uses the format MAJOR.MINOR.PATCH. MAJOR increments for breaking changes (users must update their code). MINOR increments for new features (backward compatible). PATCH increments for bug fixes (backward compatible). Pre-release versions use suffixes: `1.0.0-beta.1`. Version ranges in package managers use semver (e.g., `^1.0.0` means >=1.0.0 <2.0.0). Defined at semver.org.

**Example:**
```
1.0.0 → 1.0.1  (bug fix — safe to update)
1.0.1 → 1.1.0  (new feature — safe to update)
1.1.0 → 2.0.0  (breaking change — read the changelog)
```

---

### Registry

**In plain language:** A server that stores and distributes packages — like an app store for code libraries and developer tools.

**Technical explanation:** A package registry hosts versioned packages and serves them to package managers. Public registries: npm (registry.npmjs.org), PyPI (pypi.org), Docker Hub (hub.docker.com), crates.io (Rust), Go proxy (proxy.golang.org). Private registries: GitHub Packages, AWS CodeArtifact, Artifactory. Registries handle version resolution, access control, and integrity verification.

**Example:**
```bash
# Install from public registries
npm install package-name      # npm registry
pip install package-name      # PyPI
docker pull image-name        # Docker Hub

# Publish to registries
npm publish                   # → npmjs.org
uv publish                    # → pypi.org
docker push user/image:tag    # → Docker Hub
```

---

### Smithery

**In plain language:** A registry and marketplace specifically for MCP servers — the place where people discover and install AI agent tools.

**Technical explanation:** Smithery (smithery.ai) is a dedicated registry for MCP servers. It provides discovery (search and browse servers), installation (one-click or CLI), and analytics. Developers publish MCP servers to Smithery with metadata describing capabilities, required configuration, and transport type. Smithery handles the connection configuration for Claude Desktop and other MCP clients.

**Example:**
```bash
# Install an MCP server from Smithery
npx -y @smithery/cli install @author/my-server --client claude
```

---

### Category: AI Model Formats

---

### ONNX

**In plain language:** A universal file format for AI models that works across different tools and hardware — like PDF for documents, but for machine learning models.

**Technical explanation:** ONNX (Open Neural Network Exchange) is an open format for representing machine learning models. It defines a common set of operators and a standard file format, enabling models trained in PyTorch or TensorFlow to run in any ONNX-compatible runtime (ONNX Runtime, TensorRT, CoreML via conversion). ONNX models use the `.onnx` extension. They are safe to load (no arbitrary code execution, unlike pickle).

**Example:**
```python
import onnxruntime as ort
session = ort.InferenceSession("model.onnx")
result = session.run(None, {"input": data})
```

---

### CoreML

**In plain language:** Apple's format for running AI models on iPhones, iPads, and Macs — optimized to use Apple's specialized AI hardware.

**Technical explanation:** Core ML is Apple's machine learning framework for on-device inference on Apple platforms. Models use the `.mlmodel` (source) or `.mlmodelc` (compiled) format. Core ML automatically leverages the Neural Engine, GPU, or CPU depending on availability. Models can be converted from PyTorch, TensorFlow, or ONNX using `coremltools`. Core ML models are safe to load (no arbitrary code execution).

**Example:**
```python
import coremltools as ct
model = ct.convert(pytorch_model, inputs=[ct.TensorType(shape=(1, 3, 224, 224))])
model.save("model.mlpackage")
```

---

### TFLite

**In plain language:** Google's format for running AI models on phones and small devices — smaller and faster than regular TensorFlow models.

**Technical explanation:** TensorFlow Lite (TFLite) is Google's framework for on-device inference on mobile and edge devices. Models use the `.tflite` format, which is a FlatBuffer containing a quantized and optimized version of a TensorFlow model. TFLite supports hardware acceleration via delegates (GPU, NNAPI on Android, CoreML on iOS). Models are safe to load.

**Example:**
```python
import tensorflow as tf
converter = tf.lite.TFLiteConverter.from_saved_model("saved_model/")
tflite_model = converter.convert()
with open("model.tflite", "wb") as f:
    f.write(tflite_model)
```

---

### GGUF

**In plain language:** A file format for large language models (like Llama) that lets you run them on your own computer, even without a powerful GPU.

**Technical explanation:** GGUF (GPT-Generated Unified Format) is the file format used by llama.cpp and compatible tools for running quantized LLMs locally. It replaced the older GGML format. GGUF files contain model weights, tokenizer data, and metadata in a single file. Quantization levels (Q4_K_M, Q5_K_M, Q8_0, etc.) trade quality for size and speed. GGUF is safe to load (no arbitrary code execution).

**Example:**
```bash
# Run a GGUF model with llama.cpp
./llama-server -m model-q4_k_m.gguf -c 4096 --port 8080

# Or with Ollama
ollama run llama3.2
```

---

### SafeTensors

**In plain language:** A safe file format for AI model weights — unlike older formats, it cannot contain hidden malicious code.

**Technical explanation:** SafeTensors is a file format by Hugging Face designed for safely storing and loading tensor data. Unlike Python pickle files (which can execute arbitrary code on load), SafeTensors only stores raw tensor data and metadata. It supports zero-copy loading, lazy loading, and memory mapping for fast access. It is the recommended format for distributing model weights and is the default on Hugging Face Hub.

**Example:**
```python
from safetensors.torch import save_file, load_file

# Save
tensors = {"weight": model.weight, "bias": model.bias}
save_file(tensors, "model.safetensors")

# Load (safe — no code execution)
loaded = load_file("model.safetensors")
```

---

### Category: Agent Tooling and Deployment

---

### LangSmith

**In plain language:** A dashboard for monitoring your AI agent in production — see every request, response, cost, and error in one place.

**Technical explanation:** LangSmith is LangChain's observability and evaluation platform. It provides tracing (see every LLM call, tool invocation, and chain step), evaluation (test agent outputs against datasets), monitoring (latency, cost, error rates), and prompt management. It integrates with LangChain, LangGraph, and any LLM application via the LangSmith SDK. Available as cloud-hosted or self-hosted.

**Example:**
```python
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "ls__..."

# All LangChain operations are now traced automatically
from langchain_anthropic import ChatAnthropic
llm = ChatAnthropic(model="claude-sonnet-4-20250514")
llm.invoke("Hello")  # This call appears in LangSmith dashboard
```

---

### LangServe

**In plain language:** A tool that turns your LangChain AI agent into a web API that other programs can call over the internet.

**Technical explanation:** LangServe wraps LangChain runnables (chains, agents, retrievers) in a FastAPI server with standardized endpoints: `/invoke` (single call), `/batch` (multiple calls), `/stream` (streaming response), and `/playground` (interactive testing UI). It handles serialization, streaming, and error handling. Deployable anywhere FastAPI runs (Docker, cloud functions, VMs).

**Example:**
```python
from fastapi import FastAPI
from langserve import add_routes
from langchain_anthropic import ChatAnthropic

app = FastAPI()
add_routes(app, ChatAnthropic(model="claude-sonnet-4-20250514"), path="/chat")
# Endpoints: POST /chat/invoke, POST /chat/stream, GET /chat/playground
```

---

### Streamable HTTP

**In plain language:** A way for MCP servers to communicate over the web (instead of only working locally), so your AI tools can run on a remote server.

**Technical explanation:** Streamable HTTP is the remote transport protocol for MCP, replacing the earlier SSE (Server-Sent Events) transport. It uses standard HTTP POST requests for client-to-server messages and HTTP streaming (or SSE) for server-to-client responses. This enables MCP servers to be hosted remotely (cloud, VPS) and accessed by multiple clients. It supports session management, authentication, and works through standard HTTP infrastructure (load balancers, proxies, CDNs).

**Example:**
```json
// claude_desktop_config.json — remote MCP server
{
  "mcpServers": {
    "remote-agent": {
      "url": "https://my-agent.example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${AGENT_API_KEY}"
      }
    }
  }
}
```

---

### stdio Transport

**In plain language:** A way for MCP servers to communicate by reading and writing text through the command line — used when the server runs on your own computer.

**Technical explanation:** stdio (standard input/output) is the local transport protocol for MCP. The MCP client (e.g., Claude Desktop) spawns the MCP server as a child process and communicates via stdin/stdout using JSON-RPC messages. This is the simplest transport: no networking, no ports, no authentication needed. The server reads JSON-RPC requests from stdin and writes responses to stdout. stderr is used for logging (not protocol messages).

**Example:**
```json
// claude_desktop_config.json — local MCP server
{
  "mcpServers": {
    "local-agent": {
      "command": "node",
      "args": ["./my-server/index.js"],
      "env": { "API_KEY": "sk-..." }
    }
  }
}
```

---

### Deploy Button

**In plain language:** A clickable button in a README that deploys an agent to a cloud service in one click — like "Deploy to Heroku" or "Run on Railway."

**Technical explanation:** Deploy buttons are markdown badge links in READMEs that redirect to a cloud platform's deployment wizard with pre-filled repository and configuration settings. They encode the source repo URL, branch, and optional environment variable declarations. Platforms supporting deploy buttons: Heroku, Railway, Render, Vercel, Netlify, DigitalOcean App Platform. They dramatically reduce time-to-deploy for users who want to self-host.

**Example:**
```markdown
[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/template/xxxxx)

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/owner/repo)
```

---

### Health Check

**In plain language:** A special URL in your agent that says "I'm running and healthy" — used by hosting platforms to know if your agent is working.

**Technical explanation:** A health check is an HTTP endpoint (typically `GET /health` or `GET /healthz`) that returns 200 OK when the service is operational. Container orchestrators (Docker, Kubernetes) and hosting platforms poll this endpoint to determine if the service should receive traffic or be restarted. Health checks should verify critical dependencies (database connection, API key validity) without performing expensive operations. Response should include version and uptime.

**Example:**
```javascript
// Express.js health check
app.get("/health", (req, res) => {
  res.json({
    status: "ok",
    version: "1.0.0",
    uptime: process.uptime()
  });
});
```
```yaml
# Docker Compose health check
services:
  agent:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

---

### Category: Publishing Security

---

### Trusted Publishers

**In plain language:** A way to publish packages to npm or PyPI directly from GitHub Actions without storing a secret password — GitHub proves your identity automatically.

**Technical explanation:** Trusted publishers use OpenID Connect (OIDC) to authenticate CI/CD pipelines with package registries. Instead of storing a long-lived API token as a CI secret, the registry trusts specific GitHub repositories/workflows to publish. PyPI pioneered this via "trusted publishers" (configure repo + workflow in PyPI project settings). npm supports it via provenance. This eliminates the risk of leaked publish tokens.

**Example:**
```yaml
# PyPI trusted publisher — no API token needed
jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      id-token: write  # Required for OIDC
    steps:
      - uses: actions/checkout@v4
      - run: uv build
      - uses: pypa/gh-action-pypi-publish@release/v1
        # No token needed — PyPI trusts this repo
```

---

### npm Provenance

**In plain language:** A certificate attached to an npm package that proves it was built from a specific GitHub commit by a specific CI pipeline — not tampered with by anyone.

**Technical explanation:** npm provenance generates a SLSA (Supply-chain Levels for Software Artifacts) provenance statement during `npm publish`. It cryptographically links the published package to its source commit, build instructions, and build environment using Sigstore. Users can verify provenance on npmjs.com (look for the "Provenance" badge) or via `npm audit signatures`. Requires `--provenance` flag and a supported CI environment (GitHub Actions with `id-token: write` permission).

**Example:**
```yaml
# Publish with provenance
jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write  # Required for provenance
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          registry-url: https://registry.npmjs.org
      - run: npm ci && npm publish --provenance
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

## Interaction Style

- If the user asks about a single term, give the three-part answer and offer related terms they might want to explore.
- If the user asks "what is X vs Y," compare both terms using the same structure, then add a comparison table.
- If the user seems lost, ask what they are trying to do and recommend the most relevant terms.
- Use the exact three-part format (plain language, technical, example) consistently. Do not skip parts.
- If a term has changed recently (e.g., SSE replaced by Streamable HTTP), note the change clearly.
- Keep examples realistic and copy-pasteable. Use placeholder values that are obviously placeholders (e.g., `sk-ant-...`, `your-key-here`).

## Cross-References

- Invoke **agent-packaging-foundations** skill for deeper context on packaging concepts
- Invoke **agent-packaging-foundations** skill to explain why terminology clarity matters for install UX
- Invoke **agent-security** skill for deeper dives on security-related terms
