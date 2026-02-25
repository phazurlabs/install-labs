---
name: custom-gpt-packaging
description: "Complete guide to packaging AI agents as Custom GPTs for the ChatGPT GPT Store. Use when the user mentions: Custom GPT, GPT Store, GPT Builder, GPT Actions, GPT packaging, ChatGPT plugin, GPT knowledge files, GPT system instructions, build a GPT, create a GPT, publish GPT, GPT marketplace, GPT distribution, OpenAI GPT, GPT configuration, conversational agent, GPT monetization"
---

# Custom GPT Packaging for the GPT Store

Complete reference for building, configuring, publishing, and maintaining Custom GPTs — OpenAI's zero-install distribution format for conversational AI agents.

---

## What Is a Custom GPT?

A Custom GPT is a configured ChatGPT instance that combines four building blocks into a single shareable agent:

| Building Block | What It Does | Limit |
|----------------|-------------|-------|
| **System Instructions** | Defines personality, behavior, process, and boundaries | ~8,000 characters recommended |
| **Knowledge Files** | Documents searched via RAG to ground responses in your data | 20 files, 512 MB each |
| **GPT Actions** | OpenAPI schemas that let the GPT call external APIs | Multiple actions per GPT |
| **Built-in Capabilities** | Code Interpreter, DALL-E image generation, web browsing | Toggle on/off per GPT |

**Key mental model:** A Custom GPT is ChatGPT with a preset personality, private knowledge, and optional API superpowers. Users click "Start Chat" and get a specialized agent instantly — zero installation, zero configuration, zero technical knowledge required.

---

## When to Build a Custom GPT

Build a Custom GPT when:

- **Your audience is non-technical** — They will never run a terminal command or install a package. They open ChatGPT and talk.
- **The tool is conversational** — The primary interface is back-and-forth dialogue, not a dashboard or CLI.
- **Maximum reach matters** — ChatGPT has 200M+ weekly active users. The GPT Store is the largest AI agent marketplace.
- **Zero friction is critical** — No install, no signup (beyond existing ChatGPT account), no onboarding. Click and use.
- **Local execution is not required** — Everything runs in OpenAI's cloud. The user's device is just a chat window.

**Do NOT build a Custom GPT when:**
- You need persistent local state or file system access
- The agent must run offline
- Data privacy requirements prohibit sending data to OpenAI
- You need sub-second latency for tool calls
- The workflow is better served by a structured UI than a conversation

---

## Building Blocks in Detail

### System Instructions

System instructions are the personality and behavior definition loaded before every conversation. They are invisible to the user but control everything the GPT does.

**Recommended structure:**

```
## Identity
You are [Name], a [role] that helps users [primary task].

## Behavior Rules
- Always do X
- Never do Y
- When asked about Z, respond with [specific format]

## Process
When the user provides [input type]:
1. First, do this
2. Then, do this
3. Finally, present results as [format]

## Boundaries
- Do not answer questions about [out-of-scope topic]
- If the user asks for [edge case], respond with [specific handling]

## Tone
[Describe the voice: professional, casual, technical, encouraging, etc.]

## Examples
User: [example input]
Assistant: [example ideal response]
```

**Instruction writing principles:**

1. **Be specific, not vague.** "You are a tax advisor for US freelancers filing Schedule C" beats "You help with taxes."
2. **Define boundaries explicitly.** Without boundaries, the GPT will attempt anything and do it poorly. State what it does NOT do.
3. **Include 2-3 example exchanges.** Examples anchor the GPT's behavior more reliably than abstract rules.
4. **Specify tone in concrete terms.** "Write at an 8th-grade reading level, avoid jargon, use analogies" beats "be friendly."
5. **Stay under 8,000 characters.** Longer instructions degrade adherence. If you need more, move reference material to knowledge files.

### Knowledge Files

Knowledge files are documents the GPT searches via retrieval-augmented generation (RAG) to ground responses in your data.

**Supported formats:** PDF, DOCX, TXT, CSV, JSON, XLSX, PPTX, MD, HTML

**Limits:** 20 files maximum, 512 MB per file.

**Optimization strategies:**

| Strategy | Why It Matters |
|----------|---------------|
| OCR scanned PDFs before uploading | RAG cannot search image-based PDFs — it sees blank pages |
| Structure documents with headers | RAG retrieves chunks; headers help it find the right chunk |
| One topic per file | A focused file retrieves more relevant chunks than a giant omnibus document |
| Use tables for structured data | Tables are parsed more reliably than paragraph-form data |
| Include a summary at the top of each file | The summary helps RAG decide whether this file is relevant to the query |
| Name files descriptively | `us-tax-schedule-c-guide-2025.pdf` not `doc1.pdf` |

