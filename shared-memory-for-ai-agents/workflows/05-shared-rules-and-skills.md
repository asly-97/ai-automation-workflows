# Workflow 5: One Set of Rules and Skills for Every Tool

**For:** anyone maintaining CLAUDE.md, AGENTS.md, MEMORY.md, `.github/copilot-instructions.md`, Cursor rules and ChatGPT custom instructions at the same time.

**Pain it removes:** instruction files that drift apart, so different tools follow different versions of your rules.

## Three layers, all in Vilix AI

| Layer | Tool | Scope | Example |
|---|---|---|---|
| Personal rules | `add_user_rule` | You, everywhere | "Short answers. No emojis. Always show the command before running it." |
| Project rules | `create_project` / `update_project` | One product or client | "Postgres not SQLite. Never touch /billing. Vitest for tests." |
| Skills | `create_skill` / `get_skill` | Reusable procedures | `release-notes`, `deploy-checklist`, `new-client-onboarding` |

## Migrate in 10 minutes

1. Open your existing CLAUDE.md, AGENTS.md and custom instructions side by side.
2. In any connected tool:
   ```
   Here are my current instruction files. Merge them into Vilix AI:
   personal preferences as user rules, project-specific items as project rules
   for "<project>", and step-by-step procedures as skills.
   ```
3. Replace each tool's instruction file with the short block from `get_setup_instructions` (it tells the tool to load Vilix AI context first).
4. From now on, change a rule once: *"Update the Northstar project rules: we moved from Vitest to Jest."* Every tool follows the new rule on its next read.

## Skills in practice

- Agents check `list_skills` before non-trivial work and load the matching one with `get_skill`.
- Improve a skill once (`update_skill`) and every tool uses the improved version.
- Global skills apply everywhere; pass a `project_id` to keep a skill inside one project.

Examples: [`../config/skill-examples.md`](../config/skill-examples.md).
