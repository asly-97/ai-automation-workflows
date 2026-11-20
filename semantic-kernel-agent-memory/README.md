# Does Semantic Kernel remember between sessions?

The practical guide behind the Agentic Pod episode **"Does Semantic Kernel Remember Between Sessions? Vilix AI Remembers What the Kernel Forgets"**.

Watch: https://youtu.be/PlWfi6-Z1DA

Read the full post: https://vilix.ai/blog/does-semantic-kernel-remember-between-sessions

## The problem

Your Semantic Kernel agent holds a great conversation, then the session ends and the next one starts from nothing. Inside one session the agent seems to remember everything, because your code keeps a ChatHistory list and resends it with every call. That list lives in your process memory. Restart the process, scale out to a second server, or close the app, and it is gone. Nothing was ever stored anywhere durable, so a returning user gets a polite hello from an agent that met them yesterday and remembers none of it.

## What Semantic Kernel keeps in one session

Short term conversation state, and only that. Every turn, your app appends the user message and the model reply to a ChatHistory list and resends the whole list with the next call. That is how the agent "remembers" what was said two minutes ago: your code hands the history back to the model each time.

Two consequences follow. First, the history is unbounded by default, so a long session keeps growing the prompt and the bill with it. Semantic Kernel includes chat history reducers that truncate old messages or summarize them, which is the sanctioned way to keep the window under control. Second, the history lives in your application's memory, so it survives exactly as long as the process does.

## What happens when the session ends

Nothing automatic. There is no built in session store, no user profile that accumulates, and no record of what the agent learned about the user that carries into the next conversation. If you want the agent to greet a returning user with knowledge of the last session, you serialize the ChatHistory to JSON yourself and store it in a database or cache keyed by conversation ID, then reload it when the user comes back. The framework gives you the object; the saving and loading are your code.

Older Semantic Kernel versions shipped higher level abstractions for this, like the SemanticTextMemory APIs. Those are legacy now and the current guidance is to avoid them in new code. The modern surface is lower level: chat history for the short term, vector stores for retrieval, and agent thread memory providers for extracted memories.

## The memory providers, and what they actually do

The Agent Framework supports components that attach to an AgentThread, pull memories out of the messages added to that thread, and hand those memories back to the agent when needed. Microsoft documents two of them, and both are marked experimental.

The WhiteboardProvider is the short term one. It processes each message and extracts requirements, proposals, decisions, and actions onto a "whiteboard" that travels with the thread. Its real job is surviving truncation: when you cut the chat history down to fit the context window, the whiteboard keeps the distilled version of what mattered. You can cap how many messages it retains, and less valuable entries get removed first.

The Mem0Provider is the long term one. It sends every thread message to the Mem0 service, which extracts memories, and on each invocation it queries Mem0 for memories matching the current request and injects them into the agent's context. The scoping is the important part: configure it with a user ID and memories persist across threads for that user; configure it with a thread ID and memories stay attached to that single thread. You can also combine both providers, so one agent gets whiteboard short term context plus Mem0 long term recall.

Notice the pattern. In every case, the memory lives outside Semantic Kernel: in the Mem0 service, in a vector store you configured, in a database you wrote. The framework orchestrates the handoff. It does not operate the storage.

## The four piece build

There are four pieces, and you need all four for a production agent:

1. Persist the chat history yourself. Serialize ChatHistory to JSON and store it keyed by conversation or user ID in a database or cache. Reload it at the start of each session. Do not keep it only in server memory if you scale out or restart.
2. Bound the history with reducers. Truncate or summarize old turns so long sessions do not blow the context window or the token budget. The whiteboard provider can carry the distilled decisions so truncation does not lose the plot.
3. Attach memory providers to the agent thread. Use the whiteboard for short term extracted context within and across the turns of a thread, and a provider like Mem0 with a user level scope for memories that should follow the user across threads and sessions.
4. Wire your own long term store. For semantic retrieval over past interactions, documents, or user facts, connect a vector store and handle the write path (what gets stored, scoped to which user) and the read path (what gets injected per request) in your application code.

If you are building scheduled agents on top of Semantic Kernel, a cron job that runs every morning, step one is the one people skip. A scheduled run is a brand new session every time. Without persisted history keyed to something stable, each run wakes up knowing nothing about the previous one, no matter which memory providers you attached, because the providers' inputs start empty too.

## Every pain, and the Vilix AI fix