**What NOT to upload:**
- Entire textbooks when only 2 chapters are relevant (extract the relevant chapters)
- Scanned documents without OCR processing
- Files with primarily images, charts, or diagrams (RAG cannot interpret visual content)
- Sensitive data you would not want processed by OpenAI's systems

### GPT Actions (The Power Feature)

GPT Actions let your Custom GPT call external APIs during a conversation — fetching real-time data, creating records, triggering workflows, or performing any operation exposed via an HTTP API.

**When to use Actions:**
- The GPT needs real-time data (weather, stock prices, live inventory)
- The GPT needs to write data to an external system (CRM updates, ticket creation)
- The GPT needs to perform calculations on a remote server
- The GPT needs to authenticate as a specific user to access their data

**OpenAPI 3.1 Schema Structure:**

```yaml
openapi: "3.1.0"
info:
  title: "My API"
  version: "1.0.0"
  description: "What this API does"
servers:
  - url: "https://api.example.com/v1"
paths:
  /search:
    get:
      operationId: searchItems
      summary: "Search for items by query"
      description: "Returns matching items. Use this when the user wants to find something."
      parameters:
        - name: q
          in: query
          required: true
          schema:
            type: string
          description: "The search query"
        - name: limit
          in: query
          required: false
          schema:
            type: integer
            default: 10
          description: "Maximum results to return"
      responses:
        "200":
          description: "Successful search"
          content:
            application/json:
              schema:
                type: object
                properties:
                  results:
                    type: array
                    items:
                      type: object
                      properties:
                        id:
                          type: string
                        name:
                          type: string
                        description:
                          type: string
  /items:
    post:
      operationId: createItem
      summary: "Create a new item"
      description: "Creates an item in the system. Use when the user wants to add something."
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - name
              properties:
                name:
                  type: string
                description:
                  type: string
      responses:
        "201":
          description: "Item created"
```

**Critical schema tips:**
- `operationId` must be unique and descriptive — this is how the GPT identifies which action to call
- `description` on each operation tells the GPT WHEN to use this action. Write it as guidance: "Use this when the user wants to..."
- `summary` is the human-readable label shown in the GPT Builder preview
- Every parameter needs a `description` — the GPT uses these to understand what values to pass

**Authentication Options:**

| Type | Use Case | Setup |
|------|----------|-------|
| **None** | Public APIs with no auth | No configuration needed |
| **API Key** | Server-to-server calls with a shared key | Enter key in GPT Builder; specify header name (e.g., `Authorization: Bearer <key>`) |
| **OAuth 2.0** | User-specific data requiring user login | Configure client ID, client secret, auth URL, token URL, scopes |

**API Key** is the simplest. The key is stored by OpenAI and injected into every request. The user never sees it.

**OAuth 2.0** is required when the GPT needs to act on behalf of a specific user (e.g., accessing their Google Calendar, their Salesforce records). The user completes an OAuth flow the first time they use the action.

**Privacy Policy Requirement:** Any GPT with Actions MUST link to a privacy policy URL. This is enforced by the GPT Builder. The policy must explain what data the action collects and how it is used.

**Testing Actions:**
1. Use the Preview panel in GPT Builder to test each action
2. Check the "Debug" section to see the exact request/response
3. Test edge cases: empty inputs, invalid parameters, API errors
4. Verify error messages are helpful ("I couldn't find results for that query" not a raw 404)

---

## Built-in Capabilities

Toggle these on or off in the GPT Builder:

| Capability | What It Does | Enable When |
|------------|-------------|-------------|
| **Code Interpreter** | Runs Python in a sandboxed environment, processes uploaded files | GPT needs to analyze data, generate charts, manipulate files |
| **DALL-E** | Generates images from text prompts | GPT needs to create images, diagrams, or visual content |
| **Web Browsing** | Searches the web and reads pages in real time | GPT needs current information beyond its training data |

These are independent of Actions. A GPT can have Actions AND Code Interpreter AND web browsing simultaneously.

---

## Distribution Tiers

| Tier | Visibility | Review Required | Use Case |
|------|-----------|----------------|----------|
| **Only Me** | Only the creator | No | Development, testing, personal use |
| **Anyone with a Link** | Anyone with the URL | No | Team sharing, beta testing, private distribution |
| **Everyone** | GPT Store listing, searchable | Yes — compliance review | Public distribution, maximum reach |

### Publishing to the GPT Store ("Everyone")

Requirements:
1. GPT must comply with OpenAI's usage policies
2. Creator must have a verified Builder Profile (name + website or social link)
3. GPT must have a name, description, and profile image
4. If Actions are used, a privacy policy URL must be provided
5. Submit for review — typically approved within hours to days

