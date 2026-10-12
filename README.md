# AI Automation Workflows

Real automation workflows I run day to day as a solo founder. Each folder is one complete workflow: the problem it solves, the architecture, the prompts, the config, and the things that broke along the way.

No toy demos. Everything here runs on a schedule and does real work.

## Workflows

| Workflow | What it does | Stack |
|---|---|---|
| [Shared memory for AI agents](./shared-memory-for-ai-agents) | Gives ChatGPT, Claude, Claude Code, Codex, Cursor, OpenClaw and Hermes one shared memory, so agents stop forgetting, stop repeating failed runs and follow one set of rules. Setup for 20+ tools, agent prompts, 5 workflows | Vilix AI, MCP, any AI tool |
| [What is cross-AI memory?](./what-is-cross-ai-memory) | Explains cross-AI memory and sets up one shared memory for ChatGPT, Claude, Claude Code, Cursor, Codex, OpenClaw and Hermes over MCP: the get_context / save_turn loop, shared rules, projects, skills and copy-paste prompts | Vilix AI, MCP, any AI tool |
| [Claude Code vs Codex](./claude-code-vs-codex) | Which AI coding agent to use, and how to use both: plan and debug in Claude Code, delegate to Codex, and keep one shared memory (decisions, rules, tasks, failures) across both over MCP. Setup commands, copy-paste prompts, FAQ | Claude Code, Codex, Vilix AI, MCP |
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
