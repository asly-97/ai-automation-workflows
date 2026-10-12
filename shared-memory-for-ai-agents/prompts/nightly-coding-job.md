# Nightly Coding Job Prompt (Claude Code or Codex, headless)

```bash
claude -p "$(cat nightly-job.md)"
# or
codex exec "$(cat nightly-job.md)"
```

`nightly-job.md`:

```
Nightly job for the <project> repo.

1. get_context("nightly job for <project>: open tasks, failed builds, heartbeat
   reports, inbox"). Read the project rules and any messages for "Claude Code"
   or "Codex".
2. get_project for <project>. Pick the highest-priority task in state "todo" that
   isn't blocked. update_task_state to "doing".
3. Check past runs for this task. If an approach is marked FAILED, don't repeat it.
4. Do the work. Follow the project rules exactly.
5. save_turn with source "Claude Code nightly" (or "Codex nightly"):
   RESULT / TASK / WHAT I CHANGED (files) / WHAT FAILED (exact errors) /
   DECISIONS (and why) / NEXT RUN SHOULD.
6. update_task_state to "done" or "blocked" (with the reason in the task body).
```
