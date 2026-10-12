# Workflow 2: Solo Founder, Phone to Laptop

**For:** one developer building and marketing one product, often around a day job.

**Pains it removes:** the spec written on the phone that's gone by the evening, re-explaining the codebase to every new chat, product and marketing context scattered across tools.

## A typical day

| When | Where | With Vilix AI |
|---|---|---|
| 5:30 AM | ChatGPT on the phone | Plan the "team invites" feature by voice. The spec is saved to the **Northstar** project in Vilix AI |
| 12:30 PM | Claude on the work break | "Write the invite email copy for the feature I planned this morning." Claude pulls the spec with `get_context` |
| 8:00 PM | Claude Code on the laptop | "Implement the team invites feature." Claude Code loads the spec, the stack and the project rules before writing code |
| 10:00 PM | Cursor | Fixing a bug. Cursor sees the decision log ("invites expire after 7 days because of abuse risk") and doesn't "fix" it away |
| Weekend | ChatGPT | "Write a build-in-public post about this week." It has the whole week, from every tool |

## Step by step

1. Connect ChatGPT (from a desktop browser once), Claude, Claude Code and Cursor: [setup guide](../setup/README.md).
2. Create one project per product with project rules:
   ```
   Create a Vilix AI project "Northstar" with project rules:
   Stack: Next.js, Postgres (Supabase), Stripe. Postgres not SQLite: we need row level security.
   Never touch billing code without asking. Tests with Vitest.
   ```
3. Add your tasks: `create_task` for each feature, then move them with `update_task_state` (todo, doing, blocked, done). Every tool sees what's active.
4. Make memory automatic in each tool: ask "Set up Vilix AI to load and save context automatically" (it uses `get_setup_instructions`).
5. Use your tools normally. Planning on the phone and building on the laptop are now one continuous conversation.

## Prompts that work well

- *"Continue the feature I planned this morning."*
- *"Why did we pick Postgres?"* (returns the original reasoning, with the tool and date)
- *"What's blocked in Northstar right now?"*
- *"Summarize everything I did this week across all tools."*
