# Does Celery remember between runs?

The practical guide behind the Agentic Pod episode **"Does Celery Remember Between Runs? Vilix AI Remembers What the Result Backend Forgets"**.

Watch: https://youtu.be/jhBPAAmylhM

Read the full post: https://vilix.ai/blog/does-celery-remember-between-runs/

## The problem

Your Celery task reports SUCCESS and stores its result, then the next run starts with only its arguments and drafts the same question to a customer who already answered. Celery keeps careful records about tasks: whether a task is waiting, running, finished, or failed, and the value a task returned. None of that gives the next scheduled task the context the last one worked with. Once a task contains an AI step that has to follow up or draft based on what happened last time, that gap becomes the bug.

## What Celery actually keeps

- The scheduler. Celery Beat is a separate process that reads a schedule, works out which tasks are due, and publishes a message to the broker for each one. Beat does not run tasks itself. Workers pick the messages up and execute them.
- Beat's default PersistentScheduler stores the last run time of each entry in a local file, usually called celerybeat-schedule. Database backed schedulers such as django-celery-beat store the same information in database rows instead, including last_run_at and total_run_count.
- The result backend. When a task finishes, Celery can store its state and return value in a backend such as Redis or a database. Each record is keyed by that task's own ID. The built-in states cover the life of one task: PENDING, STARTED when tracking is enabled, SUCCESS, FAILURE, RETRY, and REVOKED. Results also expire, and a built-in cleanup task removes old results so the store does not grow without limit.
- Worker state is narrower still. A worker started with a statedb file persists only its list of revoked task IDs between restarts. That is operational housekeeping, not a history of what tasks decided.

So Celery keeps three kinds of record: when an entry last fired, how one identified task ended, and which IDs were revoked. All three are useful. None is loaded into the next task on its own.

## What the next task starts with

A new task invocation starts with the arguments in its message. That is the whole briefing. If a scheduled job runs every morning with the same arguments, every morning's task receives the same starting point, no matter what yesterday's task learned, promised, or corrected.

Nothing in the result backend changes that by itself. Yesterday's return value sits under yesterday's task ID. Today's task does not search the backend for relevant past work, merge it into a briefing, and continue from it. Code you write has to do every step of that retrieval: know which IDs matter, fetch them before they expire, turn stored return values back into context, and handle the cases where the result was never stored or has already been cleaned up.

Beat's schedule file does not help here either. Knowing that an entry fired at 7 AM yesterday says nothing about what the task found, what a person replied, or which follow-up is now overdue. The timestamp proves the schedule ran. It carries no working memory forward.

The morning support task makes it concrete. A scheduled task checks open support threads each morning and drafts replies with an AI model. Tomorrow's task does not know that one customer already answered yesterday's question, that a human rewrote one draft and explained why, or that another thread was promised a follow-up today. It can easily draft the same question again, in fresh wording, with complete confidence.

## The store operators end up building by hand

The usual response is to build the missing layer inside the application. A database table holds the last summary per customer. A Redis key holds open items. A prompt builder queries recent rows, trims them to fit, and pastes them into today's task. Together they become a second system to design, test, and keep correct as the schedule and the model change. Every Celery team becomes a memory team.

That store also captures less than the agent needs. A summary row is the end of a run, compressed. The next decision often depends on the middle: the original request, the option that was rejected, the correction a person gave, and the reason a decision went one way instead of another. If only the final output was saved, the next run gets the conclusion without the exchange that produced it.

Scope is the other limit. The table belongs to that one Celery application. The same operator usually runs agents in other places too: another queue, an n8n or Make scenario, and AI tools used directly. Each place keeps its own partial record, and a decision made in a chat tool never reaches the morning Celery task unless someone copies it across.

## Every pain, and the Vilix AI fix

