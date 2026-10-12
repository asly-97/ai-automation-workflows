# Agent Run Template (any tool, any schedule)

Put this at the top of every scheduled or headless agent prompt: Claude Code (`claude -p`), Codex, OpenClaw, Hermes, ChatGPT scheduled tasks, n8n AI nodes. It's what turns isolated runs into a team that learns.

```
You share one memory with my other agents through Vilix AI (MCP).

BEFORE doing anything:
1. Call get_context with a one-line description of this run's job
   (e.g. "nightly dependency update for the Northstar repo").
2. Read what comes back: past runs of this job, related runs by other
   agents (check the source labels), failed attempts and their errors,
   my rules, the project rules, and any inbox messages.
3. If an inbox message is addressed to you, follow it and call ack_message.
4. If a skill matches this job (list_skills / get_skill), follow it.
5. Never repeat an approach that a previous run marked as FAILED unless
   you have a concrete new reason it will work now.

DO the job.

AFTER the job, call save_turn with:
- user_message: the job description for this run
- source: "<this agent's name, e.g. OpenClaw heartbeat, Codex nightly>"
- assistant_message, in this format:
    RESULT: success | partial | failed
    WHAT I DID: ...
    WHAT FAILED (exact error, settings, endpoint): ...
    WHAT WORKED: ...
    DECISIONS (and why): ...
    NEXT RUN SHOULD: ...
If a task in the project changed state, call update_task_state.
```

Why each part matters:

| Step | Fixes |
|---|---|
| `get_context` first | The re-explaining tax and agents that can't see each other |
| Respect FAILED records | The Groundhog Day problem (repeating the same broken call) |
| Inbox + `ack_message` | Steering agents from your phone without editing prompts |
| Skills | Same procedure in every tool, no drifting instruction files |
| `save_turn` with WHAT FAILED and DECISIONS | Lost decisions; next run starts smarter |
