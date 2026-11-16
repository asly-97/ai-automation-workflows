# Does Goose remember between sessions?

The practical guide behind the Agentic Pod episode **"Does Goose Remember Between Sessions? Vilix AI Remembers the Work It Forgets"**.

Watch: https://youtu.be/yDUnOoPboDY

Read the full post: https://vilix.ai/blog/does-goose-remember-between-sessions/

## The problem

Goose, Block's open-source AI agent, has four separate ways to carry information forward, and a new session can still start knowing nothing. Each mechanism remembers exactly what you put in, and only where you put it. The runs that need memory most, the scheduled and headless CLI jobs, are the ones that start fresh every time: batch jobs spin up a fresh session per job, so every run begins as a stranger, with no note on what last night's run tried, decided, or broke.

## What Goose keeps per session

- Resumable sessions: work is organized into sessions, and sessions persist. Close one, come back days later, resume it by name, and the accumulated context is intact. A resumed session remembers its own conversation, not you: a new session for a new task knows nothing about yesterday.
- The memory extension: a real cross-session store for coding conventions, project facts, and preferences. It is opt-in, it only knows what was stored into it by hand, and it lives on one machine. A second install has its own separate memory, with no sync, so two installs drift apart.
- .goosehints files: standing instructions loaded at session start, including nested hint files Goose finds as it explores. How the repo is organized, which commands are safe. A hint file never updates itself, so it stays stale until a human edits it, and a changed process means confident, outdated instructions in every session.
- Persistent instructions (MOIM): Model-Observed Internal Memory, text injected into working memory and re-read fresh every turn so guardrails cannot fade as a chat grows. Capped at 64 KB, it is a slot for rules, not for everything your runs have learned.

## Desktop and CLI do not remember equally

The desktop app carries persistent memory across sessions. A CLI session starts fresh, per Goose's own docs. Scheduled and headless automation runs through the CLI, so the exact runs nobody watches start blank every time, and when the work moves machines, laptop to server to phone, nothing follows the operator.

## Every pain, and the Vilix AI fix

| Pain | Why it happens | The Vilix AI fix |
|---|---|---|
| A new session knows nothing about yesterday | A resumed session remembers its own conversation only; a new session for a new task starts cold | At run start the agent calls get_context in Vilix AI and loads the full history, decisions and failures included |
| The memory extension is opt-in and manual | It only knows what someone stored into it by hand | Vilix AI saves what actually happened in the run automatically with save_turn at the end of every run |
| Memory lives on one machine and drifts | Each install keeps its own separate store, with no sync and no shared store | The same Vilix AI memory is shared over MCP with every machine and every tool, one shared store instead of local ones drifting apart |
| Hint files go stale | A .goosehints file never updates itself when the project changes | Standing rules live in Vilix AI as projects, project rules, and user rules, and repeat workflows live as skills, updated once for every session |
| The 64 KB instruction slot cannot hold learned work | Persistent instructions are a slot for rules, capped at 64 KB, not a history of runs | Vilix AI stores full conversation history, not just facts: what was tried, what was decided, and what failed |
| Headless batch jobs start as strangers | The CLI starts fresh and each batch job spins up a fresh session per job, with no note on what the last run tried | get_context at run start loads last night's run, and save_turn with a source label saves this run for the next one. Failures are saved too, so the next run skips them. Newest save wins |

Vilix AI is cloud hosted, so there is no database or server to run. It has a free forever plan and a 7 day Pro trial with no credit card, and you can export everything or delete it anytime: https://vilix.ai

## Setup: the two-call loop on every run

1. Connect Vilix AI to Goose over MCP: https://vilix.ai/get-started (under two minutes per tool).
2. At the start of every session or run, before doing the work, call get_context to load the full history: decisions, failures, and the standing rules.
3. Do the work in Goose as usual: run the task, the batch job, or the scheduled job.
4. At the end of the run, call save_turn with a source label for that run, recording what was tried, what was decided, and whether it succeeded or failed. Save failures too, with the reason.
5. Keep standing rules in Vilix AI projects and project rules, and repeat workflows as skills, so the next session and the next tool follow the same rules. Use one Vilix AI account per client so memories never mix. The agent inbox passes notes to the next agent.

## Prompts

Run start:

```
Before starting this task, call get_context in Vilix AI. List what previous sessions and runs already did on this project, every decision already made, any approach already tried and rejected, and any failed actions. Skip anything that already has a completed decision. Work only on what is still open.
```

Run end:

```
Call save_turn in Vilix AI with a source label for this run. Save: what was tried, what was checked, the decision, and whether the action succeeded or failed. Save failures too, with the reason. Save what the next session needs to know before it starts.
```

Headless batch job wrapper:

```
This job runs unattended. First call get_context in Vilix AI and read what the previous run of this job saved: what it processed, what it skipped, and what failed. Do not repeat completed work or retry a recorded failure the same way. When you finish, call save_turn in Vilix AI with this job name as the source label and save the outcome of every item, including failures.
```

## FAQ

**Does Goose remember between sessions?**
Partly. A resumed session keeps its own conversation, the memory extension keeps facts you stored by hand on one machine, .goosehints files keep standing instructions, and persistent instructions keep rules up to 64 KB. A brand new session, a second install, and every headless CLI run still start without the history of the work itself.

**What is the Goose memory extension?**
An opt-in cross-session store for conventions, project facts, and preferences. It only knows what was stored into it, and it lives on one machine with no sync.

**What are .goosehints?**
Hint files Goose loads at session start with standing instructions for a project, including nested files. They never update themselves, so they go stale until a human edits them.

**How does Vilix AI help Goose?**
get_context at run start loads the full history of past sessions and runs, decisions and failures included. save_turn at run end saves the new work back with source labels, newest wins, shared over MCP with every machine and tool, including the headless CLI runs that start fresh.

## Links

- Episode: https://youtu.be/yDUnOoPboDY
- Full post: https://vilix.ai/blog/does-goose-remember-between-sessions/
- Vilix AI: https://vilix.ai
- Connect Vilix AI: https://vilix.ai/get-started