| Pain | Why it happens | The Vilix AI fix |
|---|---|---|
| The next task starts with only its arguments | A task invocation receives its message arguments and nothing else, so every run with the same arguments starts from the same point | At task start the AI step calls get_context in Vilix AI and loads the relevant history for that customer and workflow before it acts |
| Yesterday's result is stranded under yesterday's task ID | Result backend records are keyed by task ID, expire, and are cleaned up, and no task searches them on its own | The full exchange is saved in Vilix AI with save_turn at the end of every run, so the next run reads the story, not a return value under a dead ID |
| The morning task repeats a settled question | It never saw the customer reply, the human rewrite and its reason, or the promised follow-up | get_context loads the reply, the rewrite, and the reason why, so the task continues the thread instead of restarting it |
| The hand built store becomes a second system | Summary tables, Redis keys, and prompt builders need schema, tests, and upkeep as the schedule and the model change | Vilix AI is the memory layer outside the task: cloud hosted, zero infrastructure, over MCP, with full conversation history instead of compressed summaries |
| Memory stops at one app | The hand built table lives in one Celery application, while the same work happens in other queues, automation platforms, and chat tools | The same Vilix AI memory is shared over MCP by the Celery worker, another automation platform, and the AI tools you open directly |

Vilix AI is cloud hosted, so there is no database or server to run. It stores full conversation history, not just facts or final results. It has a free forever plan and a 7 day Pro trial with no credit card, and you can export everything or delete it anytime: https://vilix.ai

## Setup: the two-call loop on every task

1. Connect Vilix AI to your tools over MCP: https://vilix.ai/get-started (under two minutes per tool). Agencies give each client one Vilix AI account, so client memory stays apart.
2. At the start of every task, before the AI step acts, call get_context to load the history that matters for that customer, project, or workflow: past exchanges, corrections, open items, and standing rules.
3. Do the work in Celery as usual: Beat schedules, workers execute, the result backend records task states.
4. Before the task ends, call save_turn with a source label for that workflow and run, recording what was decided, what is still open, and whether anything failed. Save failures too, with the reason, so the next run skips the broken path.
5. Keep standing rules in Vilix AI projects, project rules, and user rules, and repeat workflows as skills, so the Celery worker, your other automation platforms, and every other tool (Claude Code, Cursor, your phone) share the same memory. The agent inbox passes notes between agents. When two records disagree, the newest save wins.

## Prompts

Task start:

```
Before acting in this Celery task, call get_context in Vilix AI. Read what this workflow has already done for this customer: the full past exchanges, any human corrections and why they were made, open items, and follow-ups that were promised. If the customer already answered a question, never ask it again. Continue from where the last run actually left off.
```

Task end:

```
Call save_turn in Vilix AI with a source label for this Celery workflow and this run. Save: every decision made and why, every open item and its owner, every follow-up promised and its date, and every failure with its reason. Save what the next run of this task needs to know before it starts.
```

## FAQ

**Does Celery remember between runs?**
In the bookkeeping sense, yes: Beat remembers when entries last fired, the result backend remembers how identified tasks ended until results expire, and workers can persist revoked IDs. In the working sense an AI agent needs, no: the next task receives only its arguments and starts fresh unless your own code fetches and rebuilds the past for it.

**What does the result backend actually store?**
One record per task, keyed by that task's own ID, holding its state (PENDING, STARTED, SUCCESS, FAILURE, RETRY, or REVOKED) and its return value or traceback. Results expire and a cleanup task removes old ones. It is a record of how tasks ended, not a briefing the next task loads.

**Why does the morning task ask the same question again?**
Because it starts with only its arguments. It never saw yesterday's customer reply, the human rewrite of its draft, or the follow-up promised on another thread, so it drafts the same question in fresh wording with complete confidence.

**How does Vilix AI help a Celery task?**
get_context at task start loads the full history that matters, the exchange and the reasoning, not just final results. save_turn at the end saves the new exchange with a source label, failures included, newest wins, shared over MCP with every tool, so the next run in any tool starts where the last one left off.

## Links

- Episode: https://youtu.be/jhBPAAmylhM
- Full post: https://vilix.ai/blog/does-celery-remember-between-runs/
- Vilix AI: https://vilix.ai
- Connect Vilix AI: https://vilix.ai/get-started
