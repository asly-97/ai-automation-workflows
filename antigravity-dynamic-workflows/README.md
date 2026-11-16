# Antigravity Dynamic Workflows: one script, a team of agents

The practical guide behind the Agentic Pod episode **"Antigravity Dynamic Workflows: One Script, a Team of Agents"**.

Watch: https://youtu.be/dQ3MX3AbgxI

Google shipped Dynamic Workflows in Antigravity version 2.32.0 on November 13, 2026. You type /run-workflow, the agent writes a short Python script that coordinates a team of coding agents, and you read and approve that script before anything runs. This guide covers what Dynamic Workflows are, how to install or update, the step by step run, the script building blocks, the workflow shapes worth stealing, and the gotchas.

Sources: https://antigravity.google/docs/dynamic-workflows/, https://freshsynapse.com/articles/antigravity-dynamic-workflows-coding-agents-2026-11-16/, https://tmlnews.tech/stories/antigravity-dynamic-workflows-python-scripts/

## What Dynamic Workflows are

- One command, /run-workflow, turns one task into a short Python script that coordinates many agents.
- The plan is code, not chat: the script groups the work into named phases you can follow live while it runs in the background.
- Every workflow agent works in its own conversation. File reads, searches and test logs stay out of your main chat context.
- Workflow agents cannot see the script or each other. The script hands each agent everything it needs inside its prompt.
- Every agent returns a structured result that matches a JSON Schema, so the next step can check the result like data instead of guessing at prose.

## Install or update to v2.32.0

1. Open the Antigravity App or the Antigravity CLI and update to version 2.32.0 (released November 13, 2026). Dynamic Workflows shipped in that release, so an older build does not have /run-workflow.
2. Sign in with a Google AI Pro or Ultra plan. Dynamic Workflows are available on those two plans only, in both the App and the CLI.
3. There is no separate charge. Workflow runs use your plan credits.

## Run your first workflow, step by step

1. Type /run-workflow, then your task in plain words. Example from the docs style: list the Python files changed since main, then review each file for unhandled exceptions.
2. Pick a model tier per agent if you care: ask for flash lite or flash for fast checks across many files, pro for deeper reasoning. By default, agents inherit your conversation model. Workflows run on Gemini 4 Carbon, Argon and Hydrogen.
3. Read the script Antigravity shows you. This is the approval step: Antigravity asks "Allow running this workflow script?" before a single agent runs. Check the phases, the files and the models. If the plan is wrong, say no and tell the agent what to change.
4. Approve it and watch it run. The run happens in the background while you keep working, and the step view shows each phase, each log() line, and each agent card live.
5. If the workflow works, save the script to a .py file in your project. A saved script can be run again later, exactly the same way.

## The building blocks: parallel(), pipeline(), phase(), log()

- parallel() runs a set of agent jobs at the same time. Use it when the jobs do not depend on each other, like reviewing ten different files.
- pipeline() runs jobs one after another, in order. Use it when steps share files or when each step needs the result of the step before it, like test then fix per bug.
- phase() groups jobs into named stages, so a long run reads as a short plan: find changes, review, fix, verify.
- log() writes progress lines into the run view, like a one line verdict per file, so you can follow the run without opening every agent conversation.
- Structured results: define a JSON Schema for what each agent must return (for example a file name, a verdict, and a list of issues). The script can then branch on real fields instead of parsing free text.

## Three workflow shapes worth stealing

### 1. Parallel review

One phase finds the changed files. The next phase reviews every file at the same time with parallel() and logs a one line verdict per file. Fast, repeatable, and easy to scan.

### 2. Review, fix, verify with a persistent reviewer

One reviewer agent lists the bugs and keeps its notes across phases. Then one fixer per bug runs in parallel. Then the same reviewer agent checks every bug it found and confirms each one is fixed. The reviewer staying alive across the phases is the clever part: the agent that found the problems is the agent that signs them off.

### 3. Isolated branch workspace edits

For each package, one agent works in its own isolated branch workspace, makes the edit (for example swapping a deprecated helper), runs the tests, and logs the diff and the test result. Agents cannot collide because each one works in its own workspace. When steps must touch the same files, switch that part to pipeline() so the edits happen one at a time.

## Gotchas

- Parallel is all or nothing. If one item in a parallel() phase fails, the other items in that phase are canceled. Split fragile jobs into their own phases, or run them with pipeline().
- Agents start blind. A workflow agent cannot see the script or the other agents, so the script prompt for each agent must carry the full context that agent needs.
- Paid plans only. Dynamic Workflows need Google AI Pro or Ultra. There is no free tier for workflows, and runs draw on your plan quota, so a big parallel run can spend credits fast. Antigravity lead Varun Mohan announced a Gemini quota reset the same week, but quota is still the ceiling.
- Not Orchestrator, not /swarm. Orchestrator arrived November 12 and switches models inside one session. /swarm puts agents on a shared board where they talk to each other. A Dynamic Workflow is different: a fixed script you approved, agents that never talk, and results that come back as data.
- One off exploration is still a normal chat. Workflows pay off on repeatable, staged, checkable work: the same review or migration you run again and again.

## FAQ

**What is a Dynamic Workflow?**
A short Python script, written by the agent when you type /run-workflow, that coordinates a team of coding agents through named phases. You approve the script before it runs, and every agent returns a JSON Schema structured result.

**How do I get Dynamic Workflows?**
Update Antigravity (App or CLI) to version 2.32.0 or later and sign in with a Google AI Pro or Ultra plan. Then type /run-workflow with your task.

**What is the difference between parallel() and pipeline()?**
parallel() runs jobs at the same time for independent work. pipeline() runs jobs one after another when steps share files or depend on earlier results.

**Can I reuse a workflow?**
Yes. Save a script that works to a .py file in your project and run it again later. That is the main payoff: a review or migration that runs the same way every time.

**What happens if one parallel job fails?**
The other jobs in that parallel phase are canceled. Put fragile jobs in their own phase or run them in a pipeline.

**Is there an extra charge?**
No separate charge. Workflows use the credits of your Google AI Pro or Ultra plan.

## Links

- Episode: https://youtu.be/dQ3MX3AbgxI
- Antigravity Dynamic Workflows docs: https://antigravity.google/docs/dynamic-workflows/
- Launch coverage: https://freshsynapse.com/articles/antigravity-dynamic-workflows-coding-agents-2026-11-16/
- Launch coverage: https://tmlnews.tech/stories/antigravity-dynamic-workflows-python-scripts/
