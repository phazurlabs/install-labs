---
name: docker-agent-packaging
description: "Complete guide to containerizing AI agents and automations for distribution via Docker. Use when the user mentions: docker agent, containerize agent, docker compose agent, dockerfile AI, GPU docker, docker deploy agent, multi-agent docker, docker image AI, agent container, docker packaging, one-click deploy, railway deploy, render deploy, cloud run agent, docker health check, container registry, agent docker compose, vector store docker, agent infrastructure, docker distribution"
---

# Docker Agent Packaging

## When to Use Docker for Agents

Docker is the right choice when your agent has requirements that exceed what a simple package manager can handle. Use this decision framework:

| Situation | Use Docker? | Why |
|---|---|---|
| Complex ML dependencies (PyTorch, transformers, CUDA) | Yes | Reproducible environment eliminates "works on my GPU" |
| Multi-service architecture (agent + vector store + DB) | Yes | Compose orchestrates the full stack in one command |
| Server-side agent (API endpoint, webhook handler) | Yes | Standard deployment target for every cloud platform |
| GPU inference required | Yes | nvidia-container-toolkit provides clean GPU passthrough |
| Team needs identical dev environments | Yes | Dev containers eliminate onboarding friction |
| Simple CLI tool with few deps | No | Use a single binary (Go/Rust) or `uvx`/`npx` |
| Agent is just an MCP server | No | Use npm/PyPI; MCP clients handle lifecycle |
| Users are non-technical without Docker installed | No | Docker itself is a prerequisite most non-devs don't have |
| Lightweight Python script calling APIs | No | `pip install` or `uvx` is faster with zero overhead |

**Rule of thumb:** if your agent needs more than one process or has dependencies that fight each other across machines, Docker is the answer. If it is a single-process CLI tool, Docker adds overhead without value.

---

## Dockerfile for Python AI Agents

Multi-stage builds keep your runtime image small by separating build-time tools from the final artifact.

```dockerfile
# =============================================================================
# Stage 1: Build — install dependencies in an isolated layer
# =============================================================================
FROM python:3.12-slim AS builder

# Prevent Python from writing .pyc files and enable unbuffered output
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# Install system-level build dependencies (removed in runtime stage)
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc libpq-dev && \
    rm -rf /var/lib/apt/lists/*

# Copy dependency manifest first (layer caching: deps change less than code)
COPY requirements.txt .

# Install Python dependencies into a virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
RUN pip install --no-cache-dir -r requirements.txt

# =============================================================================
# Stage 2: Runtime — minimal image with only what's needed to run
# =============================================================================
FROM python:3.12-slim AS runtime

# Runtime system deps only (no compiler)
RUN apt-get update && \
    apt-get install -y --no-install-recommends libpq5 curl && \
    rm -rf /var/lib/apt/lists/*

# Create non-root user (never run agents as root)
RUN groupadd --gid 1000 agent && \
    useradd --uid 1000 --gid agent --shell /bin/bash --create-home agent

WORKDIR /app

# Copy virtual environment from builder
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Copy application source
COPY --chown=agent:agent . .

# Switch to non-root user
USER agent

# Expose the agent's API port (change to match your agent)
EXPOSE 8000

# Health check — container orchestrators use this to know if agent is alive
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# API keys are NEVER baked into the image — pass at runtime via -e or .env
# ENV ANTHROPIC_API_KEY=  (do NOT set a default value)

# Start the agent
CMD ["python", "-m", "my_agent.server"]
```

### Key decisions explained

1. **`python:3.12-slim` not `python:3.12`** -- The full image is ~1GB. Slim is ~150MB. You lose convenience packages but gain a 6x smaller image.
2. **Virtual environment inside Docker** -- Seems redundant, but it makes the `COPY --from=builder` clean: one directory to copy, no system-site-packages contamination.
3. **Non-root user** -- If the agent is compromised (prompt injection, dependency vulnerability), damage is contained.
4. **HEALTHCHECK** -- Without this, Docker has no idea if your agent is alive or deadlocked. Every orchestrator (Compose, Kubernetes, ECS) uses health checks for restart decisions.
5. **Layer ordering** -- `requirements.txt` is copied before source code. Dependencies change rarely; source changes every commit. This means `pip install` is cached on most builds.

