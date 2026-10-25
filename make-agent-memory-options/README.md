# What is the best AI agent memory for Make workflows?

The technical guide behind the Agentic Pod episode **"What Is the Best AI Agent Memory for Make Workflows?"**: why Make AI agents forget between runs, the seven memory options for Make (Data Stores, a stable Conversation ID, knowledge files, Google Sheets or Airtable, a vector database over HTTP, Mem0 and Vilix AI), when each one fits, and how Vilix AI attaches to Make's AI agent as an MCP tool.

Blog post: https://vilix.ai/blog/best-ai-agent-memory-for-make-workflows · Start free: https://vilix.ai/get-started

## The problem

Make scenario runs are stateless: nothing carries over unless you write it down and read it back. On Make's AI agent module the Conversation ID is the sharp edge. Leave it blank and every run creates a fresh agent identity that knows nothing. Give each thread a stable ID and the agent keeps continuity inside that thread, but a second scenario with its own agent starts from zero again, even for the same customer.

## The seven options

| Option | Best for | Cross-tool sharing | You operate |
|---|---|---|---|
| Make Data Stores | Small key-value state | No | Schema and write policy |
| Stable Conversation ID | Within-thread continuity | No | The ID scheme |
| Knowledge files | Static reference material | No | The documents |
| Google Sheets or Airtable | Human-readable state | No | Write policy and lookups |
| Vector database over HTTP | Large knowledge bases | Via API | Embeddings and retrieval |
| Mem0 | Self-hosting requirement | Via MCP | The deployment |
| Vilix AI | Agents across many tools | Yes, over MCP | Nothing |

- **Data Stores:** Make's built-in key-value storage. Ideal for the last processed row ID, a run counter or a "did I already send this" flag. Lookups are by exact key, you own the schema, and it only lives inside Make.
- **Stable Conversation ID:** one ID per customer, ticket or run series. Good for customer-facing agents where one thread is one conversation.
- **Knowledge files:** uploaded documents the agent can search. Great for a refund policy or tone guide, but frozen: a run cannot add to them.
- **Google Sheets or Airtable:** the scenario writes a row at the end of a run and reads the latest row at the start of the next. Easy for people to open and check. Sheets get slow and fragile past a few thousand rows.
- **Vector database over HTTP:** Pinecone, Weaviate, Qdrant or Supabase through HTTP modules, when recall has to be by meaning. You own embeddings, chunking and tuning, and each recall costs Make operations.
- **Mem0:** open source with a self-host path and an MCP server. The pick when compliance says memory stays in your infrastructure. You run the deployment.
- **Vilix AI:** a cloud memory layer that follows your agents across every tool, not just Make.

## Vilix AI inside a Make scenario

Make's AI agent module accepts MCP tools, so the memory sits one level above your scenarios:

1. Connect Vilix AI to the agent as an MCP tool (setup per tool: https://vilix.ai/get-started).
2. At the start of a run, the agent calls `get_context`. Vilix AI returns your saved decisions, projects, tasks and user rules.
3. At the end of a run, the agent calls `save_turn` with a summary of what happened.
4. Every scenario that connects to the same account reads and writes the same memory. Every save carries a source label, so you can see which tool wrote what, and when two tools disagree the newest memory wins.
5. Retrieval is semantic plus keyword search, so recall finds what you meant, not only what you typed.

The same account connects to Claude, Codex, Cursor, OpenClaw, Hermes and any other MCP tool, so context follows you out of Make.

## A quick test

1. Connect your Make AI agent and Claude to the same Vilix AI account.
2. Have a Make run save a decision, for example "we send invoices on the 1st, not the 30th".
3. Open Claude and ask about it.
4. If Claude answers with the decision, the shared memory works.

## Agencies: one Vilix AI account per client

If you build Make automations for clients, give each client their own Vilix AI account, like a Gmail account that belongs to them. Every scenario you build for that client connects to their account, so the memory belongs to their business and nothing mixes between clients.

## Which one should you pick?

- Everything stays inside Make and the state is tiny: Data Stores.
- Customer-facing agents where one thread is one conversation: a stable Conversation ID.
- State people want to open and read: a sheet.
- A hard self-hosting requirement: Mem0.
- You re-brief every agent, every morning: one shared Vilix AI memory.

## FAQ

**Can Make scenarios remember data between runs?** Yes, with a store the next run reads: a Data Store, a Google Sheet or an external memory service. Runs are stateless by default, so memory is always an explicit write-then-read.

**Does the Conversation ID give the agent long-term memory?** It gives continuity within one thread, not durable cross-agent memory.

**Do I need a vector database?** Only when recall is semantic rather than exact. For known values like a last-processed timestamp, a Data Store is faster and cheaper.

## Links

- Blog post: https://vilix.ai/blog/best-ai-agent-memory-for-make-workflows
- Vilix AI: https://vilix.ai
- Get started: https://vilix.ai/get-started
