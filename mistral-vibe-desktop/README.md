# Mistral Vibe Desktop: parallel coding agents in one window

The practical guide behind the Agentic Pod episode **"Mistral Vibe Desktop: Parallel Coding Agents in One Window"**.

Watch: https://youtu.be/mMJ77-qpV-s

Mistral launched Vibe Desktop on November 17, 2026, for macOS and Windows, with Linux coming soon. It runs local agents on your machine and remote agents in a cloud sandbox at the same time, several tasks at once in one window, where you watch them side by side, review diffs, and track pull requests. Git worktrees keep parallel tasks in separate copies of the project files. Vibe, formerly Le Chat and renamed in May 2026, has Chat, Work, and Code modes, and the desktop app uses the Vibe CLI engine locally. Paid users get Mistral Large 4, called Le Chonk, a 1 trillion parameter open weight model with weights published October 29, as the default. This guide covers what shipped, how parallel agents stay separate, how to try it step by step, pricing, and the gotchas.

## What shipped

- Vibe Desktop is a desktop app from Mistral, out now for macOS and Windows. Linux is coming soon.
- Vibe started as Le Chat. Mistral renamed it to Vibe in May 2026.
- The desktop app uses the Vibe CLI engine when it runs agents locally. The desktop window is the friendly face, and the CLI engine does the work underneath.
- Local agents run on your machine and remote agents run in a cloud sandbox, at the same time, in the same window.

## Parallel agents, worktrees and modes

- Start several tasks at once: for example a bug fix, a test update, and a small feature. One agent runs locally, another runs remotely, and you watch each one work side by side.
- Review the exact lines each agent changed (the diffs) and track pull requests without leaving the window.
- Git worktrees keep the tasks separate. A worktree gives each task its own copy of the project files, so one agent cannot overwrite another one mid task.
- Three modes set how hands on you are, per task:
  - Chat: questions and planning.
  - Work: bigger jobs you hand off and watch.
  - Code: sitting close to the edits and reviewing every change.

## The model: Mistral Large 4 (Le Chonk)

- Paid users get Mistral Large 4, called Le Chonk, as the default model in Vibe.
- It has 1 trillion parameters and is an open weight model. Mistral published the weights on October 29, so people can download the weights and run it themselves.
- Vibe connects to more than 100 tools, so agents can reach the apps and services around your code, not just the files.

## Pricing

- Free tier to start.
- Pro: $14.99 per month.
- Team: $24.99 per user per month.
- Europe deal: Pro is free for 3 months with the code LECHONK. The first batch of codes ran out and Mistral restocked them.

## How to try it, step by step

1. Download Vibe Desktop for macOS or Windows and open it.
2. Start one task in Chat mode: ask a question or plan a small job.
3. Start a second task beside it, for example in Work mode, and watch both run at the same time, one local and one in the cloud sandbox.
4. Review the diffs for each task in the same window, and track the pull requests there.
5. If you are in Europe, apply the code LECHONK for 3 free months of Pro.

## Gotchas and limits

- Linux is still waiting. The launch is macOS and Windows, with Linux coming soon.
- Parallel agents still need a human reviewer. Three agents can make three wrong changes just as fast as three right ones. The diff view helps, but you are still the reviewer.
- The Europe deal ends after 3 months, then the normal Pro price applies.
- If you only ever run one task at a time, you will not feel the gain. The win is for mornings with a pile of tasks waiting.

## FAQ

**What is Mistral Vibe Desktop?**
A desktop app from Mistral, launched November 17, 2026, for macOS and Windows, that runs local and remote coding agents at the same time in one window, with diffs and pull requests reviewed in that same window.

**How do parallel agents avoid overwriting each other?**
Git worktrees. Each task gets its own copy of the project files, so agents edit separate copies and cannot overwrite each other mid task.

**What are Chat, Work, and Code modes?**
Chat is for questions and planning, Work is for bigger jobs you hand off and watch, and Code is for sitting close to the edits and reviewing every change.

**What is Le Chonk?**
The nickname for Mistral Large 4, a 1 trillion parameter open weight model. Mistral published the weights on October 29, and paid Vibe users get it as the default.

**What does Vibe cost?**
Free to start, Pro at $14.99 per month, and Team at $24.99 per user per month. In Europe, Pro is free for 3 months with the code LECHONK (first batch out, restocked).

**Is Vibe Desktop on Linux?**
Not yet. The launch is macOS and Windows, with Linux coming soon.

## Links

- Episode: https://youtu.be/mMJ77-qpV-s