---

## GPT Store Listing Optimization

Your store listing is a landing page. Optimize every element.

### Name (2-4 words)
- Include the use case: "Tax Filing Helper" not "TaxBot Pro AI Ultra"
- Avoid buzzwords: "AI-Powered Smart Assistant" tells the user nothing
- Be findable: use words users would search for

### Description
Structure: one-sentence summary + 3 capability bullets.

```
Helps US freelancers prepare Schedule C tax filings with step-by-step guidance.

- Identifies deductible business expenses from your transaction list
- Calculates quarterly estimated tax payments
- Explains IRS rules in plain English
```

### Conversation Starters (4 prompts)
These are the first things users see. They demonstrate the GPT's best capabilities.

```
"Review my business expenses for deductions"
"Calculate my Q2 estimated tax payment"
"Explain the home office deduction rules"
"What records do I need to keep for an audit?"
```

**Rules for conversation starters:**
- Show range — each starter should demonstrate a different capability
- Be specific — "Help me with taxes" is useless; "Calculate my quarterly estimated payment" shows value
- Use the user's voice — write as the user would speak, not as marketing copy

### Profile Image
- Simple icon or symbol that communicates the domain at a glance
- Works at circle crop (GPT Store uses circular thumbnails)
- Readable at small sizes — no tiny text, no complex scenes
- Consistent with the GPT's tone (professional, playful, technical)

---

## Monetization

OpenAI offers a GPT Builder revenue sharing program:
- Revenue is based on user engagement (conversations) with your GPT by ChatGPT Plus subscribers
- Payouts are proportional to usage relative to other GPTs
- Requires a verified Builder Profile
- Available in supported countries

To maximize revenue: build a GPT that users return to repeatedly, not a one-time novelty. Utility GPTs (tax help, writing assistant, code reviewer) outperform gimmick GPTs.

---

## Updating Your GPT

| Change | Process | Availability |
|--------|---------|-------------|
| Edit instructions | Update in GPT Builder, click Save | Immediate for "Only Me" and link-shared |
| Refresh knowledge files | Upload new versions, remove old ones | Immediate |
| Update Actions schema | Paste new OpenAPI spec, test, save | Immediate |
| Change name/description | Edit in GPT Builder | Store listings may need re-review |

There is no versioning system. Changes are live the moment you save. For "Everyone" (store-listed) GPTs, significant changes to the name or description may trigger a re-review.

**Tip:** Test changes in "Only Me" mode before applying them to a public GPT. Create a duplicate GPT for testing.

---

## Limitations

| Limitation | Impact | Workaround |
|------------|--------|------------|
| Requires ChatGPT Plus ($20/mo) for users | Limits audience to paying subscribers | None — this is a platform constraint |
| No offline mode | Cannot work without internet | Build a local tool instead (CLI, desktop app) |
| No local execution | Cannot access user's file system, run local commands | Use Code Interpreter for sandboxed file processing |
| Data goes to OpenAI | Not suitable for regulated data (HIPAA, classified) | Use a local LLM or private deployment |
| No persistent state | GPT forgets everything between conversations | Use Actions to store/retrieve state in an external database |
| 128K context window | Long conversations or large knowledge retrievals may lose context | Keep knowledge files focused, summarize long conversations |
| No real-time streaming from Actions | Action responses are returned as a block, not streamed | Keep API responses concise |
| Rate limits on Actions | Actions are throttled if called too frequently | Design conversations to batch API calls |

---

## Common Pitfalls and Fixes

### 1. Vague Instructions Producing Inconsistent Behavior

**Problem:** "You are a helpful assistant that knows about marketing" produces wildly different responses each conversation.

**Fix:** Define the exact scope, process, and output format:
```
You are a B2B SaaS email marketing specialist. You help users write cold outreach email sequences of 3-5 emails. For each email, provide: subject line, body (under 150 words), and a CTA. Always ask for the target persona and value proposition before writing.
```

### 2. Scanned PDFs Without OCR

**Problem:** Knowledge file appears uploaded but the GPT says "I don't have information about that" when asked about its contents.

**Fix:** Open the PDF in Adobe Acrobat or use an OCR tool (Tesseract, macOS Preview > Export as PDF with text) before uploading. Test by searching for a specific phrase in the PDF — if Find does not work, the PDF needs OCR.

### 3. No Defined Boundaries

**Problem:** A tax-focused GPT starts giving legal advice, medical recommendations, and relationship counseling because nothing told it to stay in lane.

