# Does Salesforce Agent Context Engine remember between runs?

The practical guide behind the Agentic Pod episode **"Does Salesforce Agent Context Engine Remember Between Runs? Vilix AI Remembers the Runs Outside It"**.

Watch: https://youtu.be/_MDsHZbAc-M

Read the full post: https://vilix.ai/blog/salesforce-agent-context-engine-memory-layer-scheduled-agents/

## The problem

Salesforce opened its Agent Context Engine announcement with a story. An account executive opens a $5M expansion that the CRM marks "Commit" and asks an agent for the next best action. Working from the CRM alone, the agent drafts a confident close email. Meanwhile the enterprise already knows product usage has fallen 27%, there is an open billing dispute, an unsigned compliance rider, and a customer success agent who, after an outage, proposed holding renewal pricing flat and waiving the overages. The CRM had the deal stage. It did not have the context. Salesforce's own diagnosis: "The gap is not data. It is context."

Now run the same problem at operator scale. A lead scoring agent in n8n learns that a target account just changed its pricing page. The next morning, a follow up agent in Make re-researches the same account from scratch. A weekly reporting agent in Zapier rediscovers the same broken integration every single week. Each automation wakes up blind, re-briefs itself from scratch, and whatever one run figured out dies with it, unless the memory lives somewhere outside all of them.

## What Agent Context Engine actually is

Agent Context Engine (ACE) is Salesforce's shared context layer for AI agents, announced at Dreamforce. Any agent on a supported platform can pull context from it: agents built on Agentforce, Google ADK, AWS Bedrock, LangGraph, or the OpenAI SDK. Agents built outside Agentforce connect through SDK code or MCP tools, and every agent receives the same governed context, filtered to what it is permitted to see.

Under the hood, ACE sits on Salesforce's Data 360 customer data platform: the real time pipeline, zero copy access to data where it lives, identity resolution, tagging, and governance. On top of that foundation, ACE adds a knowledge graph and traversal engine that assembles context using GraphRAG. When a request arrives, ACE translates the incoming prompt into the minimum data needed for the maximum result, walking the graph from the resolved customer and traversing only to the facts that are relevant and permitted. The agent receives one scoped payload assembled for that one request: the resolved customer profile, the relevant memories, the real time signals about what the customer is doing right now, and the slice of the knowledge graph relevant to the scenario.

Two properties matter most:

- **Context compounds.** ACE mines the exhaust every agent produces as it works, the raw conversation and the agent's reasoning steps, for facts, preferences, and learnings, and persists them with provenance and governance. One agent's write becomes the next agent's read. Salesforce is explicit that this can require human confirmation before other agents rely on it, because an unchecked memory means one agent's guess becomes another agent's "fact."
- **Context is governed.** Assembly applies the entitlements you already enforce: object, field, and row level security, sharing rules, and data classification. Sensitive content is masked or withheld before it reaches the agent. What an agent can read and what it can write are both governed, and every memory is traceable to its source.

ACE is in preview now. Salesforce says you can start using it today with agent registration and configuration.

## The honest gap for scheduled agents

ACE is the enterprise answer to exactly this problem. It is built for companies on the Salesforce stack, with Data 360 as the foundation, and it is aimed at the full weight of enterprise governance: entitlements, masking, provenance, human confirmation. If that describes your world, the path is agent registration and the preview signup.

But if your agents live in n8n, Make, Zapier, and a few Python scripts on a cron, they sit outside that layer. They get none of its assembled context, none of its compounded memory, and none of its governance. You need the same pattern without the enterprise stack: a memory layer that sits outside every platform, connects over an open protocol, and hands every run the same continuous context. The layer, not the platform, is what remembers.

## How Vilix AI fixes each pain

| Pain for scheduled agents | The Vilix AI fix |
| --- | --- |
| A run wakes up blind and re-briefs itself from scratch | The agent calls `get_context` before it plans and loads the relevant full history |
| One run's learning dies with the run | After each run, `save_turn` with a source label saves the full exchange, so the next run starts from what actually happened |
| Extracted facts lose the reasoning behind a decision | Vilix AI keeps full conversation history, not just extracted facts, so the why can be revisited, not just the outcome |
| The same broken integration gets rediscovered weekly | Failures are saved too, so the next run skips the broken path instead of repeating it |
| Facts go stale across tools and runs | Correct a fact once in Vilix AI and the newest version is what every agent sees |
| Memory trapped inside one platform | The same memory follows the work over MCP into ChatGPT, Claude, Claude Code, Codex, Cursor, n8n style agents, and every other connected tool |
| Standing instructions re-explained every run | Projects, project rules, and user rules live in Vilix AI; repeat workflows become reusable skills; the agent inbox passes notes between agents |
| One more database to run | Vilix AI is cloud hosted with zero infrastructure: no Data 360 stack, no database to run, no memory schema to design |
| Lock in worries | Export everything in a portable format anytime, or delete individual memories or wipe the account instantly |

## Setup

1. Create a Vilix AI account at https://vilix.ai/get-started (the free plan is free forever; the 7 day Pro trial needs no credit card).
2. Connect Vilix AI to your AI tools over MCP using the get started guide. One memory store now follows the work everywhere.
3. In each scheduled agent, before it plans or acts, load context from Vilix AI (the `get_context` pattern: relevant history for that account, project, and workflow).
4. After each run ends, save the exchange back with `save_turn` and a source label for that agent and schedule, including failures and why they happened.
5. Keep Salesforce and ACE for what they are built for: governed enterprise context on the Salesforce stack. Let Vilix AI carry the memory for the runs that live outside it.

## Prompts

Load context before a scheduled run:

```
Before you plan anything, load everything memory holds about this
account and this workflow: previous runs, what was learned, what
failed and why, and the current standing rules.
Use it naturally. Do not re-research anything a previous run
already established. If memory and fresh data disagree, trust the
fresh data and note the correction so it can be saved.
```

Save the run afterwards:

```
Save this run to memory with a source label for this agent and
schedule. Keep the full exchange, what you checked, what you
learned, every decision and the reason behind it, and the outcome.
If anything failed, save the failure and why, so the next run
does not repeat it.
```

## FAQ

**Does Salesforce Agent Context Engine remember between runs?**
Yes, inside its world. ACE persists what agents produce, with provenance and governance, and assembles it for the next agent on a supported platform. Its foundation is Data 360 and the Salesforce stack.

**Do my n8n, Make, or Zapier agents get ACE memory?**
No. They live outside the Salesforce layer. Without a memory layer of their own, each scheduled run starts from scratch and whatever it learns dies with the run.

**What is the simplest fix for a scheduled agent that wakes up blind?**
Give it a memory layer outside every platform: load context with `get_context` before the run, save the exchange with `save_turn` after it. Vilix AI provides that layer over MCP, cloud hosted, with full conversation history.

**How is Vilix AI different from ACE?**
Same pattern, different world. ACE is the enterprise answer on the Salesforce stack, with Data 360, entitlements, masking, and human confirmation. Vilix AI is the operator answer: the same continuous memory for agents and tools that live outside Salesforce, with zero infrastructure to run.

## Links

- Blog post: https://vilix.ai/blog/salesforce-agent-context-engine-memory-layer-scheduled-agents/
- Vilix AI: https://vilix.ai
- Get started: https://vilix.ai/get-started
- Video: https://youtu.be/_MDsHZbAc-M
