---
name: npm-agent-packaging
description: "Packaging Node.js and TypeScript AI agents for distribution via npm. Use when the user mentions: npm, npx, node agent, typescript agent, npm publish, package.json, bin field, npm package, agent distribution, node distribution, vercel ai sdk, npm install, npm registry, npx agent, global install, scoped package, npm provenance, tsconfig, esbuild, tsup, bundling agent, npm ci, prepublishOnly, monorepo agent, npm workspace, agent cli"
---

# npm Agent Packaging

## When to Use npm Distribution

npm is the right distribution channel when:

- Your agent is built with Node.js, TypeScript, or JavaScript.
- You are packaging an MCP server (the primary consumer of npx-based distribution).
- Your agent uses the Vercel AI SDK, LangChain.js, or any Node-based AI framework.
- You want users to run your agent with a single `npx` command, zero prior setup.
- You need cross-platform distribution (macOS, Windows, Linux) without separate binaries.
- Your target audience is developers who already have Node.js installed.

npm gives you: a global registry, dependency resolution, semantic versioning, provenance attestation, and the `npx` zero-install runner. No other JavaScript distribution method matches this combination.

---

## package.json Anatomy for Agent Distribution

Every field matters. Here is the annotated structure for a distributable AI agent.

```json
{
  "name": "@your-org/agent-name",
  "version": "1.0.0",
  "description": "One-line description of what this agent does",
  "type": "module",
  "bin": {
    "agent-name": "./dist/cli.js"
  },
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "files": [
    "dist/"
  ],
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "start": "node dist/cli.js",
    "inspect": "npx @modelcontextprotocol/inspector node dist/cli.js",
    "lint": "eslint src/",
    "test": "vitest run",
    "prepublishOnly": "npm run build && npm run test"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.12.0",
    "zod": "^3.23.0"
  },
  "devDependencies": {
    "@types/node": "^22.0.0",
    "typescript": "^5.7.0",
    "vitest": "^3.0.0"
  },
  "engines": {
    "node": ">=18.0.0"
  },
  "keywords": [
    "ai-agent",
    "mcp",
    "mcp-server",
    "cli",
    "automation"
  ],
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/your-org/agent-name"
  }
}
```

### Field-by-Field Breakdown

**name** -- Use a scoped name (`@your-org/agent-name`). Scoped packages avoid name collisions on the npm registry and signal organizational ownership. Unscoped names are fine for personal projects but become a liability at scale.

**version** -- Follow semantic versioning strictly. For AI agents, consider: MAJOR = breaking changes to tool schemas or CLI interface, MINOR = new tools or capabilities, PATCH = bug fixes and prompt improvements.

**description** -- One sentence, no jargon. This appears in `npm search` results and registry listings. Front-load the action: "Searches GitHub issues via MCP" beats "An MCP server for GitHub."

**type: "module"** -- Declare ESM. Modern Node.js, the MCP SDK, and most AI libraries use ES modules. If you omit this, Node.js defaults to CommonJS, and you will fight import/require mismatches.

**bin** -- This is what makes your package executable. When a user runs `npx -y @your-org/agent-name`, npm looks up the `bin` field, downloads the package, and runs the specified file. The key in the bin object becomes the command name. The value must point to a compiled `.js` file in your `dist/` directory, never to a `.ts` source file.

**main** -- Entry point for programmatic `import`. Allows other packages to import your agent's functionality without using the CLI.

**types** -- TypeScript declaration file. Enables type checking and IDE autocompletion for consumers who import your package.

**files** -- Whitelist of files to include in the published package. Only `dist/` should ship. This excludes `src/`, `tests/`, `.github/`, `node_modules/`, and everything else. Always verify with `npm pack --dry-run`.

**engines** -- Declare the minimum Node.js version. The MCP SDK requires Node 18+. If you use `fetch` without a polyfill, you need Node 18+. If you use `node:` protocol imports, you need Node 16+. State this explicitly so npm warns users on incompatible versions.

**keywords** -- Used by the npm registry search and by MCP registries for auto-discovery. Always include `mcp` and `mcp-server` if your agent is an MCP server.

**scripts.inspect** -- MCP-specific. Launches the MCP Inspector for interactive testing. Not relevant for non-MCP agents but essential for MCP servers.

**scripts.prepublishOnly** -- Runs automatically before `npm publish`. Build and test in this hook so you never publish stale or broken code.

---

## TypeScript Compilation

### tsconfig.json for Agent Distribution

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

**Key settings explained:**

- `target: "ES2022"` -- Enables top-level await, which MCP servers and modern agents rely on.
- `module: "Node16"` / `moduleResolution: "Node16"` -- Correct ESM resolution for Node.js. Do not use `"bundler"` unless you are actually bundling.
- `declaration: true` -- Generates `.d.ts` files so consumers get type information.
- `outDir: "./dist"` -- Compiled output goes to `dist/`, matching the `files` field in package.json.
- `rootDir: "./src"` -- Preserves directory structure in output. `src/index.ts` becomes `dist/index.ts`.

