---
name: framework-packaging-guides
description: "Framework-specific packaging guides for turning agents and automations built with any major framework into installable products. Covers LangChain, LangGraph, CrewAI, AutoGen, Pydantic AI, LlamaIndex, Semantic Kernel, Vercel AI SDK, OpenAI Assistants, Anthropic Tool Use, Smolagents, n8n, Flowise, Dify, ComfyUI, browser automation agents, and custom Python/Node.js agents. Use when the user mentions: LangChain packaging, CrewAI distribution, AutoGen install, package my agent, distribute my automation, LangGraph deploy, Pydantic AI packaging, LlamaIndex deploy, n8n workflow distribution, Flowise deployment, Dify packaging, ComfyUI workflow, browser agent, Playwright agent, custom agent packaging, Semantic Kernel deploy, Vercel AI agent, OpenAI assistant packaging, smolagents distribution."
---

# Framework-Specific Packaging Guides

How to turn an agent built with any major framework into something other people can install with one command.

---

## 1. LangChain / LangGraph

**When to use:** Stateful multi-step agents with tool calling, memory, and conditional branching. LangGraph adds explicit graph-based control flow.

**Project structure:**
```
my-langgraph-agent/
├── src/my_agent/
│   ├── __init__.py
│   ├── agent.py          # graph definition
│   ├── nodes.py          # node functions
│   ├── tools.py
│   └── state.py          # TypedDict state schema
├── langgraph.json        # deployment manifest
├── pyproject.toml
├── Dockerfile
├── docker-compose.yml
└── .env.example
```

**Key configs:**
```json
// langgraph.json — required for langgraph deploy and Cloud
{ "graphs": {"my_agent": "./src/my_agent/agent.py:graph"}, "env": ".env", "python_version": "3.12", "dependencies": ["."] }
```
```toml
# pyproject.toml
[project]
name = "my-langgraph-agent"
dependencies = ["langgraph>=0.2.0", "langchain-anthropic>=0.2.0"]
[project.scripts]
my-agent = "my_agent.cli:main"
```

**Distribution:** LangGraph Cloud (`langgraph deploy`) | Docker (`docker compose up -d`) | PyPI (`pip install my-langgraph-agent`)

**Pitfalls:**
1. **Version conflicts.** LangChain ships breaking changes frequently. Pin exact versions (`langgraph==0.2.14`, not `>=0.2`). Users with other LangChain projects hit dependency hell.
2. **Missing `langgraph.json`.** Without it, `langgraph deploy` silently fails or serves the wrong graph.
3. **State schema not exported.** Library consumers need your `State` TypedDict. Export it from `__init__.py`.

---

## 2. CrewAI

**When to use:** Multi-agent systems where each agent has a distinct role, goal, and backstory. Role-based collaboration (researcher + writer + editor).

**Project structure:**
```
my-crew/
├── src/my_crew/
│   ├── __init__.py
│   ├── crew.py           # @CrewBase class
│   ├── agents.py
│   ├── tasks.py
│   ├── tools/custom_tool.py
│   └── config/
│       ├── agents.yaml
│       └── tasks.yaml
├── pyproject.toml
├── Dockerfile
└── .env.example
```

**Key configs:**
```yaml
# config/agents.yaml
researcher:
  role: "Senior Research Analyst"
  goal: "Find comprehensive data on {topic}"
  llm: anthropic/claude-sonnet-4-20250514
```
```toml
[project]
name = "my-crew"
dependencies = ["crewai[tools]>=0.80.0"]
[project.scripts]
my-crew = "my_crew.cli:main"
```

**Distribution:** PyPI (`pip install my-crew`) | Docker (`docker compose up`) | CrewAI CLI (`crewai run`, dev only)

**Pitfalls:**
1. **YAML path resolution.** CrewAI resolves config relative to the crew file. After `pip install`, the working directory changes. Use `pathlib.Path(__file__).parent / "config"` for module-relative paths.
2. **Hardcoded LLM provider.** Users may lack an OpenAI key. Make the LLM configurable via env var.
3. **Tool deps not declared.** Custom tools depend on packages not in `pyproject.toml`. The crew imports fine but crashes when the tool is invoked.

