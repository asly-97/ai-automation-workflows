# Relay.app Is Gone: Keep Your Workflow Memory Alive With Vilix AI

Relay.app shut down in 2026. Free accounts were deleted on August 15, 2026, and paying accounts on September 14, 2026. Per the company's own shutdown notice, accounts and all associated data were permanently deleted. Founder Jacob Bank rejoined Google as VP of Product for Chrome, and TechCrunch reported the end as a shutdown with a set of hires, not an acquisition.

This guide is the technical companion to Agentic Pod EP63, "Relay.app Is Gone. Vilix AI Keeps Your Workflow Memory Alive". It covers what actually died with Relay, why the same trap sits inside every automation tool, and how to put your agents' memory in a layer that survives its tools: [Vilix AI](https://vilix.ai).

- Watch the episode: https://youtu.be/evddGgI1QwM
- Read the full post: https://vilix.ai/blog/relay-app-is-gone-your-memory-should-survive/

## The problem

The migration conversation after Relay was all about workflows: export your triggers, rebuild your steps, find a new vendor. That is the easy part. A workflow is a recipe (trigger on this, send that, branch here), and a recipe fits in a JSON export.

What cannot be retyped is the ledger of how the recipe behaved:

- Which runs succeeded and which failed, and why
- The corrections applied after each bad run
- The decisions AI steps made on ambiguous inputs, and whether they turned out right
- The patterns in your data that only showed up after a thousand runs

Make's migration guide for ex-Relay users confirmed the export included workflows, sequences, run history, tables and MCP servers as JSON or CSV. But the export was only available while accounts were still active: 60 days for paying users, 30 days for free users, during a wind-down when most operators were rebuilding, not archiving. The predictable outcome: people exported the recipe and left the knowledge behind. After September 14, the knowledge stopped existing.

Relay raised $32.2 million in May 2024 and launched in 2021 aiming to become the new Zapier. Five years of funding and momentum still ended in a shutdown notice. The average lifespan of an automation tool in your stack is shorter than the lifespan of the knowledge it holds.

## This is not a Relay-specific flaw

- Your n8n execution history lives inside n8n's walls.
- Your Zapier task history lives inside Zapier.
- Your Make scenario logs live inside Make.

Wherever your automations run, your agents' memory is your vendor's data, and vendor data has an expiry date you do not control. Shutdowns are the dramatic version. Every vendor switch is the everyday version: the new system knows the steps, but not what the old system learned, so every mistake the old system had stopped making comes back.

## The test: what belongs outside your tools

If a new tool cannot function without it, it does not belong inside the old tool. By that test, an external memory layer should hold:

- Decisions and their outcomes
- Corrections after failures
- Learned patterns about your data and customers
- The full run history that lets an agent understand what "normal" looks like for your operation

## The fix: Vilix AI as the memory layer under your automations

Vilix AI is a cloud-hosted memory layer that lives outside any single tool. Every tool reads from it and writes to it over MCP, so the memory survives any individual tool dying, being replaced, or getting acquired.

| Pain | How Vilix AI fixes it |
| --- | --- |
| Run history deleted with the tool | Full conversation and run history lives in Vilix AI, not inside the automation platform |
| A rebuilt workflow starts from zero | Before a run, the agent calls `get_context` and loads the relevant decisions and outcomes |
| Failures and corrections are lost | After a run, the agent saves the exchange with `save_turn`, including failures, with a source label per tool |
| A fact changes and every tool disagrees | Correct it once in Vilix AI; the newest version wins for every connected tool |
| Standing instructions get re-explained weekly | Projects, project rules and user rules live in Vilix AI and apply across tools |
| Repeat workflows rebuilt per tool | Repeat workflows become skills in Vilix AI; agents can also leave each other notes in the agent inbox |
| Agencies run automations for many clients | One Vilix AI account per client keeps each client's memory separate and portable |
| You want out | Export everything or delete it anytime; Vilix AI is cloud hosted with zero infrastructure for you to run |

## Setup

1. Create a free Vilix AI account at https://vilix.ai/get-started (free forever plan; 7 day Pro trial, no credit card).
2. Connect each tool you use (ChatGPT, Claude, Claude Code, Codex, Cursor, n8n, Make, Zapier, or any MCP client) to the same Vilix AI account, following the per-tool steps on the get-started page.
3. Add one instruction to every agent and automation: call `get_context` before planning work, and `save_turn` with the outcome when the work finishes, including what failed and what you corrected.
4. Move your standing instructions into Vilix AI projects, project rules and user rules once, instead of pasting them into each tool.
5. For agency work, repeat with one Vilix AI account per client.

## Prompts

Before a run:

```
Before you plan anything, call get_context in Vilix AI for this workflow's history:
past failures, corrections, and the patterns learned about this data.
Follow the project rules and user rules you find there.
```

After a run:

```
Save this run to Vilix AI with save_turn and the source label for this tool.
Include what succeeded, what failed, the correction applied, and anything
a future run must not repeat. If a fact changed, save the new version so
the newest wins.
```

## FAQ

**Could Relay users have saved their run history?**
Technically yes, briefly. The shutdown notice gave free users 30 days and paying users 60 days to export, and Make's guide listed run history among the exportable data. In practice, exports during a wind-down get triaged: the recipe gets exported first and the history gets left behind. After September 14, 2026, the question became permanently unanswerable.

**Is this only a shutdown problem?**
No. Shutdowns are just the version with a deadline. Every migration between automation platforms resets the learned layer of your operation, whether or not the old vendor deletes anything. If you cannot take your agents' accumulated knowledge with you, you did not own it; you were renting it.

**What belongs in an external memory layer?**
Decisions and their outcomes. Corrections after failures. Learned patterns about your data and customers. The full run history that lets an agent understand what "normal" looks like for your operation.

**How much does Vilix AI cost to start?**
Vilix AI is free forever, with a 7 day Pro trial that asks for no credit card. Start at https://vilix.ai/get-started.

## Links

- Vilix AI: https://vilix.ai
- Get started: https://vilix.ai/get-started
- Full blog post: https://vilix.ai/blog/relay-app-is-gone-your-memory-should-survive/
- Episode: https://youtu.be/evddGgI1QwM