---

## Dockerfile for Node.js AI Agents

```dockerfile
# =============================================================================
# Stage 1: Build
# =============================================================================
FROM node:20-slim AS builder

WORKDIR /app

# Copy package files first for layer caching
COPY package.json package-lock.json ./

# Install all dependencies (including devDependencies for build step)
RUN npm ci --ignore-scripts

# Copy source and build (TypeScript compilation, bundling, etc.)
COPY . .
RUN npm run build

# Remove devDependencies after build
RUN npm prune --production

# =============================================================================
# Stage 2: Runtime
# =============================================================================
FROM node:20-slim AS runtime

RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Non-root user (node user exists in official node images)
USER node

WORKDIR /app

# Copy production node_modules and built output
COPY --from=builder --chown=node:node /app/node_modules ./node_modules
COPY --from=builder --chown=node:node /app/dist ./dist
COPY --from=builder --chown=node:node /app/package.json ./

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

CMD ["node", "dist/server.js"]
```

**Node-specific notes:**
- `npm ci` instead of `npm install` -- deterministic installs from lockfile, faster in CI.
- `npm prune --production` after build -- removes devDependencies (TypeScript, test tools) from the runtime image.
- The official `node` images include a `node` user at UID 1000. Use it.

---

## Docker Compose for Multi-Agent Systems

Most production agents are not a single container. They need a database for state, a cache for performance, and a vector store for retrieval. Compose orchestrates all of these.

```yaml
# docker-compose.yml — Multi-agent system with full infrastructure
# Start: docker compose up -d
# Stop:  docker compose down
# Logs:  docker compose logs -f agent

services:
  # =========================================================================
  # The AI agent itself
  # =========================================================================
  agent:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      # API keys — passed from host .env file, NEVER hardcoded here
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      # Internal service URLs (Docker DNS resolves service names)
      - DATABASE_URL=postgresql://agent:${POSTGRES_PASSWORD}@postgres:5432/agent_db
      - REDIS_URL=redis://redis:6379/0
      - VECTOR_STORE_URL=http://qdrant:6333
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      qdrant:
        condition: service_healthy
    restart: unless-stopped
    # Resource limits prevent runaway agents from killing the host
    deploy:
      resources:
        limits:
          memory: 2G
          cpus: "2.0"

  # =========================================================================
  # PostgreSQL — agent state, conversation history, tool results
  # =========================================================================
  postgres:
    image: pgvector/pgvector:pg16
    # pgvector image = PostgreSQL + vector extension pre-installed
    # Enables both relational storage AND vector similarity search
    environment:
      POSTGRES_USER: agent
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: agent_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U agent -d agent_db"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # =========================================================================
  # Redis — caching, rate limiting, task queues, pub/sub between agents
  # =========================================================================
  redis:
    image: redis:7-alpine
    command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # =========================================================================
  # Qdrant — dedicated vector store for RAG / semantic search
  # =========================================================================
  qdrant:
    image: qdrant/qdrant:latest
    volumes:
      - qdrant_data:/qdrant/storage
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:6333/healthz || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

# Persistent volumes survive `docker compose down` (but not `down -v`)
volumes:
  postgres_data:
  redis_data:
  qdrant_data:
```

### Why health conditions on `depends_on`

Without `condition: service_healthy`, Docker Compose only waits for the container to *start*, not for the service inside to be *ready*. Your agent will crash on boot trying to connect to a database that hasn't finished initialization. Health checks fix this.

### Choosing between pgvector and a dedicated vector store

