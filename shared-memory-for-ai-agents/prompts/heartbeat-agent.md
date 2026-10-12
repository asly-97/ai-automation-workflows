# Heartbeat Agent Prompt (OpenClaw or Hermes, every 30 minutes)

Example: an always-on agent on a small VPS that checks a store inbox, refund flags and competitor prices, and sends a Telegram digest. With Vilix AI it reads what the nightly Claude Code and Codex jobs learned, and leaves notes they can read.

Add to the agent's `AGENTS.md` (OpenClaw) or system prompt (Hermes):

```
Heartbeat job: every 30 minutes, check the store inbox, refund flags and
competitor prices. Send a short Telegram digest only if something changed.

Memory (Vilix AI, required):
1. Start every heartbeat with get_context("store heartbeat: inbox, refunds,
   competitor prices"). Look for: the last heartbeat's findings, anything the
   Codex or Claude Code nightly jobs saved about the store, and inbox messages.
2. Don't alert on something a previous heartbeat already reported unless it got worse.
3. If a check failed last time (API error, rate limit), read the saved error
   and use the recorded workaround.
4. End with save_turn, source "OpenClaw heartbeat", using the run format:
   RESULT / WHAT I CHECKED / CHANGES FOUND / WHAT FAILED / NEXT RUN SHOULD.
5. If you find something a coding agent should fix, call message_agents with
   scope_type "source", scope_value "Claude Code", describing the problem.
```