**Fix:** Add explicit boundary instructions:
```
## Boundaries
- Only answer questions about US federal and state income tax for individuals and sole proprietors
- Do not provide legal advice, investment advice, or advice on any non-tax topic
- If the user asks about something outside your scope, say: "I specialize in tax preparation. For [their topic], I'd recommend consulting a [relevant professional]."
```

### 4. Missing Privacy Policy for Actions

**Problem:** GPT Builder refuses to publish a GPT with Actions because no privacy policy URL is provided.

**Fix:** Create a simple privacy policy page. At minimum it must state: what data your API collects from GPT conversations, how it is stored, how users can request deletion. Host it on your website or use a free privacy policy generator. The URL goes in the GPT Builder under the Action configuration.

### 5. Knowledge Files Too Large or Unfocused

**Problem:** A single 200-page PDF covers 15 topics. RAG retrieves irrelevant chunks, producing confused or incorrect responses.

**Fix:** Split the document into focused files:
- `tax-deductions-home-office.pdf` (12 pages)
- `tax-deductions-vehicle.pdf` (8 pages)
- `tax-deductions-equipment.pdf` (6 pages)
- `estimated-tax-payments.pdf` (10 pages)

Smaller, focused files produce dramatically better retrieval accuracy.

### 6. Actions Schema Missing Descriptions

**Problem:** The GPT has Actions but never calls them, or calls the wrong one.

**Fix:** Add detailed `description` fields to every operation and parameter in the OpenAPI schema. The GPT decides which action to call based on these descriptions. Without them, it guesses — and guesses wrong.

---

## Custom GPT vs Other Distribution Formats

| Factor | Custom GPT | Claude Code Plugin | CLI Tool | Web App |
|--------|-----------|-------------------|----------|---------|
| Install friction | None (click to use) | Copy directory | brew/npm/pip install | Open URL |
| Technical audience required | No | Yes (developers) | Yes | No |
| Offline capable | No | Yes | Yes | No (usually) |
| Local file access | No | Yes | Yes | No |
| Data privacy | OpenAI processes all data | Local | Local | Depends |
| Customization | Instructions + knowledge + actions | Skills + commands + MCP | Full code | Full code |
| Distribution reach | 200M+ ChatGPT users | Claude Code users | Developers | Anyone |
| Monetization | GPT Store revenue share | Self-managed | Self-managed | Self-managed |
| Persistent state | Only via Actions (external DB) | File system | File system | Database |

Choose Custom GPTs for maximum reach with zero friction. Choose Claude Code plugins for developer audiences needing local execution. Choose CLI tools for automation pipelines. Choose web apps for custom interfaces.

---

## Quick Start Checklist

To ship a Custom GPT from zero to published:

1. **Define scope** — Write one sentence: "This GPT helps [who] do [what] by [how]."
2. **Write system instructions** — Identity, behavior rules, process, boundaries, tone, examples. Under 8,000 characters.
3. **Prepare knowledge files** — OCR any scanned PDFs, split large documents, name files descriptively.
4. **Configure capabilities** — Enable Code Interpreter, DALL-E, or web browsing only if the GPT needs them.
5. **Build Actions (if needed)** — Write OpenAPI 3.1 schema, configure auth, add a privacy policy URL.
6. **Set conversation starters** — 4 prompts that showcase the GPT's best capabilities.
7. **Create profile image** — Simple icon, works at circle crop, readable at small sizes.
8. **Test in "Only Me" mode** — Run through 10+ conversations covering happy paths and edge cases.
9. **Share via link** — Give the URL to 3-5 beta testers for feedback.
10. **Publish to GPT Store** — Set to "Everyone," submit for review, verify your Builder Profile.

---

## Sources & References

1. **[Creating a GPT]** — OpenAI. https://help.openai.com/en/articles/8554397-creating-a-gpt. Official guide for using GPT Builder to configure system instructions, knowledge files, capabilities, and Actions for Custom GPTs.
2. **[OpenAPI Specification v3.1.0]** — OpenAPI Initiative. https://spec.openapis.org/oas/v3.1.0. The schema standard used to define GPT Actions. Every Action endpoint, parameter, and response must conform to this specification.
3. **[OpenAI API Documentation]** — OpenAI. https://platform.openai.com/docs. Developer reference for the OpenAI platform, including authentication patterns (API keys, OAuth 2.0), rate limits, and the Assistants API that underlies GPT capabilities.
4. **[GPT Store and Builder Profile Guidelines]** — OpenAI. https://openai.com/policies/. OpenAI's usage policies, brand guidelines, and content moderation rules that all GPT Store submissions must comply with. Includes requirements for privacy policies when Actions are used.
