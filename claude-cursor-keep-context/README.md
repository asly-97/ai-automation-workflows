# How do you keep context between Claude and Cursor?

The full technical guide behind the Agentic Pod episode **"How Do You Keep Context Between Claude and Cursor?"**: what the Claude to Cursor handoff loses, the decision record that fixes it, three ways to carry it into Cursor, and how to share one memory between both tools with Vilix AI.

- 🎧 Watch the episode: [How Do You Keep Context Between Claude and Cursor? on Agentic Pod](https://youtu.be/dAwyTgAPD8c)
- 📖 Read the article: [How to keep context between Claude and Cursor](https://vilix.ai/blog/keep-context-between-claude-and-cursor)
- 🧠 Shared memory used here: [Vilix AI](https://vilix.ai) (hosted MCP server: `https://api.vilix.ai/mcp`)
- ⚡ Get started: [vilix.ai/get-started](https://vilix.ai/get-started)

---

## The short answer

Put your decisions and constraints somewhere both tools can read. Cursor does not automatically read the conversations in your Claude account. It can use repository files, chat input and the integrations you configured. So end every Claude design session with a written decision record, then make that record available to Cursor.

## What the handoff really loses

The chat is lost, but the real damage is the **constraints**: a latency budget, a library you ruled out, an error-handling convention, the reason you chose one schema over another. If those stay in the Claude thread, Cursor can propose something that conflicts with them.

## Step 1: end the Claude session with a decision record

Treat the design chat like a meeting that has to end with minutes. A good record has five short lists:

- **Decisions**: what was chosen, one line each.
- **Rejected options and why**: so Cursor does not re-propose them.
- **Constraints**: hard limits the code must respect.
- **Conventions**: naming, error handling, file layout, test expectations.
- **Open questions**: what you deliberately deferred, so a gap is not mistaken for a decision.

Prompt to end on:

```
Summarise this session as a decision record with Decisions, Rejected options, Constraints, Conventions, and Open questions. Keep each item to one line. No prose.
```

Read the record and fix anything wrong before you use it.

## Step 2: get the record into Cursor (three ways)

1. **A file in the repo.** Paste the reviewed record into `DECISIONS.md` at the repo root and commit it. Ask Cursor to read it before editing. Teammates can review it, and it survives the chat closing.
2. **Explicit editor context.** Attach the file or paste the record into the implementation chat. In a new session or a long conversation, confirm the decisions are still in context.
3. **One shared memory (Vilix AI).** Connect Claude and Cursor to the same Vilix AI account so neither handoff needs you to carry anything.

You can combine 1 and 3: `DECISIONS.md` as the canonical, reviewable record and Vilix AI for the rest of the saved conversations.

## How Vilix AI does the handoff

Vilix AI is a hosted memory both tools talk to over MCP.

- **Before each reply**, the tool calls `get_context` with your latest message and uses what comes back. Cursor reads what Claude saved.
- **After each reply**, it calls `save_turn` with the full exchange, not a summary, so the reasons are saved and not just the outcomes.
- **Source labels**: every save says where it came from (for example "Claude" or "Cursor").
- **Saved failures**: dead ends are saved too, so nobody retries them.
- **Newest wins**: correct a decision once and every connected tool sees the new truth.
- **Project rules** hold conventions (naming, error handling), **user rules** hold personal preferences, and **skills** hold reusable steps. Every connected tool loads them.
- **Agent inbox**: post one message and all your agents get it on their next read.

## A two-day example

- **Day 1, Claude.** You design a webhook ingestion service. You choose idempotency keys, reject a queue as overkill for the volume, and require retries to be safe to replay. You end with the decision record prompt.
- **End of day 1.** Review the record, commit it to `DECISIONS.md`, and check the exchange was saved to your Vilix AI account.
- **Day 2, Cursor.** Ask Cursor to read `DECISIONS.md` or call `get_context` before implementing. Check that idempotency keys, no queue and replay-safe retries come back. Review and test the handler against them.
- **Day 2, mid-build.** You find the dedupe logic mishandles duplicate IDs. Update `DECISIONS.md` and save the exchange. Next Claude session, ask for the updated record first.

## Setup

1. Create a free account at [vilix.ai](https://vilix.ai) and open [vilix.ai/get-started](https://vilix.ai/get-started).
2. Connect Claude and Cursor over MCP (`https://api.vilix.ai/mcp`). Setup takes under two minutes per tool.
3. Add the run loop below to each tool's instructions, then verify it with the two-tool test.

## Copy-paste prompts

Start of every reply:
```
Before you answer, call get_context with my latest message and use what it returns. Respect the saved decisions and constraints.
```

End of every reply:
```
After you reply, call save_turn with the exact message and your reply, and a source label ("Claude" or "Cursor"). Save failures too.
```

## Verify it: the two-tool test

1. In Claude, make a distinctive decision ("no queue, idempotency keys only") and let it save.
2. Open Cursor, connected to the same account, and ask what you decided about queues.
3. Cursor should answer from `get_context` before it writes any code.

## Agencies: one Vilix AI account per client

Give each client their own Vilix AI account, like a Gmail account that belongs to that client. Every tool you wire up for them connects to that account, so the memory belongs to their business.

## FAQ

**Can Cursor read my Claude conversations directly?**
Not automatically. It uses repository files, chat input and configured integrations. Provide a file, transfer the text, or use a memory service both tools connect to.

**Isn't a `DECISIONS.md` file enough?**
It can be for the decisions it contains. Its limits are keeping it current, making sure the tool reads it, and details left out of the summary.

**Does it work from Cursor back to Claude?**
Yes. Update the record or save the exchange, then ask Claude to read or retrieve it before the next design discussion.

**What should stay out of the record?**
Full transcripts, low-value detours and details that change every commit. Keep durable decisions, constraints, conventions and the reasons for rejected options.

**Can I export or delete what Vilix AI stores?**
Yes. Export everything or delete it anytime. Vilix AI has a free plan, and the 7-day Pro trial needs no credit card.