### Shebang Preservation

TypeScript strips shebangs during compilation. You need the shebang in the compiled output for `npx` to work. Three approaches:

**Option A: Banner plugin (recommended for tsc users)**

Add a postbuild script that prepends the shebang:

```json
{
  "scripts": {
    "build": "tsc && node -e \"const f='dist/cli.js';const c=require('fs').readFileSync(f,'utf8');if(!c.startsWith('#!'))require('fs').writeFileSync(f,'#!/usr/bin/env node\\n'+c)\""
  }
}
```

**Option B: Use tsup or esbuild (handles it natively)**

```typescript
// tsup.config.ts
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/cli.ts"],
  format: ["esm"],
  banner: { js: "#!/usr/bin/env node" },
  outDir: "dist",
  clean: true,
});
```

**Option C: Keep the shebang as a comment that survives compilation**

Place `#!/usr/bin/env node` as the first line of your `.ts` file. TypeScript preserves it if it is literally the first line (before any imports). Verify this works with your TypeScript version.

---

## The Shebang Requirement

```
#!/usr/bin/env node
```

This line must be the very first line of your CLI entry point (`dist/cli.js`). It tells the operating system to execute the file using Node.js.

Without it:
- `npx @your-org/agent` fails on macOS/Linux with "Permission denied" or attempts to run the file as a shell script.
- Windows is more forgiving (npm creates a `.cmd` wrapper), but the shebang is still best practice.

After building, always verify:

```bash
head -1 dist/cli.js
# Should output: #!/usr/bin/env node
```

---

## npx Zero-Install Pattern

The gold standard for agent distribution. The user runs one command and your agent executes immediately.

```bash
npx -y @your-org/agent-name
```

How it works:
1. npm checks if `@your-org/agent-name` is installed locally or globally.
2. If not, it downloads the latest version to a temporary cache.
3. It looks up the `bin` field in package.json.
4. It executes the specified file.
5. The package remains cached for future invocations (cache TTL varies by npm version).

The `-y` flag auto-confirms the download prompt, enabling non-interactive usage. This is critical for MCP server configs, where Claude Desktop spawns the process without user interaction.

### Passing Arguments

```bash
npx -y @your-org/agent-name --port 3000 --verbose
```

Everything after the package name is passed as command-line arguments to your entry point. Parse them with a library like `commander`, `yargs`, or Node's built-in `util.parseArgs`.

### Pinning Versions

```bash
npx -y @your-org/agent-name@2.1.0
```

Users can pin to a specific version. For MCP server configs, this prevents surprise breaking changes:

```json
{
  "command": "npx",
  "args": ["-y", "@your-org/agent-name@2.1.0"]
}
```

---

## npm install -g Pattern

For agents that users run frequently, a global install avoids re-downloading on every invocation.

```bash
npm install -g @your-org/agent-name
agent-name --help
```

The command name comes from the `bin` field key in package.json. After global install, it is available system-wide.

Trade-offs vs. npx:
- Faster startup (no download check).
- User must manually update (`npm update -g @your-org/agent-name`).
- Potential version conflicts if multiple projects need different versions.
- Requires explicit install step (worse first-run UX).

Recommendation: Document both patterns. Lead with npx for quick starts, mention global install for power users.

---

## Monorepo Agents

When you maintain multiple agents in a single repository, use npm workspaces.

```
my-agents/
  package.json          # Root with workspaces config
  packages/
    agent-search/
      package.json      # @your-org/agent-search
      src/
    agent-deploy/
      package.json      # @your-org/agent-deploy
      src/
    shared/
      package.json      # @your-org/agent-shared (internal)
      src/
```

Root package.json:

```json
{
  "name": "my-agents",
  "private": true,
  "workspaces": [
    "packages/*"
  ],
  "scripts": {
    "build": "npm run build --workspaces",
    "publish-all": "npm publish --workspaces --access public"
  }
}
```

Each agent package has its own `package.json`, `bin`, `files`, and `version`. They can depend on each other using workspace protocol (`"@your-org/agent-shared": "workspace:*"`) during development. npm resolves these to real versions at publish time.

---

## Environment Variable Handling

### Reading Environment Variables

```typescript
// Good: check at startup, fail with clear message
const apiKey = process.env.API_KEY;
if (!apiKey) {
  console.error("Error: API_KEY environment variable is required.");
  console.error("");
  console.error("Set it in your MCP config:");
  console.error('  "env": { "API_KEY": "your-key-here" }');
  console.error("");
  console.error("Or export it in your shell:");
  console.error("  export API_KEY=your-key-here");
  process.exit(1);
}
```

