# LangGraph Remembers the Thread. Vilix AI Remembers the Work

LangGraph remembers only inside one thread. Its checkpointer saves your graph's state under a thread_id, so the same conversation can resume after a crash or a restart. A new thread_id starts with a blank slate. For memory that survives across threads and sessions, you need LangGraph's store, a separate system you have to wire in deliberately, or a hosted memory layer outside the LangGraph runtime.

This guide is the technical companion to Agentic Pod EP69, "LangGraph Remembers the Thread. Vilix AI Remembers the Work". It covers what a checkpointer actually remembers, why a new thread starts from zero, what the store adds, and how to give a LangGraph agent one shared memory across threads, sessions and tools: [Vilix AI](https://vilix.ai).

- Watch the episode: https://youtu.be/r8cSqFSRkUA
- Read the full post: https://vilix.ai/blog/does-langgraph-remember-between-runs/

## The problem

A checkpointer snapshots your graph's state after every node runs and keys it by thread_id. That gives you four things:

- Conversation continuity. The same thread picks up where it left off, even after the process restarts, as long as the checkpointer itself is durable.
- Crash recovery. If a node fails mid-run, LangGraph resumes from the last saved checkpoint instead of starting over.
- Human-in-the-loop. You can pause a thread, let a person review or edit the pending state, and resume.
- Time travel. You can rewind a thread to any earlier checkpoint for debugging.

What the checkpointer remembers is where this conversation was. What it does not remember is anything outside that conversation. One more detail that bites people: the default checkpointer in tutorials, InMemorySaver, keeps checkpoints in RAM. Restart the process and everything is gone. LangGraph's own docs treat it as a development tool and recommend PostgresSaver for anything real.

Threads are isolated by design. Each thread_id is its own island of state. The checkpointer for the new thread has no way to reach into the old thread's snapshots, and it was never meant to. A checkpointer answers "what was happening in this conversation?" It has nothing to say about "what do I know about this user across all conversations?"

This is the exact point where scheduled and multi-session agents break. A nightly job that runs your LangGraph agent with a fresh thread_id every night gets a brand-new agent every night. Preferences learned on Monday, corrections made on Tuesday, facts gathered on Wednesday: all of it is invisible on Thursday, because Thursday is a different thread.

## The store, and what it costs you

The store is LangGraph's second persistence system, and it exists precisely for the cross-thread case. Where the checkpointer is automatic and thread-scoped, the store is deliberate and cross-thread:

| | Checkpointer | Store |
| --- | --- | --- |
| Scope | One thread (thread_id) | All threads |
| What it holds | Graph state snapshots | Application-defined data: preferences, facts, user profiles |
| Saved | Automatically, after each node | Only when you call store.put() |
| Read by | Resuming a known thread | Namespace lookup or search from any thread |

```python
from langgraph.store.memory import InMemoryStore

store = InMemoryStore()
store.put(
    namespace=("users", user_id),
    key="preferences",
    value={"report_format": "bullet points", "timezone": "America/Denver"},
)
graph = builder.compile(checkpointer=checkpointer, store=store)
```

Most production LangGraph agents run both: the checkpointer for the current thread's working state, the store for durable knowledge that should follow the user across sessions. InMemoryStore is the dev stand-in; production setups point the store at Postgres or Redis, on the same database instance as the checkpointer if you want operational simplicity. Checkpoint tables grow fast under sustained traffic, so you also need a retention policy, for example a scheduled job that deletes checkpoints older than N days.

The most common LangGraph memory mistake is treating the checkpointer as long-term memory. A September 2026 field guide to LangGraph memory put it bluntly: most developers migrating to LangGraph use the checkpointer as a drop-in for the old ConversationBufferMemory and expect it to remember user preferences across sessions. It does not. The preferences land in the checkpointer, scoped to the old thread_id, and a returning user on a new thread gets a blank slate. Teams usually discover this in production, when a real user reports that the agent has forgotten everything about them.

## Every pain, and the Vilix AI fix

| Pain | How Vilix AI fixes it |
| --- | --- |
| A new thread_id starts blank, so a nightly agent forgets Monday's preferences and Tuesday's corrections by Thursday | At the start of every run, the agent calls `get_context` in Vilix AI and loads prior decisions, corrections, and the full conversation history, recalled semantically across threads, sessions and tools |
| What the agent did and decided dies with the thread | At the end of a run, the agent saves what it did and decided in Vilix AI with `save_turn` and a source label, so the next run in any thread starts from it |
| Failed runs teach nothing and repeat silently | Failed runs are saved in Vilix AI too, so the next run can see the failure instead of repeating it |
| A correction made in one tool never reaches the LangGraph agent | Correct something once in Vilix AI and the newest version is what every tool retrieves, because retrieval is recency-aware and the last write wins |
| The store only saves what your nodes deliberately write, inside the LangGraph runtime | Vilix AI saves full conversation history, not just facts, and any MCP-compatible tool can read and write it: the LangGraph agent, Claude, Codex, and your n8n workflows alike |
| Durable memory means hosting Postgres or Redis and pruning checkpoint tables forever | Vilix AI is cloud hosted with zero infrastructure to run; you connect once and manage nothing |
| Standing instructions get re-explained per tool | Projects and project rules live in Vilix AI, and user rules live in Vilix AI, so every connected tool follows the same rules |
| Repeat workflows are rebuilt per tool | Repeat workflows become skills in Vilix AI, and agents can leave each other notes in the agent inbox |
| Agencies run LangGraph agents for several clients | One Vilix AI account per client keeps each client's memory separate and portable |
| You want out | List, edit, or delete memories, export everything in a portable format, or wipe the account anytime |

## Setup

1. Create a free Vilix AI account at https://vilix.ai/get-started (free forever plan; 7 day Pro trial, no credit card).
2. Connect your LangGraph agent to Vilix AI over MCP with an API key as a Bearer token to https://api.vilix.ai/mcp, following the per-tool steps on the get-started page. Connect the other AI tools your team works in (ChatGPT, Claude, Claude Code, Codex, Cursor, or any MCP client) to the same account.
3. Add a read step at the start of every run, whatever thread_id it uses: call `get_context` in Vilix AI for the user before the agent decides anything.
4. Add a write step at the end of every run: save what the agent did and decided with `save_turn` in Vilix AI, with a source label like `langgraph-agent`. Save failed runs too.
5. Keep the checkpointer for what it is good at: thread working state, crash recovery, human-in-the-loop and time travel. Put standing rules (always use bullet points for this user) in a Vilix AI project as project rules, and turn repeat flows into Vilix AI skills.
6. If you also run LangGraph's own store, key its namespaces by user_id, not thread_id: one user has many threads.

## Prompts

Start-of-run read (first step of every run, any thread):

```
Before you act, call get_context in Vilix AI for this user.
Use the saved record: prior decisions, preferences, corrections, and the
full conversation history behind them, across every earlier thread.
Do not ask the user for anything already answered in a saved conversation.
```

End-of-run write (last step of every run, success or failure):

```
Save this run to Vilix AI with save_turn, source "langgraph-agent":
the user, the thread_id, what you decided and why, what you promised,
and what stays open. If the run failed, save the failure and the reason.
```

Correction (run once, anywhere connected):

```
Correct the saved preference for this user in Vilix AI: use the corrected
version from now on. The newest version wins for every tool and thread.
```

## FAQ

**Does LangGraph remember between runs?**
Within one thread, yes: the checkpointer restores the thread's state, so the same conversation continues across runs and restarts. Across threads, no: each new thread_id starts empty. Cross-thread memory needs the store, or a memory layer outside the runtime.

**What is the difference between a LangGraph checkpointer and a store?**
A checkpointer saves thread-scoped graph state automatically after every node. A store persists application-defined data across all threads, but only what you explicitly write to it with store.put(). Production agents typically use both.

**Why does my LangGraph agent forget the user on a new conversation?**
Because the facts were saved in the checkpointer, which is scoped to the old thread_id. Move user-level facts (preferences, corrections, durable context) into the store keyed by user, or into Vilix AI, and read them at the start of each run.

**Do I need a database for LangGraph memory in production?**
For LangGraph's own persistence, yes. InMemorySaver and InMemoryStore lose everything on process restart. Use PostgresSaver for the checkpointer and a Postgres or Redis backed store for cross-thread memory, plus a retention policy for old checkpoints. With Vilix AI there is no database to run: it is cloud hosted and you connect over MCP.

**When should you reach for a hosted memory layer instead?**
Use the store plus PostgresSaver when your memory never needs to leave the LangGraph runtime. Reach for Vilix AI when you want zero infrastructure to operate and one shared memory across every tool your agents run in.