| Factor | pgvector (in PostgreSQL) | Qdrant / Weaviate / Pinecone |
|---|---|---|
| Simplicity | One fewer service to manage | Separate container |
| Scale | Good to ~5M vectors | Built for billions |
| Filtering | Full SQL WHERE clauses | Payload filtering |
| Use case | Agent with moderate RAG needs | Production search at scale |

For most agents starting out, pgvector in PostgreSQL is sufficient and eliminates an entire service from your stack.

---

## GPU Support

### Prerequisites

The host machine needs:
1. NVIDIA GPU with compatible drivers
2. `nvidia-container-toolkit` installed (`apt install nvidia-container-toolkit`)
3. Docker Engine (not Docker Desktop on Linux; Desktop handles this automatically on macOS/Windows)

### Dockerfile with CUDA

```dockerfile
# Use NVIDIA's CUDA base image instead of python:slim
FROM nvidia/cuda:12.4.1-runtime-ubuntu22.04 AS runtime

# Install Python into the CUDA image
RUN apt-get update && \
    apt-get install -y --no-install-recommends python3.12 python3.12-venv python3-pip curl && \
    rm -rf /var/lib/apt/lists/*

# ... rest follows the same pattern as the Python Dockerfile above
```

### Compose with GPU reservation

```yaml
services:
  agent:
    build: .
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1          # Number of GPUs (or "all")
              capabilities: [gpu]
```

**Important:** `deploy.resources.reservations.devices` only works with `docker compose up` (Compose V2). The legacy `docker-compose` binary does not support it. Use `runtime: nvidia` as a fallback for older setups.

### Apple Silicon (Metal) in Docker

Docker on macOS runs a Linux VM. There is no Metal GPU passthrough into Docker containers. For Apple Silicon GPU inference, the agent must run natively (not in Docker). Use Docker only for the infrastructure services (database, vector store, cache) and run the agent process directly on the host.

---

## Image Optimization

### Base image selection

| Base Image | Size | Use When |
|---|---|---|
| `python:3.12` | ~1 GB | Never in production; useful only for debugging |
| `python:3.12-slim` | ~150 MB | Default choice for Python agents |
| `node:20-slim` | ~200 MB | Default choice for Node.js agents |
| `gcr.io/distroless/python3` | ~50 MB | Maximum security; no shell, no package manager |
| `nvidia/cuda:12.4.1-runtime-ubuntu22.04` | ~3.6 GB | When GPU inference is required |
| `alpine` | ~5 MB | Avoid for Python (musl libc breaks many scientific packages) |

### Layer ordering principle

Order Dockerfile instructions from most stable to least stable:

```dockerfile
# 1. Base image            (changes: never)
FROM python:3.12-slim
# 2. System packages       (changes: rarely)
RUN apt-get install -y libpq5
# 3. Python dependencies   (changes: weekly)
COPY requirements.txt .
RUN pip install -r requirements.txt
# 4. Application source    (changes: every commit)
COPY . .
```

Every layer after a change is invalidated. If you put `COPY . .` before `pip install`, you reinstall all dependencies on every code change.

### .dockerignore

```
# .dockerignore — keep the build context small and secure
.git
.github
.env
.env.*
!.env.example
__pycache__
*.pyc
node_modules
.venv
*.egg-info
dist
build
.pytest_cache
.mypy_cache
.ruff_cache
tests
docs
*.md
!README.md
models/           # Large model weights should not be in the image
*.gguf
*.bin
*.safetensors
docker-compose*.yml
Makefile
```

Without a `.dockerignore`, Docker sends your entire directory (including `.git`, `node_modules`, and model weights) to the build daemon. A 10GB build context makes every build slow.

---

## One-Click Deploy Buttons

These platforms let users deploy your Dockerized agent without touching a terminal.

### Railway

Create `railway.json` in your repository root:

```json
{
  "$schema": "https://railway.com/railway.schema.json",
  "build": {
    "builder": "DOCKERFILE",
    "dockerfilePath": "Dockerfile"
  },
  "deploy": {
    "startCommand": "python -m my_agent.server",
    "healthcheckPath": "/health",
    "healthcheckTimeout": 30,
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 3
  }
}
```

