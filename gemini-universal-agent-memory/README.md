# Does Google's Gemini universal agent remember between tasks?

The full technical guide behind the Agentic Pod episode **"Does Google's New Gemini Agent Remember Your Work?"**: what Google announced for its Gemini workplace agent at Gemini at Work 2026 (8 October 2026), what "you never have to re-brief it" covers, and how to give that agent, ChatGPT, Claude, Claude Code, Codex and Cursor one shared memory with Vilix AI.

- 🎧 Watch the episode: [Agentic Pod on YouTube](https://www.youtube.com/@agenticpod) (the episode link is added here once the upload is live)
- 📰 Source for the announcement: [CBS News: Google launches Gemini AI workplace agent that can write code and run tasks](https://www.cbsnews.com/news/google-gemini-ai-workplace-agent/)
- 🧠 Shared memory used here: [Vilix AI](https://vilix.ai) (hosted MCP server: `https://api.vilix.ai/mcp`)
- ⚡ Get started: [vilix.ai/get-started](https://vilix.ai/get-started)

---

## Contents

1. [The short answer](#the-short-answer)
2. [What Google announced](#what-google-announced)
3. [What "never re-brief" covers](#what-never-re-brief-covers)
4. [Pains and how Vilix AI fixes each one](#pains-and-how-vilix-ai-fixes-each-one)
5. [The run loop](#the-run-loop)
6. [Setup](#setup)
7. [Verify it: the two-tool test](#verify-it-the-two-tool-test)
8. [Copy-paste prompts](#copy-paste-prompts)
9. [Agencies: one Vilix AI account per client](#agencies-one-vilix-ai-account-per-client)
10. [FAQ](#faq)

---

## The short answer

Google says its new Gemini agent builds memory and skills from the tasks and questions it gets, so inside that agent you do not re-brief it. Most people also work in ChatGPT, Claude, Claude Code, Codex and Cursor in the same week. To give all of them, and Gemini, the same decisions, rules and history, keep them in one memory that every tool reads at the start of a task and writes at the end. That is what Vilix AI does.

## What Google announced

From CBS News coverage of the Gemini at Work 2026 keynote by Thomas Kurian, CEO of Google Cloud (8 October 2026):

| Feature | What it does |
|---|---|
| Universal agent | You delegate a task and it returns a finished product; it can act as a personal assistant or a team member such as a project manager or finance analyst |
| Code and media | Writes and runs code, and creates images and media |
| Connected apps | Gmail, Docs, Drive and Sheets, plus Slack and Microsoft 365 |
| Job-specific agents | Creates temporary agents, each with its own identity, that work together on a goal over several days |
| Cloud and memory | Runs in the cloud and builds memory and skills from the tasks and questions it receives |

Kurian's line: "You never have to re-brief it or wonder which machine you told it to do something on."

## What "never re-brief" covers

The memory described belongs to that agent: it learns from what you do with it. Your decisions in Cursor, your rules in Claude Code and your plan in ChatGPT sit in other tools. One shared layer is what lets every tool, Gemini included, start from the same picture.

## Pains and how Vilix AI fixes each one

### 1. Each tool has its own memory, so you re-brief the others

**How Vilix AI fixes it:** every tool calls `get_context` at the start of a task and `save_turn` at the end, with a `source` label such as `cursor: billing refactor`. Vilix AI stores full conversations, not a summary of facts, so the next tool starts where the last one stopped.

### 2. You cannot tell which tool decided what

**How Vilix AI fixes it:** `search_semantic(query, source?)` finds by meaning, `search_keyword(query)` by exact words, and `source` narrows to the tool that saved it.

### 3. Two tools disagree

**How Vilix AI fixes it:** the newest memory wins. Say once that a decision changed and every tool that calls `get_context` afterwards sees the change.

### 4. The next tool repeats a failed approach

**How Vilix AI fixes it:** agents save failures with what was tried, the exact error and the conclusion. The next tool's `get_context` brings it back.

### 5. Several agents on one job need the same picture

**How Vilix AI fixes it:** create a project for the job and put the rules in it.

- **Project rules** (`create_project(..., project_rules)` / `update_project`): the facts and limits for this job, for example "never delete files" or "reports use this format".
- **User rules** (`add_user_rule`): how you like to work, loaded into every tool.
- **Skills** (`create_skill`, loaded with `get_skill`): the playbook for a recurring job.
- **Agent inbox** (`message_agents(body, urgency, scope_type)`): a note to a running agent that arrives with its next Vilix AI read, so you can steer mid-job.

## The run loop

```
 ┌──────── every tool and every agent on the job (Gemini, ChatGPT, Claude, Cursor, ...) ────────┐
 │                                                                                               │
 │  1. get_context("<what this agent does>")  ──►  Vilix AI returns:                            │
 │                                                  • recent decisions (newest wins)             │
 │                                                  • user rules, project rules                  │
 │                                                  • what other agents already tried            │
 │                                                  • agent inbox notes                          │
 │  2. do the task                                                                               │
 │                                                                                               │
 │  3. save_turn(user_message, assistant_message, source="<tool + task>")                        │
 │        • what was decided, what changed, what is still open                                  │
 │        • failures too: what was tried, the exact error, the conclusion                        │
 └───────────────────────────────────────────────────────────────────────────────────────────────┘
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

**Cursor, Gemini CLI, Gemini Spark, OpenClaw, Hermes and headless agents:** create a key in Vilix AI under **Agents → Create API key** and use the header `Authorization: Bearer YOUR_VILIX_API_KEY`. Full per-tool commands: [`../mcp-memory-server-setup/`](../mcp-memory-server-setup/README.md).

**Make it automatic:** in any connected tool, ask `Set up Vilix AI to load and save context automatically.` It calls `get_setup_instructions` and returns the exact block and where it goes.

## Verify it: the two-tool test

1. In Claude Code, say: "Save to Vilix AI: reports for this project use the weekly format, one page."
2. Open Cursor or Gemini CLI and ask: "What format do reports use here? Check Vilix AI first."
3. It answers from the same memory without you retyping anything.
4. Change the decision in Cursor and ask Claude Code again: the newest decision wins.

## Copy-paste prompts

**Task opener for any agent on the job:**

```
Start by calling Vilix AI get_context with "<project>: decisions, rules and what other agents already tried for <this task>".
Do the task. When you finish, call Vilix AI save_turn with source "<tool>: <task name>".
Include what you changed, what you decided, what is still open, and any failure with the exact error.
```

**Set the job rules once:**

```
In Vilix AI, create a project "<job name>" with these project rules:
- Never delete files.
- Reports use the weekly one-page format.
```

**Steer a running agent through the inbox:**

```
Send a Vilix AI agent inbox message to my agents: pause the export, review the report format first.
```

## Agencies: one Vilix AI account per client

Agencies and automation builders give each client their own Vilix AI account, included in the retainer or paid by the client. Every agent built for that client uses that client's API key, so the memory belongs to the client's business. The agency never builds memory from scratch: each new agent is wired to the client's account with `get_context` at the start and `save_turn` at the end.

## FAQ

**Does Google's Gemini agent remember between tasks?**
Google says it builds memory and skills from the tasks and questions it receives. To share decisions, rules and history between Gemini and your other tools, keep them in one memory such as Vilix AI and have every tool call `get_context` first and `save_turn` last.

**What did Google announce?**
A universal workplace agent at Gemini at Work 2026 that writes and runs code, creates media, connects to Gmail, Docs, Drive, Sheets, Slack and Microsoft 365, and can create temporary job-specific agents that work together over several days.

**Does this work with ChatGPT, Claude, Cursor and Codex?**
Yes. They connect to the same Vilix AI account over MCP, so a decision saved from one tool is available in all of them.

**Can Gemini Spark and Gemini CLI use Vilix AI?**
Yes. Connect them with an API key as a Bearer token.

**Can I export or delete what Vilix AI stores?**
Yes. Your data is portable: export everything or delete it anytime. Vilix AI has a free plan, and the 7-day Pro trial needs no credit card.
