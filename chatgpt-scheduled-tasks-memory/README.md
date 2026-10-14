# Do ChatGPT Scheduled Tasks Remember Between Runs?

The full technical guide behind the Agentic Pod episode **"Do ChatGPT Scheduled Tasks Remember Between Runs?"**: what each kind of ChatGPT scheduled task keeps from one run to the next, why your morning briefing repeats itself, and how to give every scheduled run (in ChatGPT, n8n, Zapier, Make or your own cron job) one shared memory with Vilix AI.

- 🎧 Watch the episode: VIDEO_LINK_PENDING
- 📖 Read the article: [Do ChatGPT Scheduled Tasks Remember Between Runs? What Actually Persists](https://vilix.ai/blog/do-chatgpt-scheduled-tasks-remember-between-runs-what-actually-persists-and-what-you-still-lose)
- 🧠 Shared memory used here: [Vilix AI](https://vilix.ai) (hosted MCP server: `https://api.vilix.ai/mcp`)
- ⚡ Get started: [vilix.ai/get-started](https://vilix.ai/get-started)

---

## Contents

1. [The short answer](#the-short-answer)
2. [The three kinds of scheduled tasks](#the-three-kinds-of-scheduled-tasks)
3. [Five scheduled-task pains, and how Vilix AI fixes each one](#five-scheduled-task-pains-and-how-vilix-ai-fixes-each-one)
4. [The memory-bearing run loop](#the-memory-bearing-run-loop)
5. [What to store: three kinds of run state](#what-to-store-three-kinds-of-run-state)
6. [Setup: ChatGPT, n8n, Zapier, Make, Claude Code, Codex and headless agents](#setup)
7. [Verify it: the two-run test](#verify-it-the-two-run-test)
8. [Copy-paste prompts](#copy-paste-prompts)
9. [Agencies: one Vilix AI account per client](#agencies-one-vilix-ai-account-per-client)
10. [FAQ](#faq)

---

## The short answer

| Task type | What happens each run | Remembers the last run? |
|---|---|---|
| **Standalone** (the default from the prompt bar) | Opens a fresh chat, runs the saved prompt, reports into **Scheduled**, OpenAI's inbox for task results | No, every run starts from zero |
| **In-chat** (scheduled inside an existing chat) | Returns to the same chat and reuses its context | Yes, through the chat thread |
| **Event-triggered** (Gmail, Slack, GitHub events) | The trigger decides when, the saved prompt decides what | No, stateless by design |

If all your automation lives in one ChatGPT chat, in-chat mode is enough to stop starting over. If your workflow also runs in n8n, Zapier, Make or a cron job, give every run one shared memory it reads at the start and writes at the end. That is what Vilix AI does.

## The three kinds of scheduled tasks

**Standalone tasks.** OpenAI describes them as starting "a new chat for each scheduled run" and recommends them "when each run should be independent": a weekly report, a nightly check, a monthly summary. Results land in the Scheduled hub, which also handles one-off, recurring and monitoring tasks and looser time windows like "morning".

**In-chat tasks.** "Schedule a task inside an existing chat when you want ChatGPT to return to that chat on a schedule." The task uses the chat's existing context instead of a fresh prompt, which suits ongoing research or triage. What persists is the conversation thread: prior messages, prior findings, the back-and-forth.

**Event-triggered tasks.** They fire on Gmail, Slack or GitHub events. Several matching events arriving close together may be combined into one run.

**Where the task runs** also decides what it can see: web tasks can use connected tools but not local files; desktop tasks can reach local files while the machine stays on and the app keeps running.

## Five scheduled-task pains, and how Vilix AI fixes each one

### 1. Every standalone run starts from zero

The morning briefing reports the same news three days in a row because nothing tells run four what run three concluded.

**How Vilix AI fixes it:** the first step of every run is `get_context`. Vilix AI returns what is relevant from every earlier run: what was already handled, what is still open, and your rules. The last step is `save_turn`, which stores the full exchange with a `source` label (for example `ChatGPT scheduled task`) and a timestamp. Run four starts where run three stopped.

### 2. The same finding gets reported again and again

Without a record of what was handled, every run treats old findings as new.

**How Vilix AI fixes it:** each run saves an "already handled" entry (the alert triaged, the ticket answered, the finding reported) with `save_turn`. Failed attempts are saved too, with the exact error and the conclusion, so the next run skips the broken step instead of repeating it. When something changes ("that alert is fixed, stop reporting it"), you say it once and the newest memory wins for every agent.

### 3. A long in-chat thread buries the useful facts

After a month of daily runs, an in-chat thread holds thirty "nothing to report" turns for every one that matters.

**How Vilix AI fixes it:** `get_context` searches by meaning and by keyword across everything saved, then returns only what relates to this run. The one decision from two weeks ago comes back; the routine "all quiet" turns stay out of the way. For a targeted lookup, `search_semantic(query, source?)` and `search_keyword(query)` filter by meaning, exact terms, or by the tool that saved it.

### 4. ChatGPT can't hand its state to n8n, Zapier or Make

The scheduled task remembers what it said last Tuesday, but the n8n node that fires thirty seconds later can't see it, so n8n re-sends the alert ChatGPT already dismissed.

**How Vilix AI fixes it:** every tool connects to the same Vilix AI account over MCP. ChatGPT signs in with OAuth; n8n, Zapier, Make, cron jobs and always-on agents use a Vilix AI API key. They all read and write one memory, and every saved turn carries its source label, so the n8n run sees exactly what the ChatGPT task decided and when. To steer a running agent directly, `message_agents(body, urgency, scope_type)` posts to the **agent inbox**, and the note arrives with that agent's next Vilix AI read.

### 5. Standing decisions get pasted into every prompt

Which alerts to ignore, which repos are frozen, who owns which service, what counts as urgent. Edit the prompt in one tool and the others drift.

**How Vilix AI fixes it:** write them once.
- **Project rules** (`create_project(..., project_rules)` / `update_project`): frozen repos, owners, thresholds for one workflow or one client.
- **User rules** (`add_user_rule`): how you like things done, loaded into every tool.
- **Skills** (`create_skill`, loaded with `get_skill`): the step-by-step procedure for a recurring job, so every scheduled run in every tool follows the same playbook.
- **Tasks** (`create_task`, `update_task_state`): todo, doing, blocked, done, visible to every run.

## The memory-bearing run loop

```
 ┌──────── any scheduled run: ChatGPT task, n8n workflow, Zapier zap, Make scenario, cron ────────┐
 │                                                                                                 │
 │  1. get_context("<what this run does>")   ──►  Vilix AI returns:                                │
 │                                                  • what earlier runs handled (from ANY tool)    │
 │                                                  • open items and recent decisions (newest wins)│
 │                                                  • user rules, project rules, active tasks      │
 │                                                  • agent inbox notes for this agent             │
 │  2. do the work, skipping anything already handled                                              │
 │                                                                                                 │
 │  3. save_turn(user_message, assistant_message, source="<tool + job name>")                       │
 │        • what was handled, what was decided, what is still open                                 │
 │        • failures too: what was tried, the exact error, the conclusion                          │
 └─────────────────────────────────────────────────────────────────────────────────────────────────┘
```

Vilix AI is cloud-hosted: there is no database to babysit and nothing asleep when the cron fires. It stores full conversation turns, not just extracted facts, so the reason behind a decision survives.

## What to store: three kinds of run state

| State | Examples | Why it matters |
|---|---|---|
| **Already handled** | Alert #4812 triaged, ticket answered, finding reported on Oct 12 | Stops run four repeating run two |
| **Standing decisions** | Ignore staging alerts, `billing-api` is frozen, Dana owns payments | Keeps every tool on the same rules without editing prompts |
| **Pointers outward** | The dashboard URL, the runbook, the tracker | Things no task can find in its own thread |

Put standing decisions in project rules; put handled items and pointers in each run's `save_turn`.

## Setup

Use the **same Vilix AI account** everywhere. Server URL: `https://api.vilix.ai/mcp`.

**ChatGPT (where the scheduled task lives):** Settings → Security and login → **Developer mode**. Then Plugins → **+**, name **Vilix AI**, paste the URL, choose **OAuth**, sign in and approve. Create or open the chat that holds your task, enable **Vilix AI** under **+ → Developer mode**, and put the run loop in the task prompt (see [prompts](#copy-paste-prompts)).

**n8n, Zapier, Make and cron jobs:** create a key in Vilix AI under **Agents → Create API key**. Point your platform's MCP client step at `https://api.vilix.ai/mcp` with the header `Authorization: Bearer YOUR_VILIX_API_KEY`. In n8n, attach an MCP client tool to the AI Agent node with that endpoint and header, then add the run loop to the agent's system message.

**Claude Code:**

```bash
claude mcp add --transport http --scope user vilix https://api.vilix.ai/mcp
```

**Codex:**

```bash
codex mcp add vilix --url https://api.vilix.ai/mcp
codex mcp login vilix
```

**OpenClaw, Hermes and other always-on agents:** same API key and Bearer header as above. Full per-tool commands: [`../mcp-memory-server-setup/`](../mcp-memory-server-setup/README.md).

**Make it automatic:** in any connected tool, ask `Set up Vilix AI to load and save context automatically.` It calls `get_setup_instructions` and returns the exact block and where it goes.

## Verify it: the two-run test

1. Schedule a standalone ChatGPT task with the run-loop prompt below, every 10 minutes, on any small job (for example "check this RSS feed for new posts about MCP").
2. After the first run, open Vilix AI and confirm a saved turn with source `ChatGPT scheduled task` listing what it reported.
3. After the second run, check that it reported only new items and said which ones it skipped as already handled.
4. Trigger your n8n workflow and ask its agent: *"What did the ChatGPT scheduled task report last?"* It answers from the same memory.

## Copy-paste prompts

**Scheduled task prompt (any tool):**

```
Every run:
1. Call Vilix AI get_context with "<job name> run: what was already handled, what is open, which rules apply".
2. Do the job. Skip anything already handled. Follow the project rules.
3. Only report what is new or changed. If nothing is new, say "nothing new" in one line.
4. Call Vilix AI save_turn with source "<tool> scheduled task: <job name>". Include: items handled this run,
   decisions made, items still open, and any failure with the exact error.
5. If the job is finished for good, say so and stop asking for more runs.
```

**Set the standing rules once:**

```
In Vilix AI, create a project "<workflow or client name>" with these project rules:
- Ignore alerts from staging.
- billing-api is frozen until further notice.
- Urgent means customer-facing and unresolved for more than 30 minutes.
```

**Correct a decision for every agent:**

```
Save to Vilix AI: billing-api is no longer frozen as of today. Newest wins, all scheduled runs should treat it as normal.
```

**Steer a running agent through the inbox:**

```
Send a Vilix AI agent inbox message to my n8n agents: skip the weekly report this Friday, the team is offline.
```

## Agencies: one Vilix AI account per client

Agencies and automation builders give each client their own Vilix AI account, included in the retainer or paid by the client. Every scheduled task, n8n flow, support agent and booking agent built for that client connects with that client's API key, so the memory belongs to the client's business. The agency never builds memory from scratch: it wires each new automation into the client's account with `get_context` at the start and `save_turn` at the end.

## FAQ

**Do ChatGPT scheduled tasks remember between runs?**
Standalone tasks don't; each run starts a new chat. In-chat tasks reuse their chat's context. Event-triggered tasks carry no state. With Vilix AI, every run reads earlier runs with `get_context` and writes back with `save_turn`, whichever type you use.

**Which mode do I get by default?**
Tasks created from the prompt bar are standalone.

**Why does my scheduled task repeat the same finding?**
Each standalone run is a fresh chat with no record of what earlier runs concluded. Save an "already handled" entry with `save_turn` and have each run call `get_context` first.

**Can a ChatGPT scheduled task share state with n8n or Zapier?**
Yes, when both connect to the same Vilix AI account: ChatGPT with OAuth, n8n and Zapier with an API key. Every saved turn carries a source label, so each tool sees which run saved what.

**Is ChatGPT Memory the same as task state?**
No. ChatGPT Memory is personalization: preferences and general context. Task state is exact: what was done, what was decided, what counts as new. Vilix AI keeps both as full conversation turns, searchable from every tool.

**How do I keep the memory clean over months of daily runs?**
`get_context` returns only what is relevant to the current run, newest entries win when facts change, and standing decisions live in project rules instead of the run history.

**What does Vilix AI cost?**
There is a free plan, and a 7-day Pro trial with no credit card. You can export or delete everything anytime. [vilix.ai/get-started](https://vilix.ai/get-started)