Deploy button for your README:

```markdown
[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/template/YOUR_TEMPLATE_ID)
```

To create a template: push your repo to GitHub, go to railway.com/new, create a project from it, then click "Generate Template" in project settings.

### Render

Create `render.yaml` in your repository root:

```yaml
services:
  - type: web
    name: my-agent
    runtime: docker
    healthCheckPath: /health
    envVars:
      - key: ANTHROPIC_API_KEY
        sync: false  # User must provide this during deploy
      - key: DATABASE_URL
        fromDatabase:
          name: agent-db
          property: connectionString

databases:
  - name: agent-db
    plan: free
    databaseName: agent_db
```

Deploy button:

```markdown
[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/YOUR_ORG/YOUR_REPO)
```

### Google Cloud Run

Cloud Run deploys any container with an HTTP endpoint. Deploy button:

```markdown
[![Run on Google Cloud](https://deploy.cloud.run/button.svg)](https://deploy.cloud.run?git_repo=https://github.com/YOUR_ORG/YOUR_REPO)
```

Requirements for your Dockerfile:
- Must listen on the port specified by `$PORT` environment variable (Cloud Run sets this)
- Must respond to HTTP requests (not just a CLI agent)
- Must start in under 300 seconds

```dockerfile
# Cloud Run compatibility: respect the PORT env var
CMD ["sh", "-c", "uvicorn my_agent.server:app --host 0.0.0.0 --port ${PORT:-8000}"]
```

### HuggingFace Spaces (Docker SDK)

Create a `Dockerfile` and set the Space SDK to Docker in the repo's `README.md` header:

```markdown
---
title: My Agent
emoji: 🤖
colorFrom: blue
colorTo: purple
sdk: docker
app_port: 8000
---
```

The Dockerfile must expose the port specified in `app_port`. HuggingFace Spaces provides free CPU instances and paid GPU instances.

### Replicate (Cog)

Replicate uses Cog, a wrapper around Docker for ML models:

```yaml
# cog.yaml
build:
  python_version: "3.12"
  python_packages:
    - "anthropic>=0.40.0"
    - "langchain>=0.3.0"
  gpu: true

predict: "predict.py:Predictor"
```

```python
# predict.py
from cog import BasePredictor, Input

class Predictor(BasePredictor):
    def setup(self):
        """Load model weights — runs once when container starts."""
        self.agent = initialize_agent()

    def predict(self, prompt: str = Input(description="Agent prompt")) -> str:
        """Run the agent — called on every request."""
        return self.agent.run(prompt)
```

Replicate handles GPU provisioning, scaling, and billing. Ideal for agents that need GPU inference and you want per-request pricing.

---

## Environment Variable Patterns

### The golden rule

**API keys are runtime configuration, not build configuration.** They go in `docker run -e` or `.env`, never in the Dockerfile or image layers.

```bash
# CORRECT: pass at runtime
docker run -e ANTHROPIC_API_KEY=sk-ant-... my-agent

# CORRECT: use an env file
docker run --env-file .env my-agent

# WRONG: baked into Dockerfile (visible in image history)
# ENV ANTHROPIC_API_KEY=sk-ant-...

# WRONG: passed as build arg (cached in image layers)
# ARG ANTHROPIC_API_KEY
```

### .env.example template

Ship this in your repository. Users copy it to `.env` and fill in their values.

```bash
# .env.example — copy to .env and fill in your values
# Required
ANTHROPIC_API_KEY=           # Get from https://console.anthropic.com/keys
OPENAI_API_KEY=              # Get from https://platform.openai.com/api-keys (optional if using Anthropic only)

# Infrastructure (defaults work for docker compose)
DATABASE_URL=postgresql://agent:changeme@postgres:5432/agent_db
REDIS_URL=redis://redis:6379/0
VECTOR_STORE_URL=http://qdrant:6333

# Optional
LOG_LEVEL=info               # debug, info, warning, error
MAX_TOKENS=4096              # Max tokens per agent response
MODEL=claude-sonnet-4-20250514    # Model to use
POSTGRES_PASSWORD=changeme   # Change in production
```

