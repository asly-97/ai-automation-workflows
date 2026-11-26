# Jira Automation Remembers the Rule. Vilix AI Remembers the Work

A Jira Automation rule carries no memory from one execution to the next. Variables and smart values built during a run end when that run ends. The next run starts from its trigger and the current state of your issues. Anything it knows about last time is what an earlier run wrote onto an issue or a project.

This guide is the technical companion to Agentic Pod EP67, "Jira Automation Remembers the Rule. Vilix AI Remembers the Work". It covers what a rule keeps while it runs, what survives after it finishes, and how to give a scheduled AI rule a real memory outside Jira: [Vilix AI](https://vilix.ai).

- Watch the episode: https://youtu.be/F6WgbdpphPA
- Read the full post: https://vilix.ai/blog/does-jira-automation-remember-between-runs/

## The problem

Inside one execution, a rule has real working memory. Smart values pull data from the trigger issue and from anything the rule looks up: fields, comments, dates, lists. The Create variable action lets you define your own smart value and reuse it in later actions and conditions in the same rule. That variable is always a string, and its scope is the flow that created it. It is a scratchpad for one execution: compose a summary, hold a calculated date, carry a value out of a branch. When the execution finishes, the scratchpad is discarded.

Three things survive after the rule finishes, each one Jira data rather than rule memory:

- The issues themselves. Any field a rule sets, comment it adds, or transition it performs stays on the issue, and a later run can read those fields like any other data. The issue becomes the storage.
- Entity properties. Jira stores key-value JSON data on entities such as issues, projects, and users. Automation includes a Set entity property action, and a later run can read a value back with a smart value such as {{issue.properties.usernameEmail}}, or through the REST API and JQL. Atlassian documents a maximum of 32,768 bytes per value, warns that anyone who can edit the entity can modify its property, and states that private or personal data should never be stored in entity properties. Concurrent writes are not merged; the latest saved value is retained.
- The audit log. Every execution is recorded so a person can inspect what ran and where it failed. The rule cannot read it back as context for its next decision.

A scheduled trigger running a JQL search offers one narrow option: only include issues that have changed since the last time the rule executed. It compares against when issues were last updated, and an issue the previous run itself updated counts as changed, because the rule's own edit moved its timestamp. That is a filter, not a memory. It tells the new run which issues changed, not what the previous run concluded about them, which ones it skipped and why, or what it promised next. Operators have asked Atlassian for the missing piece directly: feature request AUTO-891 asks for persistent smart values remembered from prior executions. The request exists because the product does not do it today.

## What a scheduled AI rule actually needs

Add an AI step and the gap changes character. The rule no longer just moves an issue between states. It reads a thread, weighs history, and decides: escalate, wait, reply, or leave it alone. The decision is the valuable part, and it is exactly what field values and small properties were not designed to hold. The next run needs the reasoning, not only the result: that the agent already asked this customer for logs twice, that a human took over the thread on Tuesday, that last week's summary was corrected.

## Every pain, and the Vilix AI fix

| Pain | How Vilix AI fixes it |
| --- | --- |
| Create variable values die at the end of every run | At the end of a run, the agent saves what it did and decided in Vilix AI with `save_turn` and a source label, so the next run starts from it |
| The next run has no record of prior decisions or why issues were skipped | At the start of a run, the agent calls `get_context` in Vilix AI and loads prior decisions, corrections, and the full conversation history behind them |
| Failed runs teach nothing and repeat silently | Failed runs are saved in Vilix AI too, so the next run can see the failure instead of repeating it |
| A correction made in one tool never reaches the rule | Correct something once in Vilix AI and the newest version is what every tool retrieves, because retrieval is recency-aware and the last write wins |
| Entity properties are small, per entity, and editable by anyone who can edit the entity | Memory belongs to your Vilix AI account, outside Jira, and you can list, edit, or delete memories, export everything, or wipe the account instantly |
| Standing escalation rules get re-explained per tool | Projects and project rules live in Vilix AI, and user rules live in Vilix AI, so every connected tool follows the same rules |
| Repeat triage workflows are rebuilt per tool | Repeat workflows become skills in Vilix AI, and agents can leave each other notes in the agent inbox |
| Memory is trapped inside one Jira site and one issue at a time | The same Vilix AI context is available to the scheduled rule in Jira, a session in Claude or Codex, and the next automation in n8n or Make, over MCP |
| Agencies run Jira automations for several clients | One Vilix AI account per client keeps each client's memory separate and portable |
| You want out | Export everything in a portable format or delete it anytime; Vilix AI is cloud hosted with zero infrastructure for you to run |

## Setup

1. Create a free Vilix AI account at https://vilix.ai/get-started (free forever plan; 7 day Pro trial, no credit card).
2. Connect your scheduled agent to Vilix AI over MCP with an API key as a Bearer token to https://api.vilix.ai/mcp, following the per-tool steps on the get-started page. Connect the AI tools your team plans work in (ChatGPT, Claude, Claude Code, Codex, Cursor, or any MCP client) to the same account.
3. Add a read step at the start of every scheduled run: call `get_context` in Vilix AI for the issue or customer before the agent decides anything.
4. Add a write step at the end of every run: save what the agent did and decided with `save_turn` in Vilix AI, with a source label like `jira-automation`. Save failed runs too.
5. Put standing rules (escalate after two unanswered asks, never re-ask for logs already received) in a Vilix AI project as project rules, and turn repeat triage flows into Vilix AI skills.

## Prompts

Start-of-run read (first step of every scheduled rule with an AI step):

```
Before you act, call get_context in Vilix AI for this issue.
Use the saved record: prior decisions, what was already asked, any human
takeover, and any corrections. Do not ask the customer for anything that
was already requested or answered in a saved conversation.
```

End-of-run write (last step of every run, success or failure):

```
Save this run to Vilix AI with save_turn, source "jira-automation":
the issue, what you decided (escalate, wait, reply, leave alone), why,
what you asked or promised, and what stays open. If the run failed,
save the failure and the reason.
```

Correction (run once, anywhere connected):

```
Correct the saved summary for this issue in Vilix AI: use the corrected
version from now on. The newest version wins for every tool and rule.
```

## FAQ

**Does a Create variable value persist to the next execution?**
No. A variable created with the Create variable action is a string smart value for use in later actions of the same rule execution. When the execution ends, the variable is gone. Carrying a value into a later run requires writing it to an issue field or an entity property and reading it back.

**Can a Jira Automation rule read its own audit log?**
No. The audit log records executions for people to inspect and debug. A rule has no action that reads a previous execution's log as input, so log history never becomes context for the next run.

**Does Jira Automation remember between runs if the rule runs on a schedule?**
The schedule changes how the rule starts, not what it remembers. A scheduled trigger can limit its JQL to issues changed since the last execution, a timestamp filter. The run still begins with no variables, no conclusions, and no record of its own prior decisions beyond what was written onto Jira data.

**Do I need to replace Jira Automation?**
No. Keep the rule. The pattern in this guide gives the rule's AI step one memory outside Jira through Vilix AI, so every run starts from the last run's reasoning instead of from zero.

**What does Vilix AI cost to start?**
The free plan is free forever. Full Pro can be tried for 7 days with no credit card required.

## Links

- Vilix AI: https://vilix.ai
- Get started: https://vilix.ai/get-started
- Full post: https://vilix.ai/blog/does-jira-automation-remember-between-runs/
- Episode: https://youtu.be/F6WgbdpphPA