---

## 3. AutoGen / AG2 (Microsoft)

**When to use:** Multi-agent conversations where agents talk to each other autonomously, with optional human-in-the-loop.

**Project structure:**
```
my-autogen-agent/
├── src/my_agent/
│   ├── __init__.py
│   ├── agents.py         # AssistantAgent, UserProxyAgent
│   ├── group_chat.py     # GroupChat + GroupChatManager
│   ├── tools.py
│   └── config.py         # OAI_CONFIG_LIST builder
├── OAI_CONFIG_LIST.example
├── pyproject.toml
├── Dockerfile
└── .env.example
```

**Key configs:**
```json
// OAI_CONFIG_LIST.example (user renames and fills in keys)
[{"model": "claude-sonnet-4-20250514", "api_key": "YOUR_KEY", "api_type": "anthropic"}]
```
```toml
[project]
name = "my-autogen-agent"
dependencies = ["autogen-agentchat>=0.4.0", "autogen-ext>=0.4.0"]
```

**Distribution:** PyPI (`pip install my-autogen-agent`) | Docker (`docker compose up`)

**Pitfalls:**
1. **Namespace confusion.** The project rebranded from `pyautogen` to `autogen-agentchat` + `autogen-ext`. Old tutorials use `import autogen`; new packages use `from autogen_agentchat import ...`. Pin the correct name.
2. **OAI_CONFIG_LIST file management.** Users forget to create it or place it in the wrong directory. Provide a CLI `init` command that generates it from env vars.
3. **Code execution by default.** `UserProxyAgent` executes code. In Docker this is fine; on bare metal, document this prominently and offer `code_execution_config=False`.

---

## 4. Pydantic AI

**When to use:** Type-safe, dependency-injected agents with structured outputs. Library-first: your agent is a Python object, not a service.

**Project structure:**
```
my-pydantic-agent/
├── src/my_agent/
│   ├── __init__.py
│   ├── agent.py          # Agent() with system prompt, result_type, deps_type
│   ├── models.py         # Pydantic response models
│   ├── tools.py          # @agent.tool functions
│   └── cli.py
├── pyproject.toml
└── .env.example
```

**Key configs:**
```toml
[project]
name = "my-pydantic-agent"
dependencies = ["pydantic-ai>=0.1.0"]
[project.scripts]
my-agent = "my_agent.cli:main"
```

**Distribution:** PyPI only (`pip install my-pydantic-agent`). It is a library.

**Pitfalls:**
1. **Hardcoded model string.** Make `Agent("anthropic:claude-sonnet-4-20250514")` configurable via `AGENT_MODEL` env var so users can switch providers.
2. **Deps not documented.** Pydantic AI uses dependency injection. Document what consumers must provide at `agent.run()` call time.
3. **No CLI shipped.** Library-first authors forget the CLI entrypoint. Add `[project.scripts]` so end users can run standalone.

---

## 5. LlamaIndex

**When to use:** RAG pipelines or agents that reason over documents. Deep vector store and data connector integrations.

**Project structure:**
```
my-llamaindex-agent/
├── src/my_agent/
│   ├── __init__.py
│   ├── agent.py          # agent/query engine
│   ├── index.py          # index construction + persistence
│   ├── tools.py          # QueryEngineTool wrappers
│   └── ingest.py         # data loading + chunking
├── storage/              # persisted index (gitignored)
├── pyproject.toml
├── docker-compose.yml    # includes vector store service
└── .env.example
```

**Key configs:**
```toml
[project]
name = "my-llamaindex-agent"
dependencies = [
    "llama-index-core>=0.11.0", "llama-index-llms-anthropic>=0.3.0",
    "llama-index-vector-stores-chroma>=0.2.0", "llama-index-readers-file>=0.2.0",
]
```

**Distribution:** Docker Compose with vector store (`docker compose up -d`) | llama-deploy (managed) | PyPI (library)

