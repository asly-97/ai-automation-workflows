# What Is Cross-AI Memory? One Shared Memory for ChatGPT, Claude, Cursor, Codex and Every MCP Tool

What is cross-AI memory, and how do ChatGPT, Claude, Claude Code, Cursor and Codex share one memory? This is the full technical guide behind the Agentic Pod episode **"What Is Cross-AI Memory? (And How Do Your AI Tools Share One?)"**.

- 🎧 Watch the episode: [What Is Cross-AI Memory? (And How Do Your AI Tools Share One?)](https://youtu.be/HjHgTpAbRqA) on Agentic Pod
- 📖 Read the article: [What is cross-AI memory?](https://vilix.ai/blog/cross-ai-memory)
- 🧠 Memory layer used here: [Vilix AI](https://vilix.ai)
- ⚙️ Setup for 20+ AI tools: [`shared-memory-for-ai-agents/setup/`](../shared-memory-for-ai-agents/setup/README.md)

---

## Contents

1. [Cross-AI memory in one sentence](#cross-ai-memory-in-one-sentence)
2. [Why your AI tools don't share memory today](#why-your-ai-tools-dont-share-memory-today)
3. [How MCP makes cross-AI memory possible](#how-mcp-makes-cross-ai-memory-possible)
4. [Five pains, and how Vilix AI fixes each one](#five-pains-and-how-vilix-ai-fixes-each-one)
5. [Three surfaces, one memory](#three-surfaces-one-memory)
6. [Setup: connect your tools](#setup-connect-your-tools)
7. [Copy-paste prompts](#copy-paste-prompts)
8. [Who it's for](#who-its-for)
9. [FAQ](#faq)

---

## Cross-AI memory in one sentence

Cross-AI memory is a persistent memory layer shared between several AI tools: instead of each tool keeping its own isolated history, ChatGPT, Claude, Claude Code, Cursor, Codex and other connected clients read from and write to the same store.

```
   ChatGPT   Claude   Claude Code   Cursor   Codex   OpenClaw   Hermes   Copilot ...
      │        │          │           │        │         │         │        │
      └────────┴──────────┴───────────┴───┬────┴─────────┴─────────┴────────┘
                                          │  MCP (Model Context Protocol)
                                 ┌────────▼────────┐
                                 │    Vilix AI     │
                                 │  conversations  │
                                 │  projects/tasks │
                                 │  rules, skills  │
                                 │  agent inbox    │
                                 └─────────────────┘
```

## Why your AI tools don't share memory today

ChatGPT has memory. Claude has Projects. Cursor has its rules file. Each one keeps context inside its own product, so it goes nowhere else:

- You explain your stack to Claude, switch to Cursor, and explain it again.
- A decision you made in ChatGPT is missing from your next Codex session.
- You re-paste the same rules and preferences into every tool's settings.

Cross-AI memory decouples memory from any one tool and makes it shared infrastructure, the same way one database is shared by every service in your app. The memory lives in your Vilix AI account, tied to you, not to a single assistant.

## How MCP makes cross-AI memory possible

The **Model Context Protocol (MCP)** is an open standard that lets AI tools connect to external servers as context providers. Before MCP, adding persistent memory to an AI tool meant a browser extension or a custom integration for every tool. MCP standardizes the interface, so **one server serves many tools**.

A memory server exposes tools the AI calls:

| Tool | When | What it does |
|---|---|---|
| `get_context` | Before every reply / at the start of an agent run | Loads recent messages, related past conversations from every tool, your user rules, and live project and task state |
| `save_turn` | After composing the reply / at the end of a run | Saves the exact exchange with a `source` label (which tool saved it) and a timestamp, then returns a `chat_id` to reuse in the same conversation |

```
 any tool (ChatGPT, Claude, Cursor, Codex, OpenClaw, Hermes ...)
   1. get_context(user_prompt, chat_id?)  ──► Vilix AI returns relevant memory
   2. compose the reply with that context
   3. save_turn(user_message, assistant_message, source, chat_id?)
   4. send the saved reply
```

## Five pains, and how Vilix AI fixes each one

### 1. Re-explaining your stack every time you switch tools

**How Vilix AI fixes it:** every connected tool calls `get_context` before it replies. Vilix AI searches everything you've saved from every tool (semantic and keyword search) and returns what's relevant: the stack, the decisions, yesterday's progress. You open Cursor and say "let's keep going," and it already knows.

### 2. Decisions made in one tool are missing in another

**How Vilix AI fixes it:** `save_turn` stores the full exchange, so the *why* behind a decision survives, labeled with the tool that saved it and a timestamp. When two memories disagree, the **newest wins**: say "we're not doing that anymore" once in ChatGPT on your phone, and Codex sees it tonight.

### 3. The same rules pasted into every tool

**How Vilix AI fixes it:** write rules once.
- **User rules** (`add_user_rule`): personal directives like "short answers" or "always use TypeScript", loaded into every new chat in every connected tool.
- **Project rules** (`create_project(..., project_rules)` / `update_project`): stack, conventions and tone for one project.

### 4. One extension or integration per tool

**How Vilix AI fixes it:** Vilix AI is MCP-native. Add `https://api.vilix.ai/mcp` as a custom MCP connector in each tool and sign in. No browser extension, under two minutes per tool.

### 5. Agents that repeat each other's mistakes

**How Vilix AI fixes it:** agents save failed runs like successful ones (what was tried, the error, the conclusion), so the next agent's `get_context` surfaces it. **Skills** (`create_skill` / `get_skill`) hold reusable procedures any agent can load, and the **agent inbox** (`message_agents`) lets you leave a note from your phone that your agents pick up on their next run.

## Three surfaces, one memory

| Surface | What it holds | Tools |
|---|---|---|
| **Cross-tool memory** | Past conversations, decisions and facts, retrievable when relevant | `get_context`, `search_semantic`, `search_keyword`, `save_turn` |
| **Projects and tasks** | A lightweight project manager whose state auto-injects into context, so every tool sees what's open, what's done and the current goal | `create_project`, `create_task`, `update_task_state` |
| **Rules** | `user_rules` (personal style) and `project_rules` (per-project stack, conventions, tone) | `add_user_rule`, `update_project` |

## Setup: connect your tools

1. Create your account at [vilix.ai/get-started](https://vilix.ai/get-started) (free 7-day Pro trial, no credit card).
2. Add the MCP endpoint in each tool:
   ```
   https://api.vilix.ai/mcp
   ```
   - **ChatGPT:** turn on **Settings → Security and login → Developer mode**, open [chatgpt.com/plugins](https://chatgpt.com/plugins), select **+**, name it **Vilix AI**, paste the URL, choose **OAuth**, then **Create** and sign in.
   - **Claude:** **Customize → Connectors → + → Add custom connector**, name it **Vilix AI**, paste the URL, **Add → Connect**, sign in.
   - **Claude Code:** `claude mcp add --transport http --scope user vilix https://api.vilix.ai/mcp`, then `/mcp` → **vilix** → sign in.
   - **Codex:** `codex mcp add vilix --url https://api.vilix.ai/mcp` then `codex mcp login vilix`.
   - **Cursor:** add `"vilix": { "url": "https://api.vilix.ai/mcp" }` under `mcpServers` in `~/.cursor/mcp.json`, restart, then **Customize → MCPs → Vilix AI** and sign in.
   - **OpenClaw / Hermes and other always-on agents:** create a key in Vilix AI under **Agents → Create API key** and send `Authorization: Bearer YOUR_VILIX_API_KEY`.
   - Full per-tool steps: [`shared-memory-for-ai-agents/setup/`](../shared-memory-for-ai-agents/setup/README.md).
3. Make it automatic: ask any connected tool *"Set up Vilix AI to load and save context automatically."* It calls `get_setup_instructions` and gives you the exact custom-instructions block and where it goes.
4. Verify the loop in a real conversation: the tool should call `get_context` before replying and `save_turn` before sending.

## Copy-paste prompts

**Save your first shared memory (in tool A):**
```
Remember in Vilix AI: Northstar is my invoicing SaaS. Next.js + Stripe. The pricing page ships first.
```

**Check it crossed over (in tool B):**
```
What am I building and what ships first?
```

**Set a rule once for every tool:**
```
Add a Vilix AI user rule: always answer with short, concise responses.
```

**Create a project with rules:**
```
Create a Vilix AI project "Northstar" with project rules: Next.js App Router, TypeScript strict, Stripe for billing, no new dependencies without asking.
```

**Agent run wrapper (for scheduled agents):**
```
At the start of this run, call get_context with the task. At the end, call save_turn with what you did, what failed and why, and the next step. Use source "<agent name>".
```

## Who it's for

- **AI agencies and automation builders:** give each client their own Vilix AI account (like a Gmail account that belongs to the client), included in your retainer or paid by the client. Every agent and automation you build for that client connects with that account's API key, so the memory belongs to the client's business and you never build memory from scratch.
- **People who sell standalone agents** (support, sales, booking): plug in Vilix AI as the agent's memory, one account per customer.
- **Solo founders:** plan in ChatGPT on your phone, build in Claude Code or Cursor at night, same memory.

## FAQ

**What is cross-AI memory?**
A persistent memory layer shared between several AI tools. One store holds your context, decisions and preferences, and connected clients read from and write to it through MCP.

**How does Vilix AI implement cross-AI memory?**
Vilix AI connects as a custom MCP server at `api.vilix.ai/mcp`. The AI calls `get_context`, composes its reply, calls `save_turn` with the exact exchange, then sends the saved reply. Memory lives server-side in your Vilix AI account.

**Which AI tools work with it?**
ChatGPT, Claude, Claude Code, Codex, Cursor, OpenClaw, Hermes, Grok, Manus, GitHub Copilot, Windsurf, Lovable and other MCP-compatible tools.

**Is it different from ChatGPT memory or Claude Projects?**
Yes. Tool-specific memory stays inside its own product. Cross-AI memory makes saved information available to every connected tool.

**Do I need a browser extension?**
No. Vilix AI is MCP-native: add `api.vilix.ai/mcp` as a custom connector in each tool.

---

Built with [Vilix AI](https://vilix.ai), the shared memory layer for every AI tool. Questions or a workflow you want covered? Open an issue.
