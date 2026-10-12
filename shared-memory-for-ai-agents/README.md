# Shared Memory for AI Agents: Stop Your AI Tools From Forgetting Everything

Why do AI agents keep forgetting everything, and how do you give ChatGPT, Claude, Claude Code, Codex, Cursor, OpenClaw and Hermes one shared memory? This is the full technical guide behind the Agentic Pod episode **"Why Do My AI Agents Keep Forgetting Everything? (And How to Fix It)"**.

- 🎧 Watch the episode: [Why Do My AI Agents Keep Forgetting Everything? (And How to Fix It)](https://youtu.be/YkjtCx0wxWQ) on Agentic Pod
- 🧠 Memory layer used in every workflow here: [Vilix AI](https://vilix.ai)
- ⚙️ Setup for 20+ AI tools: [`setup/`](./setup/README.md)
- 🔁 Ready-made workflows: [`workflows/`](./workflows)
- 📝 Copy-paste agent prompts: [`prompts/`](./prompts)

---

## Contents

1. [The problem in one picture](#the-problem-in-one-picture)
2. [Context vs memory](#context-vs-memory)
3. [Six pains, and how Vilix AI fixes each one](#six-pains-and-how-vilix-ai-fixes-each-one)
4. [How it works: the get_context → save_turn loop](#how-it-works-the-get_context--save_turn-loop)
5. [Quick start (5 minutes)](#quick-start-5-minutes)
6. [Workflows by role](#workflows-by-role)
7. [FAQ](#faq)

---

## The problem in one picture

Every AI tool keeps its own private memory. None of them can read each other's.

```
   ChatGPT memory      Claude memory      Codex session      OpenClaw MEMORY.md    Cursor chat
   ┌──────────┐        ┌──────────┐       ┌──────────┐       ┌──────────┐          ┌──────────┐
   │ notes A  │        │ notes B  │       │ (gone    │       │ notes D  │          │ notes E  │
   │          │        │          │       │  after   │       │          │          │          │
   └──────────┘        └──────────┘       │  run)    │       └──────────┘          └──────────┘
        ▲                   ▲             └──────────┘            ▲                     ▲
        └───────────────────┴──────── you, copy-pasting ──────────┴─────────────────────┘
```

So you become the integration layer. As one operator put it on Reddit: *more tools, more islands, more of me playing human router.*

With a shared memory layer, every tool reads and writes the same place:

```
   ChatGPT   Claude   Claude Code   Codex   Cursor   OpenClaw   Hermes   Copilot   Grok ...
      │        │          │           │        │         │         │        │        │
      └────────┴──────────┴───────────┴────┬───┴─────────┴─────────┴────────┴────────┘
                                           │  MCP (Model Context Protocol)
                                  ┌────────▼────────┐
                                  │    Vilix AI     │
                                  │  one memory:    │
                                  │  conversations, │
                                  │  projects,      │
                                  │  rules, skills, │
                                  │  agent inbox    │
                                  └─────────────────┘
```

## Context vs memory

A common question (asked word for word on Reddit): *"What is the difference between persistent memory and context?"*

| | Context | Memory |
|---|---|---|
| What it is | Everything the model can see **right now**, in this conversation | What **survives** after the conversation ends |
| Analogy | The desk | The filing cabinet |
| Lifetime | One chat or one agent run | Across chats, days, devices and tools |
| Default in most AI tools | Cleared on every new chat | Private to that one tool (if it exists at all) |

Vilix AI is one filing cabinet shared by all your tools. At the start of a chat or an agent run, the right files from the cabinet are put on the desk automatically.

## Six pains, and how Vilix AI fixes each one

These come from real operators, founders and agencies (verified Reddit threads in r/AI_Agents, r/SaaS, r/n8n, r/LocalLLaMA).

### 1. The re-explaining tax

> "The re-explaining is the tax nobody budgets for. I timed a Friday of it recently and got about three hours of restating context." (r/SaaS)

**How Vilix AI fixes it:** every chat or agent run starts with `get_context(user_prompt)`. Vilix AI searches all your saved conversations from every tool (semantic + keyword search) and returns only what's relevant: the stack, the decisions, yesterday's progress. You stop pasting project docs, and you stop paying tokens to re-read them.

### 2. Agents that can't see each other

OpenClaw writes to its own memory files, a Codex session disappears when the run ends, a ChatGPT scheduled task only sees its own thread.

**How Vilix AI fixes it:** every agent ends its run with `save_turn`, which stores the full exchange plus a `source` label (which tool saved it) and a timestamp. Any other agent, in any other tool, finds it with `get_context` or `search_semantic(query, source?)`. The OpenClaw heartbeat reads what last night's Codex job learned before it spends a token. To steer agents directly, use the **agent inbox**: `message_agents(body, urgency, scope_type)` delivers your note inside the next Vilix AI read of the agents you target.

### 3. The Groundhog Day problem (repeated failures)

> "I've literally watched my agents burn through tokens trying the exact same broken API call the previous one just failed at. It's like paying for Groundhog Day." (r/AI_Agents)

**How Vilix AI fixes it:** failed attempts are saved like successes. The run record includes what was tried, the exact error, and the conclusion. The next agent's `get_context` surfaces it, so run 41 starts from what runs 1 to 40 learned. See [`config/run-record-format.md`](./config/run-record-format.md).

### 4. Lost decisions

> "what gets lost is never facts, it's decisions." (r/AI_Agents)

**How Vilix AI fixes it:** Vilix AI stores **full conversation turns**, so the *why* behind a decision survives ("Postgres, because we need row level security"), searchable by meaning from any tool. Every result carries a timestamp and the newest wins, so saying "we're not doing that anymore" once makes it the truth for every agent.

### 5. Instruction files that drift

CLAUDE.md for Claude Code, AGENTS.md for Codex, MEMORY.md for OpenClaw, custom instructions in ChatGPT. Update one, forget three.

**How Vilix AI fixes it:** write rules once.
- **Personal rules** (`add_user_rule`): your style and standing instructions, loaded into every new chat in every tool.
- **Project rules** (`create_project(..., project_rules)` / `update_project`): stack, conventions, never-do-X items for one project.
- **Skills** (`create_skill`): reusable procedures (deploy, release notes, client onboarding) that any agent in any tool can load with `get_skill`.

Fix it in one place, it's fixed everywhere. Examples: [`config/project-rules-examples.md`](./config/project-rules-examples.md), [`config/skill-examples.md`](./config/skill-examples.md).

### 6. Context dies between devices and between clients

Solo founders plan on the phone in ChatGPT and build at night in Claude Code. Agencies juggle 3 to 7 client contexts a day.

**How Vilix AI fixes it:** one account, one memory on phone and laptop, in every tool. One **project** per product or per client, each with its own rules and tasks (`create_task`, `update_task_state`: todo, doing, blocked, done). Every tool sees what's active, blocked and done.

## How it works: the get_context → save_turn loop

```
 ┌─────────────── any tool: ChatGPT, Claude Code, Codex, OpenClaw, Hermes ... ───────────────┐
 │                                                                                           │
 │  1. get_context(user_prompt, chat_id?)   ──►  Vilix AI returns:                           │
 │                                                 • related past turns from ALL tools       │
 │                                                 • recent messages                         │
 │                                                 • personal rules + active projects        │
 │                                                 • inbox messages for this agent           │
 │  2. do the work (with that context on the desk)                                           │
 │                                                                                           │
 │  3. save_turn(user_message, assistant_message, source, chat_id?)                          │
 │        • full exchange, including what failed and why                                     │
 │        • labeled with the tool that saved it + timestamp                                  │
 │        • retried saves are deduplicated server-side                                       │
 └───────────────────────────────────────────────────────────────────────────────────────────┘
```

Full tool list with parameters: [`reference/mcp-tools.md`](./reference/mcp-tools.md).

## Quick start (5 minutes)

1. **Create your account** at [vilix.ai](https://vilix.ai). There's a free plan and a 7-day free trial of Pro, no credit card.
2. **Connect your first tool.** The MCP endpoint is:
   ```
   https://api.vilix.ai/mcp
   ```
   Most tools use one-time OAuth (sign in and approve). Always-on agents like OpenClaw and Hermes use a personal API key: in Vilix AI open **Agents → Create API key** and send it as `Authorization: Bearer YOUR_VILIX_API_KEY`. Step-by-step for every tool: [`setup/README.md`](./setup/README.md).
3. **Save your first memory.** In the connected tool, type:
   ```
   Remember in Vilix AI: Northstar is my invoicing SaaS. We use Next.js and Stripe. The pricing page comes first.
   ```
4. **Connect a second tool** and ask: *"What am I building and what comes first?"* It answers from the memory the first tool saved.
5. **Make it automatic.** Ask any connected tool: *"Set up Vilix AI to load and save context automatically."* It calls `get_setup_instructions` and gives you the exact block to paste and where. For agents, use the template in [`prompts/agent-run-template.md`](./prompts/agent-run-template.md).

## Workflows by role

| Workflow | For | What you get |
|---|---|---|
| [Multi-agent operator: one memory for every agent](./workflows/01-multi-agent-operator.md) | People running OpenClaw, Hermes, Claude Code, Codex, ChatGPT scheduled tasks, n8n | Agents that read each other's runs, never repeat failures, and take orders from your phone via the inbox |
| [Solo founder: phone to laptop](./workflows/02-solo-founder-phone-to-laptop.md) | One developer building one product | Spec written in ChatGPT on the phone is there when Claude Code or Cursor opens the repo |
| [Agency: one brain per client](./workflows/03-agency-one-project-per-client.md) | Automation freelancers and small studios | Per-client rules, searchable build history, reusable skills across clients |
| [Switching tools without starting over](./workflows/04-switching-tools-without-starting-over.md) | Vibe coders moving from Lovable/Bolt to Cursor or Claude Code | The new tool knows the app, its decisions and what was tried |
| [One set of rules and skills for every tool](./workflows/05-shared-rules-and-skills.md) | Anyone with CLAUDE.md, AGENTS.md, custom instructions | Rules and procedures written once, used everywhere |

## FAQ

**Why does ChatGPT forget my project the next day?**
A new chat starts with an empty context window. ChatGPT's own memory stores short extracted facts and only inside ChatGPT. With Vilix AI connected, ChatGPT loads the relevant history with `get_context` at the start of the chat, including what you discussed in other tools.

**How do I share context between Claude Code and Codex?**
Connect both to Vilix AI (`claude mcp add --transport http --scope user vilix https://api.vilix.ai/mcp` and `codex mcp add vilix --url https://api.vilix.ai/mcp`). Both read and write the same memory, labeled by source.

**How do I stop my agents from repeating the same failed API call?**
Save failures in the run record (`save_turn` with what was tried, the error and the conclusion) and start every run with `get_context`. Template: [`prompts/agent-run-template.md`](./prompts/agent-run-template.md).

**What's the difference between persistent memory and context?**
Context is what the model sees now. Persistent memory survives between chats. See [Context vs memory](#context-vs-memory).

**Does it work on my phone?**
Yes. Add the connector once from a desktop browser, then the same memory works in the ChatGPT and Claude mobile apps.

**Can I keep clients separate?**
Yes. One Vilix AI project per client, each with its own project rules and tasks.

**Can I export or delete my memory?**
Yes. Export everything anytime from the dashboard (`vilix-export-{timestamp}.ndjson`), delete single memories or wipe the account. Your data isn't sold and isn't used for training.

---

Built with [Vilix AI](https://vilix.ai), the shared memory layer for every AI tool. Questions or a workflow you want covered? Open an issue.
