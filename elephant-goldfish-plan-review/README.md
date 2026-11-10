# What is Google's Elephant-Goldfish plugin, and how do you install it?

The practical guide behind the Agentic Pod episode **"What Is Google's Elephant-Goldfish Plugin? Review the Plan Before the Code"**. Google open-sourced the plugin its engineers use to get AI coding plans reviewed by agents that deliberately do not know the backstory.

Tagline from the repo: *"Design with an elephant. Review with a goldfish."*

## What it is

- An open-source plugin for AI coding assistants that helps you **design and review software before code is written**.
- Repo: [google/elephant-goldfish-model](https://github.com/google/elephant-goldfish-model). License: **Apache 2.0**. The README says it is **not an officially supported Google product**.
- **The elephant** is your main session. It holds the project context and drafts the specification.
- **The goldfish** are fresh reviewer sessions that never saw your conversation. They read the draft and the source material it references, so unstated assumptions and blind spots show up.
- **You** make the trade-offs and approve the specification before implementation.

## The seven commands

In Antigravity and Claude Code the commands use the `/egm:` prefix, for example `/egm:egm-brainstorm`. In Codex, pick the EGM skill from the skill picker.

| Command | What it does |
|---|---|
| `/egm-brainstorm` | Tests an idea and writes a concept brief (`concepts_brief.md`) |
| `/egm-draft` | Turns a brief or notes into a design document (`design_draft.md`) |
| `/egm-review` | Reviews a design with fresh agents, for example `/egm-review design_draft.md` |
| `/egm-resolve` | Works through the review findings so you agree on each change |
| `/egm-bootstrap` | Creates or updates documentation for an existing codebase |
| `/egm-review-loop` | Runs a configured group of reviewers for a specific task |
| `/egm-drift-report` | Compares checked-in documents with the current source code |

## Install

Claude Code:

```
claude plugin marketplace add google/elephant-goldfish-model
claude plugin install egm@egm
```

Codex CLI:

```
codex plugin marketplace add google/elephant-goldfish-model
codex plugin add egm@egm
```

Antigravity CLI:

```
agy plugin install https://github.com/google/elephant-goldfish-model
```

The README also lists the ChatGPT and Codex desktop apps, Antigravity App and IDEs, Claude Cowork and ChatGPT Work. Start a **new session** after installing.

## Try it on one real feature

1. Pick a small feature you have been putting off and write down the idea.
2. Run `/egm-brainstorm`, then `/egm-draft` to get `design_draft.md`.
3. Run `/egm-review design_draft.md` and read what the goldfish complains about.
4. Run `/egm-resolve`, decide each finding yourself, and approve the spec.
5. Only now let the agent write code.

## Gotchas

- The goldfish needs **fresh subagent contexts**. If your agent cannot create them, EGM stops the review instead of faking one.
- It **does not replace tests or human judgment**. It reviews the design, not whether the code runs.
- One independent analysis notes there is **no security-focused reviewer by default**. The reviewer personas are customizable, so you can add one.
- Not worth it for a tiny change. Use it for features that start from a loose idea.
- Google says thousands of its engineers use it daily. That is their reported adoption, not a measured defect-reduction rate.

## FAQ

**Why a reviewer that forgets?**
The session that wrote the plan cannot see its own blind spots. A reviewer with only the document has to ask "what does this mean?", and every unanswered question is an unstated assumption.

**Is it only for Google's tools?**
No. It supports Antigravity, Claude Code and Codex, among others.

**Do I still need to review the code?**
Yes. This is design review, and it is meant to sit before implementation, not replace it.

## Links

- Repo: https://github.com/google/elephant-goldfish-model
- Google Open Source Blog: https://opensource.googleblog.com/2026/10/introducing-elephant-goldfish-model-plugin.html
- Coverage: https://helpnetsecurity.com/2026/10/26/google-elephant-goldfish-model-plugin
- Episode: [Agentic Pod EP36 on YouTube](https://youtu.be/Rd8EqM2R2B8)
