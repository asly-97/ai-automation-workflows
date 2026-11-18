# Does a Slack AI agent remember between runs?

The practical guide behind the Agentic Pod episode **"Does a Slack AI Agent Remember Between Runs? Vilix AI Remembers the Channel, Not Just the Thread"**.

Watch: https://youtu.be/eVNzhaX3v-I

Read the full post: https://vilix.ai/blog/slack-ai-agent-remembers-the-thread-not-the-channel/

## The problem

Your Slack AI agent is brilliant inside one thread and blank in the next one. A custom Slack agent reads only the thread it was summoned in, so that thread is its whole working memory. Open a new thread and ask what the team decided about the Acme refund last week, and the agent has nothing: silence first, then a confident answer the team never made. The thread ended, so the memory ended.

## What Slack search actually gives you

- In early 2026, Slack rebuilt Slackbot as a context aware agent. It pulls from your channels, your files, and your canvases, and it drafts briefings in your voice, inside what you are allowed to see.
- That is retrieval, not memory. Slack stores the messages, so the agent can search them, the way you use the search bar, just faster.
- The agent itself keeps nothing: not what it did, not what it decided. Close the chat, open a new one, and it starts from search alone.
- A custom agent you build yourself gets even less: it reads the thread it was summoned in, and that thread is its whole working memory. A new thread is a blank slate, and a DM about another topic is blank there too.

## The thread boundary in production

- Summarize this thread: great. What did the last three Acme threads say: useless, because each one lives in a different thread.
- The digest agent runs at 7 AM in a fresh thread. Yesterday's digest is not in its memory, so it cannot say which tickets are still open.
- The on-call assistant hits a second incident on the same service two days later. It has no idea what fixed the first one: the weird database flag and the runbook step are all in another thread, which is to say, nowhere.
- The worst failure is the invented answer. An agent that cannot remember a decision often invents one instead: same confidence, wrong facts, inside a thread your team trusts.

## Why search is not memory

Retrieval waits for the right question. Memory carries the context forward on its own. An agent with memory checks its own state before it answers: it spots the customer who complained twice, and it follows up on the task it promised yesterday. Search never volunteers any of that.

Builders can wire it up themselves: write every decision into a database at the end of each run, and read it back at the start of the next. It works, and now your memory is a second codebase: schema, versioning, failure modes, maintained forever. Every agent team becomes a database team.

## Every pain, and the Vilix AI fix

| Pain | Why it happens | The Vilix AI fix |
|---|---|---|
| A new thread starts blank | A custom agent reads only the summoned thread, so a new thread or DM inherits nothing | At thread start the agent calls get_context in Vilix AI and loads the full history that matters, the decision and the reasoning behind it |
| Old decisions get invented | The agent cannot remember a decision, so it fills the gap with a confident answer the team never made | The real decision is saved in Vilix AI as a full conversation, not just facts, so the agent reads it back instead of inventing one |
| The 7 AM digest cannot say what is still open | Yesterday's digest lives in a thread the new run never reads | save_turn at the end of every run saves the outcome with a source label for that thread and run, so the next digest reads yesterday's record first and open tickets carry over |
| The on-call agent never saw the first fix | The first incident's fix sits in another thread the second incident never opens | The fix is saved in Vilix AI, failures included, so the on-call agent reads the first incident's record before it starts |
| The DIY fix becomes a second codebase | Hand wired memory means your own schema, versioning, and failure modes, maintained forever | Vilix AI is the shared memory layer: cloud hosted, zero infrastructure, over MCP, with projects, project rules, user rules, skills, and the agent inbox |

Vilix AI is cloud hosted, so there is no database or server to run. It stores full conversation history, not just facts. It has a free forever plan and a 7 day Pro trial with no credit card, and you can export everything or delete it anytime: https://vilix.ai

## Setup: the two-call loop on every thread and run

1. Connect Vilix AI to your tools over MCP: https://vilix.ai/get-started (under two minutes per tool). Agencies give each client one Vilix AI account, so client memory stays apart.
2. At the start of every thread or run, before answering, call get_context to load the history that matters: past decisions with their reasoning, open items, and the standing rules.
3. Do the work in Slack as usual: read the thread, search the channel, draft the reply or the digest.
4. At the end, call save_turn with a source label for that Slack thread and that run, recording what was decided, what is still open, and whether anything failed. Save failures too, with the reason.
5. Keep standing rules in Vilix AI projects, project rules, and user rules, and repeat workflows as skills, so the Slack bot, the scheduled runs, and every other tool (Claude Code, Cursor, your phone) share the same memory. The agent inbox passes notes between agents. When two records disagree, the newest save wins.

## Prompts

Thread start:

```
Before answering in this thread, call get_context in Vilix AI. Read what this channel has already decided and done: past decisions with their reasoning, open tickets, and anything that failed. If a decision exists, use it and cite it. Never invent a decision that is not in the record.
```

Thread end:

```
Call save_turn in Vilix AI with a source label for this Slack thread. Save: every decision made and why, every open item and who owns it, and every failure with its reason. Save what the next thread on this channel needs to know before it answers.
```

Digest run:

```
This digest runs on a schedule in a fresh thread. First call get_context in Vilix AI and read yesterday's digest record: which tickets were open, which closed, and what was promised. Carry open tickets over, do not repeat closed ones. When you finish, call save_turn in Vilix AI with this run as the source label and save the new open and closed state.
```

## FAQ

**Does a Slack AI agent remember between runs?**
Inside one thread, yes: the thread is its working memory. A new thread, a new DM, or a scheduled run in a fresh thread starts blank. Slack AI search can retrieve stored messages, channels, files, and canvases, but that is retrieval over what people wrote, not memory of what the agent did or decided.

**What does Slack AI search actually give an agent?**
Search over the messages Slack stores, plus files and canvases, inside what the user is allowed to see. It answers when you ask the right question. It does not carry context forward on its own, and it holds nothing about the agent's own past decisions, open items, or failures.

**Why does a Slack agent invent answers?**
When an agent cannot remember a decision, it often fills the gap with a plausible one, stated with full confidence. The fix is a real record the agent reads first: the decision and its reasoning, saved where the next thread can load it.

**How does Vilix AI help a Slack agent?**
get_context at thread start loads the full history that matters, decisions with reasoning, not just facts. save_turn at the end saves the thread with a source label, failures included, newest wins, shared over MCP with every tool, so the next thread, the 7 AM digest, and the on-call agent all read the record first instead of starting blank.

## Links

- Episode: https://youtu.be/eVNzhaX3v-I
- Full post: https://vilix.ai/blog/slack-ai-agent-remembers-the-thread-not-the-channel/
- Vilix AI: https://vilix.ai
- Connect Vilix AI: https://vilix.ai/get-started
