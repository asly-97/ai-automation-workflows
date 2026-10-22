# Does an Agentforce agent remember between scheduled runs?

The full technical guide behind the Agentic Pod episode **"Does Agentforce Remember Between Scheduled Runs?"**: what Agentforce keeps inside one session, why a scheduled run starts blank, what Salesforce's long-horizon runtime changes, and how to give an Agentforce agent (and every other scheduled agent in your stack: n8n, Claude Code, Codex, OpenClaw, Hermes or a cron job) one shared memory with Vilix AI.

**Watch the episode:** https://youtu.be/ekL-Ax_Mcf0

- 📖 Read the article: [Your Agentforce Agent Remembers the Session. It Does Not Remember Last Night's Run.](https://vilix.ai/blog/agentforce-remembers-the-session-not-last-nights-run)
- 🧠 Shared memory used here: [Vilix AI](https://vilix.ai) (hosted MCP server: `https://api.vilix.ai/mcp`)
- ⚡ Get started: [vilix.ai/get-started](https://vilix.ai/get-started)

---

## The short answer

| What | Lives as long as | Holds |
|---|---|---|
| Context variables | One messaging session | Pre-chat form data, Messaging Session fields, carried-over session info. Set when the session starts and fixed during it |
| Custom variables | One session | Action outputs: whether the customer authenticated, intermediate results, flags that unlock topics |
| CRM and Data Cloud | Permanently | The data the agent reads: accounts, cases, orders. A data source, not the agent's own memory |

Every scheduled execution is a fresh session, so the agent can read the CRM again but starts without what its last run worked out. Salesforce has announced a long-horizon runtime (memory across sessions, durable execution, dynamic steering) with Hunter, its outbound sales agent, first. It is in pilot, with general availability expected in November 2026.

## Three pains, and how Vilix AI fixes each one

### 1. The scheduled run wakes up blank

Session state ends with the session. Tuesday's insight (deals that mention one competitor stall after the second call) is not in a variable, the chat or the CRM.

**How Vilix AI fixes it:** the first step of the run is `get_context`. Vilix AI returns what earlier runs and other tools saved, plus your rules, so Wednesday's run starts with Tuesday's lesson. The last step is `save_turn`, which stores the full exchange with a `source` label such as `agentforce: nightly pipeline review`.

### 2. The CRM is not the agent's memory

The record says three cases are open. It does not say that last night's run already investigated them, merged two duplicates and escalated the third. So tonight's run pays token cost to re-derive the same conclusions.

**How Vilix AI fixes it:** `save_turn` stores the working knowledge and the full conversation, not just extracted facts, so a run can revisit what was actually decided. Failed attempts are saved with the exact error, so the next run skips the dead end. When a conclusion changes, say it once: the newest memory wins for every tool. `search_semantic(query, source?)` and `search_keyword(query)` find a specific earlier run.

### 3. Waiting for a roadmap, and one platform only

A runtime in pilot is a roadmap bet, and it covers the agents Salesforce puts on it.

**How Vilix AI fixes it:** the memory lives outside any single platform. The same Vilix AI account serves Agentforce, n8n, Claude Code, Codex and your phone apps over MCP, so what one agent learns is available to the others. Standing instructions live in one place:

- **User rules** (`add_user_rule`): how you like things done, loaded into every tool.
- **Project rules** (`create_project(..., project_rules)`): thresholds, owners and tone for one workstream or client.
- **Skills** (`create_skill`, loaded with `get_skill`): the step-by-step procedure for a recurring job.
- **Agent inbox** (`message_agents`): a note to a running agent that arrives with its next Vilix AI read.

## The memory-bearing run loop

```
 1. get_context("<what this run does>")   ->  what earlier runs saved, open items, newest decisions, rules
 2. do the work with that history in view
 3. save_turn(user_message, assistant_message, source="<tool + job name>")
       what was done, what was decided, what is still open, and any failure with the exact error
```

## Setup

Use the same Vilix AI account everywhere. Server URL: `https://api.vilix.ai/mcp`.

- **Scheduled jobs, n8n, Make, Zapier, cron:** create a key in Vilix AI under **Agents, Create API key**, then point any MCP client or HTTP step at `https://api.vilix.ai/mcp` with the header `Authorization: Bearer YOUR_VILIX_API_KEY`, and add the run loop to the agent's instructions.
- **Claude Code:** `claude mcp add --transport http --scope user vilix https://api.vilix.ai/mcp`
- **Codex:** `codex mcp add vilix --url https://api.vilix.ai/mcp` then `codex mcp login vilix`
- **ChatGPT and Claude:** add a custom connector with the server URL and sign in with OAuth.
- **Make it automatic:** in any connected tool, ask `Set up Vilix AI to load and save context automatically.`

More per-tool commands: [`../mcp-memory-server-setup/`](../mcp-memory-server-setup/README.md).

## Verify it: the two-run test

1. Schedule a job with the prompt below, every 10 minutes, on a small task such as "review the open items and report what changed".
2. After the first run, open Vilix AI and confirm a saved turn with your source label.
3. After the second run, check that it refers to what the first run reported instead of starting blank.
4. Ask Claude Code or Codex: "What did the nightly job report last?" It answers from the same memory.

## Copy-paste prompts

**Scheduled agent instructions (any MCP-connected tool):**

```
Every run:
1. Call Vilix AI get_context with "<job name> run: what earlier runs reported, what is open, which rules apply".
2. Do the job with that history in view. Mark anything already reported as "still open" instead of new.
3. Report only what is new or changed. If nothing is new, say "nothing new" in one line.
4. Call Vilix AI save_turn with source "<tool>: <job name>". Include: what you reported, decisions made,
   items still open, and any failure with the exact error.
```

**Save a pattern for every agent:**

```
Save to Vilix AI: deals that mention <competitor> stall after the second call. Flag them first in every pipeline review.
```

**Steer a running agent through the inbox:**

```
Send a Vilix AI agent inbox message to my scheduled agents: pause the Friday report, the team is offline.
```

## Agencies: one Vilix AI account per client

Agencies and automation builders give each client their own Vilix AI account, like a Gmail account that belongs to that client. It is included in the retainer or paid by the client. Every agent and workflow built for that client connects with that client's API key, so the memory belongs to the client's business. The agency never builds memory from scratch: it wires each new automation into the client's account with `get_context` at the start and `save_turn` at the end.

## FAQ

**Does an Agentforce agent remember between sessions?**
Per Salesforce's documentation, context variables and custom variables are session state. They are set during a session and are gone when it ends. Agentforce reads CRM and Data Cloud data, but that is a data source, not a record of what a previous run concluded.

**Is the CRM the agent's memory?**
No. It holds deal and case facts. It does not hold the agent's judgment from earlier runs.

**What is the long-horizon runtime?**
A Salesforce runtime for work that spans days and weeks, with memory across sessions, durable execution and dynamic steering. Hunter is the first agent on it. It is in pilot, with general availability expected in November 2026.

**Can I use Vilix AI with agents that are not Agentforce?**
Yes. Connect n8n, Claude Code, Codex, Cursor, ChatGPT, Claude, OpenClaw, Hermes or any MCP client to the same account and they share one memory.

**Can I export or delete what Vilix AI stores?**
Yes. Export everything or delete it anytime. Vilix AI has a free plan, and the 7-day Pro trial needs no credit card.
