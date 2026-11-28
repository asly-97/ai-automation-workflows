# Kore.ai Remembers the Conversation. Vilix AI Remembers the Work

Kore.ai has some of the best documented conversation memory in conversational AI. Its Context object persists data across all intents in a dialog, so the chat picks up where it left off, even after a timeout. None of that memory reaches a scheduled agent. The 6am run that works for the same customers starts with no run history, no yesterday decisions, and no promises made in chat.

This guide is the technical companion to Agentic Pod EP71, "Kore.ai Remembers the Conversation. Vilix AI Remembers the Work". It covers what Kore.ai memory actually covers, where it stops, what a scheduled run needs instead, and how to give every agent in your stack one shared memory: [Vilix AI](https://vilix.ai).

- Watch the episode: https://youtu.be/J2ENZ9VcsrI
- Read the full post: https://vilix.ai/blog/kore-ai-remembers-the-conversation-scheduled-agents-blank/

## The problem

According to Kore.ai's own documentation, the platform maintains a Context object: a container that persists data for dialog execution and across all intents, including dialog tasks, action tasks, alert and info tasks, and FAQs. The NLP engine populates it with the identified intent, the entities extracted, and the conversation history. In practice that gives you four things:

- Context carried across intents. You ask the cost of an economy flight from London to Paris on August 15th, the agent answers 242 euros, and when you say "Great, I would like to book it," the agent already knows the flight details. Entity values captured in one intent pre-populate entities in another, so a destination city from a flight-status check flows straight into a weather check.
- Follow-up intent detection. "What are the extra charges?" resolves to "what are the charges for a Premium Economy seat?" if you were just discussing Premium Economy benefits.
- Context tags and session variables. Developers can emit context tags from script nodes, entity nodes, and dialog tasks, and conversations can resume after timeouts with the previous context intact.
- A developer-defined memory policy. The bot maintains session, user, and enterprise context throughout all interactions, and developers define what information the bot stores in memory, and for how long.

That is genuinely capable conversation memory. It is also, crucially, a conversation memory. Every capability above is scoped to a conversation the user is having with the assistant. The Context object loads for the chat session. It answers one question very well: "what happened in this conversation so far?"

Your scheduled agents need an answer to a different question: "what happened in the last run?" A nightly support agent triages tickets, drafts replies, and flags the ones that need a human. A morning briefing agent reads dashboards and writes a summary. A follow-up agent checks which open threads went quiet. None of these runs are conversations. None of them load a Context object. When the 6am run starts, it does not know what yesterday's run decided and why, what it already tried on a stubborn thread (and what failed), what a customer was promised in yesterday's chat, which preferences were learned over weeks, or which follow-ups were already sent.

## What a scheduled run actually needs

A scheduled agent does not need a better context window. It needs a memory that outlives the run, in three kinds:

- Run history. What happened in the last run: decisions made, actions taken, what worked, what failed. Without it, the agent retries the same failed approach every morning, or repeats actions it already completed, like sending a follow-up email that already went out.
- Commitments and state. Promises made, threads in flight, tasks left open. The agent that resolved 40 tickets last night should start this morning knowing exactly which 12 are still open and what each one is waiting on.
- Learned preferences. The durable stuff: how this customer likes to be contacted, what solutions worked for this problem class before, which escalation paths actually get results.

Kore.ai stores intents and entities for the conversation. Your scheduled agents need full run records, searchable later, writable from any tool in your stack. That is a different memory job, which is why bolting a chatbot platform's memory onto a scheduling platform never quite works: the memory was designed for the chat, not for the clock.

## Every pain, and the Vilix AI fix

| Pain | How Vilix AI fixes it |
| --- | --- |
| The 6am scheduled agent knows none of yesterday's decisions, so it retries failed fixes and repeats completed actions | At the start of every run, the agent calls `get_context` in Vilix AI and loads prior decisions, actions and full conversation history, recalled semantically across runs, sessions and tools |
| What a customer was promised in chat ("we will call you back tomorrow") never reaches the scheduled agent | Chat and scheduled agents read and write the same Vilix AI memory over MCP, so a promise saved by the chat is read by the morning run |
| Follow-ups get sent twice because the run cannot see what already went out | At the end of a run, the agent saves what it did with `save_turn` in Vilix AI with a source label, so the next run starts from the record |
| Failed runs teach nothing and repeat silently | Failed runs are saved in Vilix AI too, so the next run can see the failure instead of repeating it |
| A correction made in one tool never reaches the others | Correct something once in Vilix AI and the newest version is what every tool retrieves, because retrieval is recency-aware and the last write wins |
| Each tool keeps its own private memory (Kore.ai chat, n8n, Make, custom scripts) | Vilix AI stores full conversation history, not just extracted entities, and any MCP-compatible tool can read and write it |
| Durable memory means hosting and pruning your own store forever | Vilix AI is cloud hosted with zero infrastructure to run; you connect once and manage nothing |
| Standing instructions get re-explained per tool | Projects and project rules live in Vilix AI, and user rules live in Vilix AI, so every connected tool follows the same rules |
| Repeat workflows are rebuilt per tool | Repeat workflows become skills in Vilix AI, and agents can leave each other notes in the agent inbox |
| Agencies run agents for several clients | One Vilix AI account per client keeps each client's memory separate and portable |
| You want out | List, edit, or delete memories, export everything in a portable format, or wipe the account anytime |

## Setup

1. Create a free Vilix AI account at https://vilix.ai/get-started (free forever plan; 7 day Pro trial, no credit card).
2. Connect your tools to Vilix AI over MCP: a headless agent connects with an API key as a Bearer token to https://api.vilix.ai/mcp, following the per-tool steps on the get-started page. Connect the Kore.ai assistant's surrounding stack, your n8n or Make automations, and the AI tools your team works in (ChatGPT, Claude, Claude Code, Codex, Cursor, or any MCP client) to the same account.
3. Add a read step at the start of every scheduled run: call `get_context` in Vilix AI before the agent decides anything, so it loads yesterday's decisions, open commitments and learned preferences.
4. Add a write step at the end of every run: save what the agent did and decided with `save_turn` in Vilix AI, with a source label like `morning-briefing-agent`. Save failed runs too.
5. Keep Kore.ai's Context object for what it is good at: carrying the conversation across intents. Put standing rules (this customer prefers email, not phone) in a Vilix AI project as project rules, and turn repeat flows into Vilix AI skills.

## Prompts

Start-of-run read (first step of every scheduled run):

```
Before you start, call get_context in Vilix AI for this customer and this workflow.
Load: yesterday's run record, open commitments and promises, follow-ups already sent,
approaches already tried and what failed, and the customer's learned preferences.
Do not repeat a completed action. Do not retry a failed approach without saying why it will work this time.
```

End-of-run write (last step of every run):

```
Save this run to Vilix AI with save_turn and the source label for this agent.
Record: decisions made and why, actions taken, promises made to customers,
threads left open and what each is waiting on, anything that failed and the exact error,
and any new preference you learned. Save this even if the run failed.
```

## FAQ

**Does Kore.ai remember between conversations?**
Inside a conversation, yes: the Context object carries context across intents, entities pre-populate later intents, and a session can resume after a timeout. That memory is scoped to the conversation and the chat platform.

**Why does my scheduled agent wake up blank if my chatbot remembers everything?**
Because the scheduled run is not a conversation. It never loads the Context object, and the chat's memory was never written anywhere the run can read. The two systems share no memory by default.

**What should a scheduled agent store after each run?**
Decisions and why, actions taken, promises made, open threads and their state, failures with errors, and learned preferences. Full run records, not just extracted facts, so the next run can reason from what actually happened.

**How does Vilix AI connect to a headless scheduled agent?**
Over MCP with an API key as a Bearer token to https://api.vilix.ai/mcp. The agent calls `get_context` at the start of a run and `save_turn` at the end, the same loop every other connected tool uses.

## Links

- Watch EP71: https://youtu.be/J2ENZ9VcsrI
- Full post: https://vilix.ai/blog/kore-ai-remembers-the-conversation-scheduled-agents-blank/
- Vilix AI: https://vilix.ai
- Get started: https://vilix.ai/get-started
