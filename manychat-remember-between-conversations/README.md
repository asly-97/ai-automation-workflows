# Does ManyChat remember between conversations?

The practical guide behind the Agentic Pod episode **"Does ManyChat Remember Between Conversations? Vilix AI Remembers What ManyChat Forgets"**.

Watch: https://youtu.be/c5SpDfVTGNA

Read the full post: https://vilix.ai/blog/manychat-ai-agent-memory-between-conversations

## The problem

Your ManyChat bot handles a great live conversation. It uses the contact's name, remembers their preferences, and answers with full awareness of the thread. Then the conversation ends. When the same contact returns, the AI starts a fresh conversation with no access to what was said last time. Your team can read the old chat in the Inbox. The AI agent opening the new conversation cannot. It does not browse past chats, it does not recall last week's objection, and it does not know the contact already asked about pricing twice. Every returning conversation pays the full cost of rediscovery.

## What ManyChat remembers inside one conversation

Within a single conversation, memory works the way you would hope. ManyChat ships a Memory setting with a Context Retention toggle. Turn it on and the AI keeps user inputs, names, preferences, and recent answers available throughout the conversation. If someone says their name is Josh and later asks about pricing, the bot answers with their name in it. AI Replies, AI Steps, and Goals all draw on this same live context. For one shot interactions, that is enough. The limit is the boundary of the conversation itself: context retention keeps the current chat coherent and carries nothing into the next one.

## Tags and custom fields: persistence, not memory

Tags and custom fields are ManyChat's persistence layer. A tag marks a contact: qualified lead, pricing asked, VIP. A custom field stores a value: name, email, budget, last product viewed. Both survive between conversations, both drive segmentation and branching, and both are the standard community answer to session management: save the state you care about to fields and load it back at the start of the next run.

What they are not is memory. A custom field is a label you designed in advance. The AI cannot ask it an open question like "what did this contact worry about last month?" and get an answer, because fields only hold what you explicitly wrote into them, in the shape you defined. A field that says `budget_5000` records the outcome without the reason the contact hesitated. You, the operator, become the memory architect, and the approach collapses under real conversation history.

## The four options for cross conversation memory

1. **The write back pattern.** At the end of each conversation, save the facts that matter into custom fields or tags. At the start of the next conversation, inject the saved fields into the AI's context. Cheap and inspectable. The ceiling is the schema you designed.
2. **The external CRM.** Push conversation outcomes into your CRM and pull the contact record back before the AI replies. Better for sales teams because the record is shared with humans. Same ceiling, plus integration maintenance.
3. **The summary pipeline.** After each conversation, have a model write a short summary and store it somewhere queryable, then feed the latest summary into the next conversation. This captures the why, not just the what. The work is the pipeline: summarization prompts, storage, retrieval, and keeping it all running.
4. **A shared memory layer.** One store the agent writes to after each conversation and reads from before the next, reachable from every conversation, every flow, and every other tool you run. This is the pattern that changes the scaling math.

## How Vilix AI fixes each pain

| Pain in ManyChat | The Vilix AI fix |
| --- | --- |
| Returning contact treated like a stranger | The agent calls `get_context` before the next conversation and loads the relevant full history |
| Fields record outcomes without reasons | Vilix AI keeps full conversation history, not just extracted facts, so the next conversation revisits what the contact actually said |
| No way to search past conversations by meaning | Semantic retrieval with keyword matching alongside it: names and order numbers match literally, concepts match by meaning |
| Facts go stale across tools | Correct a fact once in Vilix AI and the newest version is what every agent sees; save with `save_turn` and a source label after each conversation |
| Failed approaches get retried | Failures are saved too, so the next conversation skips the broken path |
| Memory trapped inside one bot | The same memory follows the work over MCP into ChatGPT, Claude, Claude Code, Codex, Cursor, and every other connected tool |
| Standing instructions re-explained every time | Projects, project rules, and user rules live in Vilix AI; repeat workflows become reusable skills; the agent inbox passes notes between agents |
| One more database to run | Vilix AI is cloud hosted with zero infrastructure: no database to run and no memory schema to design |
| Lock in worries | Export everything in a portable format anytime, or delete individual memories or wipe the account instantly |

## Setup

1. Create a Vilix AI account at https://vilix.ai/get-started (free plan is free forever; the 7 day Pro trial needs no credit card).
2. Connect Vilix AI to your AI tools over MCP using the get started guide. One memory store now follows the work everywhere.
3. In your ManyChat flow, before the AI replies to a returning contact, load the contact's context from Vilix AI (the `get_context` pattern: relevant history for that contact and project).
4. After each conversation ends, save the exchange back with `save_turn` and a source label for the bot and flow, so the next conversation starts from what actually happened.
5. Keep tags and custom fields for what they are good at: segmentation and branching. Let Vilix AI carry the conversation history itself.

## Prompts

Load context before replying:

```
Before you reply, load everything we know about this contact from memory:
previous conversations, objections, preferences, and outcomes.
Use it naturally. Do not re-ask anything they already told us.
If memory and the current message disagree, trust the current message
and note the correction so it can be saved.
```

Save the conversation afterwards:

```
Save this conversation to memory with a source label for this bot.
Keep the full exchange, the contact's stated needs, every objection
and the reason behind it, what we promised, and the outcome.
If anything failed, save the failure and why, so we do not repeat it.
```

## FAQ

**Does ManyChat AI have long term memory?**
Not natively. It has in conversation context retention and persistent tags and custom fields per contact, but no built in long term memory that lets the AI recall previous conversations. Cross conversation recall requires an external store or a memory layer.

**Can the ManyChat AI see old conversations in the Inbox?**
Your team can read them there. The AI cannot. Past conversations are visible to humans in the Inbox, not to the agent starting the new conversation.

**Do tags and custom fields count as agent memory?**
They are durable per contact storage and the standard workaround. But they store only what you explicitly save in a fixed schema, and the AI cannot semantically search them. They are a filing system, not a memory.

**What is the simplest fix for a bot that treats repeat contacts as strangers?**
The write back pattern is the cheapest start. The durable fix is a shared memory layer like Vilix AI: save each conversation once, load the relevant history before the next one, in every tool.

## Links

- Blog post: https://vilix.ai/blog/manychat-ai-agent-memory-between-conversations
- Vilix AI: https://vilix.ai
- Get started: https://vilix.ai/get-started
- Video: https://youtu.be/c5SpDfVTGNA
