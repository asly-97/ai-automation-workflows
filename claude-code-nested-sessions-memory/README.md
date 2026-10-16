# Do Claude Code Child Sessions Share Memory?

The full technical guide behind the Agentic Pod episode **"Do Claude Code Child Sessions Share Memory?"**: what the Claude Code desktop update of 14 October 2026 added (nested child sessions, saved environments, Local, Cloud and SSH run modes), what a child session starts with, and how to give the parent, every child, Cursor, Codex and your other tools one shared memory with Vilix AI.

- 🎧 Watch the episode: [Agentic Pod on YouTube](https://youtu.be/pVqxQxrn4aA)
- 📰 Source for the update: [AlphaSignal: Claude Code now runs nested sessions and saves dev environments](https://alphasignal.ai/news/anthropic-s-claude-code-now-runs-nested-sessions-and-saves-dev-environments)
- 🧠 Shared memory used here: [Vilix AI](https://vilix.ai) (hosted MCP server: `https://api.vilix.ai/mcp`)
- ⚡ Get started: [vilix.ai/get-started](https://vilix.ai/get-started)

---

## Contents

1. [The short answer](#the-short-answer)
2. [What the update shipped](#what-the-update-shipped)
3. [The sticky note problem](#the-sticky-note-problem)
4. [Pains and how Vilix AI fixes each one](#pains-and-how-vilix-ai-fixes-each-one)
5. [The parent and child run loop](#the-parent-and-child-run-loop)
6. [Setup](#setup)
7. [Verify it: the two-child test](#verify-it-the-two-child-test)
8. [Copy-paste prompts](#copy-paste-prompts)
9. [Agencies: one Vilix AI account per client](#agencies-one-vilix-ai-account-per-client)
10. [FAQ](#faq)

---

## The short answer

A child session starts with the task its parent hands it. Your decision from Tuesday, the naming rule from last week and the reason you dropped a library are only there if they are written into that task. To give every session the same picture, put the decisions in one memory that every session reads at the start and writes at the end. That is what Vilix AI does, across Claude Code, Cursor, Codex, ChatGPT and the rest.

## What the update shipped

From the 14 October 2026 Claude Code desktop update (rolling out to Pro, Max, Team and Enterprise plans):

| Feature | What it does |
|---|---|
| Nested child sessions | A parent session splits parallel work into child sessions. Each child handles its assigned task and reports the result to the parent when finished. |
| Saved environments | One setup can combine several folders, an SSH host or a cloud target, and an optional worktree per folder. |
| Three run modes | Local (your machine), Cloud (Anthropic-managed, sessions continue after the app closes) and SSH (a remote host). |
| Session dashboard | The sidebar shows which sessions are active or waiting for input, with a live to-do list and turn recap per session. |
| Side questions | The `/btw` command opens a side question in another tab. |

Every active session consumes model usage, so re-explaining the same background to each child is not free.

## The sticky note problem

Think of each child as an intern handed one sticky note. The note holds whatever the parent wrote into the task. Anything else you decided earlier, in this session, another session, or another tool entirely (Cursor, Codex, ChatGPT), is not on the note unless you type it again. The more children you run, the more times you type it.

## Pains and how Vilix AI fixes each one

### 1. Every child starts without your decisions

**How Vilix AI fixes it:** the task given to each child starts with "call Vilix AI `get_context` first". Vilix AI returns your recent decisions, rules, open items and what earlier sessions already tried, so the sticky note only needs to say what to do, not everything that is true.

### 2. Nobody knows which child decided what

**How Vilix AI fixes it:** each child ends with `save_turn(user_message, assistant_message, source)`. Vilix AI stores the full exchange with a source label such as `claude-code child: auth refactor`, so you and the parent can see which session, in which tool, decided what. `search_semantic(query, source?)` and `search_keyword(query)` filter by meaning, exact terms, or the tool that saved it.

### 3. The next child walks into the same wall

**How Vilix AI fixes it:** failures are saved too, with what was tried, the exact error and the conclusion. The next child's `get_context` brings that back, so it skips the broken approach.

### 4. You change your mind halfway through

**How Vilix AI fixes it:** say it once. In Vilix AI the newest memory wins, so every child that starts after the change already has the new decision, and so does every other tool you use.

### 5. Standing rules re-typed into every task

**How Vilix AI fixes it:** write them once.

- **User rules** (`add_user_rule`): how you like to work (for example, always use pnpm), loaded into every tool.
- **Project rules** (`create_project(..., project_rules)` / `update_project`): the facts for one repo or client (for example, never touch the billing folder).
- **Skills** (`create_skill`, loaded with `get_skill`): the step-by-step procedure for a recurring job, so every child follows the same playbook.
- **Agent inbox** (`message_agents(body, urgency, scope_type)`): a note to a running agent that arrives with its next Vilix AI read.

### 6. Plans that must outlive the session

Cloud mode keeps sessions running after you close the app, but your decisions still need a home that outlasts any one session. **How Vilix AI fixes it:** the plan for the next session sits in the same memory as the decisions, so any tool can pick it up with `get_context`.

## The parent and child run loop

```
 ┌──────── parent session and every child session (Claude Code, Cursor, Codex, ...) ────────┐
 │                                                                                           │
 │  1. get_context("<what this session does>")  ──►  Vilix AI returns:                      │
 │                                                    • recent decisions (newest wins)       │
 │                                                    • user rules, project rules            │
 │                                                    • what other sessions already tried    │
 │                                                    • agent inbox notes for this agent     │
 │  2. do the task                                                                           │
 │                                                                                           │
 │  3. save_turn(user_message, assistant_message, source="<tool + task>")                    │
 │        • what was decided, what changed, what is still open                               │
 │        • failures too: what was tried, the exact error, the conclusion                    │
 └───────────────────────────────────────────────────────────────────────────────────────────┘
```

## Setup

Use the **same Vilix AI account** in every tool. Server URL: `https://api.vilix.ai/mcp`.

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

**Cursor, OpenClaw, Hermes and headless agents:** create a key in Vilix AI under **Agents → Create API key** and use the header `Authorization: Bearer YOUR_VILIX_API_KEY`. Full per-tool commands: [`../mcp-memory-server-setup/`](../mcp-memory-server-setup/README.md).

**Make it automatic:** in any connected tool, ask `Set up Vilix AI to load and save context automatically.` It calls `get_setup_instructions` and returns the exact block and where it goes.

## Verify it: the two-child test

1. In Claude Code, tell the parent session: "Save to Vilix AI: we use pnpm, never npm, in this repo."
2. Ask the parent to start two child sessions, each told only: "Call Vilix AI get_context first, then add a script to package.json and install a dev dependency. Save what you did with save_turn."
3. Check that both children used pnpm without being told in the task.
4. Open Cursor or Codex and ask: "What did the child sessions do in this repo?" It answers from the same memory.

## Copy-paste prompts

**Task opener for every child session:**

```
Start by calling Vilix AI get_context with "<repo or project>: decisions, rules and what earlier sessions tried for <this task>".
Do the task. When you finish, call Vilix AI save_turn with source "claude-code child: <task name>".
Include what you changed, what you decided, what is still open, and any failure with the exact error.
```

**Set the standing rules once:**

```
In Vilix AI, create a project "<repo or client name>" with these project rules:
- Use pnpm, never npm.
- Never touch the billing folder.
- Every pull request needs a test.
```

**Change a decision for every session:**

```
Save to Vilix AI: we dropped <library> as of today. Newest wins, every session should stop using it.
```

**Steer a running agent through the inbox:**

```
Send a Vilix AI agent inbox message to my Claude Code agents: pause the migration, review the schema first.
```

## Agencies: one Vilix AI account per client

Agencies and automation builders give each client their own Vilix AI account, included in the retainer or paid by the client. Every parent and child session built for that client uses that client's API key, so the memory belongs to the client's project. The agency never builds memory from scratch: each new session is wired to the client's account with `get_context` at the start and `save_turn` at the end.

## FAQ

**Do Claude Code child sessions share memory?**
Each child starts with the task it was handed by the parent. To share decisions, rules and history between the parent, every child and your other tools, keep them in one memory such as Vilix AI and have every session call `get_context` first and `save_turn` last.

**What are Claude Code nested sessions?**
A parent session can split parallel work into child sessions. Each child handles its assigned task and reports the result back to the parent when finished.

**Which run modes are there?**
Local, Cloud and SSH. Cloud sessions continue after you close the app.

**Which plans get the update?**
It is rolling out through a desktop app update for Pro, Max, Team and Enterprise plans.

**Do child sessions use more usage?**
Every active session consumes model usage, so the practical number of children depends on your plan limits, machine resources and how much you can review.

**Does this work with Cursor, Codex and ChatGPT?**
Yes. They connect to the same Vilix AI account over MCP, so a decision saved from one tool is available in all of them.

**Can I export or delete what Vilix AI stores?**
Yes. Your data is portable: export everything or delete it anytime. Vilix AI has a free plan, and the 7-day Pro trial needs no credit card.
