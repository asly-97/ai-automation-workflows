# Omi CLI: a terminal coding agent on 13 models

Omi CLI is a terminal coding agent from BuildItOmi. It launched on Product Hunt on November 22, 2026, finishing #8 of the day with 97 points at capture. You give it a coding task in plain words, inside your own repo. It reads the repo, plans multi-step work, shows every edit as a diff you approve, runs commands and tests in a loop until the tests pass, and checks your app in a real browser.

## What it does

- Reads the local repo first with list, glob, grep and read tools, then plans the work in steps.
- Edits files with edit, write and delete, and every edit arrives as a diff you approve: yes, no, or always.
- Runs commands and tests in a continuous loop via /test, fixing failures until the suite passes.
- The /undo command restores every file the last task changed, even without Git.
- Checks the app in a real browser: it opens the app, clicks through it, and reports console errors and failed requests.
- Auto picks a model per turn from 13 models, including DeepSeek, Gemini, Gemma, gpt-oss and GLM, so cheap edits stay cheap and hard problems get a stronger model.
- Parallel subagents (version 0.5.0) split a big job into tasks, each in its own Git worktree, and merge back file by file with standard conflict markers. There is never a silent winner.
- Background agents keep working after you close the terminal, and cloud agents can open pull requests.

## Step by step

1. Check your setup. You need Node.js 18.17 or newer, on macOS or Linux. On Windows, use WSL.
2. Install it:

   ```bash
   npm i -g @builditomi/cli
   ```

3. Open your project folder and start it:

   ```bash
   omi
   ```

4. Describe the task in plain words, for example: "Fix the failing login test and run the suite until it passes."
5. Read the plan, then approve each diff: yes, no, or always when you trust the run.
6. Let the /test loop run until the tests pass. If the result is wrong, use /undo to restore the files from the last task.
7. For a bigger job, let Omi split it into subagent tasks. Review each worktree merge and resolve any conflict markers yourself.

## Models and pricing

- Auto mode picks a model per turn from the 13 included models (DeepSeek, Gemini, Gemma, gpt-oss, GLM and others).
- Bring your own key if you want a specific provider model.
- Run a local model through Ollama if you want the work to stay on your machine.
- Free plan: limits refill every 5 hours and weekly. Pro is $20/mo, Pro+ is $60/mo, Max is $200/mo.
- Honest note: frontier models like Claude and GPT are not in the included plan list. You can still reach them with your own key.

## Gotchas and limits

- The --sandbox mode keeps commands inside the project with no network, on macOS and Linux. Use it on code you do not fully trust.
- The maker reports 10 out of 10 on a first public benchmark, but it was 10 small tasks judged by hidden tests. Treat it as promising, not proven.
- It is brand new software (0.5.x). Expect rough edges and fast changes.
- Diff approval is the safety model. Skipping reviews with always on a repo you care about removes that safety.

## Who it is for

Terminal-first builders who want approval on every edit, a test loop that keeps going, and parallel agents without paying for five separate subscriptions. If you live in a graphical IDE and never open a terminal, this will feel like a step sideways.

## FAQ

**Does Omi CLI work on Windows?**
Only through WSL, the Linux layer inside Windows. Native targets are macOS and Linux.

**What happens if two subagents edit the same lines?**
The merge brings the changes back file by file and marks conflicts with standard conflict markers. You resolve them. No version silently wins.

**Can I use my own model?**
Yes. Bring your own API key, or run a local model through Ollama.

**What does it cost to start?**
Nothing. The free plan's limits refill every 5 hours and weekly, which is enough to test it on a real bug.

## Watch the episode

Agentic Pod EP62: Omi CLI Runs 13 Models in Your Terminal
https://youtu.be/SvAmbQy4S8k

Sources: https://vibecodingwiki.com/news , https://www.producthunt.com/products/omi-cli , https://cli.builditomi.com
