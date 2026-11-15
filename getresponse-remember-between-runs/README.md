# Does GetResponse remember between runs?

The practical guide behind the Agentic Pod episode **"Does GetResponse Remember Between Runs? Vilix AI Remembers the Work It Forgets"**.

Watch: https://youtu.be/mDB-OtabWAU

Read the full post: https://vilix.ai/blog/does-getresponse-remember-between-runs/

## The problem

A GetResponse contact can sit in your account with its tags, custom fields, and a 1 to 5 engagement score, and the next workflow run still starts knowing nothing about the work already done on it. GetResponse remembers the contact perfectly. It remembers the work not at all. A weekly re-engagement job that skipped a contact for a good reason last week cannot see that reason this week, so it can repeat a decision that was already thrown out.

## What a contact actually keeps

- Tags currently assigned. Tags accumulate until removed: a product interest, a customer status, who started a workflow. A tag records that a label applies, never why it was applied or what was decided before it.
- Custom fields currently stored: name, plan, signup source, or preference. A form, an import, an API call, or a workflow sets them, and every later write replaces the earlier value. The contact card shows the latest value, never the history.
- The score currently on the contact. Workflow scoring adds or subtracts points and branches the path. Engagement scoring is a separate 1 to 5 read that cannot explain itself.

## Inside one run, and between two workflows

- Inside one run: conditions admit a contact (tag assigned, field changed, score given), actions send, tag, score, copy, move, or remove, and Wait elements resume the contact at the right element. That resume is workflow progress, not memory.
- Between workflows: workflows share no working state. A second workflow sees only what the first one wrote back: a tag, a field, a list move, or a score change.
- If nobody wrote the conclusion down, it is unrecoverable for the next workflow. Forcing reasons into custom fields makes it worse: fields multiply, values get overwritten, and only the latest conclusion survives.

## Every pain, and the Vilix AI fix

| Pain | Why it happens | The Vilix AI fix |
|---|---|---|
| The label is there, the reason is gone | A tag records that a label applies, never why it was applied or what was decided before it | At run start the workflow calls get_context in Vilix AI and loads who was already reviewed and every decision already made |
| Custom fields keep only the latest value | Every later write replaces the earlier value, so the history of values is lost | Vilix AI stores full conversation history, not just facts: the reasoning and the failures too, not only the latest conclusion |
| The score cannot explain itself | Engagement scoring is a 1 to 5 read, and workflow scoring only adds or subtracts points | Vilix AI saves why a contact was scored, skipped, or chosen, with save_turn at the end of every run |
| Between workflows there is no shared working state | The next workflow sees only what was written back to the contact record | The same Vilix AI memory is shared over MCP with every tool and workflow, so the second workflow reads what the first one saved |
| The weekly job repeats a discarded decision | No note on why a contact was skipped, which angle was tried and rejected, or what was planned next | Failures are saved in Vilix AI too, so the next run skips what already failed. Newest save wins |
| Standing rules get lost between runs | GetResponse has nowhere to keep project rules or repeat workflows | Projects and project rules, plus user rules, live in Vilix AI. Skills hold the repeat workflows, and the agent inbox holds notes for the next agent |

Vilix AI is cloud hosted, so there is no database or server to run. It has a free forever plan and a 7 day Pro trial with no credit card, and you can export everything or delete it anytime: https://vilix.ai

## Setup: the two-call loop on every run

1. Connect Vilix AI to your workflow tool over MCP: https://vilix.ai/get-started (under two minutes per tool).
2. At the start of every run, before reviewing contacts, call get_context to load who was already reviewed, the decisions, and the failures.
3. Do the work in GetResponse as usual: review the quiet contacts, pick who gets a new angle, and decide who should be left alone.
4. At the end of the run, call save_turn with a source label on every save, recording each contact reviewed, what was decided, which angle was tried, and whether it succeeded or failed.
5. Keep standing rules in Vilix AI projects and project rules, and repeat workflows as skills, so the next run and the next tool follow the same rules. Use one Vilix AI account per client so memories never mix.

## Prompts

Run start:

```
Before reviewing contacts, call get_context in Vilix AI. List the contacts already reviewed, the decision made for each one, any angle already tried and rejected, and any failed actions. Skip every contact that already has a completed decision. Work only on contacts with no completed decision.
```

Run end:

```
Call save_turn in Vilix AI with a source label for this workflow. Save: each contact reviewed, what was checked, the decision, the angle tried, and whether the action succeeded or failed. Save failures too, with the reason. Save who should be left alone and why.
```

Second workflow handoff:

```
Call get_context in Vilix AI and read what the first workflow saved about these contacts. Do not repeat any decision it already made or any angle it already rejected. Continue from its decisions and save your own results back with save_turn when you finish.
```

## FAQ

**Does GetResponse remember between runs?**
The contact record remembers: tags currently assigned, custom fields currently stored, and the score right now. It does not remember the work: why a contact was skipped, which angle was tried and rejected, or what was planned next.

**What do custom fields keep?**
Only the latest value. A later write replaces the earlier one, so the contact card never shows the history of values.

**What does a tag record?**
That a label applies. Never why it was applied or what was decided before it.

**How does Vilix AI help GetResponse workflows?**
get_context at run start loads the full history of reviewed contacts, decisions and failures. save_turn at run end saves the new work back with source labels, newest wins, shared over MCP with every connected workflow.

## Links

- Episode: https://youtu.be/mDB-OtabWAU
- Full post: https://vilix.ai/blog/does-getresponse-remember-between-runs/
- Vilix AI: https://vilix.ai
- Connect Vilix AI: https://vilix.ai/get-started
