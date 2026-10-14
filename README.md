# AI Automation Workflows

Real automation workflows I run day to day as a solo founder. Each folder is one complete workflow: the problem it solves, the architecture, the prompts, the config, and the things that broke along the way.

No toy demos. Everything here runs on a schedule and does real work.

## Workflows

| Workflow | What it does | Stack |
|---|---|---|
| [Shared memory for AI agents](./shared-memory-for-ai-agents) | Gives ChatGPT, Claude, Claude Code, Codex, Cursor, OpenClaw and Hermes one shared memory, so agents stop forgetting, stop repeating failed runs and follow one set of rules. Setup for 20+ tools, agent prompts, 5 workflows | Vilix AI, MCP, any AI tool |
| [What is cross-AI memory?](./what-is-cross-ai-memory) | Explains cross-AI memory and sets up one shared memory for ChatGPT, Claude, Claude Code, Cursor, Codex, OpenClaw and Hermes over MCP: the get_context / save_turn loop, shared rules, projects, skills and copy-paste prompts | Vilix AI, MCP, any AI tool |
| [Claude Code vs Codex](./claude-code-vs-codex) | Which AI coding agent to use, and how to use both: plan and debug in Claude Code, delegate to Codex, and keep one shared memory (decisions, rules, tasks, failures) across both over MCP. Setup commands, copy-paste prompts, FAQ | Claude Code, Codex, Vilix AI, MCP |
| [How to set up an MCP memory server](./mcp-memory-server-setup) | Connects ChatGPT, Claude, Claude Code, Codex, Cursor, OpenClaw and Hermes to one MCP memory server: OAuth vs API key per tool, setup commands, making memory automatic, the Northstar cross-tool test, copy-paste prompts, FAQ | Vilix AI, MCP, any AI tool |
| [ChatGPT memory vs Claude Projects](./chatgpt-memory-vs-claude-projects) | What ChatGPT memory and Claude Projects each remember, Claude memory and chat search, and how to keep one shared memory when you switch between ChatGPT, Claude, Cursor and Codex: the get_context / save_turn loop, newest-wins corrections, project rules, user rules, skills, agent inbox, setup, handoff test, prompts, FAQ | ChatGPT, Claude, Vilix AI, MCP |
| [Muse Wonder memory](./muse-wonder-memory) | What Meta's Muse Wonder and its Active Memory remember, the MemoryBench scoreboard, who leads which benchmark row, and how to give Muse, Claude, ChatGPT and Codex one shared memory over MCP: get_context / save_turn, newest-wins, user and project rules, projects, skills, agent inbox, setup, prompts, FAQ | Muse, Claude, ChatGPT, Codex, Vilix AI, MCP |
| [Stop copy-pasting AI context](./stop-copy-pasting-ai-context) | Stops you pasting the same context into ChatGPT, Claude, Cursor and Codex: why each AI app keeps its own memory, the get_context / save_turn loop, a Cursor to Claude handoff, newest-wins, user and project rules, skills, agent inbox, setup for every tool, the no-paste test, prompts, FAQ | ChatGPT, Claude, Cursor, Codex, Vilix AI, MCP |
| [Do ChatGPT scheduled tasks remember between runs?](./chatgpt-scheduled-tasks-memory) | What standalone, in-chat and event-triggered ChatGPT scheduled tasks keep between runs, why scheduled tasks repeat findings, and how every scheduled run in ChatGPT, n8n, Zapier and Make shares one memory: the get_context / save_turn run loop, saved failures, newest-wins, project and user rules, skills, agent inbox, setup, the two-run test, prompts, FAQ | ChatGPT, n8n, Zapier, Make, Vilix AI, MCP |
| [AI visibility content pipeline](./ai-visibility-content-pipeline) | Writes 3 draft articles every morning for one real question people ask AI tools, I approve one, it publishes, indexes, and tracks rankings | Claude, Notion, Ghost, Vilix AI, IndexNow |

More coming. Each one gets its own folder with the same structure:

```
workflow-name/
  README.md     problem, architecture, setup, troubleshooting
  prompts/      the agent prompts
  config/       schemas and data formats
```

## Using these

Everything is MIT licensed. Copy, adapt, break it. If something is unclear or you hit an issue, open an issue and I'll answer it.
