# How to Set Up an MCP Memory Server for ChatGPT, Claude, Codex and Cursor

How do you give ChatGPT, Claude, Claude Code, Codex, Cursor, OpenClaw and Hermes one shared memory? You connect every tool to one MCP memory server, pick the right sign-in path for each tool, make the memory loop automatic, and then prove it works with a quick cross-tool test. This is the full technical guide behind the Agentic Pod episode **"How Do You Set Up an MCP Memory Server for ChatGPT, Claude and Cursor?"**.

- 🎧 Watch the episode: [MCP memory server setup on Agentic Pod](https://youtu.be/-CMI-csqde8)
- 📖 Read the article: [How to set up an MCP memory server for ChatGPT, Claude & Cursor](https://vilix.ai/blog/mcp-memory-server-setup-guide)
- 🧠 Memory server used here: [Vilix AI](https://vilix.ai) (hosted, Streamable HTTP: `https://api.vilix.ai/mcp`)
- ⚡ Per-tool setup with screenshots: [vilix.ai/get-started](https://vilix.ai/get-started)
- ⚙️ Setup for 20+ tools: [`../shared-memory-for-ai-agents/setup/`](../shared-memory-for-ai-agents/setup/README.md)

---

## Contents

1. [What an MCP memory server does](#what-an-mcp-memory-server-does)
2. [Five pains, and how Vilix AI fixes each one](#five-pains-and-how-vilix-ai-fixes-each-one)
3. [Step 1: pick the sign-in path per tool](#step-1-pick-the-sign-in-path-per-tool)
4. [Step 2: connect each tool](#step-2-connect-each-tool)
5. [Step 3: make memory automatic](#step-3-make-memory-automatic)
6. [Step 4: run the Northstar test](#step-4-run-the-northstar-test)
7. [Copy-paste prompts](#copy-paste-prompts)
8. [Troubleshooting](#troubleshooting)
9. [FAQ](#faq)

---

## What an MCP memory server does

MCP (Model Context Protocol) is the standard way AI tools call outside tools. A memory server is one of those outside tools. It gives every connected AI a few moves: save an exchange, pull back the relevant records, and search what was saved before.

```
   ChatGPT   Claude   Claude Code   Codex   Cursor   OpenClaw   Hermes ...
      │        │          │           │        │         │         │
      └────────┴──────────┴─────┬─────┴────────┴─────────┴─────────┘
                                │  MCP (Streamable HTTP)
                       ┌────────▼────────┐
                       │    Vilix AI     │   https://api.vilix.ai/mcp
                       │  one memory:    │
                       │  conversations, │
                       │  projects,      │
                       │  rules, skills, │
                       │  agent inbox    │
                       └─────────────────┘
```

The core loop in Vilix AI is two calls:

```
get_context(user_prompt)  →  answer  →  save_turn(user_message, assistant_message, source)
```

- `get_context` runs at the start and returns the relevant decisions, rules, open tasks and recent turns from **every** connected tool.
- `save_turn` runs at the end and stores the full exchange with a **source label** (ChatGPT, Claude, Cursor, Codex...) and a timestamp, so any other tool can see where a decision came from.

Vilix AI also exposes search (`search_semantic`, `search_keyword`, `recent_messages`), projects and tasks, user rules and project rules, reusable skills, and the agent inbox (`message_agents`). MCP `tools/list` shows the full current inventory.

## Five pains, and how Vilix AI fixes each one

### 1. Every tool keeps its own private memory

ChatGPT, Claude and Cursor each remember (at most) their own chats, so you end up pasting the same project brief into every tool.

**How Vilix AI fixes it:** every tool connects to the same Vilix AI account. `get_context` pulls relevant memory saved by any tool, and `save_turn` writes each exchange back with its source label. Monday's decision in Claude is in Cursor on Tuesday.

### 2. OAuth, API keys, headers: which one?

**How Vilix AI fixes it:** two clear paths into the same account. Tools with a browser sign-in (ChatGPT, Claude, Claude Code, Codex, Cursor) use OAuth. Headless agents on a server (OpenClaw, Hermes, Gemini CLI, any custom agent) use a personal API key sent as `Authorization: Bearer YOUR_VILIX_API_KEY`. Both land in one Vilix AI memory.

### 3. A green "connected" icon is not the finish line

A connected tool still needs to actually call the memory tools.

**How Vilix AI fixes it:** ask the tool to set itself up. Vilix AI's `get_setup_instructions` returns the exact auto-memory block for that tool and where to paste it (custom instructions, `CLAUDE.md`, `AGENTS.md`, Cursor User Rules). From then on the tool calls `get_context` before it answers and `save_turn` after, reusing one `chat_id` per conversation.

### 4. Old facts and corrections

**How Vilix AI fixes it:** Vilix AI keeps the full conversation history with timestamps, and the newest memory wins. Correct something once, in any tool, and every other tool picks up the correction. Failed attempts are saved too, so no agent repeats a dead end.

### 5. Many clients, many agents

**How Vilix AI fixes it:** agencies give each client their own Vilix AI account, like a Gmail account that belongs to that client, included in the retainer or paid by the client. Every agent and automation built for that client connects with that client's API key, so the memory belongs to the client's business and the agency never builds memory from scratch. Inside one account, projects with project rules keep each stack separate, skills store reusable procedures, and the agent inbox lets you send a task from ChatGPT on your phone to a coding agent that picks it up on its next run.

## Step 1: pick the sign-in path per tool

| Tool | Path | Where |
|---|---|---|
| ChatGPT (Plus, Pro, Business, Enterprise, Edu) | OAuth | Settings → Security and login → Developer mode, then Plugins → + |
| Claude (web, desktop) | OAuth | Customize → Connectors → Add custom connector |
| Claude Code | OAuth | `claude mcp add` + `/mcp` |
| Codex (CLI, app, IDE) | OAuth | `codex mcp add` + `codex mcp login` |
| Cursor | OAuth | `~/.cursor/mcp.json` + Customize → MCPs |
| OpenClaw | API key | `openclaw mcp add ... --header` |
| Hermes | API key | `hermes mcp add ... --auth header` |

Use the **same Vilix AI account** everywhere. Create API keys in Vilix AI under **Agents → Create API key** and treat them like passwords: keep them out of chats, screenshots and committed config.

## Step 2: connect each tool

**ChatGPT**

1. Settings → Security and login → turn on **Developer mode**.
2. Open Plugins and select **+** to create an app. Name it **Vilix AI**, paste `https://api.vilix.ai/mcp`, choose **OAuth**, then **Create**, sign in and approve.
3. In a chat: **+ → Developer mode → Vilix AI**.

**Claude (web and desktop)**

1. **Customize → Connectors → + → Add custom connector**.
2. Name **Vilix AI**, URL `https://api.vilix.ai/mcp`, **Add → Connect**, sign in and approve.
3. In a chat, enable **Vilix AI** under **+ → Connectors**.

**Claude Code**

```bash
claude mcp add --transport http --scope user vilix https://api.vilix.ai/mcp
```

Start `claude`, run `/mcp`, select **vilix** and complete the browser sign-in.

**Codex**

```bash
codex mcp add vilix --url https://api.vilix.ai/mcp
codex mcp login vilix
```

Type `/mcp` inside Codex to check it. Codex app: **Settings → MCP servers → Add server** (Streamable HTTP).

**Cursor**

```json
{
  "mcpServers": {
    "vilix": { "url": "https://api.vilix.ai/mcp" }
  }
}
```

Merge into `~/.cursor/mcp.json`, restart Cursor, enable **Vilix AI** under **Customize → MCPs**, sign in, and chat in **Agent mode**.

**OpenClaw**

```bash
openclaw mcp add vilix \
  --url https://api.vilix.ai/mcp \
  --transport streamable-http \
  --header "Authorization=Bearer YOUR_VILIX_API_KEY"
openclaw mcp doctor vilix --probe
```

**Hermes**

```yaml
# ~/.hermes/config.yaml
mcp_servers:
  vilix:
    url: https://api.vilix.ai/mcp
    headers:
      Authorization: Bearer YOUR_VILIX_API_KEY
```

Then `hermes mcp test vilix` and start a new agent session.

More tools (GitHub Copilot, Windsurf, Grok, Manus, Gemini Spark, Gemini CLI, Devin, Lovable, Cline, Continue, Muse): [`../shared-memory-for-ai-agents/setup/`](../shared-memory-for-ai-agents/setup/README.md).

## Step 3: make memory automatic

In each connected tool, ask:

```
Set up Vilix AI to load and save context automatically.
```

The tool calls `get_setup_instructions` and gives you the exact block for that tool and where it goes. The pattern it installs:

```
get_context  →  answer  →  save_turn  →  return the answer
```

For scheduled agents and headless runs, start every run with `get_context` and end it with `save_turn` (see [`../shared-memory-for-ai-agents/prompts/agent-run-template.md`](../shared-memory-for-ai-agents/prompts/agent-run-template.md)).

## Step 4: run the Northstar test

Use a fictional project with no private information.

1. **Tool A** (say Claude): `Save to Vilix AI: Northstar is an invoicing app. The pricing page comes first. The launch date is 20 October.` Check that `save_turn` ran.
2. **Tool B** (say Cursor), new chat: `Retrieve Northstar from Vilix AI. What are the priority and launch date?`
3. **Tool A** again: `Northstar's launch moved to 27 October.`
4. **Tool B** again: same question. It should now answer 27 October (newest memory wins) and can tell you which tool saved it.
5. Ask Tool B for something you never saved, like the customer's phone number. A good answer separates saved facts, the correction and missing information.

## Copy-paste prompts

**Start of a work session (any tool):**

```
Load my Vilix AI context for this project: open tasks, project rules, and the last decisions from any tool. Then summarize where we left off in 5 bullets.
```

**Save a decision:**

```
Save this to Vilix AI as a decision for project <name>: <decision>. Include why we chose it and what we rejected.
```

**Hand off to another agent through the inbox:**

```
Send a Vilix AI agent inbox message to my coding agent: <task>. Include the acceptance criteria and the files involved.
```

**Save a reusable procedure as a skill:**

```
Save this as a Vilix AI skill named "<name>": <steps>. Any agent should load it before doing this task.
```

## Troubleshooting

| Symptom | Fix |
|---|---|
| Tool B can't see what Tool A saved | Check both tools are signed in to the same Vilix AI account, and that `save_turn` ran in Tool A |
| 401 Unauthorized | Reconnect OAuth, or create a fresh API key under Agents → Create API key |
| Tool doesn't list `get_context` / `save_turn` | Enable the Vilix AI tools in that chat (ChatGPT Developer mode, Claude connectors menu, Cursor Agent mode) |
| Tool answers without loading memory | Run Step 3 again so the auto-memory instructions are in place |

Exact, current instructions per client: [vilix.ai/get-started](https://vilix.ai/get-started). Transport and tool reference: [vilix.ai/docs/mcp](https://vilix.ai/docs/mcp).

## FAQ

**What is an MCP memory server?**
An MCP server that gives AI tools memory moves: save exchanges, retrieve relevant context, and search past work. Vilix AI is a hosted one at `https://api.vilix.ai/mcp`.

**Can ChatGPT, Claude and Cursor share one memory?**
Yes. Connect all three to the same Vilix AI account. Each one calls `get_context` at the start and `save_turn` at the end, so they read and write the same memory.

**OAuth or API key?**
OAuth for tools with a browser sign-in (ChatGPT, Claude, Claude Code, Codex, Cursor). A personal API key as a Bearer header for headless agents (OpenClaw, Hermes, server-side agents). Both use the same account.

**How do I know it's actually saving?**
Run the Northstar test above: save in one tool, recall in another, correct it, and ask again.

**I run an agency. How should I set this up?**
One Vilix AI account per client. Every agent you build for that client connects with that client's API key, so the memory belongs to the client.

**Does it work with OpenClaw and Hermes?**
Yes, with a Vilix AI API key sent as `Authorization: Bearer YOUR_VILIX_API_KEY`.

---

Start here: [vilix.ai/get-started](https://vilix.ai/get-started) · [Vilix AI](https://vilix.ai) · [Article](https://vilix.ai/blog/mcp-memory-server-setup-guide)