**Pitfalls:**
1. **Vector store not included.** Ship a `docker-compose.yml` that starts ChromaDB/Qdrant alongside the agent, or default to local file-based storage.
2. **Index not persisted.** Call `index.storage_context.persist()`. Users will re-ingest on every restart otherwise.
3. **Subpackage explosion.** LlamaIndex split into `llama-index-core`, `llama-index-llms-*`, `llama-index-vector-stores-*`, etc. Miss one and get `ImportError` at runtime.

---

## 6. Semantic Kernel (Microsoft)

**When to use:** Enterprise AI orchestration in Python or C#/.NET with strong typing and plugin architecture.

**Project structure (Python):**
```
my-sk-agent/
├── src/my_agent/
│   ├── __init__.py
│   ├── agent.py          # Kernel setup, planner
│   └── plugins/
│       ├── search_plugin.py
│       └── math_plugin.py
├── pyproject.toml
└── .env.example
```

**Key configs:** `dependencies = ["semantic-kernel>=1.0.0"]` (Python) | `<PackageReference Include="Microsoft.SemanticKernel" Version="1.*" />` (C#)

**Distribution:** PyPI (`pip install`) | NuGet (`dotnet add package`) | Docker | Azure Container Apps

**Pitfalls:**
1. **Azure-centric defaults.** SK defaults to Azure OpenAI. Users with direct OpenAI/Anthropic keys need different service registration. Document both.
2. **Plugin discovery.** Plugins register at kernel construction. Provide a config-driven plugin loading hook for extensibility.
3. **Python vs C# drift.** The SDKs lack feature parity. Document which features exist in each.

---

## 7. Vercel AI SDK

**When to use:** Streaming AI agents in JS/TS, typically with Next.js. Provides React hooks for streaming, tool calling, and multi-step loops.

**Project structure:**
```
my-ai-app/
├── app/
│   ├── page.tsx              # useChat() UI
│   └── api/chat/route.ts     # streamText() agent logic
├── lib/
│   ├── agent.ts
│   └── tools/
├── package.json
├── Dockerfile
└── .env.example
```

**Key configs:**
```json
{"dependencies": {"ai": "^4.0.0", "@ai-sdk/anthropic": "^1.0.0", "next": "^15.0.0"}}
```

**Distribution:** Vercel deploy (`vercel deploy`) | npm library (`npm install my-ai-agent`) | Docker

**Pitfalls:**
1. **Server-side only.** Agent logic runs in API routes. Client-side imports fail. Use `"use server"` or split packages.
2. **Streaming requires compatible hosting.** `streamText()` uses SSE. Some platforms (older Lambda, buffering proxies) break streaming.
3. **Missing provider package.** `@ai-sdk/anthropic` or `@ai-sdk/openai` must be in `dependencies`. Missing = runtime error, not build error.

---

## 8. OpenAI Assistants API

**When to use:** Agent runtime hosted on OpenAI's infrastructure. You distribute a thin client, not the agent.

**Project structure:**
```
my-assistant-client/
├── src/my_assistant/
│   ├── __init__.py
│   ├── client.py         # creates/retrieves assistant
│   ├── threads.py        # thread + message management
│   ├── setup.py          # one-time assistant creation
│   └── cli.py
├── assistant_config.json # assistant params (non-secret)
├── pyproject.toml
└── .env.example
```

**Key configs:**
```json
// assistant_config.json
{"name": "My Assistant", "model": "gpt-4o", "tools": [{"type": "file_search"}, {"type": "code_interpreter"}]}
```

**Distribution:** Custom GPT (zero-install, GPT Store) | PyPI wrapper (`pip install my-assistant-client && my-assistant-setup`) | npm wrapper

**Pitfalls:**
1. **Assistant ID management.** Create the assistant on the user's account via a setup script. Sharing your ID ties usage to your billing.
2. **Custom GPT limitations.** GPTs cannot call external APIs without Actions (OpenAPI spec). If your agent needs external tools, a GPT alone is insufficient.
3. **Thread state is server-side.** Store thread IDs locally (SQLite/JSON) so users can resume conversations.

---

## 9. Anthropic Tool Use (Direct Claude API)

**When to use:** Calling Claude directly with tool definitions, no framework. Full control over the agentic loop.

**Project structure:**
```
my-claude-agent/
├── src/my_agent/
│   ├── __init__.py
│   ├── agent.py          # agentic loop: call model -> execute tools -> repeat
│   ├── tools.py          # tool schemas (JSON) + implementations
│   ├── prompts.py
│   └── cli.py
├── pyproject.toml
└── .env.example
```

**Key configs:**
```toml
[project]
name = "my-claude-agent"
dependencies = ["anthropic>=0.40.0"]
[project.scripts]
my-agent = "my_agent.cli:main"
```

**Distribution:** MCP server (`uvx my-claude-agent` in Claude Desktop config) | PyPI (`pip install`) | npm (TS version) | Docker

**Pitfalls:**
1. **Tool execution not sandboxed.** You run tool results yourself. Sandbox tools that execute shell commands or write files.
2. **No loop termination guard.** Without `max_turns=10`, a confused agent loops forever burning tokens.
3. **Streaming + tools.** Must handle `content_block_start/delta/stop` for `tool_use` blocks. Many tutorials skip this, silently dropping tool calls in streaming mode.

---

## 10. Smolagents (HuggingFace)

**When to use:** Lightweight, code-generating agents from HuggingFace. Simple tool-using agents that can run on HuggingFace Spaces for free.

**Project structure:**
```
my-smolagent/
├── src/my_agent/
│   ├── __init__.py
│   ├── agent.py          # CodeAgent or ToolCallingAgent
│   └── tools.py          # @tool functions
├── app.py                # Gradio UI for Spaces
├── pyproject.toml
└── .env.example
```

**Distribution:** HuggingFace Spaces (push to Space repo, users access via browser) | PyPI (`pip install my-smolagent`) | Docker

**Pitfalls:**
1. **CodeAgent runs generated Python.** Dangerous in untrusted environments. Use `ToolCallingAgent` for production or sandbox with `E2BSandbox`.
2. **HF_TOKEN required.** Gated models or Inference API need a HuggingFace token. The missing-token error is unhelpful.
3. **Tool serialization.** Tools must be serializable for `agent.push_to_hub()`. Closures and unpicklable objects fail silently.

---

## 11. n8n (Visual Workflow Automation)

**When to use:** You built an automation workflow in n8n's visual editor and want to share it.

**Project structure:**
```
my-n8n-workflow/
├── workflows/my_workflow.json    # exported from n8n UI
├── credentials/credentials.example.json
├── custom-nodes/                 # optional: npm packages
│   └── n8n-nodes-my-custom/
├── docker-compose.yml
└── .env.example
```

**Key configs:**
```yaml
# docker-compose.yml
services:
  n8n:
    image: n8nio/n8n:1.60.0   # pin version
    ports: ["5678:5678"]
    volumes: ["./workflows:/home/node/.n8n/workflows"]
```

**Distribution:** n8n template library (one-click import) | Docker with pre-loaded workflows (`docker compose up -d`) | Custom community node via npm

**Pitfalls:**
1. **Credentials not portable.** Exports include credential IDs, not values. Users must recreate every credential. Ship a `credentials.example.json` with setup instructions.
2. **Webhook URLs are instance-specific.** Document which trigger nodes need URL reconfiguration after import.
3. **Version mismatch.** Pin the n8n Docker image version. Workflows from v1.60 may use nodes missing in v1.50.

---

## 12. Flowise

**When to use:** LangChain-powered chatflows built with Flowise's visual drag-and-drop builder.

**Project structure:**
```
my-flowise-chatflow/
├── chatflows/my_chatflow.json   # exported from Flowise UI
├── docker-compose.yml
└── .env.example
```

**Distribution:** Docker (`docker compose up -d`, import chatflow via API or UI) | Flowise Cloud (upload JSON)

**Install via API:** `curl -X POST http://localhost:3000/api/v1/chatflows -H "Content-Type: application/json" -d @chatflows/my_chatflow.json`

**Pitfalls:**
1. **API keys in export.** Flowise may embed keys in the chatflow JSON. Inspect and sanitize before committing.
2. **Custom component deps.** Target instance must have the same custom components installed. List them in the README.
3. **Hardcoded connection strings.** Chatflows referencing a specific Pinecone index or Chroma URL break in other environments.

---

## 13. Dify

**When to use:** No-code/low-code agent builder. Distribute via Dify Cloud or self-hosted Docker.

**Project structure:**
```
my-dify-app/
├── dsl/my_app.yml        # exported Dify DSL (no secrets included)
├── docker-compose.yml    # Dify self-hosted stack (6+ containers)
└── .env.example
```

**Distribution:** Dify DSL import (via UI) | Dify Cloud (hosted, users access via URL) | Docker self-hosted | API-only

**Pitfalls:**
1. **No credentials in DSL.** Users must configure model providers in Dify settings before the app works. Document required providers and models.
2. **Heavy stack.** Self-hosted runs 6+ containers (API, worker, web, DB, Redis, vector store). Document minimum requirements: 4GB RAM, 20GB disk.
3. **Version incompatibility.** DSL format changes between versions. Pin the Dify version in your Docker compose.

---

## 14. ComfyUI

**When to use:** AI image generation pipelines built in ComfyUI's node editor.

**Project structure:**
```
my-comfyui-workflow/
├── workflows/
│   ├── my_workflow.json       # visual format
│   └── my_workflow_api.json   # API format (programmatic)
├── custom_nodes/my-node/      # optional
│   ├── __init__.py
│   ├── nodes.py               # NODE_CLASS_MAPPINGS
│   └── requirements.txt
├── MODELS_REQUIRED.md         # download URLs + expected paths
└── docker-compose.yml
```

**Distribution:** Workflow JSON (drag-drop into ComfyUI) | Custom node git repo (ComfyUI Manager: Install via URL) | Docker

**Pitfalls:**
1. **Missing models.** Workflows reference filenames like `sd_xl_base_1.0.safetensors` that must exist locally. Ship `MODELS_REQUIRED.md` with download URLs. Never bundle multi-GB files.
2. **Node dependency conflicts.** Two custom nodes requiring incompatible `torch` versions break each other. Test with a clean install.
3. **GPU requirements undocumented.** A workflow needing 24GB VRAM will OOM on 8GB. Document minimum VRAM and whether CPU fallback works.

---

## 15. Browser Automation Agents (Playwright / Puppeteer)

**When to use:** Agents that automate web browsers for scraping, form filling, or interacting with web apps lacking APIs.

**Project structure:**
```
my-browser-agent/
├── src/my_agent/
│   ├── __init__.py
│   ├── agent.py          # LLM + browser orchestration
│   ├── browser.py        # page interactions
│   └── cli.py
├── pyproject.toml
├── Dockerfile            # USE mcr.microsoft.com/playwright/python base image
└── .env.example
```

**Key config:** `dependencies = ["playwright>=1.45.0", "anthropic>=0.40.0"]`

**Distribution:** Docker strongly recommended (`docker compose up`) | PyPI + `playwright install chromium` | npm + `npx playwright install`

**Pitfalls:**
1. **Browser binaries not installed.** Playwright/Puppeteer download browsers separately. This is the #1 support issue. Ship Docker or add a post-install script.
2. **System library deps.** Headless Chromium needs `libnss3`, `libatk-bridge2.0-0`, etc. Use the official Playwright Docker base image.
3. **Headless vs headed.** Test in headless mode. Default to `headless=True` in production. Rendering differences cause silent failures.

---

## 16. Custom Python Agents (No Framework)

**When to use:** Calling Anthropic/OpenAI SDKs directly. Simplest and most portable when you do not need framework features.

**Project structure:**
```
my-agent/
├── src/my_agent/
│   ├── __init__.py
│   ├── agent.py          # prompt -> LLM -> tool -> repeat
│   ├── tools.py
│   └── cli.py            # argparse or click
├── pyproject.toml
├── Dockerfile
└── .env.example
```

**Key configs:**
```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
[project]
name = "my-agent"
dependencies = ["anthropic>=0.40.0", "click>=8.0.0"]
[project.scripts]
my-agent = "my_agent.cli:main"
```

**Distribution:** PyPI / pipx (`pipx install my-agent`) | MCP server (`uvx my-agent --mcp` in Claude Desktop config) | Docker | Single-file via `pipx run`

**Pitfalls:**
1. **No isolation.** Global `pip install` causes conflicts. Recommend `pipx install` for CLI tools.
2. **No CLI entrypoint.** Without `[project.scripts]`, users are stuck with `python -m my_agent`. Always add one.
3. **Missing `__init__.py` exports.** If also a library, ensure `from my_agent import Agent` works.

---

## 17. Custom Node.js Agents (No Framework)

**When to use:** Building agents in JS/TS using Anthropic/OpenAI SDKs directly. Ideal for JS ecosystems and npm distribution.

**Project structure:**
```
my-agent/
├── src/
│   ├── agent.ts
│   ├── tools.ts
│   └── index.ts          # CLI entry (needs #!/usr/bin/env node shebang)
├── dist/                 # compiled JS (gitignored)
├── package.json
├── tsconfig.json
└── .env.example
```

**Key configs:**
```json
{"name": "my-agent", "type": "module", "bin": {"my-agent": "./dist/index.js"},
 "files": ["dist"], "scripts": {"build": "tsc", "prepublishOnly": "npm run build"},
 "dependencies": {"@anthropic-ai/sdk": "^0.30.0"}, "engines": {"node": ">=20.0.0"}}
```

**Distribution:** npm (`npx my-agent`) | MCP server (`npx my-agent --mcp` in Claude Desktop config) | Docker

**Pitfalls:**
1. **`dist/` not built before publish.** Add `"prepublishOnly": "npm run build"` and `"files": ["dist"]`.
2. **Missing shebang.** CLI entry needs `#!/usr/bin/env node` at the top or the `bin` entry is not executable.
3. **ESM vs CJS confusion.** Set `"type": "module"`. Test with `npx` -- it surfaces module resolution errors hidden by `npm run dev`.

---

## Cross-Cutting Decision Matrix

| Framework | Best Distribution Target | Install Command | Difficulty |
|---|---|---|---|
| **LangChain / LangGraph** | Docker / LangGraph Cloud | `docker compose up` / `langgraph deploy` | Medium |
| **CrewAI** | PyPI | `pip install my-crew` | Medium |
| **AutoGen / AG2** | PyPI + Docker | `pip install my-autogen-agent` | Medium-High |
| **Pydantic AI** | PyPI | `pip install my-pydantic-agent` | Low |
| **LlamaIndex** | Docker Compose (+ vector store) | `docker compose up` | Medium-High |
| **Semantic Kernel** | PyPI / NuGet | `pip install` / `dotnet add package` | Medium |
| **Vercel AI SDK** | Vercel / npm | `vercel deploy` / `npm install` | Low-Medium |
| **OpenAI Assistants** | Custom GPT / PyPI wrapper | GPT Store / `pip install` | Low |
| **Anthropic Tool Use** | MCP server / PyPI | `uvx my-agent` / `pip install` | Low-Medium |
| **Smolagents** | HF Spaces / PyPI | Push to Space / `pip install` | Low |
| **n8n** | Docker + workflows | `docker compose up` | Medium |
| **Flowise** | Docker + chatflow | `docker compose up` | Medium |
| **Dify** | DSL import / Docker | Import YAML / `docker compose up` | Medium-High |
| **ComfyUI** | Workflow JSON + Manager | Drag-drop JSON | Medium |
| **Browser agents** | Docker (strongly recommended) | `docker compose up` | High |
| **Custom Python** | PyPI (pipx) | `pipx install my-agent` | Low |
| **Custom Node.js** | npm | `npx my-agent` | Low |

### How to Read This Matrix

- **Difficulty** = how hard for the *end user* to install, not how hard to package.
- **Audience is developers:** PyPI or npm. They manage their own environments.
- **Audience is non-developers:** Docker, managed cloud (Vercel, Dify Cloud, HF Spaces), or Custom GPT.
- **Agent needs infrastructure** (DB, vector store, queue): Docker Compose for self-hosted. Managed cloud otherwise.
- **Agent is primarily tools** (no UI, no state): Wrap as MCP server. Users get it inside Claude Desktop or Claude Code with zero standalone setup.

---

## Sources & References

1. **[LangChain Python Documentation]** — LangChain, Inc. https://python.langchain.com/docs/ Reference for the LangChain framework including chains, agents, tools, memory, and retrieval patterns.
2. **[LangGraph Documentation]** — LangChain, Inc. https://langchain-ai.github.io/langgraph/ Guide to building stateful, multi-step agent graphs with LangGraph, including deployment via LangGraph Cloud.
3. **[CrewAI Documentation]** — CrewAI, Inc. https://docs.crewai.com/ Reference for building multi-agent systems with role-based collaboration, YAML configuration, and task orchestration.
4. **[AutoGen Documentation]** — Microsoft. https://microsoft.github.io/autogen/ Guide to multi-agent conversation frameworks including AssistantAgent, GroupChat, and code execution patterns.
5. **[Pydantic AI Documentation]** — Pydantic. https://ai.pydantic.dev/ Reference for type-safe, dependency-injected AI agents with structured outputs and tool definitions.
6. **[LlamaIndex Documentation]** — LlamaIndex. https://docs.llamaindex.ai/ Comprehensive guide to RAG pipelines, data connectors, vector store integrations, and query engines for document-based agents.
7. **[Semantic Kernel Documentation]** — Microsoft. https://learn.microsoft.com/en-us/semantic-kernel/ Reference for Microsoft's AI orchestration SDK covering Python and C#/.NET plugin architecture and planner patterns.
8. **[Vercel AI SDK Documentation]** — Vercel. https://sdk.vercel.ai/docs Guide to building streaming AI applications with React hooks, tool calling, and multi-step agent loops in TypeScript.
9. **[OpenAI API Documentation]** — OpenAI. https://platform.openai.com/docs Reference for the OpenAI Assistants API, function calling, threads, and the GPT Builder for Custom GPT distribution.
10. **[Anthropic API Documentation]** — Anthropic. https://docs.anthropic.com/ Official reference for Claude API tool use, message streaming, and agentic loop patterns.
11. **[Smolagents Documentation]** — Hugging Face. https://huggingface.co/docs/smolagents/ Guide to lightweight code-generating and tool-calling agents with HuggingFace Spaces deployment.
12. **[n8n Documentation]** — n8n GmbH. https://docs.n8n.io/ Reference for building visual workflow automations, custom nodes, Docker self-hosting, and template distribution.
13. **[Flowise Documentation]** — FlowiseAI. https://docs.flowiseai.com/ Guide to building LangChain-powered chatflows with a visual drag-and-drop builder, including Docker deployment and API import.
14. **[Dify Documentation]** — Dify. https://docs.dify.ai/ Reference for the no-code/low-code agent builder, DSL format, self-hosted Docker stack, and Dify Cloud deployment.
15. **[ComfyUI]** — comfyanonymous. https://github.com/comfyanonymous/ComfyUI Open-source node-based UI for AI image generation pipelines, including workflow JSON format and custom node development.
16. **[Playwright Documentation]** — Microsoft. https://playwright.dev/docs/intro Cross-browser automation library for building browser-based agents, with Docker base images and headless mode support.
17. **[Puppeteer Documentation]** — Google. https://pptr.dev/ Node.js library for controlling headless Chrome/Chromium, used for browser automation agents and web scraping.
