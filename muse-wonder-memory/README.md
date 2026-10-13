# Muse Wonder Memory: Share Active Memory With Claude, ChatGPT and Codex

Does Meta's Muse Wonder remember between conversations? Yes. Its Active Memory carries context across chats and tops Meta's own MemoryBench. This guide covers what Meta launched, what the scoreboard says, why no single model wins every task, and how to give Muse, Claude, ChatGPT and Codex one shared memory with Vilix AI over MCP. It's the full technical guide behind the Agentic Pod episode **"Meta's Muse Wonder Remembers Everything. Your Other AIs Don't."**.

- 🎧 Watch the episode: Muse Wonder on Agentic Pod (link added after upload)
- 📖 Read the article: [Muse Wonder Remembers You. Now Make Claude, ChatGPT and Codex Remember Too.](https://vilix.ai/blog/muse-wonder-active-memory-claude-chatgpt-codex/)
- 🧠 Memory layer used here: [Vilix AI](https://vilix.ai)
- ⚙️ Setup for 20+ AI tools: [`shared-memory-for-ai-agents/setup/`](../shared-memory-for-ai-agents/setup/README.md)

---

## Contents

1. [What Meta launched](#what-meta-launched)
2. [The MemoryBench scoreboard](#the-memorybench-scoreboard)
3. [What Active Memory and Active Focus do](#what-active-memory-and-active-focus-do)
4. [No model wins every row](#no-model-wins-every-row)
5. [Four pains, and how Vilix AI fixes each one](#four-pains-and-how-vilix-ai-fixes-each-one)
6. [Setup: connect Muse, Claude, ChatGPT and Codex to Vilix AI](#setup-connect-muse-claude-chatgpt-and-codex-to-vilix-ai)
7. [Copy-paste prompts](#copy-paste-prompts)
8. [Who it's for](#who-its-for)
9. [FAQ](#faq)

---

## What Meta launched

On October 12, 2026, Meta Superintelligence Labs launched two frontier models built for agents that do personal and professional work:

| | Muse Wonder | Muse Wonder Full |
|---|---|---|
| Version | Fast | Full |
| Where | Meta Model API and Muse Code, available now | Muse Code higher subscription plans |
| Context window | 1 million tokens | 1 million tokens |
| API price at launch | about $4 per 1M input tokens, $20 per 1M output | |

Meta says Wonder will soon power the subscription version of the Muse personal agent app.

## The MemoryBench scoreboard

MemoryBench is Meta's own evaluation of how well a model remembers and uses context over a long interaction history. Scores at maximum effort:

| Model | MemoryBench |
|---|---|
| Muse Wonder | 85.6 |
| Muse Wonder Full | 85.5 |
| Claude Opus 5.5 | 81.5 |
| GPT-6 Astra | 81.4 |
| Claude Fable 5.1 | 79.0 |

It's Meta's internal benchmark and independent testing is still coming in. The bigger signal: memory is now something frontier labs compete on and publish scores for.

## What Active Memory and Active Focus do

- **Active Memory** carries important information across conversations and pulls it back in when it becomes relevant. Mention your project on Monday, and on Thursday Muse already knows it.
- **Active Focus** keeps the model working when a task gets hard: it tries other approaches and recovers from errors.

## No model wins every row

Meta's own table shows different leaders for different work:

| Work | Leader |
|---|---|
| Legal agent work (Harvey LAB), finance research (Vals Finance Agent), business workflows (AutomationBench), long-horizon coding (DeepSWE, SWE-Marathon) | Muse Wonder Full |
| Agentic terminal coding (Terminal-Bench), from-scratch programming (ProgramBench), professional tool use (JobBench), Artificial Analysis Index | Claude Opus 5.5 |
| Professional document work (GDP.pdf) | GPT-6 Astra |

So the smart setup is routing: plan in ChatGPT on your phone, build in Claude Code or Codex, and hand legal and finance research to an agent running Muse Wonder. Each model does what it's best at, and one shared memory keeps them all on the same page.

```
  ChatGPT (plan)     Claude Code / Codex (build)     Muse Wonder agent (legal, finance)
        │                       │                                │
        └───────────────────────┼────────────────────────────────┘
                                │  MCP
                       ┌────────▼────────┐
                       │    Vilix AI     │
                       │ full conversations, user + project rules,
                       │ projects + tasks, skills, agent inbox
                       └─────────────────┘
```

## Four pains, and how Vilix AI fixes each one

### 1. Each lab's memory stays in its own app

Muse's Active Memory knows what you told Muse. Claude's memory knows what you told Claude. ChatGPT's memory knows what you told ChatGPT.

**How Vilix AI fixes it:** every tool connects to the same Vilix AI account over MCP. Before a reply, the tool calls `get_context`, and Vilix AI returns your relevant history from every connected tool, your rules and your open tasks. After the reply, the tool calls `save_turn`, and the exchange is saved with a source label (`Muse`, `Claude`, `Codex`) and a timestamp. What you told Muse is there for Claude, and the other way around.

### 2. Every switch means re-briefing

The project, the decisions, the client's tone, what already failed.

**How Vilix AI fixes it:** Vilix AI keeps full conversation history, not just extracted facts, and pulls back only what's relevant. Failed attempts are saved like successful ones, so the next model skips the dead end. When two tools disagree, the newest memory wins, so you correct something once and it's corrected everywhere.

### 3. Rules live in three different settings pages

**How Vilix AI fixes it:** keep personal preferences as **user rules** ("short answers", "TypeScript strict") and the stack, conventions and tone as **project rules** on a Vilix AI project. Every connected model gets the same current version through `get_context`.

### 4. Agents on different models don't share procedures or task state

**How Vilix AI fixes it:** **projects and tasks** are injected into every model's context, so each one sees what's open and what's done. **Skills** are saved procedures any agent can load, so you build a procedure once and every agent follows it. The **agent inbox** lets you leave a note from your phone, and your agents read it on their next memory read.

## Setup: connect Muse, Claude, ChatGPT and Codex to Vilix AI

1. Create your account at [vilix.ai/get-started](https://vilix.ai/get-started) (free plan forever, 7-day Pro trial, no credit card).
2. Use the same Vilix AI account everywhere. Server URL: `https://api.vilix.ai/mcp`.

**Muse:** in Vilix AI open **Agents → Create API key** and copy it. In a Muse chat, say *"Connect Vilix AI so you have my context."* and paste the key into the secure link Muse sends.

**ChatGPT:** Settings → Security and login → **Developer mode**. Then Plugins → **+**, name **Vilix AI**, paste the URL, choose **OAuth**, sign in and approve. In a chat: **+ → Developer mode → Vilix AI**.

**Claude (web, desktop):** **Customize → Connectors → Add custom connector**, name **Vilix AI**, paste the URL, **Connect**. Enable it in a chat under **+ → Connectors**.

**Claude Code:**
```bash
claude mcp add --transport http --scope user vilix https://api.vilix.ai/mcp
# then inside Claude Code: /mcp -> vilix -> sign in
```

**Codex:**
```bash
codex mcp add vilix --url https://api.vilix.ai/mcp
codex mcp login vilix
```

**Server-side agents on the Meta Model API:** send the Vilix AI API key as `Authorization: Bearer YOUR_VILIX_API_KEY` with transport Streamable HTTP. Treat the key like a password.

3. Make it automatic: in each tool, ask *"Set up Vilix AI to load and save context automatically."* It calls `get_setup_instructions` and gives you the exact block and where it goes.
4. Verify: tell Muse a decision, then open Claude and ask *"What did I decide in Muse today?"*

Other tools (Cursor, OpenClaw, Hermes, Gemini, Grok and more): [`shared-memory-for-ai-agents/setup/`](../shared-memory-for-ai-agents/setup/README.md).

## Copy-paste prompts

**In Muse, after a research session:**
```
Save to Vilix AI: the findings from this contract review, the risks we flagged and why, and create a task in the "Acme legal" project for each follow-up.
```

**In Claude Code, picking it up:**
```
Load context from Vilix AI for the "Acme legal" project. What did Muse flag today, and which tasks are still open?
```

**In ChatGPT on your phone:**
```
Send a Vilix AI agent inbox message to my agents: use the "Acme" tone rules for every client draft this week.
```

**Save a procedure once for every model:**
```
Save a Vilix AI skill "client contract review": steps, checklist and output format. Any agent on any model should follow it.
```

## Who it's for

- **AI agencies and automation builders:** give each client their own Vilix AI account, included in your retainer or paid by the client. Wire every agent you build for that client, on any model, to that client's key. When Muse Wonder wins a task you run for a client, swap the model and the client's whole history comes along. You never build memory from scratch.
- **People who sell standalone agents:** plug Vilix AI in as the agent's memory, one account per customer, and pick the best model for the job.
- **Solo founders and indie devs:** if you pay for two or three AI subscriptions, one memory layer ends the re-briefing tax.

## FAQ

**Does Muse Wonder remember between conversations?**
Yes. Active Memory carries important information across conversations and brings it back when it's relevant.

**Can Claude or ChatGPT see what Muse remembers?**
Each lab's built-in memory stays in its own product. Connect all of them to Vilix AI over MCP and they read the same context.

**Where can I use Muse Wonder today?**
Through the Meta Model API and Muse Code. Meta says it will roll out to Muse app subscribers soon.

**Is Muse supported by Vilix AI?**
Yes. Muse connects with a Vilix AI API key, next to ChatGPT, Claude, Claude Code, Codex, Cursor, OpenClaw, Hermes, Gemini Spark, Gemini CLI, Grok, Manus, GitHub Copilot, Windsurf and any MCP-compatible AI.

**Is Vilix AI free?**
There's a free plan forever with no credit card, a 7-day Pro trial with no credit card, and paid Pro and Power plans. You can export or delete your memory anytime.
