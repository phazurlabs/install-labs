---
name: mcp-server-packaging
description: "Complete guide to packaging AI agents as MCP (Model Context Protocol) servers. Use when the user mentions: MCP, Model Context Protocol, MCP server, MCP tool, MCP resource, MCP prompt, claude desktop config, claude_desktop_config.json, stdio transport, streamable http, mcpb, desktop extension, mcp registry, smithery, mcp inspector, mcp sdk, mcp packaging, mcp distribution, mcp install, remote mcp, local mcp server, mcp environment variables, mcp config injection"
---

# MCP Server Packaging

## What Is MCP

Model Context Protocol (MCP) is the open standard for connecting AI systems to external tools, data sources, and services. Developed by Anthropic and adopted across the AI ecosystem, MCP defines a structured interface between an AI host (Claude Desktop, Claude Code, Cursor, Windsurf) and external capabilities.

MCP servers expose three primitive types:

| Primitive  | Purpose                                | Example                                      |
|------------|----------------------------------------|----------------------------------------------|
| **Tools**  | Functions the AI can call              | `search_database`, `create_ticket`           |
| **Resources** | Data the AI can read                | `file://config.yaml`, `db://users/123`       |
| **Prompts**   | Reusable prompt templates           | `summarize-code`, `review-pr`                |

MCP is to AI agents what USB is to peripherals: a universal plug that lets any compliant host talk to any compliant server without custom integration code.

### Why MCP Is the #1 Distribution Target

- Claude Desktop, Claude Code, Cursor, Windsurf, Cline, and Zed all support MCP natively.
- A single MCP server works across every compatible host with zero code changes.
- The protocol handles capability negotiation, lifecycle management, and error propagation.
- Registry infrastructure (official registry, Smithery, mcp.so) is maturing rapidly.

---

## MCP Server Architecture

### Transport Layers

MCP supports two transport mechanisms. Choosing the right one is the most consequential architectural decision.

**stdio (Local Servers)**

The host spawns the MCP server as a child process. Communication happens over stdin/stdout using JSON-RPC 2.0 messages, one per line.

- Zero network configuration. No ports, no firewall rules.
- Server lifecycle is managed by the host (started on demand, killed on exit).
- Best for: CLI tools, filesystem access, local development utilities.
- The server MUST NOT write anything to stdout except valid JSON-RPC messages. Use stderr for logging.

**Streamable HTTP (Remote Servers)**

The server runs as a web service. The host connects via HTTP with optional Server-Sent Events (SSE) for streaming.

- Server runs independently in the cloud or on a remote machine.
- Supports authentication, multi-tenant usage, and horizontal scaling.
- Best for: SaaS integrations, shared team tools, managed services.
- Replaces the deprecated SSE-only transport from the earlier MCP specification.

### Lifecycle Management

1. **Initialize**: Host sends `initialize` with its capabilities. Server responds with its own capabilities and supported protocol version.
2. **Initialized**: Host sends `initialized` notification. Server can begin accepting requests.
3. **Operation**: Host calls tools, reads resources, renders prompts.
4. **Shutdown**: Host closes the transport. For stdio, this means closing stdin and terminating the child process.

### Tool Registration

Tools are the most common primitive. Each tool declares a name, description, and input schema (JSON Schema).

```typescript
server.tool(
  "search_issues",
  "Search GitHub issues by query string",
  {
    query: z.string().describe("Search query"),
    repo: z.string().describe("Repository in owner/name format"),
    state: z.enum(["open", "closed", "all"]).default("open")
  },
  async ({ query, repo, state }) => {
    const results = await github.searchIssues(repo, query, state);
    return {
      content: [{ type: "text", text: JSON.stringify(results, null, 2) }]
    };
  }
);
```

### Resource Registration

Resources expose read-only data. They use URI templates for dynamic access.

```typescript
server.resource(
  "issue",
  new ResourceTemplate("github://issues/{owner}/{repo}/{number}", {
    list: async () => ({ resources: cachedIssueList })
  }),
  async (uri, { owner, repo, number }) => ({
    contents: [{ uri: uri.href, mimeType: "application/json", text: JSON.stringify(issue) }]
  })
);
```

