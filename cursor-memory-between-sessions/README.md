# Does Cursor remember between sessions?

The full technical guide behind the Agentic Pod episode **"Does Cursor Remember Between Sessions?"**: what Cursor keeps (chat history, Memories, rules files, the codebase index), what none of that carries into tomorrow's chat, why scheduled and background runs start blank, and how to give Cursor, Claude Code, ChatGPT, Codex and every other tool you use one shared memory with Vilix AI.

- 📖 Read the article: [Cursor Keeps Your Chats. It Still Forgets What You Taught It.](https://vilix.ai/blog/cursor-keeps-your-chats-it-still-forgets-what-you-taught-it)
- 🧠 Shared memory used here: [Vilix AI](https://vilix.ai) (hosted MCP server: `https://api.vilix.ai/mcp`)
- ⚡ Get started: [vilix.ai/get-started](https://vilix.ai/get-started)

---

## The short answer

| What Cursor keeps | Lives as long as | What it holds |
|---|---|---|
| Chat history | As long as you keep the chat | The conversation. Its knowledge only exists inside that chat's context window |
| Memories | Across chats, inside one IDE | A facts store. The management interface has changed between versions |
| Rules files (`.cursor/rules/`, `AGENTS.md`) | As long as the file exists | How to work: conventions, commands, style. Reloaded every session, written and updated by hand |
| Codebase index | While the project is indexed | A vector index for semantic code search. It finds code, not the reason you wrote it |

None of these stores the decisions, rejected options and lessons from yesterday's work in a form the next session, or another tool, can search. Scheduled and background runs start with a fresh context every time, and a run history is a log, not memory.

## Four pains, and how Vilix AI fixes each one

### 1. Yesterday's decision lives in one chat

You tell Cursor "we dropped that library, it broke our build". Today's chat has never heard of it and proposes the library again.

**How Vilix AI fixes it:** the first step of a task is `get_context`. Vilix AI returns the relevant decisions, preferences and rejected options saved by earlier sessions. The last step is `save_turn`, which stores the full exchange (not a thin summary), so the reason is still there next week.

### 2. Hand-written rules files go stale

Somebody has to write and update `.cursor/rules/` and `AGENTS.md`, and nobody updates six repos after every change.

**How Vilix AI fixes it:** standing instructions live in Vilix AI. **Project rules** (`create_project(..., project_rules)`) are read at the start of every task by every connected agent, and **user rules** (`add_user_rule`) apply to all your tools. Change a rule once and every agent gets the new version. **Skills** (`create_skill`, loaded with `get_skill`) hold step-by-step procedures you want repeated.

### 3. Scheduled and background runs start blank

Every run starts with a fresh context, so run one hundred is as blind as run one, and it pays full price to re-derive what the last run already worked out.

**How Vilix AI fixes it:** the run begins with `get_context("<what this run does>")` and ends with `save_turn(..., source="Cursor: <job name>")`. Failed attempts are saved with the exact error, so the next run skips the dead end. When a conclusion changes, the newest memory wins.

### 4. Cursor's memory stays inside Cursor

You also work in Claude Code, ChatGPT or Codex, and none of them see what Cursor learned.

**How Vilix AI fixes it:** every tool connects to the same Vilix AI account over MCP. Every save carries a **source label** (Cursor, Claude Code, ChatGPT, Codex, OpenClaw, Hermes), so you can see which tool wrote what, and `search_semantic(query, source?)` and `search_keyword(query)` find a specific earlier decision. The **agent inbox** (`message_agents`) lets you send a note to a running agent that arrives with its next Vilix AI read.

## The memory loop

```
 1. get_context("<what this task does>")  ->  past decisions, rejected options, rules, open items
 2. do the work with that history in view
 3. save_turn(user_message, assistant_message, source="Cursor: <task or job>")
       what was done, what was decided and why, what is still open, any failure with the exact error
```

## Setup in Cursor

Use the same Vilix AI account in every tool. Server URL: `https://api.vilix.ai/mcp`.

1. Add Vilix AI as an MCP server in Cursor and sign in with OAuth. Per-tool steps with screenshots: [vilix.ai/get-started](https://vilix.ai/get-started).
2. Make it automatic: in Cursor, ask `Set up Vilix AI to load and save context automatically.` Vilix AI's `get_setup_instructions` returns the exact block for Cursor and where to paste it (Cursor User Rules), so the agent calls `get_context` before it works and `save_turn` after.
3. Other tools: **Claude Code** `claude mcp add --transport http --scope user vilix https://api.vilix.ai/mcp`, **Codex** `codex mcp add vilix --url https://api.vilix.ai/mcp` then `codex mcp login vilix`, **ChatGPT and Claude** a custom connector with the server URL and OAuth.
4. Headless agents and scheduled jobs: create a key under **Agents, Create API key** and send `Authorization: Bearer YOUR_VILIX_API_KEY` to `https://api.vilix.ai/mcp`.

More per-tool commands: [`../mcp-memory-server-setup/`](../mcp-memory-server-setup/README.md).

## Verify it: the two-session test

1. In Cursor, make a small decision out loud: "We are not using library X because it broke the build. Save that."
2. Open a brand new Cursor chat in the same project and ask: "Which libraries did we rule out, and why?"
3. It should answer from `get_context`, not from the open chat. Check Vilix AI for a saved turn labelled with your source.
4. Ask Claude Code or Codex the same question. It answers from the same memory.

## Where each thing goes

If it describes the project, put it in a file (`AGENTS.md`, `.cursor/rules/`). If it describes the work, put it in searchable memory. Files for facts, memory for lessons.

## Copy-paste prompts

**Task instructions for any MCP-connected tool:**

```
Every task:
1. Call Vilix AI get_context with "<project>: what we decided, what we ruled out, which rules apply to <this task>".
2. Do the work with that history in view. Do not re-propose anything marked as rejected.
3. Call Vilix AI save_turn with source "Cursor: <task>". Include what you did, decisions and the reason,
   what is still open, and any failure with the exact error.
```

**Save a decision for every tool:**

```
Save to Vilix AI: we dropped <library> because it broke the build. Do not propose it again.
```

## Agencies: one Vilix AI account per client

Agencies and automation builders give each client their own Vilix AI account, like a Gmail account that belongs to that client. It is included in the retainer or paid by the client. Every agent and automation built for that client connects with that client's API key, so the memory belongs to the client's business. The agency never builds memory from scratch: it wires each new project into the client's account with `get_context` at the start and `save_turn` at the end.

## FAQ

**Does Cursor remember between sessions?**
It keeps chat history, Memories, rules files and a codebase index. Those cover what you said and what you wrote down. They are not a searchable record of yesterday's decisions that the next session, or another tool, can query.

**Do scheduled or background Cursor runs remember the last run?**
Each run starts with a fresh context. Run history and logs are not agent memory.

**Do I still need rules files?**
Yes, for facts about the project (conventions, commands). Put decisions and lessons in memory.

**Can I use Vilix AI with tools other than Cursor?**
Yes. Connect ChatGPT, Claude, Claude Code, Codex, OpenClaw, Hermes, n8n or any MCP client to the same account and they share one memory.

**Can I export or delete what Vilix AI stores?**
Yes. Export everything or delete it anytime. Vilix AI has a free plan, and the 7-day Pro trial needs no credit card.
