# Parallel Code: race your coding agents in separate worktrees

Parallel Code is a free, open source (MIT license) Electron desktop app for macOS and Linux. It launched on November 21, 2026. It runs terminal coding agents side by side, and every task gets its own branch and isolated Git worktree, so agents never edit the same files at the same time.

Project: https://parallelcode.app
News source: https://vibecodingwiki.com/news
Author: Johannes Millan (GitHub johannesjo), author of Super Productivity
Episode: EP60 of Agentic Pod, https://youtu.be/-WLNctyi8_c

## What it does

- Runs six terminal coding agents side by side: Claude Code, Codex CLI, Copilot CLI, OpenCode, Gemini CLI and Antigravity CLI.
- Drives the real CLIs you already installed, with no API wrapper in between.
- Creating a task automatically makes a branch from the configured base, an isolated Git worktree for it, and symlinks for node_modules and other gitignored directories, then spawns the agent inside that tree.
- Diff review and guided change tours show what each agent changed.
- AI Arena mode races agents head to head on the same task, so you can merge the best diff.
- Phone monitoring over Wi-Fi or Tailscale lets you check progress away from your desk.
- The branch is merged back from the sidebar when you pick a winner.

The project is solo led, at about 1,060 stars and 144 forks. The latest release noted at launch coverage was v5.1.0 (2026-10-29).

## Try it step by step

1. Download Parallel Code from parallelcode.app and install it on macOS or Linux. There is no Windows build.
2. Open your project and set the base branch that new tasks should start from.
3. Create a task, write the prompt, and pick which CLI runs it: Claude Code, Codex CLI, Copilot CLI, OpenCode, Gemini CLI or Antigravity CLI.
4. Let the app create the branch and worktree and spawn the agent inside it.
5. Review the diff, follow the guided change tour, run your tests on that branch, and merge it from the sidebar if it passes.

## Race two agents on one bug

1. Pick one small, real bug with a clear pass or fail test.
2. In AI Arena, give the same task and the same prompt to two agents, for example Claude Code and Codex CLI.
3. Let both finish in their separate worktrees. Neither touches the other agent's files.
4. Compare the two diffs side by side and read both guided tours.
5. Run your tests on each branch.
6. Merge the better diff from the sidebar and drop the losing branch.

Your cost is only the agent subscriptions you already pay for. Parallel Code itself is free.

## Gotchas and limits

- macOS and Linux only. No Windows build.
- Fast moving surface: four major versions in eight months, so expect the interface and workflow to keep changing.
- Solo led project: one maintainer carries the project, though the MIT licensed code is open for anyone to read and contribute to.
- Every worktree is a real checkout and every agent is a real process, so disk and memory use grows with each parallel task.
- Always read the winning diff in full before merging. A race gives you choices, not a guarantee.

## Who it is for

Developers who already run two or more terminal coding agents and lose time running them one at a time. If you use a single agent occasionally, the extra machinery is probably not worth it yet.

## FAQ

**Is Parallel Code free?**
Yes. It is free and open source under the MIT license. You pay only for your own agent subscriptions.

**Does Parallel Code wrap the agent APIs?**
No. It drives the real CLIs directly, with no API wrapper.

**What happens when I create a task?**
The app makes a branch from your configured base, creates an isolated Git worktree, symlinks node_modules and other gitignored directories, and spawns the agent inside that tree.

**What is AI Arena?**
A mode that races agents head to head on the same task in separate worktrees, so you can compare diffs and merge the best one.