### Prompt Registration

Prompts are reusable templates that the host can render with arguments.

```typescript
server.prompt(
  "review-pr",
  "Review a pull request for code quality and security",
  { pr_url: z.string().describe("Pull request URL") },
  async ({ pr_url }) => ({
    messages: [
      { role: "user", content: { type: "text", text: `Review this PR: ${pr_url}` } }
    ]
  })
);
```

---

## Distribution Methods (Ranked by UX Quality)

### 1. Desktop Extension (.mcpb) -- Best UX

A `.mcpb` file is a ZIP archive containing a manifest, optional binaries, and an icon. The user downloads the file, opens it, and Claude Desktop presents an "Install" dialog. One click and the server is configured.

```
my-server.mcpb
  manifest.json    # Server metadata, config schema, transport config
  icon.png         # 512x512 icon displayed in the Extensions panel
  bin/             # Optional bundled binaries
```

The manifest declares how to launch the server, what environment variables it needs, and what permissions it requests.

### 2. Remote URL (Streamable HTTP) -- Zero Install

The server runs in the cloud. The user pastes a URL into Claude Desktop's "Add Integration" dialog or into their config file.

```json
{
  "mcpServers": {
    "my-saas-tool": {
      "url": "https://mcp.example.com/v1",
      "headers": {
        "Authorization": "Bearer ${MY_API_KEY}"
      }
    }
  }
}
```

No local installation. No Node.js or Python required on the user's machine. The server operator handles updates, scaling, and availability. This is the future of MCP distribution for SaaS products.

### 3. Claude Code CLI

For developers using Claude Code, the CLI provides a single command to add an MCP server.

```bash
claude mcp add my-server -- npx -y @org/my-server
```

This writes the server configuration to Claude Code's settings. Supports `--scope` flag for user-level vs. project-level configuration.

```bash
# Project-level (stored in .mcp.json, shareable via git)
claude mcp add my-server --scope project -- npx -y @org/my-server

# User-level (stored in user settings)
claude mcp add my-server --scope user -- npx -y @org/my-server
```

### 4. npx / uvx One-Liner

The most common distribution method today. Users add a JSON block to their config file.

**Node.js (npx):**
```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["-y", "@org/my-server"],
      "env": {
        "API_KEY": "user-provides-this"
      }
    }
  }
}
```

**Python (uvx):**
```json
{
  "mcpServers": {
    "my-server": {
      "command": "uvx",
      "args": ["my-server"],
      "env": {
        "API_KEY": "user-provides-this"
      }
    }
  }
}
```

The `-y` flag in npx auto-confirms the install prompt. uvx is the uv tool runner that creates an isolated environment per invocation.

### 5. Docker

For servers with complex dependencies or when sandboxing is required.

```json
{
  "mcpServers": {
    "my-server": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-e", "API_KEY",
        "ghcr.io/org/my-server:latest"
      ],
      "env": {
        "API_KEY": "user-provides-this"
      }
    }
  }
}
```

Note: Docker containers use stdio transport. The `-i` flag is required to keep stdin open. The `--rm` flag cleans up the container on exit.

### 6. Manual JSON Config -- Worst UX

The user hand-edits `claude_desktop_config.json` (macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`). This is error-prone and should be a last resort. Always provide a copy-paste-ready JSON block in your README.

---

## Complete package.json for MCP Server (npm)

```json
{
  "name": "@your-org/mcp-server-example",
  "version": "1.0.0",
  "description": "MCP server that provides example tools for Claude",
  "type": "module",
  "bin": {
    "mcp-server-example": "./dist/index.js"
  },
  "files": [
    "dist/"
  ],
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "start": "node dist/index.js",
    "inspect": "npx @modelcontextprotocol/inspector node dist/index.js",
    "prepublishOnly": "npm run build"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.12.0",
    "zod": "^3.23.0"
  },
  "devDependencies": {
    "@types/node": "^22.0.0",
    "typescript": "^5.7.0"
  },
  "engines": {
    "node": ">=18.0.0"
  },
  "keywords": [
    "mcp",
    "mcp-server",
    "model-context-protocol",
    "claude",
    "ai-tools"
  ],
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/your-org/mcp-server-example"
  }
}
```

### Minimal MCP Server Entry Point (TypeScript)

```typescript
#!/usr/bin/env node

