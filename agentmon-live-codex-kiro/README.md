# AgentMon Live: watch Claude Code, Codex and Kiro in one view

AgentMon Live is a free, open source (Apache 2.0) coding monitor from Codenotary. It launched in November 2026 for Claude Code, and the update reported by FOSS Force on November 20, 2026 added OpenAI Codex and AWS Kiro (Kiro Crew). One local web view now lists sessions from all three tools.

Project: https://github.com/codenotary/agentmon-live
News source: https://fossforce.com/2026/11/agentmon-live-widens-its-watch-to-codex-and-kiro/
Episode: EP58 of Agentic Pod, https://youtu.be/946pAXh3HMQ

## What it does

- Shows every tool call, subagent, file change, test run and token, live, in a pane beside your chat and in a browser view.
- Helps you verify completed agent work and spot tasks that still need attention.
- Tracks cost, context and quota limits where the tool provides them, so you can cut token waste.
- A done gate checks for unfinished work before the agent stops: a failed test or an open todo sends it back once.
- The web view is local only (127.0.0.1, port 47311) and masks secrets before anything reaches it.

Dennis Zimmer, CTO and co-founder of Codenotary: the goal is visibility across the tools developers use, without forcing them into a particular AI ecosystem.

## Install

Every path needs `node` on your PATH for the web view.

### Claude Code (2.1.308 or later)

In a Claude Code session:

```
/plugin install agentmon-live --marketplace codenotary/agentmon-live
```

Answer `y` to add the marketplace and pick a scope. The pane opens by itself when the terminal is wide enough, or with `/agentmon`. Press `o`, or run `/agentmon open`, to open the web view.

### Codex

The web view lists every Codex chat on the machine (CLI, `codex exec`, and the IDE extension) by reading the session files Codex writes, so nothing has to be installed in Codex for sessions to appear.

To have AgentMon start the web view with each Codex session, install it as a Codex plugin:

```
codex plugin marketplace add codenotary/agentmon-live
codex plugin add agentmon-live@agentmon-live
```

Then open `/hooks` in Codex and trust AgentMon Live's `SessionStart` and `Stop` hooks. The `Stop` hook is the done gate for Codex. Open http://127.0.0.1:47311/ for the web view.

Remove it with `codex plugin remove agentmon-live@agentmon-live`.

### Kiro Crew

The web view lists every open Kiro Crew chat beside your Claude Code sessions, on any harness, by reading the log Kiro Crew writes. Nothing needs to be installed in Kiro Crew for the chats to appear.

To run AgentMon as a tab beside the chat, install it as a Kiro Crew app:

```
git clone https://github.com/codenotary/agentmon-live
kirocrew app install <path>/agentmon-live
```

Then open **Library** in Kiro Crew, find AgentMon Live under **Disabled**, and click **Enable**. Kiro Crew asks you to trust the app before its code runs; read the hook code first, as the project itself recommends. Open a chat, click **+** above the side panel, and pick **AgentMon Live**.

Remove it with `kirocrew app uninstall agentmon-live`.

## Verify completed work

1. Let the agent finish a task with the monitor running.
2. Open the web view and check the Verification section: the last test, lint, typecheck and build command, each marked ok, failed or unknown.
3. On Claude Code, let the done gate work: if the agent claims done while a test failed or a todo is open, it gets sent back once with what is open.
4. Check the Files section for edits you did not ask for, and the session log for the turns that burned the most tokens.

## Cut token waste

- Watch the context gauge per session; each Codex chat counts against its own window and is never summed with a Claude Code session.
- Use the session log to find repeated failed calls and long loops, then fix the prompt or the setup that caused them.
- On Claude Code, `/agentmon bench` compares your sessions with and without AgentMon.

## Gotchas and limits

- Monitoring data differs between tools. The done gate, Focus and project learning are Claude Code features.
- On Kiro Crew, the done gate, Focus and learning stay empty, and there are no test counts because the Kiro Crew log holds no output.
- On Codex, approvals, subagent cards, cost in dollars and learning stay empty. The pane is part of Claude Code; stock Codex has no plugin sidebar, so use the web view.
- The web view lets in only a browser of the account that runs it. On Windows, an address typed by hand shows no sessions; use the one-time code from `/agentmon open`.
- A session shows only while it runs, and a very short run with no tool call can end before its first post and never show.
- Sessions inside a VM or container cannot reach the server directly; mount the spool folder as described in the project guide so they show up marked as sandbox sessions.

## Who it is for

Developers who run more than one coding agent in a week and want one place to see what each agent did, what it cost, and whether its work checks out, without moving to a single vendor's ecosystem.
