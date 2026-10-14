# What Are OpenAI Dots? Can You Control What They Remember?

The full technical guide behind the Agentic Pod episode **"What Are OpenAI Dots? Can You Control What They Remember?"**: what OpenAI's always-on Dots are, how their memory works, what OpenAI's Dots FAQ says about viewing and deleting dot memories, and how to give every agent you run (a dot, Claude Code, Codex, n8n, OpenClaw, Hermes) one shared memory you can inspect, correct, delete and export, with Vilix AI.

- 🎧 Watch the episode: [What Are OpenAI Dots? Can You Control What They Remember? on Agentic Pod](https://youtu.be/GnIKVovI9is)
- 📖 Read the article: [OpenAI Dots Explained: The Always-On Agent With Memory You Can't Control](https://vilix.ai/blog/openai-dots-explained-the-always-on-agent-with-memory-you-cant-control/)
- 🧠 Shared memory used here: [Vilix AI](https://vilix.ai) (hosted MCP server: `https://api.vilix.ai/mcp`)
- ⚡ Get started: [vilix.ai/get-started](https://vilix.ai/get-started)

---

## Contents

1. [The short answer](#the-short-answer)
2. [What a dot is](#what-a-dot-is)
3. [How Dots memory works](#how-dots-memory-works)
4. [Four memory pains, and how Vilix AI fixes each one](#four-memory-pains-and-how-vilix-ai-fixes-each-one)
5. [The memory-bearing run loop](#the-memory-bearing-run-loop)
6. [Setup: ChatGPT, Claude Code, Codex, n8n, OpenClaw and Hermes](#setup)
7. [Verify it: the one-wrong-memory test](#verify-it-the-one-wrong-memory-test)
8. [Copy-paste prompts](#copy-paste-prompts)
9. [Agencies: one Vilix AI account per client](#agencies-one-vilix-ai-account-per-client)
10. [FAQ](#faq)

---

## The short answer

| Question | OpenAI Dots today | With Vilix AI as the shared memory |
|---|---|---|
| Where does the agent's memory come from? | ChatGPT memory of your preferences plus the dot's own notes | Full conversation turns saved by every connected agent, searchable by meaning and keyword |
| Can I view, edit or delete one memory? | Not today, per OpenAI's Dots privacy FAQ; deleting the dot clears its context | Yes: list, update or delete any single memory from any connected AI or the dashboard |
| Does disconnecting an app remove what was learned? | Disconnecting stops new access; what the dot already learned stays | Delete the specific memory; source labels show which tool saved it |
| Which tools can read the memory? | ChatGPT, Slack and Teams | Every MCP-compatible tool: ChatGPT, Claude, Claude Code, Codex, Cursor, OpenClaw, Hermes, n8n and more |
| Can I export it? | Not covered for dot memories | Export everything anytime from the dashboard, or wipe the account instantly |

Dots are a strong always-on agent. If you also want memory you control, and memory every other agent in your stack can read, keep it in one shared layer: Vilix AI.

## What a dot is

OpenAI announced Dots at its DevDay keynote on September 29, 2026. A dot is a persistent agent you set up inside ChatGPT: you give it a name and a goal, and it keeps working on that goal in the background.

- Runs on the **GPT-6 Astra** model.
- Gets its **own cloud computer and its own browser**.
- Does what OpenAI calls **proactive research**: it reads connected apps with read-only tools, forms notes about what it finds, and pursues ongoing responsibilities like monitoring customer feedback or tracking a recurring pipeline.
- Connects to **more than 4,000 applications** through OpenAI's plugin ecosystem.
- You reach it through **ChatGPT, Slack and Microsoft Teams**, with text messages on the way.
- The first dot is included with **ChatGPT Pro and Business Premium**. Specialist dots with their own identities, credentials and tools are positioned as the next step.

## How Dots memory works

A dot draws on two sources:

1. **ChatGPT's existing memory** of your preferences.
2. **Its own notes** about your ongoing responsibilities and working habits.

That context carries across ChatGPT, Slack and Teams, so a preference you state in a chat can shape what the dot does overnight. The job survives the night because the memory does. This confirms what automation builders have said for years: the winning setup is the model plus persistent memory plus tools, waking up with context already loaded.

## Four memory pains, and how Vilix AI fixes each one

### 1. One wrong memory, and no way to fix just that one

OpenAI's Dots privacy, security and safety FAQ says you currently can't view, delete or directly modify individual dot memories, including details that came in from plugins. To clear a dot's context you delete the dot, which also removes its conversations, saved memories and scheduled tasks.

**How Vilix AI fixes it:** every memory is a saved conversation turn you can list, update or delete one at a time, from any connected AI or from the dashboard at [app.vilix.ai](https://app.vilix.ai). Fix one wrong note in one place, and every agent sees the fix, because every agent reads the same memory. When a decision changes, you say it once and the **newest memory wins** for every agent.

### 2. Disconnecting an app doesn't remove what was learned

Per OpenAI's documentation, disconnecting a service stops new access but doesn't delete information the dot already built into its context. If a dot read a CRM overnight and you want one customer gone, the only route is deleting the whole dot.

**How Vilix AI fixes it:** every turn is saved with `save_turn` and a **source label** (which tool or agent saved it) plus a timestamp. Find the memory with `search_semantic(query, source?)` or `search_keyword(query)`, then delete just that one. Nothing else is touched: the rest of the history, your rules and your tasks stay.

### 3. The memory stays inside one ecosystem

A dot's memory works across ChatGPT, Slack and Teams. Your n8n workflows, scheduled reports, and coding agents in Claude Code or Codex can't read what the dot spent weeks learning.

**How Vilix AI fixes it:** one cloud-hosted memory that every MCP-compatible tool connects to: ChatGPT, Claude, Claude Code, Codex, Cursor, OpenClaw, Hermes, Gemini CLI, GitHub Copilot, Windsurf, n8n and more. Every run starts with `get_context` and ends with `save_turn`, so the Codex job reads what last night's n8n run decided, and the other way around. Failures are saved too, with the exact error, so no agent repeats a broken step.

### 4. Accumulated context you can't move or audit

Weeks of accumulated judgment (preferences, decisions, what was already handled) is the part that compounds. You want to see it, audit it and take it with you.

**How Vilix AI fixes it:** export everything anytime from the dashboard (`vilix-export-{timestamp}.ndjson`), or wipe the account instantly. Standing knowledge lives in places you can read and edit:
- **Project rules** (`create_project(..., project_rules)` / `update_project`): conventions and never-do-X items for one product or one client.
- **User rules** (`add_user_rule`): how you like things done, loaded into every tool.
- **Skills** (`create_skill`, loaded with `get_skill`): step-by-step procedures any agent can follow.
- **Agent inbox** (`message_agents(body, urgency, scope_type)`): steer a running agent; the note arrives with its next Vilix AI read.

## The memory-bearing run loop

```
 ┌──────── any agent run: a dot's task, Claude Code, Codex, n8n, OpenClaw, Hermes, cron ─────────┐
 │                                                                                               │
 │  1. get_context("<what this run does>")   ──►  Vilix AI returns:                              │
 │                                                  • what earlier runs handled (from ANY tool)  │
 │                                                  • open items and decisions (newest wins)     │
 │                                                  • user rules, project rules, active tasks    │
 │                                                  • agent inbox notes for this agent           │
 │  2. do the work                                                                               │
 │                                                                                               │
 │  3. save_turn(user_message, assistant_message, source="<tool + job name>")                     │
 │        • what was handled, decided, still open                                                │
 │        • failures too: what was tried, the exact error, the conclusion                        │
 │                                                                                               │
 │  Any time: list, update or delete one memory · export everything · wipe the account           │
 └───────────────────────────────────────────────────────────────────────────────────────────────┘
```

## Setup

Use the **same Vilix AI account** everywhere. Server URL: `https://api.vilix.ai/mcp`.

**ChatGPT:** Settings → Security and login → **Developer mode**. Then Plugins → **+**, name **Vilix AI**, paste the URL, choose **OAuth**, sign in and approve. Enable **Vilix AI** in the chat under **+ → Developer mode**.

**Claude Code:**

```bash
claude mcp add --transport http --scope user vilix https://api.vilix.ai/mcp
```

**Codex:**

```bash
codex mcp add vilix --url https://api.vilix.ai/mcp
codex mcp login vilix
```

**n8n, OpenClaw, Hermes and other always-on agents:** create a key in Vilix AI under **Agents → Create API key** and send it as `Authorization: Bearer YOUR_VILIX_API_KEY` to `https://api.vilix.ai/mcp`. In n8n, attach an MCP client tool to the AI Agent node with that endpoint and header. Full per-tool commands: [`../mcp-memory-server-setup/`](../mcp-memory-server-setup/README.md).

**Make it automatic:** in any connected tool, ask `Set up Vilix AI to load and save context automatically.` It calls `get_setup_instructions` and returns the exact block and where it goes.

## Verify it: the one-wrong-memory test

1. In Claude Code, tell your agent: *"Save to Vilix AI: our biggest customer is Acme, contact is Dana."* It calls `save_turn`.
2. In ChatGPT, ask: *"Who is our biggest customer and who is the contact?"* It answers from Vilix AI.
3. Delete that single memory from the Vilix AI dashboard (or ask any connected AI to delete it).
4. Ask again in Codex or n8n. The fact is gone everywhere, and every other memory is still there.

## Copy-paste prompts

**Run loop for any agent:**

```
Every run:
1. Call Vilix AI get_context with "<job name>: what was already handled, what is open, which rules apply".
2. Do the job. Skip anything already handled. Follow the project rules.
3. Call Vilix AI save_turn with source "<tool>: <job name>". Include items handled, decisions made,
   items still open, and any failure with the exact error.
```

**Correct one memory for every agent:**

```
Save to Vilix AI: Dana no longer owns the Acme account; it's Priya as of today. Newest wins for every agent.
```

**Find and remove one detail:**

```
Search Vilix AI for memories about <customer name> saved by source "<tool>". Show them to me, then delete only those.
```

**Set standing rules once:**

```
In Vilix AI, create a project "<client or product>" with these project rules:
- Monitor customer feedback daily, report only new themes.
- Never store payment details in memory.
```

**Steer a running agent:**

```
Send a Vilix AI agent inbox message to my OpenClaw agent: pause the competitor report until Monday.
```

## Agencies: one Vilix AI account per client

Agencies and automation builders give each client their own Vilix AI account, included in the retainer or paid by the client. Every agent and automation built for that client (support agents, lead-gen flows, n8n pipelines, OpenClaw and Hermes agents) connects with that client's API key, so the memory belongs to the client's business. The agency never builds memory from scratch, and when a client asks to remove one detail, it's one delete, not a rebuilt agent.

## FAQ

**What are OpenAI Dots?**
Always-on, personal agents in ChatGPT, announced at DevDay on September 29, 2026. Each runs on GPT-6 Astra with its own cloud computer and browser, pursues a goal in the background, and connects to more than 4,000 apps. You reach it in ChatGPT, Slack and Teams.

**What do Dots remember?**
ChatGPT's memory of your preferences plus the dot's own notes about your ongoing work, carried across ChatGPT, Slack and Teams.

**Can I delete a single dot memory?**
Per OpenAI's Dots privacy FAQ, not today: you can't view, delete or directly modify individual dot memories. Deleting the dot clears its context along with its conversations and scheduled tasks.

**Does disconnecting an app erase what my dot learned from it?**
OpenAI's documentation says disconnecting stops new access but doesn't delete what the dot already built into its context.

**How do I get memory I can edit and delete across every tool?**
Connect your agents to Vilix AI over MCP. Every run calls `get_context` first and `save_turn` last, and you can list, update or delete any single memory from any connected AI or the dashboard. The newest memory wins when facts change.

**Can my other agents read the same memory?**
Yes. ChatGPT, Claude, Claude Code, Codex, Cursor, OpenClaw, Hermes, n8n and any MCP-compatible tool connect to the same Vilix AI account, and every saved turn carries its source label.

**What does Vilix AI cost?**
There is a free plan, and a 7-day Pro trial with no credit card. You can export or delete everything anytime. [vilix.ai/get-started](https://vilix.ai/get-started)