### Docker Secrets (production)

For production deployments (Docker Swarm, Kubernetes), use secrets instead of environment variables:

```yaml
services:
  agent:
    secrets:
      - anthropic_api_key
    environment:
      - ANTHROPIC_API_KEY_FILE=/run/secrets/anthropic_api_key

secrets:
  anthropic_api_key:
    external: true  # Created via: echo "sk-ant-..." | docker secret create anthropic_api_key -
```

Your agent code reads the file:

```python
import os

def get_secret(name: str) -> str:
    """Read secret from Docker secret file or environment variable."""
    file_path = os.environ.get(f"{name}_FILE")
    if file_path and os.path.exists(file_path):
        return open(file_path).read().strip()
    return os.environ.get(name, "")
```

---

## Health Checks

Every containerized agent needs a health endpoint. Without one, orchestrators cannot distinguish a running container from a deadlocked one.

### Minimal health endpoint (FastAPI)

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/health")
async def health():
    """Liveness check — is the process alive?"""
    return {"status": "ok"}

@app.get("/ready")
async def ready():
    """Readiness check — can the agent handle requests?"""
    checks = {
        "database": await check_database(),
        "vector_store": await check_vector_store(),
        "api_key_set": bool(os.environ.get("ANTHROPIC_API_KEY")),
    }
    all_ok = all(checks.values())
    return {"ready": all_ok, "checks": checks}
```

### Three types of probe

| Probe | Question | Failure Action |
|---|---|---|
| **Startup** | Has the container finished initializing? | Keep waiting (don't restart yet) |
| **Liveness** | Is the process alive and not deadlocked? | Kill and restart the container |
| **Readiness** | Can the agent handle requests right now? | Stop sending traffic, but don't restart |

In Compose, `HEALTHCHECK` serves as both liveness and readiness. In Kubernetes, configure all three separately.

---

## Container Registries

After building your image, push it to a registry so others can pull it.

### Docker Hub

```bash
# Tag and push
docker tag my-agent:latest yourusername/my-agent:latest
docker tag my-agent:latest yourusername/my-agent:v1.0.0
docker push yourusername/my-agent:latest
docker push yourusername/my-agent:v1.0.0
```

### GitHub Container Registry (ghcr.io)

Tied to your GitHub repo. Free for public repositories.

```bash
# Authenticate
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Tag and push
docker tag my-agent:latest ghcr.io/your-org/my-agent:latest
docker push ghcr.io/your-org/my-agent:latest
```

### Google Artifact Registry

```bash
# Authenticate
gcloud auth configure-docker us-docker.pkg.dev

# Tag and push
docker tag my-agent:latest us-docker.pkg.dev/PROJECT/REPO/my-agent:latest
docker push us-docker.pkg.dev/PROJECT/REPO/my-agent:latest
```

**Tagging strategy:** Always push both `:latest` and a version tag (`:v1.0.0`). Users who want stability pin a version. Users who want the latest pull `:latest`. Never use only `:latest` -- there is no way to roll back.

---

## CI/CD: GitHub Actions

Build and push your image automatically on every tagged release.

```yaml
# .github/workflows/docker-publish.yml
name: Build and Push Docker Image

on:
  push:
    tags: ["v*"]  # Trigger on version tags: v1.0.0, v1.2.3, etc.

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata (tags, labels)
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

This workflow:
1. Triggers when you push a tag like `v1.0.0`
2. Logs into GitHub Container Registry using the built-in `GITHUB_TOKEN`
3. Tags the image with the semver version, major.minor, and commit SHA
4. Uses GitHub Actions cache for Docker layer caching (dramatically speeds up rebuilds)

---

## Common Pitfalls

### 1. Running as root

The default Docker user is root. If your agent has a vulnerability (prompt injection, dependency exploit), the attacker has root inside the container. Always create and switch to a non-root user.