### Using dotenv (for Development)

```typescript
import "dotenv/config"; // Loads .env file if present

const apiKey = process.env.API_KEY;
```

Include `dotenv` as a regular dependency (not devDependency) if your agent supports `.env` files. But document that MCP server configs should use the `env` field instead.

### Never Hardcode Credentials

```typescript
// NEVER do this
const API_KEY = "sk-abc123secretkey";

// ALWAYS do this
const API_KEY = process.env.API_KEY;
```

Hardcoded keys end up in the npm registry, in Git history, and in your users' node_modules. There is no way to fully retract a published npm package.

---

## Bundling

### When to Bundle

Bundle your agent into a single file when:
- You want to minimize install time (no dependency resolution).
- You have many small internal dependencies.
- You are distributing a single-purpose CLI tool.
- You want to reduce the attack surface (fewer dependencies = fewer supply chain risks).

Do NOT bundle when:
- Your dependencies include native modules (they cannot be bundled).
- You want consumers to `import` individual modules from your package.
- Bundle size would exceed 10MB (npm has a 50MB limit, but large packages are hostile to users).

### esbuild (Fastest)

```json
{
  "scripts": {
    "build": "esbuild src/cli.ts --bundle --platform=node --target=node18 --outfile=dist/cli.js --format=esm --banner:js='#!/usr/bin/env node'"
  }
}
```

### tsup (esbuild Wrapper with Better Defaults)

```typescript
// tsup.config.ts
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/cli.ts"],
  format: ["esm"],
  target: "node18",
  platform: "node",
  banner: { js: "#!/usr/bin/env node" },
  outDir: "dist",
  clean: true,
  // Mark native modules as external
  external: ["better-sqlite3"],
});
```

### Verifying Bundle

```bash
# Check file size
ls -lh dist/cli.js

# Verify it runs standalone
node dist/cli.js --help

# Verify shebang
head -1 dist/cli.js
```

---

## Publishing

### First-Time Setup

```bash
# Create an npm account (if you don't have one)
npm adduser

# Login
npm login

# Create your organization (for scoped packages)
# Do this at https://www.npmjs.com/org/create
```

### Publishing Workflow

```bash
# 1. Verify package contents
npm pack --dry-run

# 2. Check for sensitive files
# Look for .env, credentials, secrets in the file list

# 3. Bump version
npm version patch   # 1.0.0 -> 1.0.1
npm version minor   # 1.0.0 -> 1.1.0
npm version major   # 1.0.0 -> 2.0.0

# 4. Publish (prepublishOnly runs build + test automatically)
npm publish --access public

# 5. Verify
npx -y @your-org/agent-name@latest --help
```

### Scoped Packages

Scoped packages (`@org/name`) require `--access public` on first publish. After that, subsequent publishes default to the access level of the previous version.

### npm Provenance

Provenance attestation creates a verifiable link between your published package and the source code that produced it. Enable it in CI:

```bash
npm publish --provenance --access public
```

Requirements:
- Must run in a supported CI environment (GitHub Actions, GitLab CI).
- Repository must be public.
- Generates a Sigstore signature linking the package to a specific commit and workflow.

Users see a "Provenance" badge on npmjs.com, proving the package was built from the linked source.

---

## CI/CD: GitHub Actions Workflow

```yaml
name: Publish to npm

on:
  push:
    tags:
      - "v*"

permissions:
  contents: read
  id-token: write  # Required for provenance

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "22"
          registry-url: "https://registry.npmjs.org"

      - run: npm ci
      - run: npm run build
      - run: npm test

      - run: npm publish --provenance --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

**Workflow:**
1. Create a Git tag: `git tag v1.2.0`
2. Push the tag: `git push origin v1.2.0`
3. GitHub Actions builds, tests, and publishes automatically.
4. npm provenance links the package to this exact commit and workflow run.

---

## README Pattern for Agents

Your README is your landing page. Structure it for scanning.

```markdown
# @your-org/agent-name

One-sentence description of what this agent does.

## Quick Start

\`\`\`bash
npx -y @your-org/agent-name
\`\`\`

## Configuration

### For Claude Desktop

Add to your `claude_desktop_config.json`:

\`\`\`json
{
  "mcpServers": {
    "agent-name": {
      "command": "npx",
      "args": ["-y", "@your-org/agent-name"],
      "env": {
        "API_KEY": "your-api-key"
      }
    }
  }
}
\`\`\`

### For Claude Code

\`\`\`bash
claude mcp add agent-name -- npx -y @your-org/agent-name
\`\`\`

### Environment Variables

| Variable  | Required | Description          |
|-----------|----------|----------------------|
| API_KEY   | Yes      | Your API key         |

## Tools

| Tool           | Description                     |
|----------------|---------------------------------|
| search_items   | Search items by query           |
| create_item    | Create a new item               |

## License

MIT
```