import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "my-server",
  version: "1.0.0",
});

server.tool(
  "hello",
  "Say hello to someone",
  { name: z.string() },
  async ({ name }) => ({
    content: [{ type: "text", text: `Hello, ${name}!` }],
  })
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

The shebang (`#!/usr/bin/env node`) is mandatory for npx execution. Without it, the OS does not know to run the file with Node.js.

---

## Complete pyproject.toml for MCP Server (Python)

```toml
[project]
name = "mcp-server-example"
version = "1.0.0"
description = "MCP server that provides example tools for Claude"
readme = "README.md"
license = { text = "MIT" }
requires-python = ">=3.10"
dependencies = [
    "mcp[cli]>=1.6.0",
]
keywords = ["mcp", "mcp-server", "model-context-protocol", "claude", "ai-tools"]

[project.scripts]
mcp-server-example = "mcp_server_example:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

The `[project.scripts]` entry is what makes `uvx mcp-server-example` work. It creates an executable that calls the `main()` function in your package.

### Minimal MCP Server Entry Point (Python)

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-server")

@mcp.tool()
def hello(name: str) -> str:
    """Say hello to someone."""
    return f"Hello, {name}!"

def main():
    mcp.run(transport="stdio")

if __name__ == "__main__":
    main()
```

---

## MCP Registries

### Official MCP Registry

**URL:** `https://registry.modelcontextprotocol.io`

The canonical registry maintained by the MCP project. Servers are indexed from npm and PyPI based on naming conventions and metadata. To get listed:

1. Publish your package to npm or PyPI.
2. Include `mcp-server` in your package keywords/classifiers.
3. Name your package with the `mcp-server-` prefix (recommended).
4. Submit via the registry's GitHub repository if auto-discovery misses you.

### Smithery

**URL:** `https://smithery.ai`

Marketplace and managed hosting platform for MCP servers. Provides:

- One-command install: `npx @smithery/cli install @org/server --client claude`
- Managed hosting for remote (Streamable HTTP) servers.
- Analytics dashboard showing usage and error rates.
- User reviews and ratings.

To publish to Smithery, add a `smithery.yaml` file to your repository root:

```yaml
startCommand:
  type: stdio
  configSchema:
    type: object
    properties:
      apiKey:
        type: string
        description: "Your API key"
    required:
      - apiKey
  commandFunction:
    - |-
      (config) => ({
        command: "node",
        args: ["dist/index.js"],
        env: { API_KEY: config.apiKey }
      })
```

### Other Registries

| Registry   | URL                    | Focus                                    |
|------------|------------------------|------------------------------------------|
| mcp.so     | https://mcp.so         | Community directory, categories, search  |
| Glama      | https://glama.ai/mcp   | Curated MCP server listings              |
| PulseMCP   | https://pulsemcp.com   | Server discovery and monitoring          |

---

## Config Injection Patterns

When building an installer or setup script, you may need to programmatically add your server to the user's Claude Desktop configuration.

### Config File Locations

| Platform | Path                                                                 |
|----------|----------------------------------------------------------------------|
| macOS    | `~/Library/Application Support/Claude/claude_desktop_config.json`   |
| Windows  | `%APPDATA%\Claude\claude_desktop_config.json`                       |
| Linux    | `~/.config/Claude/claude_desktop_config.json`                       |

### Safe Config Injection (Node.js)

```typescript
import { readFileSync, writeFileSync, mkdirSync } from "fs";
import { join } from "path";
import { homedir } from "os";

function getConfigPath(): string {
  const home = homedir();
  switch (process.platform) {
    case "darwin":
      return join(home, "Library", "Application Support", "Claude", "claude_desktop_config.json");
    case "win32":
      return join(process.env.APPDATA || "", "Claude", "claude_desktop_config.json");
    default:
      return join(home, ".config", "Claude", "claude_desktop_config.json");
  }
}

function injectMcpServer(serverName: string, serverConfig: object): void {
  const configPath = getConfigPath();
  let config: Record<string, any> = {};

  try {
    config = JSON.parse(readFileSync(configPath, "utf-8"));
  } catch {
    // File doesn't exist yet, start with empty config
    mkdirSync(join(configPath, ".."), { recursive: true });
  }

  config.mcpServers = config.mcpServers || {};
  config.mcpServers[serverName] = serverConfig;

  writeFileSync(configPath, JSON.stringify(config, null, 2));
}
```

Always read-modify-write. Never overwrite the entire config file blindly, or you will destroy the user's other MCP server configurations.

---

## Environment Variable Handling

### In Config (Recommended for Desktop)

```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["-y", "@org/my-server"],
      "env": {
        "API_KEY": "sk-abc123",
        "DATABASE_URL": "postgres://localhost/mydb"
      }
    }
  }
}
```

### In Server Code (Fail Gracefully)

```typescript
const apiKey = process.env.API_KEY;
if (!apiKey) {
  console.error("ERROR: API_KEY environment variable is required.");
  console.error("Add it to your MCP server config under the 'env' field.");
  process.exit(1);
}
```

Never crash silently. Always tell the user exactly which variable is missing and where to set it.

### Keychain Integration (Advanced)

For sensitive credentials, consider reading from the system keychain instead of environment variables. On macOS, use `security find-generic-password`. On Linux, use `secret-tool`. This avoids storing API keys in plaintext config files.

---

## Testing

### MCP Inspector

The official interactive testing tool for MCP servers.

```bash
# Test a local server
npx @modelcontextprotocol/inspector node dist/index.js

# Test a Python server
npx @modelcontextprotocol/inspector uvx my-server

# Test a remote server
npx @modelcontextprotocol/inspector https://mcp.example.com/v1
```

The Inspector opens a web UI where you can:
- See all registered tools, resources, and prompts.
- Call tools with custom inputs and inspect the JSON-RPC messages.
- Verify the initialize handshake.
- Debug transport issues.

### Manual Testing with Claude Desktop

1. Add your server to `claude_desktop_config.json`.
2. Restart Claude Desktop (Cmd+Q, reopen).
3. Open a new conversation and look for the hammer icon (tools) in the composer.
4. Click it to verify your tools appear in the list.
5. Ask Claude to use your tool by name: "Use the search_issues tool to find open bugs."

### Automated Testing

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { InMemoryTransport } from "@modelcontextprotocol/sdk/inMemory.js";
import { Client } from "@modelcontextprotocol/sdk/client/index.js";

// Create server and client connected via in-memory transport
const server = createYourServer();
const [clientTransport, serverTransport] = InMemoryTransport.createLinkedPair();

await server.connect(serverTransport);

const client = new Client({ name: "test-client", version: "1.0.0" });
await client.connect(clientTransport);

// Call a tool
const result = await client.callTool({ name: "hello", arguments: { name: "World" } });
assert(result.content[0].text === "Hello, World!");
```

---

## Publishing Workflow

### npm (Node.js/TypeScript)

```bash
# 1. Build
npm run build

# 2. Verify package contents
npm pack --dry-run

# 3. Publish
npm publish --access public

# 4. Verify npx works
npx -y @your-org/mcp-server-example
```

### PyPI (Python)

```bash
# 1. Build
python -m build

# 2. Upload to PyPI
python -m twine upload dist/*

# 3. Verify uvx works
uvx mcp-server-example
```

### Post-Publish Checklist

- [ ] Verify `npx -y @org/server` or `uvx server-name` launches without errors.
- [ ] Submit to the official MCP registry.
- [ ] Submit to Smithery if you want managed hosting.
- [ ] Add README badges: npm version, MCP compatible, license.
- [ ] Include a copy-paste config block in your README for Claude Desktop users.
- [ ] Test on macOS, Windows, and Linux if your server does filesystem operations.

---

## Common Pitfalls

### Missing Shebang
**Symptom:** `npx @org/server` fails with "Permission denied" or "not recognized as a script."
**Fix:** Add `#!/usr/bin/env node` as the very first line of your entry point. Ensure the built `.js` file in `dist/` retains it (TypeScript strips it by default -- use a build plugin or prepend it in your build script).

### Writing to stdout
**Symptom:** Claude Desktop shows "Server disconnected" or garbled responses.
**Fix:** Never use `console.log()` in an MCP server using stdio transport. All stdout is part of the JSON-RPC stream. Use `console.error()` or `server.sendLoggingMessage()` instead.

### Crashing on Missing Environment Variables
**Symptom:** Server starts and immediately exits. No useful error message.
**Fix:** Check for required env vars at startup, print a human-readable error to stderr, and exit with code 1.

### Hardcoded API Keys
**Symptom:** Security vulnerability. Keys end up in npm registry or Git history.
**Fix:** Always read credentials from environment variables. Document required env vars in your README.

### Not Testing Cross-Platform
**Symptom:** Works on macOS, breaks on Windows (path separators, missing commands).
**Fix:** Use `path.join()` instead of string concatenation. Test with `npx` on all target platforms. Use `process.platform` for platform-specific logic.

### stdio vs HTTP Confusion
**Symptom:** Server configured as stdio but listening on a port, or configured as HTTP but reading from stdin.
**Fix:** Match transport to config. If the config uses `"command"` and `"args"`, the server must use `StdioServerTransport`. If the config uses `"url"`, the server must be an HTTP server.

### Publishing src/ Instead of dist/
**Symptom:** Package is bloated. TypeScript source ships to users. Potential breakage if users lack build tools.
**Fix:** Set `"files": ["dist/"]` in package.json. Run `npm pack --dry-run` before publishing to verify only compiled output is included.

### Forgetting prepublishOnly
**Symptom:** You publish stale code because you forgot to rebuild before `npm publish`.
**Fix:** Add `"prepublishOnly": "npm run build"` to your scripts. npm runs this automatically before every publish.

### Version Mismatch with MCP SDK
**Symptom:** "Unsupported protocol version" errors in Claude Desktop.
**Fix:** Keep your `@modelcontextprotocol/sdk` dependency up to date. The protocol version in the SDK must match what the host expects. Pin to a stable version and test after upgrades.

---

## Sources & References

1. **[Model Context Protocol Specification]** — Anthropic. https://spec.modelcontextprotocol.io/. The canonical protocol specification defining transports (stdio, Streamable HTTP), lifecycle, primitives (tools, resources, prompts), and capability negotiation.
2. **[MCP TypeScript SDK]** — Model Context Protocol Project. https://github.com/modelcontextprotocol/typescript-sdk. Official TypeScript/Node.js SDK for building MCP servers and clients. Includes `McpServer`, `StdioServerTransport`, `InMemoryTransport`, and the MCP Inspector.
3. **[MCP Python SDK]** — Model Context Protocol Project. https://github.com/modelcontextprotocol/python-sdk. Official Python SDK providing the `FastMCP` high-level API and low-level protocol primitives for building MCP servers.
4. **[npm Documentation]** — npm, Inc. https://docs.npmjs.com/. Official reference for package.json fields, publishing workflows, `npx` execution, scoped packages, and the `prepublishOnly` lifecycle script.
5. **[Python Packaging User Guide]** — Python Packaging Authority (PyPA). https://packaging.python.org/. Authoritative guide for pyproject.toml, build backends (hatchling, setuptools), wheel distribution, and PyPI publishing via twine.
6. **[Smithery — MCP Server Marketplace]** — Smithery. https://smithery.ai. Managed hosting and distribution platform for MCP servers, providing one-command installs, analytics, and the `smithery.yaml` configuration format.
7. **[MCP Inspector]** — Model Context Protocol Project. Bundled with the MCP TypeScript SDK (`npx @modelcontextprotocol/inspector`). Interactive web-based testing tool for verifying MCP server tool registration, JSON-RPC handshake, and transport behavior.
8. **[JSON-RPC 2.0 Specification]** — JSON-RPC Working Group. https://www.jsonrpc.org/specification. The wire protocol underlying all MCP communication. Defines request/response format, error codes, and notification messages used by stdio and HTTP transports.