| Pain | Why it happens | The Vilix AI fix |
|---|---|---|
| The next session starts with a blank ChatHistory | ChatHistory lives in process memory and is never stored anywhere durable by default | At session start the agent calls get_context in Vilix AI and loads the relevant history for that user and project before it acts |
| Persisting and reloading history is all your code | There is no built in session store; you serialize to JSON, key it, store it, and reload it yourself | The full exchange is saved in Vilix AI with save_turn at the end of every session, with a source label for that agent and run |
| Long sessions grow the prompt and the bill | History is unbounded by default and resent in full on every call | Vilix AI retrieval returns the relevant part of the history, mixing semantic search with keyword matching so exact strings like order IDs and policy names are found literally |
| Truncation loses the middle of the exchange | Reducers and summaries keep the end result and drop the original request, the rejected option, and the correction a person gave | Vilix AI stores full conversation history, not just facts or final results, so the next session reads the exchange and the reasoning |
| Providers still leave you operating the storage | Whiteboard and Mem0 providers orchestrate memory that lives in services and stores you configure and run | Vilix AI is the hosted memory layer: cloud hosted, zero infrastructure, over MCP, one API key connects the agent |
| Memory stops at one framework | A hand built store belongs to that one Semantic Kernel app, while the same work happens in chat tools and other agents | The same Vilix AI memory is shared over MCP by the Semantic Kernel agent, ChatGPT, Claude, Cursor, and every other connected tool |

Vilix AI is cloud hosted, so there is no database or server to run. It stores full conversation history, not just facts or final results. Standing rules live in projects, project rules, and user rules, repeat workflows become skills, and the agent inbox passes notes between agents. When two records disagree, the newest save wins. It has a free forever plan and a 7 day Pro trial with no credit card, and you can export everything or delete it anytime: https://vilix.ai

## Setup: the two-call loop on every session

1. Connect Vilix AI to your tools over MCP: https://vilix.ai/get-started (under two minutes per tool). Agencies give each client one Vilix AI account, so client memory stays apart.
2. At the start of every session, before the agent acts, call get_context to load the history that matters for that user, project, or workflow: past exchanges, corrections, open items, and standing rules.
3. Do the work in Semantic Kernel as usual: ChatHistory for the live conversation, reducers to bound it, thread providers where they fit.
4. Before the session ends, call save_turn with a source label for that agent and run, recording what was decided, what is still open, and whether anything failed. Save failures too, with the reason, so the next session skips the broken path.
5. Keep standing rules in Vilix AI projects, project rules, and user rules, and repeat workflows as skills, so the Semantic Kernel agent and every other tool (Claude Code, Cursor, your phone) share the same memory. The agent inbox passes notes between agents. When two records disagree, the newest save wins.

## Prompts

Session start:

```
Before acting in this session, call get_context in Vilix AI. Read what this user and project already covered: the full past exchanges, any human corrections and why they were made, open items, and decisions already taken. If a question was already answered, never ask it again. Continue from where the last session actually left off.
```

Session end:

```
Call save_turn in Vilix AI with a source label for this Semantic Kernel agent and this session. Save: every decision made and why, every open item and its owner, every follow-up promised and its date, and every failure with its reason. Save what the next session needs to know before it starts.
```

## FAQ

**Does Semantic Kernel remember between sessions?**
No, not by itself. Inside one session, Semantic Kernel keeps the conversation in a ChatHistory object that lives in your process. When the session ends, that object is gone unless your code saved it somewhere. Nothing is retained across sessions by default; the persistence layer is always yours to build.

**What do the WhiteboardProvider and Mem0Provider actually store?**
The WhiteboardProvider extracts requirements, proposals, decisions, and actions from thread messages onto a whiteboard that travels with the thread, mainly so distilled context survives truncation. The Mem0Provider sends thread messages to the Mem0 service, which extracts memories, and queries them back per request, scoped by user ID across threads or by thread ID inside one thread. Both are experimental, and in both cases the memory lives outside Semantic Kernel.

**Why does my scheduled Semantic Kernel agent forget everything overnight?**
Because a scheduled run is a brand new session every time. Its ChatHistory starts empty, and the memory providers' inputs start empty too, unless your code persisted history keyed to something stable and reloads it at the start of the run.

**How does Vilix AI help a Semantic Kernel agent?**
get_context at session start loads the full history that matters, the exchange and the reasoning, not just final results. save_turn at the end saves the new exchange with a source label, failures included, newest wins, shared over MCP with every tool, so the next session in any tool starts where the last one left off.

## Links

- Episode: https://youtu.be/PlWfi6-Z1DA
- Full post: https://vilix.ai/blog/does-semantic-kernel-remember-between-sessions
- Vilix AI: https://vilix.ai
- Connect Vilix AI: https://vilix.ai/get-started