Install command goes first. Before description, before badges, before the table of contents. The user's first question is "how do I use this?" Answer it in the first 5 lines.

---

## Common Pitfalls

### Missing bin Field
**Symptom:** `npx @your-org/agent` downloads the package but does nothing, or throws "command not found."
**Fix:** Add a `bin` field to package.json mapping a command name to your compiled entry point.

### Missing Shebang
**Symptom:** "Permission denied" or "exec format error" on macOS/Linux.
**Fix:** Ensure `#!/usr/bin/env node` is the first line of the file referenced by `bin`. Verify after TypeScript compilation.

### Publishing src/ Instead of dist/
**Symptom:** Package size is 5x larger than expected. Users see TypeScript files in `node_modules`. Potential compilation errors on users' machines.
**Fix:** Set `"files": ["dist/"]` in package.json. Run `npm pack --dry-run` and inspect the file list before every publish.

### Not Setting engines
**Symptom:** Your agent crashes on Node 14 with a cryptic syntax error because you used top-level await or optional chaining.
**Fix:** Add `"engines": { "node": ">=18.0.0" }` to package.json. npm warns users if their Node version is incompatible.

### Importing Heavy Dependencies at Top Level
**Symptom:** `npx @your-org/agent --help` takes 8 seconds because it loads a 50MB ML model on import.
**Fix:** Use dynamic `import()` for heavy dependencies. Load them only when the specific tool or command that needs them is invoked.

```typescript
// Bad: loads on every invocation
import { HeavyModel } from "heavy-ml-lib";

// Good: loads only when needed
async function runAnalysis() {
  const { HeavyModel } = await import("heavy-ml-lib");
  // ...
}
```

### Forgetting .npmignore or files Field
**Symptom:** Tests, GitHub workflows, `.env.example`, and other dev files ship to users.
**Fix:** Use the `files` whitelist in package.json (preferred over `.npmignore`). The `files` field is an allowlist; only listed paths are included.

### Breaking npx Cache Assumptions
**Symptom:** Users report they are running an old version even after you published an update.
**Fix:** npx caches packages. Users can force a fresh download with `npx -y @your-org/agent@latest`. Document this in your troubleshooting section.

### No prepublishOnly Script
**Symptom:** You publish a package with stale `dist/` from a previous build that does not match the current source.
**Fix:** Add `"prepublishOnly": "npm run build && npm run test"` to scripts. This runs automatically before every `npm publish`, ensuring the build is fresh and tests pass.

### ESM/CJS Mismatch
**Symptom:** "Cannot use import statement outside a module" or "require is not defined in ES module scope."
**Fix:** Set `"type": "module"` in package.json for ESM. Ensure tsconfig uses `"module": "Node16"`. If you must support both ESM and CJS, use tsup's dual format output (`format: ["esm", "cjs"]`) and configure package.json `exports` field.

### Publishing Without Testing npx
**Symptom:** Package installs fine via `npm install` but fails via `npx` because the bin entry point has an error.
**Fix:** After every publish, run `npx -y @your-org/agent@latest` in a clean environment. Consider adding this as a post-publish CI step.

---

## Sources & References

1. **[npm Documentation]** — npm, Inc. https://docs.npmjs.com/ Comprehensive reference for the npm CLI, registry, and package management ecosystem.
2. **[npm package.json Reference]** — npm, Inc. https://docs.npmjs.com/cli/configuring-npm/package-json Complete specification for all package.json fields including bin, files, engines, scripts, and exports.
3. **[npx Documentation]** — npm, Inc. https://docs.npmjs.com/cli/commands/npx Reference for the npx package runner, covering zero-install execution, version pinning, and cache behavior.
4. **[npm Provenance Statements]** — npm, Inc. https://docs.npmjs.com/generating-provenance-statements Guide to generating Sigstore-based provenance attestations that link published packages to their source repository and CI workflow.
5. **[Node.js ECMAScript Modules Documentation]** — Node.js Foundation. https://nodejs.org/api/esm.html Official documentation for ES module support in Node.js, including the `"type": "module"` field and module resolution algorithm.
6. **[TypeScript Documentation]** — Microsoft. https://www.typescriptlang.org/docs/ Reference for TypeScript configuration (tsconfig.json), compilation, and declaration file generation used in agent distribution.
7. **[esbuild]** — Evan Wallace. https://esbuild.github.io/ Extremely fast JavaScript bundler used for creating single-file agent distributions with shebang banner support.
8. **[tsup]** — EGOIST. https://tsup.egoist.dev/ Zero-config TypeScript bundler built on esbuild, providing format output (ESM/CJS), declaration generation, and CLI banner injection.
9. **[Sigstore]** — Linux Foundation. https://www.sigstore.dev/ Open-source project for signing, verifying, and protecting software supply chains, underpinning npm provenance attestation.
