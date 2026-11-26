# Pydantic Pax: a terminal coding agent you can rebuild

Pydantic released Pax on November 25, 2026, alongside a beta of the Pydantic AI Harness. Pax (`pydantic-ai-pax`, version 0.75.0 on PyPI) is the official Pydantic AI coding agent for your terminal, built on the Harness SDK. The built in agent is `Coder()`, a bundle of Harness tools and behaviors for coding, and every part of Pax is a plugin. It runs any model, and `--agent` runs your own Pydantic AI agent in the Pax terminal UI.

Watch the episode: https://youtu.be/7d2US4bPtU4 (Agentic Pod EP68)

Primary source: the Pydantic launch post, https://pydantic.dev/articles/pax-harness-beta

## What is what

- **Pydantic AI** is the agent framework: the loop, models from every major provider, tools, and structured output.
- **Pydantic AI Harness** is the official capability library on top of it, with more than 100 reusable capabilities you can add to any agent: file and shell tools, sandboxes, guardrails, sub-agents, context management, Agent Skills, and more.
- A **capability** is a bundle of tools, instructions, and hooks. `Coder()` is the coding capability: it turns any agent into a coding agent.
- **Pax** is the terminal UI in front of `Coder()`. It is both a daily driver coding agent and a showcase of what the Harness SDK can build.

## Install and first run

You need Python 3.11 or newer, and an API key (or local setup) for the model you want to run. Pax brings no model of its own.

Try it without installing:

```bash
uvx pydantic-ai-pax
```

Or install it to keep:

```bash
uv tool install pydantic-ai-pax
pax
```

### The macOS gotcha

macOS ships with an unrelated `/bin/pax` archiver installed by default. If `pax` does not start Pax right after installing, open a new terminal (or run `rehash` in zsh / `hash -r` in bash) so your shell picks up the new PATH entry, or run `pydantic-ai-pax` directly.

### First session checklist

1. Start Pax in a small, well tested repo you know well.
2. Ask it to explain one file and how it is used. Check the answer against the code yourself.
3. Ask for one small edit, then run the project tests.
4. Press Ctrl+X, then Ctrl+S for speculative execution if you want Pax to start safe file reads while the model is still streaming.
5. Note that code mode is on by default: the model writes a short Python snippet that searches and reads files and returns only what matters, instead of passing tool results back and forth turn by turn.

## Plugins: everything is a plugin

A Pax plugin is a capability one layer up: tools and hooks for the agent, plus terminal pieces like `/commands`, status bar items, and logins.

- Open `/plugins` in Pax to toggle `coder`, `sandbox`, `observability` (Pydantic Logfire tracing), `mcp`, and the rest on or off.
- Turn the coding agent itself off with `/plugins disable coder` and you have a clean chat client with no file or shell tools.
- Any Pydantic AI capability works as a plugin by its import path, and Pax can write new plugins for itself when you ask.

## Coder() in your own code

`Coder()` is not locked inside Pax. In plain Python, add it to any agent:

```python
from pydantic_ai import Agent
from pydantic_ai_harness import Coder

agent = Agent('anthropic:claude-sonnet-5-5', capabilities=[Coder()])
```

With standalone `Coder()`, add `Skills()` alongside it if you want Agent Skills loaded, the way the default Pax coder plugin does.

## Bring your own agent with --agent

`--agent` replaces the built in agent with any Pydantic AI `Agent` from your codebase. Pax then loads no plugins and runs your agent as-is: it keeps your agent's model, tools, and instructions, and gives it the full terminal (`--resume`, `--worktree`, `-p`, and `-m`).

```python
# my_agents.py
from pydantic_ai import Agent
from pydantic_ai.capabilities import LocalWorkspace
from pydantic_ai_harness import Coder

reviewer = Agent(
    'openai:gpt-6.1-sol',
    instructions='You review diffs against our internal engineering guidelines.',
    capabilities=[LocalWorkspace('.'), Coder()],
)

@reviewer.tool_plain
def get_guideline(topic: str) -> str:
    """Look up an internal engineering guideline by topic."""
    ...
```

```bash
pax --agent my_agents:reviewer
pax -a my_agents:reviewer -p "Review the diff on this branch against our guidelines"
```

The point: build an agent in your repo, test it in a real terminal, and ship the exact same `Agent` object in your service.

## Sandboxes and workspaces

File and shell tools talk to a workspace, so the same agent runs on your machine or in a cloud sandbox without changing tool code.

- In Pax, swap where a session runs before the first prompt: `/sandbox e2b` (or boxd, daytona, opensandbox, modal, render, sprites, kubernetes) for a cloud sandbox, or `/sandbox local smol` / `/sandbox local boxlite` for a local micro VM.
- Pax syncs your branch and uncommitted diff while every edit, build, and `!` command runs inside the sandbox, and `/resume` reconnects automatically.
- On proxy enabled providers (e2b, boxd, daytona, opensandbox), your GitHub token never enters the sandbox; the egress proxy injects it on outbound GitHub requests.
- In code, swap `LocalWorkspace('.')` for a sandbox capability such as `E2BSandbox`, and `Coder()`'s tools run there instead.

## Migrating from another coding agent

- `/resume` shows your existing Claude Code and Codex sessions next to Pax's own, or import one directly with `pax --resume-claude` or `pax --resume-codex` and keep going on any model.
- `repo_context` reads `AGENTS.md` and `CLAUDE.md`, and the default coder plugin loads Agent Skills from `.agents/skills` and `.claude/skills`.

## Gotchas and limits (read before you rely on it)

- **Beta, API not frozen.** Harness 1.0 is expected in about a month. Until then the public API can still change, so pin versions and expect small plugin tweaks.
- **Computer use is not in Pax yet.** The Harness can give an agent built with the SDK a full desktop with `ComputerUse()` on Docker or a sandbox today. In Pax, computer use arrives in a later release as `/computer-use`, run in a new conversation before `/sandbox PROVIDER`, and it needs `/permissions full`.
- **Crash recovery is not in Pax yet.** Durable execution that survives crashes and restarts is a Harness capability for agents built with the SDK. The launch post states plainly that it is not in Pax yet.
- **Benchmarks are vendor claims.** The launch post reports single runs from November on Harness-Bench SWE81 (81 tasks, eight models): Pax averaged 58.5, narrowly ahead of OpenCode at 58.2, deepagents-code at 57.6, and pi at 57.3. Pax cost 9.21 dollars a run on GPT-6.2 Sol versus Codex at 21.05 dollars, and 24.61 dollars on Claude Opus 5.5 versus Claude Code at 25.10 dollars. A relaxed rescore was applied identically to every harness, and Pydantic promises the full methodology and reruns at 1.0. Treat the numbers as a reason to test Pax on your own repo, not as a settled ranking.
- **The macOS `/bin/pax` clash** above is the most likely first run failure. Fresh terminal, rehash, or the long binary name fixes it.

## Who it is for

- Python teams already building on Pydantic AI who want a terminal agent that matches their stack.
- Builders who want one coding agent that runs any model and can be reshaped with plugins.
- Anyone who wants to develop an agent in their own repo and test the exact same agent in a real terminal before shipping it.

## Links

- Launch post: https://pydantic.dev/articles/pax-harness-beta
- Episode: https://youtu.be/7d2US4bPtU4
