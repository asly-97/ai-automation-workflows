# Do Windmill scheduled AI steps remember between runs?

The full technical guide behind the Agentic Pod episode **"Do Windmill Scheduled AI Steps Remember Between Runs?"**: what Windmill's script state, flow state, AI agent memory and resources each keep, why a scheduled AI step runs stateless, and how to give every scheduled job in your stack (Windmill, n8n, Claude Code, Codex, OpenClaw, Hermes or your own cron job) one shared memory with Vilix AI.

- 🎧 Watch the episode: [Do Windmill Scheduled AI Steps Remember Between Runs? on Agentic Pod](https://youtu.be/2xzrhK6UHKw)
- 📖 Read the article: [Windmill Keeps the State You Save by Hand. Your Scheduled AI Steps Still Run Stateless.](https://vilix.ai/blog/windmill-keeps-the-state-you-save-by-hand-your-scheduled-ai-steps-still-run-stateless/)
- 🧠 Shared memory used here: [Vilix AI](https://vilix.ai) (hosted MCP server: `https://api.vilix.ai/mcp`)
- ⚡ Get started: [vilix.ai/get-started](https://vilix.ai/get-started)

---

## Contents

1. [The short answer](#the-short-answer)
2. [The four kinds of persistence in Windmill](#the-four-kinds-of-persistence-in-windmill)
3. [Four pains, and how Vilix AI fixes each one](#four-pains-and-how-vilix-ai-fixes-each-one)
4. [The memory-bearing run loop](#the-memory-bearing-run-loop)
5. [Setup](#setup)
6. [Verify it: the two-run test](#verify-it-the-two-run-test)
7. [Copy-paste prompts](#copy-paste-prompts)
8. [Agencies: one Vilix AI account per client](#agencies-one-vilix-ai-account-per-client)
9. [FAQ](#faq)

---

## The short answer

| Windmill feature | Lives as long as | Holds |
|---|---|---|
| Script state (get and set state helpers) | Across runs of that script | The small values your code explicitly saves, like a last run time or a sync checkpoint |
| Flow user state | The flow job | A working key-and-value space for the steps of one execution |
| AI agent step memory | A conversation id, if one is passed | Recent messages, or the whole conversation with older parts compressed. Off by default |
| Resources and variables | The workspace | Connections, credentials, thresholds and settings |

Schedules, webhooks, evaluations and plain runs pass no memory id, so an AI agent step in a scheduled flow runs stateless. Windmill is excellent at making a job repeatable. A shared memory outside the job is what makes it remember. That is what Vilix AI does.

## The four kinds of persistence in Windmill

1. **Script state.** The strongest cross-run tool: a script sets a value and reads it on its next execution. It holds exactly what your code places there. The reasoning behind a ranking, an exception a reviewer approved, or a dead end a run explored is not saved unless you designed the script to store it.
2. **Flow user state.** Any step of a flow can read and write it during one execution. When the flow job ends, the space ends, and tomorrow's job starts fresh.
3. **AI agent step memory.** In a chat flow, each message arrives with a conversation identity, so the step knows which history to load. A scheduled run has no such identity, so it runs stateless. You can pin a memory id on the step, fixed for every run or built from an input such as a customer reference. A single fixed id merges every scheduled job into one thread, and per-key ids create silos you manage.
4. **Resources and variables.** Stable facts of a workspace, permissioned by path. They hold settings, not what the last run found.

## Four pains, and how Vilix AI fixes each one

### 1. The scheduled AI step starts blank

A schedule passes no memory id, so the AI step has no history to load, and memory is off by default.

**How Vilix AI fixes it:** the first step of the run is `get_context`. Vilix AI returns what earlier runs and other tools saved, plus your rules, so the step starts with history even though Windmill passed no id. The last step is `save_turn`, which stores the full exchange with a `source` label such as `windmill: morning lead review`.

### 2. Script state is a shelf you stock by hand

Only values your code writes are there, and richer history means building a private memory system one script at a time.

**How Vilix AI fixes it:** `save_turn` stores the full conversation, not a distilled fact, so the reasoning and wording behind a decision are there for the next run. Failures are saved too, with the exact error and the conclusion, so the next run skips the broken step. `search_semantic(query, source?)` and `search_keyword(query)` find a specific earlier run by meaning or exact terms.

### 3. Flow state ends with the job

Inside the run it shares. Across runs it does not.

**How Vilix AI fixes it:** the memory lives in Vilix AI, outside any single job, so a value learned in Monday's run is in Tuesday's `get_context`. When a decision changes, say it once and the newest memory wins for every tool.

### 4. One thread or many silos, and no sharing with other tools

A fixed memory id merges unrelated jobs. Per-key ids split them into silos that live inside one flow. Neither is visible to your other automation tools or to a person chatting with an assistant.

**How Vilix AI fixes it:** use **projects**. Each project holds its own memory, so jobs about different subjects stay apart, while every connected tool (Windmill jobs, n8n, Claude Code, Codex, Cursor, ChatGPT, Claude) shares what is relevant. Standing instructions live in one place:

- **User rules** (`add_user_rule`): how you like things done, loaded into every tool.
- **Project rules** (`create_project(..., project_rules)` / `update_project`): thresholds, owners and tone for one workstream or one client.
- **Skills** (`create_skill`, loaded with `get_skill`): the step-by-step procedure for a recurring job, so every scheduled agent follows the same playbook.
- **Agent inbox** (`message_agents(body, urgency, scope_type)`): a note to a running agent that arrives with its next Vilix AI read.

## The memory-bearing run loop

```
 ┌──────── any scheduled job: Windmill script or flow, n8n, Claude Code, Codex, OpenClaw, Hermes, cron ────────┐
 │                                                                                                           │
 │  1. get_context("<what this run does>")   ──►  Vilix AI returns:                                          │
 │                                                  • what earlier runs saved (from ANY tool)                │
 │                                                  • open items and recent decisions (newest wins)          │
 │                                                  • user rules, project rules, active tasks                │
 │                                                  • agent inbox notes for this agent                       │
 │  2. do the work with that history in view                                                                 │
 │                                                                                                           │
 │  3. save_turn(user_message, assistant_message, source="<tool + job name>")                                │
 │        • what was done, what was decided, what is still open                                              │
 │        • failures too: what was tried, the exact error, the conclusion                                    │
 └───────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

## Setup

Use the **same Vilix AI account** everywhere. Server URL: `https://api.vilix.ai/mcp`.

**Windmill scripts, n8n, Make, Zapier and cron jobs:** create a key in Vilix AI under **Agents → Create API key**. Point any MCP client or HTTP step in your job at `https://api.vilix.ai/mcp` with the header `Authorization: Bearer YOUR_VILIX_API_KEY`, and add the run loop to the AI step's instructions.

**Claude Code:**

```bash
claude mcp add --transport http --scope user vilix https://api.vilix.ai/mcp
```

**Codex:**

```bash
codex mcp add vilix --url https://api.vilix.ai/mcp
codex mcp login vilix
```

**ChatGPT and Claude:** add a custom connector with the server URL and sign in with OAuth.

**OpenClaw, Hermes and other always-on agents:** same API key and Bearer header as above. Full per-tool commands: [`../mcp-memory-server-setup/`](../mcp-memory-server-setup/README.md).

**Make it automatic:** in any connected tool, ask `Set up Vilix AI to load and save context automatically.` It calls `get_setup_instructions` and returns the exact block and where it goes.

## Verify it: the two-run test

1. Schedule a job with the run-loop prompt below, every 10 minutes, on a small task such as "check the open items list and report what changed".
2. After the first run, open Vilix AI and confirm a saved turn with your source label.
3. After the second run, check that it refers to what the first run reported instead of starting blank.
4. Ask Claude Code or Codex: *"What did the Windmill job report last?"* It answers from the same memory.

## Copy-paste prompts

**Scheduled AI step instructions (any MCP-connected tool):**

```
Every run:
1. Call Vilix AI get_context with "<job name> run: what earlier runs reported, what is open, which rules apply".
2. Do the job with that history in view. Mark anything already reported as "still open" instead of new.
3. Report only what is new or changed. If nothing is new, say "nothing new" in one line.
4. Call Vilix AI save_turn with source "windmill: <job name>". Include: what you reported, decisions made,
   items still open, and any failure with the exact error.
```

**Keep different subjects apart:**

```
In Vilix AI, create a project "<customer or workstream name>" with these project rules:
- Report only changes since the last run.
- Owner for escalations: <name>.
- Ignore test records.
```

**Correct a decision for every agent:**

```
Save to Vilix AI: the Acme escalation is resolved as of today. Newest wins, all scheduled jobs should stop reporting it.
```

**Steer a running agent through the inbox:**

```
Send a Vilix AI agent inbox message to my scheduled agents: pause the Friday report, the team is offline.
```

## Agencies: one Vilix AI account per client

Agencies and automation builders give each client their own Vilix AI account, like a Gmail account that belongs to that client. It is included in the retainer or paid by the client. Every Windmill flow, n8n workflow and agent built for that client connects with that client's API key, so the memory belongs to the client's business. The agency never builds memory from scratch: it wires each new automation into the client's account with `get_context` at the start and `save_turn` at the end.

## FAQ

**Do Windmill scheduled AI steps remember between runs?**
By Windmill's documentation, schedules, webhooks, evaluations and plain runs pass no memory id, so they run stateless, and agent memory is off by default. Script state can carry small values you save by hand. For run history, use a shared memory such as Vilix AI.

**What does Windmill script state hold?**
The values your script sets with the state helpers, for example a last run time or a sync checkpoint. It holds what your code explicitly stores.

**Does flow state carry over to tomorrow's run?**
No. Flow user state lives as long as the flow job, and tomorrow's job starts with a fresh state.

**Should I pin one memory id on the agent step?**
A single fixed id merges every scheduled job into one thread. Per-key ids split memory into silos that live inside that flow. A shared memory with projects keeps subjects apart and shares them across tools.

**How do several scheduled agents share what they found?**
Connect them all to the same Vilix AI account over MCP. Every run reads with `get_context` and writes with `save_turn`, and every saved turn carries a source label, so each agent sees what the others did.

**Can I export or delete what Vilix AI stores?**
Yes. Your data is portable: export everything or delete it anytime. Vilix AI has a free plan, and the 7-day Pro trial needs no credit card.
