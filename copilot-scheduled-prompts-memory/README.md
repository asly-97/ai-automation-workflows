# Do Copilot Scheduled Prompts Remember Yesterday?

The full technical guide behind the Agentic Pod episode **"Do Copilot Scheduled Prompts Remember Yesterday?"**: what a Microsoft 365 Copilot scheduled prompt keeps from one run to the next, why Copilot Memory is not run history, the three things every run forgets, and how to give every scheduled agent in your stack (n8n, Claude Code, Codex, OpenClaw, Hermes or your own cron job) one shared memory with Vilix AI.

- 🎧 Watch the episode: Agentic Pod (link added after upload)
- 📖 Read the article: [Your Copilot Scheduled Prompt Runs Every Morning. It Does Not Remember Yesterday Morning.](https://vilix.ai/blog/copilot-scheduled-prompts-dont-remember-yesterday/)
- 🧠 Shared memory used here: [Vilix AI](https://vilix.ai) (hosted MCP server: `https://api.vilix.ai/mcp`)
- ⚡ Get started: [vilix.ai/get-started](https://vilix.ai/get-started)

---

## Contents

1. [The short answer](#the-short-answer)
2. [How scheduled prompts work](#how-scheduled-prompts-work)
3. [Copilot Memory vs run history](#copilot-memory-vs-run-history)
4. [Five scheduled-run pains, and how Vilix AI fixes each one](#five-scheduled-run-pains-and-how-vilix-ai-fixes-each-one)
5. [The memory-bearing run loop](#the-memory-bearing-run-loop)
6. [Inside Copilot: amnesia-first prompts and the ledger](#inside-copilot-amnesia-first-prompts-and-the-ledger)
7. [Setup: n8n, Claude Code, Codex, OpenClaw, Hermes and headless agents](#setup)
8. [Verify it: the repeat-item test](#verify-it-the-repeat-item-test)
9. [Copy-paste prompts](#copy-paste-prompts)
10. [Agencies: one Vilix AI account per client](#agencies-one-vilix-ai-account-per-client)
11. [FAQ](#faq)

---

## The short answer

| What a scheduled run keeps | Remembered next run? |
|---|---|
| The saved prompt text | Yes, it is re-run exactly as saved |
| Your preferences (report format, writing style) through Copilot Memory | Yes |
| Patterns Copilot infers from your chats | Yes |
| What yesterday's run found or already reported | No |
| Which alert was a false positive, which summary you acted on | No |

For simple daily digests, write prompts that assume amnesia. For anything that needs to know what the last run did, keep a ledger. For a stack of scheduled agents across several tools, give them one shared memory they read at the start and write at the end. That is what Vilix AI does.

## How scheduled prompts work

Scheduled Prompts is a Microsoft 365 Copilot feature for prompts you run regularly:

1. Run the prompt once in Copilot Chat.
2. Hover over it in your chat history and select **Schedule this prompt**.
3. Set the time and frequency. Each user can have up to **10** scheduled prompts.
4. Every result lands back in your Copilot chat history, with newly run results shown in bold.

Think of it as a cron job for a prompt. What gets saved and re-executed is the prompt text, so every run is a fresh execution of the same static text at the scheduled time.

## Copilot Memory vs run history

Copilot Memory personalizes Copilot. It builds memory two ways:

- **Saved memories:** things you tell it to keep, like your preferred report format.
- **Chat history inferences:** patterns it recognizes from your conversations.

You can review, edit and delete saved memories in the personalization settings. Microsoft is explicit that Copilot Memory is not the same as Copilot conversation history. So your scheduled prompt remembers that you like bullet points; run state, like "I flagged the vendor thread on Monday", is a different category.

## Five scheduled-run pains, and how Vilix AI fixes each one

### 1. Deduplication: the same item every morning

A prompt that summarizes "new emails" keeps no list of what it already summarized. An unresolved item gets flagged Monday and Tuesday, then ages out of the window.

**How Vilix AI fixes it:** the first step of every run is `get_context`. Vilix AI returns what earlier runs already reported, what is still open, and your rules, so the run can say "already told you about this" or skip it. The last step is `save_turn`, which stores the full exchange with a `source` label (for example `n8n: morning inbox digest`) and a timestamp.

### 2. Adaptation: the same logic every run

The saved text never changes, so a run can't say "last run found nothing, so this time look somewhere else."

**How Vilix AI fixes it:** each run saves its result ("nothing in the inbox, checked the shared mailbox instead") with `save_turn`, and the next run's `get_context` brings it back, so the agent picks its next step from what actually happened. When a decision changes, you say it once and the newest memory wins for every agent.

### 3. Outcomes: no record of what happened

Preferences describe how you like things presented. They don't record which alert was a false positive or which summary you acted on.

**How Vilix AI fixes it:** Vilix AI stores full conversation turns, not just distilled facts, so the record of what each run did and why is there when you need it. Failures are saved too, with the exact error and the conclusion, so the next run skips the broken step.

### 4. The ledger needs upkeep

The documented workaround is a ledger file in SharePoint or OneDrive that the prompt reads and updates every run. It only stays accurate if every run keeps it accurate.

**How Vilix AI fixes it:** for agents connected over MCP, Vilix AI is the ledger that keeps itself. `save_turn` at the end of every run writes the entry; `get_context` at the start reads back only what is relevant, searching by meaning and by keyword. It is cloud-hosted, so there is nothing to deploy and no file to tidy. For a targeted lookup, `search_semantic(query, source?)` and `search_keyword(query)` filter by meaning, exact terms, or by the tool that saved it.

### 5. Several scheduled agents, no shared state

Most operators run several scheduled agents: a Copilot prompt for the daily digest, an n8n workflow for enrichment, a coding agent for reports. Each one keeps its own preferences and none share run history.

**How Vilix AI fixes it:** every agent connects to the same Vilix AI account over MCP. Headless agents use an API key as a Bearer token; Claude Code, Codex, ChatGPT and Claude sign in with OAuth. Every saved turn carries its source label, so each agent sees what the others did and when. Standing rules live in one place:

- **User rules** (`add_user_rule`): how you like things done, loaded into every tool.
- **Project rules** (`create_project(..., project_rules)` / `update_project`): senders to ignore, owners, thresholds for one workstream or one client.
- **Skills** (`create_skill`, loaded with `get_skill`): the step-by-step procedure for a recurring job, so every scheduled agent follows the same playbook.
- **Agent inbox** (`message_agents(body, urgency, scope_type)`): a note to a running agent that arrives with its next Vilix AI read.

## The memory-bearing run loop

```
 ┌──────── any scheduled agent: n8n workflow, Claude Code, Codex, OpenClaw, Hermes, cron ────────┐
 │                                                                                                │
 │  1. get_context("<what this run does>")   ──►  Vilix AI returns:                               │
 │                                                  • what earlier runs reported (from ANY tool)  │
 │                                                  • open items and recent decisions (newest wins)│
 │                                                  • user rules, project rules, active tasks     │
 │                                                  • agent inbox notes for this agent            │
 │  2. do the work: skip what was already reported, adapt to what the last run found              │
 │                                                                                                │
 │  3. save_turn(user_message, assistant_message, source="<tool + job name>")                      │
 │        • what was reported, what was decided, what is still open                               │
 │        • failures too: what was tried, the exact error, the conclusion                         │
 └────────────────────────────────────────────────────────────────────────────────────────────────┘
```

## Inside Copilot: amnesia-first prompts and the ledger

Keep Copilot Memory for what it does well, your standing preferences, and design the scheduled prompt itself to stand alone:

- Use **relative time windows** ("in the last 24 hours", "since Monday") instead of context-dependent references ("the email I sent").
- Ask for **structured output** (priority list, action items) that is useful without follow-up questions.
- **Scope the data** ("emails from these three senders", "files in this folder").
- For run-to-run state inside Copilot, keep a **ledger file** in SharePoint or OneDrive and tell the prompt to read it at the start and record what it reported.

When your automation grows past one tool, move the run history for your MCP-connected agents into Vilix AI so every agent wakes up with the same picture.

## Setup

Use the **same Vilix AI account** everywhere. Server URL: `https://api.vilix.ai/mcp`.

**n8n, Make, Zapier and cron jobs:** create a key in Vilix AI under **Agents → Create API key**. Point your platform's MCP client step at `https://api.vilix.ai/mcp` with the header `Authorization: Bearer YOUR_VILIX_API_KEY`. In n8n, attach an MCP client tool to the AI Agent node with that endpoint and header, then add the run loop to the agent's system message.

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

## Verify it: the repeat-item test

1. Schedule an n8n (or cron) agent with the run-loop prompt below, every 10 minutes, on a small job such as "list unanswered emails from the vendor@ address".
2. After the first run, open Vilix AI and confirm a saved turn with your source label listing what it reported.
3. Leave one item unanswered. After the second run, check that it marks that item as "already reported" instead of presenting it as new.
4. Ask Claude Code or Codex: *"What did the morning digest agent report last?"* It answers from the same memory.

## Copy-paste prompts

**Scheduled agent prompt (any MCP-connected tool):**

```
Every run:
1. Call Vilix AI get_context with "<job name> run: what was already reported, what is open, what the last run found, which rules apply".
2. Do the job. Mark anything already reported as "still open" instead of new. If the last run found nothing, try the next source.
3. Report only what is new or changed. If nothing is new, say "nothing new" in one line.
4. Call Vilix AI save_turn with source "<tool>: <job name>". Include: items reported this run, decisions made,
   items still open, where you looked, and any failure with the exact error.
```

**Amnesia-first Copilot scheduled prompt:**

```
Summarize emails received in the last 24 hours from <senders>. Output a priority list with three sections:
Needs a reply today, FYI, Ignore. One line per email: sender, subject, why it is in that section.
Before you start, read <ledger file> and mark items already listed there as "still open". Then add today's items to it.
```

**Set the standing rules once:**

```
In Vilix AI, create a project "<workstream or client name>" with these project rules:
- Ignore newsletters and automated receipts.
- The vendor@ thread is owned by Dana.
- Urgent means a customer is waiting for more than 4 hours.
```

**Correct a decision for every agent:**

```
Save to Vilix AI: the vendor invoice thread is resolved as of today. Newest wins, all scheduled agents should stop reporting it.
```

**Steer a running agent through the inbox:**

```
Send a Vilix AI agent inbox message to my n8n agents: skip the digest on Friday, the team is offline.
```

## Agencies: one Vilix AI account per client

Agencies and automation builders give each client their own Vilix AI account, included in the retainer or paid by the client. Every scheduled agent, n8n flow, support agent and booking agent built for that client connects with that client's API key, so the memory belongs to the client's business. The agency never builds memory from scratch: it wires each new automation into the client's account with `get_context` at the start and `save_turn` at the end.

## FAQ

**Do Microsoft Copilot scheduled prompts remember between runs?**
They keep the prompt text, your formatting preferences and the patterns in your chat history. What yesterday's run found or already reported is run state, which you keep in a ledger inside Copilot, or in Vilix AI for your MCP-connected agents.

**Is Copilot Memory the same as conversation history?**
No. Microsoft describes Copilot Memory as saved preferences and inferred details. It is personalization, not a record of what each run did.

**How many scheduled prompts can I have?**
Up to 10 per user.

**Why does my scheduled prompt flag the same email every day?**
Each run is a fresh execution of the same text with no list of what it already reported, so an open item keeps appearing until it leaves the time window. Track reported items in a ledger, or with `save_turn` and `get_context` in Vilix AI.

**How do several scheduled agents share what they found?**
Connect them all to the same Vilix AI account over MCP. Every run reads with `get_context` and writes with `save_turn`, and every saved turn carries a source label, so each agent sees what the others did.

**Can I export or delete what Vilix AI stores?**
Yes. Your data is portable: export everything or delete it anytime. Vilix AI has a free plan, and the 7-day Pro trial needs no credit card.
