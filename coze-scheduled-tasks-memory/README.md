# Does Coze remember between runs?

The practical guide behind the Agentic Pod episode **"Does Coze Remember Between Runs? Vilix AI Remembers What Its Scheduled Tasks Forget"**.

Watch: https://youtu.be/Jx39tSc0RFE

Read the full post: https://vilix.ai/blog/coze-has-long-term-memory-its-scheduled-tasks-still-wake-up-blank/

## The problem

Coze has long-term memory, but it remembers user conversations, not the bot's own scheduled work. A daily deal digest bot posts at 8 AM, repeats the same product three days in a row, and calls expired deals fresh, because a scheduled run does not inherit the long-term memory summaries from user chats and gets nothing about what the previous run posted, skipped, or decided. The bot works exactly as configured. It remembered the users. It never remembered its own work.

## What Coze memory actually covers

- Long-term memory: when a user chats with your bot across sessions, Coze summarizes those chats, and later chats can use those summaries. Responses get more personal over time, and a returning user is not greeted like a stranger.
- That memory is memory of the back and forth with a user. It is not memory of what the bot did alone last night. Different store, different job.
- A scheduled task is the bot acting on its own at set times with its configured instruction: daily updates, reminders, a fresh roundup. It is not continuing a user conversation, so it does not inherit the user chat summaries, and it gets nothing from yesterday's scheduled run either.

## What persists and what vanishes

What persists between runs:

- The bot configuration: persona, prompt, plugins, knowledge base, the schedule. Static. How it behaves, not what it has done.
- The long-term memory from user chats. It exists, but a scheduled run does not pull it in, and even if it did, it describes users, not the last run.
- External data the plugins can fetch fresh at run time.

What vanishes between runs:

- What the last run did: posted items, skipped items, errors, corrections. All gone.
- Decisions with reasons: why it picked those five deals, which sources it already used up.
- Running state: a counter, a last processed marker. Nothing remembers across runs unless a write happens somewhere a later run reads, and Coze long-term memory never writes run records.

## Every pain, and the Vilix AI fix

| Pain | Why it happens | The Vilix AI fix |
|---|---|---|
| A scheduled run starts blank | A scheduled run is not continuing a user conversation, so it inherits no long-term memory summaries | At run start the agent calls get_context in Vilix AI and loads the last run record first |
| The bot repeats what it already posted | No run ever told the next run what it posted, skipped, or decided | save_turn at the end of every run saves what was posted, skipped, and decided, with the reasons, under a source label for that run |
| Expired deals get called fresh | Decisions and corrections vanish between runs, so nothing marks an item used up | The next run reads the full history in Vilix AI before picking, so used items and expired deals are skipped |
| A broken source gets retried forever | Errors and failures are never recorded anywhere a later run reads | Failures are saved in Vilix AI too, so tomorrow skips the broken source. Newest save wins |
| Standing rules live only in the bot config | Persona and deal rules sit in one bot's configuration and no other tool can follow them | Standing rules live in Vilix AI as projects, project rules, and user rules, and the repeat workflow lives as a skill, shared over MCP with Coze, Claude Code, ChatGPT, Codex, Cursor, and every other tool |

Vilix AI is cloud hosted, so there is no database or server to run. It stores full conversation history, not just facts. It has a free forever plan and a 7 day Pro trial with no credit card, and you can export everything or delete it anytime: https://vilix.ai

## Setup: the two-call loop on every run

1. Connect Vilix AI to your tools over MCP: https://vilix.ai/get-started (under two minutes per tool). Use one Vilix AI account per client so memories never mix.
2. At the start of every scheduled run, before doing the work, call get_context to load the last run record: what was posted, skipped, and decided, with the reasons, plus the standing rules.
3. Do the run's work in Coze as usual: fetch fresh data with the plugins, pick the items, post the digest.
4. At the end of the run, call save_turn with a source label for that scheduled run, recording what was posted, what was skipped, and whether anything failed. Save failures too, with the reason.
5. Keep the persona and deal rules in Vilix AI projects and project rules, and the repeat workflow as a skill, so the next run and every other tool follow the same rules. The agent inbox passes notes to the next agent.

## Prompts

Run start:

```
Before starting this scheduled run, call get_context in Vilix AI. Read the record of the previous run of this bot: what it posted, what it skipped, what it decided and why, and anything that failed. Do not repost anything already posted, do not call expired items fresh, and do not retry a recorded failure the same way. Work only from fresh items.
```

Run end:

```
Call save_turn in Vilix AI with a source label for this scheduled run. Save: every item posted, every item skipped and why, every decision with its reason, and every failure with its reason. Save what tomorrow's run needs to know before it starts.
```

Scheduled bot wrapper:

```
This bot runs unattended on a schedule. First call get_context in Vilix AI and read what the previous run saved: posted items, skipped items, decisions, failures. Never repeat a posted item. When you finish, call save_turn in Vilix AI with this bot and run as the source label and save the outcome of every item, including failures.
```

## FAQ

**Does Coze remember between runs?**
It remembers user conversations: long-term memory summarizes past chats with a user so later chats get more personal. A scheduled run does not inherit those summaries and gets nothing about what the previous run posted, skipped, or decided, so scheduled work starts blank every time.

**What is a Coze scheduled task?**
A way for the bot to act on its own at set times with its configured instruction, like a daily update or reminder. The bot starts the conversation this time, instead of continuing a user's conversation.

**What persists between Coze runs?**
The bot configuration, the long-term memory from user conversations, and fresh external data from plugins. What vanishes is the work itself: posted and skipped items, errors and corrections, decisions with reasons, and running state like counters and last processed markers.

**How does Vilix AI help Coze?**
get_context at run start loads the last run record, full history, not just a summary. save_turn at run end saves what happened with a source label, failures included, newest wins, shared over MCP with every tool, so the next scheduled run reads the record first instead of starting blank.

## Links

- Episode: https://youtu.be/Jx39tSc0RFE
- Full post: https://vilix.ai/blog/coze-has-long-term-memory-its-scheduled-tasks-still-wake-up-blank/
- Vilix AI: https://vilix.ai
- Connect Vilix AI: https://vilix.ai/get-started
