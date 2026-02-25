# Install Labs — References & Bibliography

> Canonical reference list for AI agent packaging, distribution, and installation.
> All sources are real, publicly accessible resources. Last verified: February 2026.

---

## Table of Contents

1. [Model Context Protocol (MCP)](#1-model-context-protocol-mcp)
2. [Claude Code & Anthropic Platform](#2-claude-code--anthropic-platform)
3. [Python Packaging](#3-python-packaging)
4. [npm / Node.js Packaging](#4-npm--nodejs-packaging)
5. [Docker & Containerization](#5-docker--containerization)
6. [OpenAI / Custom GPTs](#6-openai--custom-gpts)
7. [AI Agent Frameworks](#7-ai-agent-frameworks)
8. [Security Standards & Tools](#8-security-standards--tools)
9. [Cloud Deployment Platforms](#9-cloud-deployment-platforms)
10. [Software Distribution Standards](#10-software-distribution-standards)
11. [MCP Registries & Marketplaces](#11-mcp-registries--marketplaces)
12. [Cognitive Science & UX of Installation](#12-cognitive-science--ux-of-installation)
13. [Python Enhancement Proposals (PEPs)](#13-python-enhancement-proposals-peps)
14. [Supplementary Standards & Specifications](#14-supplementary-standards--specifications)

---

## 1. Model Context Protocol (MCP)

The Model Context Protocol is the open standard for connecting AI assistants to external tools and data sources. It is the primary distribution channel for tool-based AI agents.

1. **Model Context Protocol Specification**
   Anthropic. *MCP Specification.*
   https://spec.modelcontextprotocol.io/
   The authoritative protocol specification defining the JSON-RPC 2.0-based transport, capability negotiation, tool/resource/prompt primitives, and lifecycle management that all MCP servers must implement.

2. **Model Context Protocol — GitHub Organization**
   Anthropic. *modelcontextprotocol.*
   https://github.com/modelcontextprotocol
   The official GitHub organization hosting the protocol specification, SDKs, reference servers, and the MCP Inspector tool. Primary hub for MCP open-source development.

3. **MCP TypeScript SDK**
   Anthropic. *@modelcontextprotocol/sdk (TypeScript).*
   https://github.com/modelcontextprotocol/typescript-sdk
   Official TypeScript/JavaScript SDK for building MCP servers and clients. The primary SDK for npm-distributed MCP servers using `npx` as the install vector.

4. **MCP Python SDK**
   Anthropic. *mcp (Python).*
   https://github.com/modelcontextprotocol/python-sdk
   Official Python SDK for building MCP servers and clients. Uses FastMCP for high-level server authoring, supporting `uvx` and `pip` as install vectors.

5. **MCP Servers — Official Repository**
   Anthropic. *modelcontextprotocol/servers.*
   https://github.com/modelcontextprotocol/servers
   Collection of reference MCP server implementations (filesystem, GitHub, Slack, PostgreSQL, etc.). Essential reference for packaging patterns and configuration conventions.

6. **MCP Inspector**
   Anthropic. *MCP Inspector.*
   https://github.com/modelcontextprotocol/inspector
   Interactive debugging and testing tool for MCP servers. Critical for pre-ship validation of tool schemas, transport behavior, and error handling before distribution.

7. **"Introducing the Model Context Protocol"**
   Anthropic. *Anthropic Blog.* November 25, 2024.
   https://www.anthropic.com/news/model-context-protocol
   Anthropic's official announcement of MCP as an open standard, explaining the motivation for a universal protocol connecting AI models to data sources and tools.

8. **MCP Specification — Architecture Overview**
   Anthropic. *MCP Docs — Architecture.*
   https://modelcontextprotocol.io/docs/concepts/architecture
   Detailed explanation of the client-host-server architecture, including how MCP clients (Claude Desktop, IDEs) discover and connect to MCP servers during installation.

---

## 2. Claude Code & Anthropic Platform

Claude Code is the primary host environment for Install Labs and the target platform for Claude Code plugin distribution.

9. **Claude Code Documentation**
   Anthropic. *Claude Code — Overview.*
   https://docs.anthropic.com/en/docs/claude-code
   Official documentation for Claude Code, covering installation, configuration, slash commands, custom instructions (CLAUDE.md), and the plugin system that Install Labs targets.

10. **Anthropic API Documentation**
    Anthropic. *API Reference.*
    https://docs.anthropic.com/
    Complete API reference for Claude models including tool use (function calling), streaming, vision, and the Messages API. Foundation for understanding how agents interact with Claude.

11. **Claude Desktop — MCP Configuration**
    Anthropic. *Claude Desktop MCP Setup.*
    https://modelcontextprotocol.io/quickstart/user
    Guide for end users configuring MCP servers in Claude Desktop via `claude_desktop_config.json`. Defines the JSON configuration format that Install Labs generates for MCP server packaging.

12. **Anthropic Tool Use Documentation**
    Anthropic. *Tool Use (Function Calling).*
    https://docs.anthropic.com/en/docs/build-with-claude/tool-use
    Documentation for Claude's native tool use capability, which underpins how MCP servers expose tools to Claude. Essential for understanding the tool schema format.

13. **Claude Code — Custom Slash Commands**
    Anthropic. *Claude Code Custom Commands.*
    https://docs.anthropic.com/en/docs/claude-code/tutorials/custom-slash-commands
    Documentation for creating custom slash commands in Claude Code, the mechanism Install Labs uses for its `/agent-guide`, `/pick-target`, and other command implementations.

---

## 3. Python Packaging

Python packaging standards govern how Python-based AI agents are distributed via PyPI, pip, and uv.

14. **Python Packaging User Guide**
    Python Packaging Authority (PyPA). *Python Packaging User Guide.*
    https://packaging.python.org/
    The authoritative guide for Python packaging, covering project structure, build backends, metadata, and distribution. The canonical reference for all `pyproject.toml`-based agent packaging.

15. **PyPI — The Python Package Index**
    Python Packaging Authority (PyPA). *PyPI.*
    https://pypi.org/
    The official repository for Python packages. The primary distribution channel for Python-based AI agents, supporting both library installs (`pip install`) and tool installs (`uvx`, `pipx`).

16. **pip Documentation**
    Python Packaging Authority (PyPA). *pip.*
    https://pip.pypa.io/
    Documentation for pip, the standard Python package installer. Covers dependency resolution, requirements files, extras, and the install lifecycle that agent packagers must understand.

17. **uv — An Extremely Fast Python Package Installer**
    Astral. *uv.*
    https://github.com/astral-sh/uv
    Rust-based Python package and project manager replacing pip, pip-tools, pipx, and virtualenv. Increasingly the preferred install vector for MCP servers via `uvx` due to its speed and reliability.

18. **uv Documentation**
    Astral. *uv Docs.*
    https://docs.astral.sh/uv/
    Comprehensive documentation for uv covering project management, tool installation (`uv tool install` / `uvx`), lock files, and Python version management relevant to agent distribution.

19. **Hatchling — Modern Python Build Backend**
    Ofek Lev. *Hatch.*
    https://hatch.pypa.io/
    Documentation for Hatch and its build backend Hatchling. A modern, standards-compliant build system recommended for new Python agent packages due to its simplicity and PEP compliance.

20. **setuptools Documentation**
    Python Packaging Authority (PyPA). *setuptools.*
    https://setuptools.pypa.io/
    Documentation for setuptools, the legacy-but-still-widely-used Python build backend. Required knowledge for packaging agents that depend on older libraries using `setup.py` or `setup.cfg`.

21. **twine — PyPI Upload Tool**
    Python Packaging Authority (PyPA). *twine.*
    https://twine.readthedocs.io/
    Secure upload tool for publishing Python packages to PyPI. Handles Trusted Publisher authentication (OIDC) for CI/CD-based agent publishing workflows.

22. **TestPyPI**
    Python Packaging Authority (PyPA). *TestPyPI.*
    https://test.pypi.org/
    Testing instance of PyPI for validating package uploads before publishing to production. Essential step in the agent pre-ship checklist.

---

## 4. npm / Node.js Packaging

npm packaging standards govern how Node.js and TypeScript-based AI agents (including most MCP servers) are distributed.

23. **npm Documentation**
    npm, Inc. *npm Docs.*
    https://docs.npmjs.com/
    Official documentation for the npm package manager, covering `package.json`, publishing, scoped packages, access control, and the npm registry. Primary reference for Node.js agent distribution.

24. **package.json Specification**
    npm, Inc. *package.json.*
    https://docs.npmjs.com/cli/v10/configuring-npm/package-json
    Complete specification for `package.json` fields including `bin`, `main`, `exports`, `engines`, `scripts`, and `files`. Defines the manifest format for all npm-distributed agents.

25. **npx Documentation**
    npm, Inc. *npx.*
    https://docs.npmjs.com/cli/v10/commands/npx
    Documentation for `npx`, the npm package runner that executes packages without global installation. The standard install-free execution vector for MCP servers distributed via npm.

26. **npm Provenance Statements**
    npm, Inc. *Generating Provenance Statements.*
    https://docs.npmjs.com/generating-provenance-statements
    Documentation for npm's supply chain security feature that cryptographically links published packages to their source repository and build. Recommended for all agent publishers.

27. **Node.js Documentation**
    OpenJS Foundation. *Node.js Docs.*
    https://nodejs.org/docs/latest/api/
    Official Node.js API documentation. Required reference for understanding the runtime environment, ES modules vs CommonJS, and `node:` built-in modules used by TypeScript MCP servers.

28. **Node.js — package.json "exports" Field**
    OpenJS Foundation. *Packages — Conditional Exports.*
    https://nodejs.org/docs/latest/api/packages.html#conditional-exports
    Documentation for the `exports` field in `package.json`, which controls module resolution for dual CJS/ESM packages. Critical for TypeScript agents targeting both module systems.

29. **TypeScript Documentation**
    Microsoft. *TypeScript Handbook.*
    https://www.typescriptlang.org/docs/
    Official TypeScript documentation. Most MCP servers are written in TypeScript; understanding `tsconfig.json` compilation settings is essential for correct `npm publish` output.

---

## 5. Docker & Containerization

Docker packaging enables self-contained agent distribution with all dependencies, GPU support, and one-click cloud deployment.

30. **Docker Documentation**
    Docker, Inc. *Docker Docs.*
    https://docs.docker.com/
    Comprehensive Docker documentation covering images, containers, volumes, networking, and registries. The foundational reference for containerized agent distribution.

31. **Dockerfile Best Practices**
    Docker, Inc. *Building Best Practices.*
    https://docs.docker.com/build/building/best-practices/
    Official guide for writing production-quality Dockerfiles including multi-stage builds, layer caching, `.dockerignore`, non-root users, and minimal base images — all critical for agent containers.

32. **Docker Compose Specification**
    Docker, Inc. *Compose Specification.*
    https://docs.docker.com/compose/compose-file/
    Specification for `docker-compose.yml` files defining multi-container applications. Used for agents that require databases, vector stores, or other service dependencies.

33. **Docker Multi-Stage Builds**
    Docker, Inc. *Multi-Stage Builds.*
    https://docs.docker.com/build/building/multi-stage/
    Documentation for multi-stage Dockerfiles that separate build and runtime environments. Essential technique for keeping agent container images small (build tools excluded from final image).

34. **Docker Security Best Practices**
    Docker, Inc. *Security Best Practices.*
    https://docs.docker.com/build/building/best-practices/#security
    Security guidance for Docker images including non-root execution, secrets management, image scanning, and read-only filesystems. Applied in Install Labs' `/secure` command.

35. **Docker Hub**
    Docker, Inc. *Docker Hub.*
    https://hub.docker.com/
    The primary public registry for Docker images. Distribution channel for containerized agents and base images (Python, Node.js) used in agent Dockerfiles.

36. **NVIDIA Container Toolkit**
    NVIDIA. *NVIDIA Container Toolkit Documentation.*
    https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/
    Documentation for GPU passthrough in Docker containers. Required for packaging AI agents that depend on GPU inference (local LLMs, embedding models, image generation).

---

## 6. OpenAI / Custom GPTs

Custom GPTs represent a major distribution channel for conversational AI agents via the ChatGPT platform.

37. **OpenAI — GPT Actions Documentation**
    OpenAI. *Actions in GPTs.*
    https://platform.openai.com/docs/actions
    Official documentation for GPT Actions, which allow Custom GPTs to call external APIs. Defines the OpenAPI schema format and authentication patterns for action-based agent distribution.

38. **OpenAI — GPTs Overview**
    OpenAI. *GPTs.*
    https://openai.com/index/introducing-gpts/
    OpenAI's announcement and overview of Custom GPTs, explaining the GPT Builder interface, knowledge files, conversation starters, and the GPT Store distribution model.

39. **OpenAPI Specification 3.1.0**
    OpenAPI Initiative. *OpenAPI Specification v3.1.0.*
    https://spec.openapis.org/oas/v3.1.0
    The industry-standard API description format required for GPT Actions. Install Labs generates OpenAPI 3.1 schemas as part of Custom GPT packaging.

40. **OpenAI Platform Documentation**
    OpenAI. *API Reference.*
    https://platform.openai.com/docs/
    Complete OpenAI API documentation covering the Assistants API, function calling, and tool definitions. Provides context for how Custom GPTs relate to the broader OpenAI agent ecosystem.

41. **OpenAI — GPT Store**
    OpenAI. *GPT Store.*
    https://openai.com/index/introducing-the-gpt-store/
    Documentation and announcement of the GPT Store marketplace. Covers listing requirements, revenue sharing, and review guidelines relevant to GPT distribution strategy.

---

## 7. AI Agent Frameworks

Install Labs provides framework-specific packaging guides for all major agent frameworks. These are the official documentation sources.

### 7a. LangChain Ecosystem

42. **LangChain Documentation**
    LangChain, Inc. *LangChain Python Docs.*
    https://python.langchain.com/docs/
    Official documentation for LangChain, the most widely adopted AI agent framework. Covers chains, agents, tools, memory, and retrieval — all of which affect packaging decisions.

43. **LangGraph Documentation**
    LangChain, Inc. *LangGraph.*
    https://langchain-ai.github.io/langgraph/
    Documentation for LangGraph, LangChain's graph-based agent orchestration framework. Its stateful, multi-step agent patterns require specific packaging considerations for persistence and checkpointing.

44. **LangServe Documentation**
    LangChain, Inc. *LangServe.*
    https://python.langchain.com/docs/langserve/
    Documentation for deploying LangChain runnables as REST APIs. Relevant to agent packaging as it provides the HTTP serving layer for containerized LangChain agents.

### 7b. Multi-Agent Frameworks

45. **CrewAI Documentation**
    CrewAI, Inc. *CrewAI Docs.*
    https://docs.crewai.com/
    Official documentation for CrewAI, a multi-agent orchestration framework. Its crew/agent/task abstractions require specific packaging patterns for distributing complete multi-agent systems.

46. **AutoGen Documentation**
    Microsoft. *AutoGen.*
    https://microsoft.github.io/autogen/
    Documentation for Microsoft's AutoGen (now AG2) multi-agent framework. Covers conversable agents, group chat, and code execution patterns that affect containerization and security packaging.

47. **Semantic Kernel Documentation**
    Microsoft. *Semantic Kernel.*
    https://learn.microsoft.com/en-us/semantic-kernel/
    Documentation for Microsoft's Semantic Kernel, an SDK for integrating LLMs into applications. Supports C#, Python, and Java — requiring multi-language packaging guidance.

### 7c. Structured Output & Type-Safe Frameworks

48. **Pydantic AI Documentation**
    Pydantic. *Pydantic AI.*
    https://ai.pydantic.dev/
    Documentation for Pydantic AI, a type-safe agent framework leveraging Pydantic's validation. Its strong typing and dependency injection patterns produce particularly clean, well-structured packages.

49. **LlamaIndex Documentation**
    LlamaIndex. *LlamaIndex Docs.*
    https://docs.llamaindex.ai/
    Documentation for LlamaIndex, a data framework for connecting LLMs to external data. Its index persistence, vector store integrations, and data connectors require specific packaging for data artifacts.

### 7d. Web & Full-Stack Frameworks

50. **Vercel AI SDK Documentation**
    Vercel. *AI SDK.*
    https://sdk.vercel.ai/docs
    Documentation for Vercel's AI SDK providing React hooks and streaming primitives. Relevant for packaging full-stack AI applications with server-side agent logic and client-side streaming UI.

51. **OpenAI Assistants API**
    OpenAI. *Assistants Overview.*
    https://platform.openai.com/docs/assistants/overview
    Documentation for OpenAI's Assistants API with built-in tools (code interpreter, retrieval, function calling). Packaging differs from other frameworks as state is managed server-side by OpenAI.

### 7e. Lightweight & Research Frameworks

52. **Smolagents Documentation**
    Hugging Face. *Smolagents.*
    https://huggingface.co/docs/smolagents/
    Documentation for Hugging Face's lightweight agent library. Its minimal dependency footprint and code-agent pattern make it particularly suitable for pip/uvx distribution.

### 7f. No-Code / Low-Code Platforms

53. **n8n Documentation**
    n8n GmbH. *n8n Docs.*
    https://docs.n8n.io/
    Documentation for n8n, a workflow automation platform. Its Docker-native architecture and webhook-based integrations require specific packaging patterns for self-hosted distribution.

54. **Flowise Documentation**
    FlowiseAI. *Flowise Docs.*
    https://docs.flowiseai.com/
    Documentation for Flowise, a drag-and-drop LLM flow builder. Distributed via npm (`npx flowise start`) and Docker, making it a reference example for visual agent packaging.

55. **Dify Documentation**
    Dify. *Dify Docs.*
    https://docs.dify.ai/
    Documentation for Dify, an open-source LLM app development platform. Its Docker Compose-based deployment and plugin architecture inform packaging patterns for platform-type agents.

56. **ComfyUI — GitHub Repository**
    comfyanonymous. *ComfyUI.*
    https://github.com/comfyanonymous/ComfyUI
    Node-based Stable Diffusion interface with a unique workflow-as-JSON format. Its model weight management and custom node distribution system present unique packaging challenges.

---

## 8. Security Standards & Tools

Security is a cross-cutting concern in agent distribution. These references inform Install Labs' `/secure` command and the agent security skill.

### 8a. Security Standards & Frameworks

57. **OWASP Top 10**
    OWASP Foundation. *OWASP Top 10.*
    https://owasp.org/www-project-top-ten/
    The industry-standard awareness document for web application security risks. Applied to agent packaging for identifying API exposure, injection, and misconfiguration vulnerabilities.

58. **OWASP Top 10 for LLM Applications**
    OWASP Foundation. *OWASP Top 10 for LLMs.*
    https://owasp.org/www-project-top-10-for-large-language-model-applications/
    LLM-specific security risks including prompt injection, insecure output handling, and supply chain vulnerabilities. Directly applicable to securing AI agent packages.

59. **NIST Secure Software Development Framework (SSDF)**
    National Institute of Standards and Technology. *NIST SP 800-218.*
    https://csrc.nist.gov/Projects/ssdf
    NIST framework for secure software development practices. Provides the supply chain security foundation applied to agent build pipelines and dependency management.

60. **NIST AI Risk Management Framework**
    National Institute of Standards and Technology. *AI RMF 1.0.*
    https://www.nist.gov/artificial-intelligence/executive-order-safe-secure-and-trustworthy-artificial-intelligence
    Framework for managing risks in AI systems. Contextualizes agent packaging security within the broader AI safety and trustworthiness landscape.

### 8b. Supply Chain Security Tools

61. **Sigstore**
    Sigstore Project. *Sigstore.*
    https://www.sigstore.dev/
    Keyless code signing infrastructure for software supply chains. Underpins npm provenance and is increasingly relevant for verifying the authenticity of published agent packages.

62. **pip-audit**
    Python Packaging Authority (PyPA). *pip-audit.*
    https://github.com/pypa/pip-audit
    Tool for scanning Python environments and packages for known vulnerabilities using the OSV database. Recommended in Install Labs' pre-ship checklist for Python agent packages.

63. **Trivy — Container Security Scanner**
    Aqua Security. *Trivy.*
    https://github.com/aquasecurity/trivy
    Comprehensive vulnerability scanner for containers, filesystems, and Git repositories. Used for scanning Docker-based agent images for OS and library vulnerabilities before distribution.

64. **TruffleHog — Secrets Detection**
    Truffle Security. *TruffleHog.*
    https://github.com/trufflesecurity/trufflehog
    Secrets detection tool that scans Git history, filesystems, and S3 buckets for leaked credentials. Critical pre-ship check to ensure API keys and tokens are not embedded in agent packages.

65. **Socket Security**
    Socket, Inc. *Socket.*
    https://socket.dev/
    Supply chain security platform that detects malicious and risky npm/PyPI packages. Relevant for auditing agent dependencies before distribution.

### 8c. Model Security & Formats

66. **SafeTensors Format**
    Hugging Face. *SafeTensors Documentation.*
    https://huggingface.co/docs/safetensors/
    Safe, zero-copy tensor serialization format that prevents arbitrary code execution during model loading. Required format for distributing model weights with AI agents (replacing pickle-based formats).

67. **GGUF Format Specification**
    ggerganov. *GGUF Specification.*
    https://github.com/ggerganov/ggml/blob/master/docs/gguf.md
    Specification for the GGUF (GPT-Generated Unified Format) used by llama.cpp and derived projects. The standard format for distributing quantized LLM weights with local inference agents.

---

## 9. Cloud Deployment Platforms

Cloud platforms provide one-click deployment targets for containerized AI agents. These references cover the platforms supported by Install Labs' Docker packaging skill.

68. **Railway Documentation**
    Railway Corporation. *Railway Docs.*
    https://docs.railway.app/
    Documentation for Railway, a cloud platform with Git-based deployments. Supports one-click deploy buttons, Dockerfile detection, and environment variable injection — ideal for agent distribution.

69. **Render Documentation**
    Render. *Render Docs.*
    https://docs.render.com/
    Documentation for Render's cloud platform supporting web services, background workers, and cron jobs. Its `render.yaml` infrastructure-as-code format enables declarative agent deployment.

70. **Google Cloud Run Documentation**
    Google Cloud. *Cloud Run Docs.*
    https://cloud.google.com/run/docs
    Documentation for Google Cloud Run, a fully managed serverless container platform. Its scale-to-zero pricing model is particularly suited to on-demand AI agent deployment.

71. **Hugging Face Spaces Documentation**
    Hugging Face. *Spaces Docs.*
    https://huggingface.co/docs/hub/spaces
    Documentation for Hugging Face Spaces, a hosting platform for ML demos and applications. Supports Gradio, Streamlit, and Docker Spaces — a primary distribution channel for ML-based agents.

72. **Replicate — Cog**
    Replicate. *Cog — Containers for Machine Learning.*
    https://github.com/replicate/cog
    Open-source tool for packaging ML models into Docker containers with a standard prediction interface. Enables distributing inference-based agents to the Replicate platform.

73. **Fly.io Documentation**
    Fly.io. *Fly.io Docs.*
    https://fly.io/docs/
    Documentation for Fly.io, an edge computing platform running containers globally. Its `fly.toml` configuration and Machines API support low-latency agent deployment.

74. **Vercel Documentation**
    Vercel. *Vercel Docs.*
    https://vercel.com/docs
    Documentation for Vercel's frontend and serverless platform. Relevant for deploying full-stack AI agents with Next.js frontends and serverless API routes.

---

## 10. Software Distribution Standards

Foundational standards and principles that govern software packaging and distribution across all platforms.

75. **Semantic Versioning 2.0.0**
    Tom Preston-Werner. *SemVer.*
    https://semver.org/
    The versioning standard (MAJOR.MINOR.PATCH) used by npm, PyPI, and virtually all package managers. Governs how agent updates communicate breaking changes, new features, and bug fixes to users.

76. **The Twelve-Factor App**
    Adam Wiggins. *The Twelve-Factor App.*
    https://12factor.net/
    Methodology for building software-as-a-service applications. Factor III (Config via environment variables), Factor V (Build/Release/Run), and Factor X (Dev/Prod parity) are directly applicable to agent packaging.

77. **XDG Base Directory Specification**
    freedesktop.org. *XDG Base Directory Specification.*
    https://specifications.freedesktop.org/basedir-spec/latest/
    Standard for user-specific data, configuration, and cache file locations on Unix-like systems. Determines where agent configuration files (`~/.config/`), data, and caches should be stored.

78. **SPDX License List**
    SPDX Project. *SPDX License List.*
    https://spdx.org/licenses/
    Standardized list of open-source license identifiers used in `package.json` (`license` field) and `pyproject.toml` (`license` field). Required for correct licensing metadata in agent packages.

79. **Keep a Changelog**
    Olivier Lacan. *Keep a Changelog.*
    https://keepachangelog.com/
    Convention for maintaining human-readable changelogs. Recommended practice for agent packages to communicate changes between versions to users and integrators.

80. **Conventional Commits**
    Conventional Commits Contributors. *Conventional Commits 1.0.0.*
    https://www.conventionalcommits.org/
    Specification for structured commit messages enabling automated changelog generation and semantic version bumping. Useful for agent packages with CI/CD-based release pipelines.

### 10a. Platform-Specific Distribution

81. **Apple Developer — Notarizing macOS Software**
    Apple, Inc. *Notarizing macOS Software Before Distribution.*
    https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution
    Apple's documentation on code signing and notarization requirements for macOS distribution. Relevant for agents distributed as native macOS applications or Electron apps.

82. **Apple Developer — Code Signing Guide**
    Apple, Inc. *Code Signing Guide.*
    https://developer.apple.com/library/archive/documentation/Security/Conceptual/CodeSigningGuide/
    Guide to macOS code signing concepts and workflows. Required for understanding Gatekeeper restrictions that affect agent installation on macOS.

83. **Microsoft — Authenticode Code Signing**
    Microsoft. *Introduction to Code Signing.*
    https://learn.microsoft.com/en-us/windows/win32/seccrypto/cryptography-tools
    Documentation for Windows Authenticode digital signatures. Relevant for agents distributed as Windows executables or installers.

---

## 11. MCP Registries & Marketplaces

Emerging marketplaces and registries for discovering and installing MCP servers — the primary discovery mechanism for MCP-based agents.

84. **Smithery**
    Smithery. *Smithery — MCP Server Registry.*
    https://smithery.ai
    MCP server registry and marketplace providing one-click installation, categorized browsing, and the `@smithery/cli` for automated MCP server setup. A primary distribution channel for MCP agents.

85. **mcp.so**
    mcp.so. *MCP Server Directory.*
    https://mcp.so
    Community-curated directory of MCP servers with search, categorization, and installation instructions. Functions as a discovery layer for the MCP ecosystem.

86. **Glama MCP Directory**
    Glama. *MCP Servers.*
    https://glama.ai/mcp
    Curated MCP server directory by Glama, featuring server descriptions, tool listings, and installation instructions. Provides an alternative discovery channel for MCP agent distribution.

87. **PulseMCP**
    PulseMCP. *MCP Server Directory.*
    https://pulsemcp.com
    MCP server discovery platform tracking the growing ecosystem of MCP servers across categories. Useful for competitive analysis and positioning when distributing MCP agents.

88. **mcp.run**
    mcp.run. *MCP Server Hosting.*
    https://mcp.run
    Platform for hosting and running MCP servers remotely, enabling serverless MCP distribution without requiring users to install and run servers locally.

---

## 12. Cognitive Science & UX of Installation

Academic and practitioner research that informs the design of agent installation flows, packaging decisions, and distribution strategy.

### 12a. Foundational Cognitive Science

89. **Sweller, J. (1988). "Cognitive Load During Problem Solving: Effects on Learning."**
    *Cognitive Science,* 12(2), 257-285.
    DOI: https://doi.org/10.1207/s15516709cog1202_4
    Foundational paper on Cognitive Load Theory. Directly applicable to agent install flows: excessive configuration options, dependency resolution messages, and error states increase extraneous cognitive load, reducing install completion rates.

90. **Hick, W. E. (1952). "On the Rate of Gain of Information."**
    *Quarterly Journal of Experimental Psychology,* 4(1), 11-26.
    DOI: https://doi.org/10.1080/17470215208416600
    Establishes that decision time increases logarithmically with the number of choices (Hick's Law). Applied to agent packaging: offering fewer, clearer install options (e.g., `npx` vs. full clone) reduces abandonment at the installation decision point.

91. **Miller, G. A. (1956). "The Magical Number Seven, Plus or Minus Two."**
    *Psychological Review,* 63(2), 81-97.
    DOI: https://doi.org/10.1037/h0043158
    Classic paper on working memory capacity limits. Informs the design of agent configuration: install instructions should present no more than 5-7 configuration parameters before chunking into advanced/optional sections.

### 12b. Behavior Design & Persuasion

92. **Fogg, B. J. (2009). "A Behavior Model for Persuasive Design."**
    *Proceedings of the 4th International Conference on Persuasive Technology.*
    DOI: https://doi.org/10.1145/1541948.1541999
    The Fogg Behavior Model (B = MAT: Behavior = Motivation x Ability x Trigger). Applied to install completion: high motivation (agent solves a real problem) must be paired with high ability (one-command install) and a clear trigger (install button/command) for successful adoption.

93. **Cialdini, R. B. (2006). *Influence: The Psychology of Persuasion.***
    Harper Business. Revised Edition.
    ISBN: 978-0061241895
    Six principles of persuasion (reciprocity, commitment, social proof, authority, liking, scarcity). Social proof (star counts, download numbers) and authority (official registry listing) directly influence agent adoption decisions.

### 12c. Usability & Design Practice

94. **Nielsen, J. (1994). *Usability Engineering.***
    Morgan Kaufmann Publishers.
    ISBN: 978-0125184069
    Seminal work establishing the 10 usability heuristics. Applied to installer UX: visibility of system status (install progress), error prevention (dependency pre-checks), and help/documentation (clear error messages) are critical for install completion.

95. **Nielsen, J. (1994). "10 Usability Heuristics for User Interface Design."**
    Nielsen Norman Group.
    https://www.nngroup.com/articles/ten-usability-heuristics/
    The canonical online reference for Nielsen's heuristics. Applied to agent packaging: "Match between system and real world" (use familiar install commands), "Aesthetic and minimalist design" (minimal configuration surface).

96. **Krug, S. (2014). *Don't Make Me Think, Revisited.***
    New Riders. Third Edition.
    ISBN: 978-0321965516
    Defines the "don't make me think" principle for web usability. Applied to agent install flows: the ideal installation requires zero decisions — a single command that works on the first try with sensible defaults.

97. **Norman, D. (2013). *The Design of Everyday Things.***
    Basic Books. Revised and Expanded Edition.
    ISBN: 978-0465050659
    Introduces mental models, affordances, signifiers, and mapping in design. Applied to agent packaging: users' mental model of "install" (one command, immediate use) must match the actual install experience. Conceptual model mismatches (unexpected configuration, manual steps) cause abandonment.

### 12d. Developer Experience Research

98. **Beyer, B., Jones, C., Petoff, J., & Murphy, N. R. (2016). *Site Reliability Engineering.***
    O'Reilly Media.
    Available at: https://sre.google/sre-book/table-of-contents/
    Google's SRE handbook. Chapters on release engineering, simplicity, and monitoring inform agent distribution practices: reproducible builds, progressive rollouts, and health check endpoints.

99. **Winters, T., Manshreck, T., & Wright, H. (2020). *Software Engineering at Google.***
    O'Reilly Media.
    Available at: https://abseil.io/resources/swe-book
    Google's software engineering practices. Chapters on dependency management ("The Diamond Dependency Problem"), versioning policy, and large-scale changes inform agent package dependency strategy.

---

## 13. Python Enhancement Proposals (PEPs)

Specific PEPs that define the Python packaging standards implemented by Install Labs' PyPI packaging skill.

100. **PEP 517 — A Build-System Independent Format for Source Trees**
     Python Software Foundation. *PEP 517.*
     https://peps.python.org/pep-0517/
     Defines the build-backend interface in `pyproject.toml` (`[build-system]` table). Foundation for modern Python packaging that decouples build backends from the installer.

101. **PEP 518 — Specifying Minimum Build System Requirements**
     Python Software Foundation. *PEP 518.*
     https://peps.python.org/pep-0518/
     Establishes `pyproject.toml` as the build configuration file and defines the `[build-system]` table with `requires` and `build-backend` keys. The file that replaced `setup.py` for modern Python agents.

102. **PEP 621 — Storing Project Metadata in pyproject.toml**
     Python Software Foundation. *PEP 621.*
     https://peps.python.org/pep-0621/
     Standardizes the `[project]` table in `pyproject.toml` for declaring package metadata (name, version, dependencies, entry points). The primary metadata format Install Labs generates for Python agents.

103. **PEP 660 — Editable Installs for pyproject.toml Based Builds**
     Python Software Foundation. *PEP 660.*
     https://peps.python.org/pep-0660/
     Defines editable installs (`pip install -e .`) for PEP 517 build backends. Essential for agent development workflows where developers need live-reload during packaging iteration.

104. **PEP 723 — Inline Script Metadata**
     Python Software Foundation. *PEP 723.*
     https://peps.python.org/pep-0723/
     Allows embedding dependency metadata in single-file Python scripts via `# /// script` comments. Enables distributing simple Python agents as single files with declared dependencies, executable via `uv run`.

105. **PEP 440 — Version Identification and Dependency Specification**
     Python Software Foundation. *PEP 440.*
     https://peps.python.org/pep-0440/
     Defines Python version numbering and dependency specifier syntax (`>=`, `~=`, `==`). Required knowledge for declaring agent dependency version constraints in `pyproject.toml`.

---

## 14. Supplementary Standards & Specifications

Additional standards and references that support specific aspects of agent packaging and distribution.

### 14a. API & Schema Standards

106. **JSON Schema Specification**
     JSON Schema. *JSON Schema.*
     https://json-schema.org/
     The vocabulary for annotating and validating JSON documents. Used in MCP tool input schemas, OpenAPI definitions for GPT Actions, and `package.json` / `pyproject.toml` validation.

107. **JSON-RPC 2.0 Specification**
     JSON-RPC Working Group. *JSON-RPC 2.0.*
     https://www.jsonrpc.org/specification
     The remote procedure call protocol used by MCP for client-server communication. Understanding JSON-RPC is essential for debugging MCP server transport issues during development and testing.

### 14b. CI/CD & Automation

108. **GitHub Actions Documentation**
     GitHub. *GitHub Actions.*
     https://docs.github.com/en/actions
     Documentation for GitHub's CI/CD platform. Used for automating agent package builds, tests, and publishing (npm publish, PyPI upload, Docker push) on every release.

109. **GitHub — Trusted Publishers (PyPI OIDC)**
     GitHub. *Publishing Packages to PyPI with a Trusted Publisher.*
     https://docs.github.com/en/actions/use-cases-and-examples/publishing-packages/publishing-python-packages
     Guide for configuring keyless PyPI publishing from GitHub Actions using OpenID Connect. The recommended authentication method for automated Python agent publishing.

### 14c. Configuration & Environment

110. **dotenv Specification**
     motdotla. *dotenv.*
     https://github.com/motdotla/dotenv
     The de facto standard for loading environment variables from `.env` files. Used by virtually all AI agents for API key configuration; must be in `.gitignore` and excluded from published packages.

111. **direnv — Environment Switcher**
     direnv contributors. *direnv.*
     https://direnv.net/
     Tool for loading/unloading environment variables based on the current directory. Alternative to dotenv for managing agent configuration in development environments.

### 14d. Documentation & Developer Onboarding

112. **README-driven Development**
     Tom Preston-Werner. *Readme Driven Development.* August 2010.
     https://tom.preston-werner.com/2010/08/23/readme-driven-development.html
     Essay advocating writing the README before writing code. Applied to agent packaging: the install instructions and README are the product — writing them first exposes packaging friction before code is written.

113. **The Documentation System (Diataxis)**
     Daniele Procida. *Diataxis.*
     https://diataxis.fr/
     Framework organizing documentation into tutorials, how-to guides, reference, and explanation. Applied to agent packaging: install instructions are a "how-to guide," while configuration reference is "reference documentation."

### 14e. Licensing

114. **Choose a License**
     GitHub. *Choose an Open Source License.*
     https://choosealicense.com/
     Plain-language guide to open-source licenses. Essential resource for agent authors selecting a license before publishing to npm, PyPI, or Docker Hub.

115. **Creative Commons Licenses**
     Creative Commons. *About The Licenses.*
     https://creativecommons.org/licenses/
     License suite commonly used for model weights, training data, and documentation distributed alongside AI agents. Understanding CC-BY, CC-BY-SA, and CC-BY-NC is important for model weight redistribution.

---

## Citation Format

References in this document follow a modified APA style adapted for software documentation:

```
Author/Organization. "Title." Publication/Platform.
URL
Annotation describing relevance to agent packaging.
```

For academic papers, standard APA format is used with DOI links where available.

---

## Contributing

To suggest additions or corrections to this reference list, verify that:

1. The URL is publicly accessible and returns a 200 status
2. The source is an official/primary source (not a blog repost or summary)
3. The annotation clearly explains relevance to AI agent packaging and distribution
4. Academic references include DOI links where available

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-02-24 | Initial comprehensive reference list (115 references, 14 sections) |

---

**Install Labs v2.0.0** | **phazurlabs**
