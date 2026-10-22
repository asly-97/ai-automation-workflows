# How do you give OpenAI Codex a memory with Vilix AI?

The full technical guide behind the Agentic Pod episode **"How Do You Give OpenAI Codex a Memory?"**: why Codex starts every session cold, how to connect it to Vilix AI over MCP in two commands, how the Codex CLI, the IDE extension and the ChatGPT desktop app share one config, and how get_context and save_turn bring your saved decisions, projects, tasks and user rules into every Codex task.

Blog post: https://vilix.ai/blog/use-vilix-with-openai-codex · Start free: https://vilix.ai/get-started

## The problem

Codex is a coding agent, not a chat app. It only knows what is in front of it, so every session starts without your stack, your conventions, or the decisions you made in ChatGPT, Claude or Cursor. The usual workaround is pasting the same project brief every morning.

## The fix: Vilix AI as Codex's memory

Vilix AI is one shared memory over MCP (the plug that lets an AI app use outside tools). Connect Codex once and it can retrieve what your other tools saved.

## Setup

1. Register Vilix AI as an MCP server in Codex:

```
codex mcp add vilix --url https://api.vilix.ai/mcp
```

2. Sign in with OAuth (no API key needed):

```
codex mcp login vilix
```

   Or open the IDE extension's MCP settings and choose Authenticate for the server. Complete the browser sign-in. You can revoke access at any time.

3. Add the Vilix AI instructions so Codex calls the memory tools: `get_context` with the latest user message before composing a reply, `save_turn` with the exact exchange afterwards, reusing `chat_id` within a conversation.
4. Verify that the Vilix AI tools are listed in Codex and that both calls succeed. Registering the server does not make the calls happen; the instructions do.

## One config for CLI, IDE and ChatGPT desktop

The ChatGPT desktop app, the Codex CLI and the IDE extension share user-level MCP configuration in `~/.codex/config.toml`. A server configured there is available to all of them. Project-scoped settings in `.codex/config.toml` apply to that project and can depend on trust settings. If you already added Vilix AI in the ChatGPT desktop app, check the existing list before running `codex mcp add` again.

## What Codex gets back

- Relevant saved context: architecture decisions, constraints and rules saved from your other connected tools.
- Your projects and tasks, so "pick up the billing task" means something concrete.
- Your user rules (short personal directives such as "always show the test first"), delivered with the context.
- Every save carries a source label, so you can see which tool wrote what, and the newest memory wins when two notes disagree.

## A two-minute test

1. Save a decision in Claude, for example "we dropped library X because it broke the build".
2. Ask Codex about it in a new session.
3. If Codex answers with the decision, the loop works.

## Agencies: one Vilix AI account per client

Give each client their own Vilix AI account, like a Gmail account that belongs to them, and connect Codex with that account. Each client's decisions stay with that client, and you never build memory from scratch.

## Plans

The free plan has no time limit and Pro includes a 7-day trial.

## Links

- Blog post: https://vilix.ai/blog/use-vilix-with-openai-codex
- Vilix AI: https://vilix.ai
- Get started: https://vilix.ai/get-started