### 2. No health check

Without `HEALTHCHECK`, Docker reports the container as "running" even when the agent inside is deadlocked, out of memory, or crashed in a way that keeps the process alive. Add a health check to every agent container.

### 3. Baking secrets into the image

```dockerfile
# NEVER DO THIS — secrets are visible in `docker history`
ENV ANTHROPIC_API_KEY=sk-ant-abc123
# Also NEVER DO THIS — build args are cached in layers
ARG API_KEY
```

Anyone who pulls your image can extract these with `docker history --no-trunc`.

### 4. Using full base images

`python:3.12` is 1GB. `python:3.12-slim` is 150MB. `alpine` is 5MB but breaks many Python packages. Slim is the right default.

### 5. No .dockerignore

Without `.dockerignore`, your `.git` directory (potentially hundreds of MB), `node_modules`, `.env` files with real secrets, and model weights all get sent to the Docker daemon on every build. Builds become slow and images become bloated.

### 6. Missing .env.example

Users clone your repo, run `docker compose up`, and get cryptic errors because environment variables are not set. Ship a `.env.example` that documents every required variable with a comment explaining where to get the value.

### 7. No depends_on health conditions

```yaml
# BAD — agent starts before postgres is ready, crashes, restarts in a loop
depends_on:
  - postgres

# GOOD — agent waits until postgres is accepting connections
depends_on:
  postgres:
    condition: service_healthy
```

### 8. Single `:latest` tag

If you only push `:latest`, users cannot pin a version, cannot roll back, and cannot reproduce builds. Always tag with semver in addition to `:latest`.

### 9. Model weights in the image

Baking a 4GB GGUF file into the Docker image means every pull downloads 4GB. Instead, download model weights on first run and cache them in a named volume:

```yaml
volumes:
  - model_cache:/app/models
```

### 10. No resource limits

An agent in an infinite loop or processing a massive document can consume all host memory and CPU. Always set `deploy.resources.limits` in Compose or `--memory` / `--cpus` flags in `docker run`.

---

## Sources & References

1. **[Docker Documentation]** — Docker, Inc. https://docs.docker.com/ Comprehensive reference for Docker Engine, CLI, and container concepts.
2. **[Dockerfile Best Practices]** — Docker, Inc. https://docs.docker.com/build/building/best-practices/ Official guidance on writing production Dockerfiles, including layer ordering, multi-stage builds, and security.
3. **[Docker Compose File Specification]** — Docker, Inc. https://docs.docker.com/compose/compose-file/ Complete reference for the Compose file format, services, volumes, networks, and deploy configuration.
4. **[Docker Multi-Stage Builds]** — Docker, Inc. https://docs.docker.com/build/building/multi-stage/ Guide to using multi-stage builds to reduce image size by separating build-time and runtime layers.
5. **[NVIDIA Container Toolkit]** — NVIDIA Corporation. https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/ Documentation for GPU passthrough in Docker containers, including installation, configuration, and runtime options.
6. **[Railway Documentation]** — Railway Corp. https://docs.railway.app/ Platform documentation for deploying Docker containers with one-click deploy buttons, environment management, and scaling.
7. **[Render Documentation]** — Render. https://docs.render.com/ Platform documentation for deploying Docker services, including render.yaml infrastructure-as-code and deploy buttons.
8. **[Google Cloud Run Documentation]** — Google Cloud. https://cloud.google.com/run/docs Reference for deploying containerized HTTP services on Cloud Run, including PORT env var handling and scaling configuration.
9. **[HuggingFace Spaces Documentation]** — Hugging Face. https://huggingface.co/docs/hub/spaces Guide to deploying Docker-based applications on HuggingFace Spaces, including SDK configuration and GPU instance options.
10. **[Replicate Cog]** — Replicate, Inc. https://github.com/replicate/cog Open-source tool for packaging ML models as Docker containers with a standardized prediction interface for deployment on Replicate.
