# Does the Vercel AI SDK remember between sessions?

The full technical guide behind the Agentic Pod episode **"Does the Vercel AI SDK Remember Between Sessions?"**: why a Vercel AI SDK agent starts blank every session, five ways to give it memory, and how to give every agent in your stack (Vercel, Claude Code, Codex, Cursor, OpenClaw, Hermes or a cron job) one shared memory with Vilix AI.

- 🎧 Watch the episode: [Does the Vercel AI SDK Remember Between Sessions? on Agentic Pod](https://youtu.be/HMe3ScWWDeo)
- 📖 Read the article: [Your Vercel AI SDK Agent Forgets Everything Between Sessions. Here Is the Fix.](https://vilix.ai/blog/vercel-ai-sdk-agent-forgets-between-sessions-fix)
- 🧠 Shared memory used here: [Vilix AI](https://vilix.ai) (hosted MCP server: `https://api.vilix.ai/mcp`)
- ⚡ Get started: [vilix.ai/get-started](https://vilix.ai/get-started)

---

## The short answer

No. A `generateText` or `streamText` call only sees the messages you pass in. The `useChat` hook keeps its state in the browser, and serverless functions lose in-memory state on cold starts and redeploys. The SDK handles the interaction layer. Persistence is your job.

## Why it hurts most for scheduled agents

In a chat, a person can re-brief the agent. A cron-triggered agent has nobody to do that. A daily lead enrichment agent researches the same leads again, spends the same tokens again and can reach a different conclusion.

## Five ways to add memory

| Option | What it keeps | The catch |
|---|---|---|
| 1. Your own database (Postgres, Supabase, Upstash Redis, Vercel KV) | Full message history | You write summaries, retention and retrieval, and it stays inside one app |
| 2. Vercel Drives (sandbox agents) | Files across runs | Recall stays manual |
| 3. Durable workflow execution | Where a run stopped | It does not keep what the run learned |
| 4. Memory providers built for the SDK | Per-user memory with scoped retrieval | An extra service to run, and your memory is tied to that provider |
| 5. One hosted memory layer over MCP (Vilix AI) | Full conversations shared across tools | Connect once per tool |

## Pains, and how Vilix AI fixes each one

### 1. The agent starts blank every run
Vilix AI is a shared memory your agent connects to over MCP. At the start of a run the agent calls `get_context`, so it reads what earlier runs saved. At the end it calls `save_turn` with the full conversation, not a summary.

### 2. Every save should say where it came from
Each `save_turn` carries a source label (for example "Vercel AI SDK agent", "Claude Code" or "Cursor"), so you can tell which tool saved what.

### 3. Dead ends repeat
Failures are saved too, so tomorrow's run does not retry the same dead end.

### 4. Tools disagree
Newest memory wins. Correct a fact once in Vilix AI and every connected tool sees the new truth.

### 5. Repeated instructions
User rules and project rules in Vilix AI are loaded by every connected tool, and reusable procedures live as skills. Use projects to keep different subjects apart.

### 6. Nudging many agents
The agent inbox lets you post one message that your agents receive on their next read. Urgent messages repeat until the agent acknowledges them.

## The memory-bearing run loop

1. Start of run: call `get_context` with the task as the prompt.
2. Work: use what came back (skip leads already enriched, respect saved decisions).
3. End of run: call `save_turn` with the full conversation, a source label and the outcome, including failures.

## Setup

1. Create a free account at [vilix.ai](https://vilix.ai) and open [vilix.ai/get-started](https://vilix.ai/get-started).
2. Connect your tools over MCP (`https://api.vilix.ai/mcp`) with an API key. Setup takes under two minutes per tool.
3. For a Vercel AI SDK agent, give the agent the MCP tools (the SDK supports MCP clients) and put the run loop above in its system prompt.

## Verify it: the two-run test

1. Run the agent once and have it save a distinctive fact.
2. Run it again from a fresh process. It should read that fact from `get_context` before it acts.
3. Ask Claude Code or Cursor, connected to the same account, about the same fact. It should know it too.

## Copy-paste prompts

Start of run:
```
Before you do anything, call get_context with a short description of today's task. Use what it returns. Do not repeat work that earlier runs already finished.
```

End of run:
```
When you finish, call save_turn with the full conversation, source "Vercel AI SDK agent", and note what worked, what failed and what to do next.
```

## Agencies: one Vilix AI account per client

Give each client their own Vilix AI account, like a Gmail account that belongs to that client. Every agent you build for them connects to that account, so the memory belongs to their business and you never build memory from scratch.

## FAQ

**Does `useChat` remember between sessions?**
It keeps messages in the browser. Close the tab and that state is gone unless you store it yourself.

**Do I still need a database?**
Use one for your app's own data. Use Vilix AI for memory that should be shared across runs and across tools.

**Can I export or delete what Vilix AI stores?**
Yes. Your data is portable: export everything or delete it anytime. Vilix AI has a free plan, and the 7-day Pro trial needs no credit card.
