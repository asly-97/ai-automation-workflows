# Workflow 1: One Memory for Every Agent (Multi-Agent Operator)

**For:** people running always-on and scheduled agents across several tools: OpenClaw or Hermes on a VPS, Claude Code and Codex nightly jobs, ChatGPT scheduled tasks, n8n pipelines.

**Pains it removes:** agents that can't see each other, the Groundhog Day problem (repeating the same failed call), re-briefing every agent, and you acting as the "human router".

## The setup before

```
OpenClaw heartbeat (VPS, every 30 min) ─► its own MEMORY.md
Claude Code nightly job                 ─► nothing saved after the run
Codex nightly job                       ─► session gone when it ends
ChatGPT scheduled task                  ─► only sees its own thread
n8n                                     ─► pipes orders into the CRM
You                                     ─► copy-paste between all of them
```

## The setup after

```
          ┌───────────────── Vilix AI (one memory) ─────────────────┐
          │ runs from every agent, labeled by source + timestamp     │
          │ failed attempts with exact errors                        │
          │ project rules, skills, tasks, agent inbox                │
          └──────▲──────────▲──────────▲──────────▲──────────▲───────┘
                 │          │          │          │          │
          OpenClaw     Claude Code    Codex     ChatGPT     you (phone)
          heartbeat    nightly        nightly   scheduled   via inbox
```

## Step by step

1. **Connect every agent** to `https://api.vilix.ai/mcp` ([setup guide](../setup/README.md)):
   - OpenClaw and Hermes with a personal API key (Vilix AI → Agents → Create API key).
   - Claude Code: `claude mcp add --transport http --scope user vilix https://api.vilix.ai/mcp`
   - Codex: `codex mcp add vilix --url https://api.vilix.ai/mcp` then `codex mcp login vilix`
   - ChatGPT: Developer mode → create the Vilix AI app with OAuth.
2. **Create one project** for the system, with project rules ([examples](../config/project-rules-examples.md)):
   ```
   Create a Vilix AI project "Store Ops" with project rules:
   Heartbeat alerts only on changes. Coding agents never push to main.
   Every run ends with save_turn in the run record format.
   ```
3. **Add the run template** to every agent prompt: [`agent-run-template.md`](../prompts/agent-run-template.md). Use the ready versions for the [heartbeat](../prompts/heartbeat-agent.md) and [nightly coding jobs](../prompts/nightly-coding-job.md).
4. **Give each agent a clear `source` name**: `OpenClaw heartbeat`, `Claude Code nightly`, `Codex nightly`, `ChatGPT weekly report`. You can filter any search by it.
5. **Steer from your phone** with the [agent inbox](../prompts/inbox-handoff.md).

## What a night looks like

| Time | Agent | What happens with Vilix AI |
|---|---|---|
| 01:00 | Codex nightly | `get_context` finds the heartbeat's report: "refund webhook returned 500 three times". Fixes the webhook handler, saves the run with the root cause |
| 01:30 | OpenClaw heartbeat | `get_context` sees Codex fixed the webhook. Doesn't re-alert. Checks prices, saves its findings |
| 02:00 | Claude Code nightly | Picks the next task from the project, sees a FAILED record from Tuesday for the same task ("pagination broke the CSV export"), uses the approach marked WHAT WORKED |
| 07:30 | You, on your phone | Ask ChatGPT "what happened overnight?" It calls `get_context` and summarizes every agent's runs by source |
| 07:32 | You | "Tell the heartbeat to pause price alerts until Monday" → `message_agents`, picked up on its next run |

## Why it works

- `get_context` searches across **all** tools, so every agent starts with the team's knowledge, not just its own.
- `save_turn` keeps the **full run**, including failures, so mistakes are made once.
- Source labels and timestamps tell agents who said what and what's newest.
- Rules and skills live once in Vilix AI, so every agent follows the same playbook.
