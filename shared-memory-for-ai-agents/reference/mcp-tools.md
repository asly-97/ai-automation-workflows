# Vilix AI MCP Tools Reference

Every tool below is available to any AI tool connected to `https://api.vilix.ai/mcp`. Your account is identified by the OAuth sign-in or API key, so no tool takes a user ID.

## Memory and search

| Tool | What it does | Key parameters |
|---|---|---|
| `get_context` | Loads recent messages plus semantically related past turns from **every** connected tool. Call it before answering or at the start of an agent run. On a new chat it also returns your personal rules, active projects and their state, and any inbox messages. | `user_prompt` (the task or question), `chat_id` (null on a new chat), `attachment_context` (short summary of any file or screenshot) |
| `save_turn` | Saves the exchange: what was asked and what was answered or done. Retries are deduplicated server-side. | `user_message`, `assistant_message`, `source` (the tool's name, e.g. `Claude Code`, `Codex`, `OpenClaw`), `chat_id`, `turn_id`, `attachment_context` |
| `search_semantic` | Search past messages by meaning. | `query`, `source` (optional filter), `limit` (up to 25) |
| `search_keyword` | Exact keyword / full-text search (IDs, error strings, names). | `query`, `source`, `limit` |
| `recent_messages` | Your latest messages, optionally from one tool. | `source`, `limit` |
| `get_conversation` | Every message of one conversation, oldest first. | `conversation_id` |

Every retrieved item carries a timestamp. When two items disagree, the newest one is the current truth.

## Projects and tasks

| Tool | What it does |
|---|---|
| `create_project(title, description?, project_rules?)` | One project per product, client or initiative. `project_rules` = stack, conventions, never-do-X items every tool follows on this project |
| `list_projects(status?)` | active, paused or done |
| `get_project(project_id)` | The project with all its tasks |
| `update_project(project_id, title?, description?, project_rules?, status?)` | Change rules once, every tool sees the new version |
| `create_task(project_id, title, body?, priority?)` | priority: low, normal, high |
| `get_task(task_id)` | One task |
| `update_task_state(task_id, state, plan_status?)` | state: todo, doing, blocked, done |
| `update_task(task_id, ...)` | Edit title, body, priority, state |

## Personal rules

| Tool | What it does |
|---|---|
| `add_user_rule(text)` | A standing instruction loaded into every new chat in every tool (style, format, never-do items) |
| `list_user_rules()` | All your rules in order |
| `update_user_rule(rule_id, text)` | Edit a rule once, it changes everywhere |

## Skills

| Tool | What it does |
|---|---|
| `create_skill(name, description?, body?, project_id?)` | Save a reusable procedure. Global if no `project_id` |
| `list_skills(project_id?)` | The skill index every agent can check before a task |
| `get_skill(ref)` | Load a skill by name or ID and follow it |
| `update_skill(skill_id, ...)` | Improve a procedure once for every tool |

## Agent inbox

| Tool | What it does |
|---|---|
| `message_agents(body, urgency?, scope_type?, scope_value?, ttl_hours?)` | Post a message to your own agents. It's delivered inside the `inbox` field of their next Vilix AI read. `urgency`: `normal` (once per chat) or `urgent` (every read until acknowledged). `scope_type`: `global`, `project`, `task` or `source` (e.g. only `OpenClaw`). `ttl_hours` up to 168 |
| `ack_message(message_id)` | The agent confirms it handled the message |

## Setup and help

| Tool | What it does |
|---|---|
| `get_setup_instructions(source?)` | Returns the exact instruction block for the calling tool and where to paste it, so memory loads and saves automatically |
| `get_vilix_guide(topic?, tool?)` | In-chat help: overview, connect, using daily, make it automatic |
