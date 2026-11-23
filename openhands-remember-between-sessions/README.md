# Does OpenHands remember between sessions?

The practical guide behind the Agentic Pod episode **"Does OpenHands Remember Between Sessions? Vilix AI Remembers Your Nightly Runs"**.

Watch: https://youtu.be/XZKz_zvGqo8

Read the full post: https://vilix.ai/blog/openhands-remembers-sessions-nightly-runs-wake-up-blank/

## The problem

You run an OpenHands agent every night at 2 AM. It sweeps the repo, triages the open pull requests, and leaves a summary of what needs your eyes in the morning. The first week is great. By week three, the nightly runs start re-flagging PRs you already reviewed, re-discovering the same flaky test, and re-asking which services are safe to touch, even though the agent resolved all of that last month.

So does OpenHands remember between sessions? The honest answer is: it remembers what it wrote down, where you told it to write it, on the machine you run it on. Everything else wakes up blank.

## What actually persists in OpenHands

There are two separate memory stories in OpenHands, and they answer different questions.

**Story one: the app's session data.** In the standard OpenHands setup, session data lives in memory. If the service restarts, previous sessions become invalid, because a new secret is generated and the old session records can no longer be recovered. The UI usually offers to resume a session, but after a restart the resume often lands you in a fresh chat anyway. The documented fix is in `config.toml`: set `file_store="local"` with an absolute `file_store_path`, and pin a fixed `jwt_secret` so session identifiers stay valid across restarts. With that configuration, sessions persist on disk. Without it, a container restart, a redeploy, or a machine reboot wipes the session list clean.

**Story two: the SDK's persistent memory.** The newer OpenHands SDK has an opt-in persistent memory system built around two `MEMORY.md` files. Pass `AgentContext(load_memory=True)` and the conversation loads memory indexes from two tiers: a user tier at `~/.openhands/memory/` and a project tier at `<workspace>/.openhands/memory/`. At session start, the contents are injected into the system prompt as a `<MEMORY_CONTEXT>` block, and the system prompt instructs the agent to maintain those files as it works: record durable, broadly useful facts in `MEMORY.md` near the end of a task, merge duplicates, prune stale entries, keep the indexes concise, never record secrets.

That is genuinely useful for a developer who opens OpenHands on the same machine every day. But it is not shared across machines, it is not visible to any other tool, and it only contains what the agent itself chose to write down.

## The scheduled-run problem

Each scheduled run is a brand-new session. The `file_store` trick helps the UI keep sessions across a restart, but it does not hand the new night's agent the decisions the last night's agent made. The SDK memory loads `MEMORY.md` from local disk, which works if the container persists and the paths are mounted, and fails silently if the scheduled job spins up a clean environment.

Even when everything is wired correctly, the memory is whatever the agent happened to record. An agent under time pressure records less. An agent that misjudged what mattered records the wrong things. There is no durable record of the full conversation, only the summary the agent wrote of itself. And stale entries linger until the agent bothers to prune them, because nothing external is curating the memory.

A nightly repo agent needs three things that neither `file_store` sessions nor `MEMORY.md` provide on their own:

- **Continuity across fresh environments.** Scheduled jobs often run in clean containers. The memory has to live where the schedule can reach it, every run, without mount tricks.
- **Full history, not just the agent's notes.** When the morning summary says "flagged PR #412 again," you want the actual conversation where the agent decided it was fine last Tuesday, not the one-line entry the agent wrote about it.
- **Memory that follows the operator, not the tool.** The same repo decisions matter to the Claude Code session where you write code, the n8n workflow that files the weekly report, and the ChatGPT thread where you plan the sprint. A memory file on the OpenHands box is invisible to all of them.

## How Vilix AI fixes each pain

| Pain for scheduled OpenHands runs | The Vilix AI fix |
| --- | --- |
| A run wakes up blank and re-briefs itself from scratch | The agent calls `get_context` before it plans and loads the relevant full history |
| One run's learning dies with the run | After each run, `save_turn` with a source label saves the full exchange, so the next run starts from what actually happened |
| The agent's own notes lose the reasoning behind a decision | Vilix AI keeps full conversation history, not just distilled facts, so the why can be revisited, not just the outcome |
| Memory trapped on one machine in one tool | The same Vilix AI account serves every connected tool over MCP: OpenHands, Claude Code, ChatGPT, n8n |
| Fresh containers lose local files silently | Vilix AI is cloud hosted with zero infrastructure to manage, so a fresh 2 AM run on a fresh machine starts with everything the last run knew |
| A fact changed and every tool needs the correction | Correct it once in Vilix AI; the newest wins everywhere |
| Standing rules get repeated every week | Projects, project rules and user rules live in Vilix AI; repeat workflows become skills; the agent inbox passes notes between agents |

OpenHands speaks MCP natively, including Streamable HTTP, so Vilix AI connects directly: the agent gets recall and retain tools, and a small rule in its instructions makes memory a habit, recalling relevant decisions at the start of each run and retaining durable facts at the end.

## Setup

1. Pin `file_store="local"` with an absolute `file_store_path` and a fixed `jwt_secret` in `config.toml`, so the OpenHands app keeps sessions across restarts. This is table stakes, not the solution.
2. Use the SDK persistent memory for the local development loop, where one developer and one machine make the two-tier `MEMORY.md` design shine.
3. Connect Vilix AI to OpenHands over MCP: https://vilix.ai/get-started
4. Add a standing rule to the agent's instructions: recall context with `get_context` before planning, and save the run with `save_turn` (with a source label like the schedule name) before finishing, failures included.
5. Run the same Vilix AI account in Claude Code, ChatGPT and your other tools, so the morning follow-up starts from the nightly run's memory.

## Prompts

Nightly run opener:

```text
Before you plan anything, call get_context in Vilix AI for this repo and this schedule.
Load: which pull requests were already reviewed or cleared, known flaky tests, which
services are safe to touch, and every decision from the last 7 nightly runs.
Then do tonight's sweep. When you finish, save_turn in Vilix AI with the source label
for this schedule: what you flagged, what you cleared, what failed, and why.
```

Morning follow-up in Claude Code:

```text
Call get_context in Vilix AI and show me what last night's OpenHands run decided,
including the reasoning, not just its summary. Start with anything it flagged that
it had already cleared on an earlier run.
```

## FAQ

**Does OpenHands remember between sessions?**
It remembers what it wrote down, where you told it to write it, on the machine you run it on. Sessions persist on disk only with `file_store="local"`, an absolute `file_store_path` and a pinned `jwt_secret`. The SDK's persistent memory is opt-in and local to one machine.

**Does the file_store fix give my nightly runs memory?**
No. It keeps the app's session list across restarts. It does not hand a new scheduled session the decisions the last run made.

**What does the SDK MEMORY.md system miss?**
Full history and sharing. It holds only the facts the agent chose to record, on local disk, invisible to every other tool, and stale entries linger until the agent prunes them.

**Why put Vilix AI under the scheduled runs?**
Because the memory then lives in the cloud, not on the container. Every run recalls and retains against the same store, from any machine, in any tool, with full conversation history. Vilix AI is cloud hosted with zero infrastructure, a free plan that stays free, a 7-day Pro trial with no credit card, and your data exportable or deletable anytime.

## Links

- Watch the episode: https://youtu.be/XZKz_zvGqo8
- Read the full post: https://vilix.ai/blog/openhands-remembers-sessions-nightly-runs-wake-up-blank/
- Vilix AI: https://vilix.ai
- Connect Vilix AI in under two minutes per tool: https://vilix.ai/get-started
