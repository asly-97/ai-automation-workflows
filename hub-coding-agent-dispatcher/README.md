# Hub: Run 100 Claude Code or Codex Sessions Like 5

Hub (by Icons8) is an open source dispatcher for Claude Code and Codex sessions. One session is a chat. Ten sessions are a mess: you cannot tell which one is stuck, which one waits for you, or who told whom what. Hub fixes the herding problem, not the coding: you write a goal in plain words in your browser or in Telegram, a coordinator session hands the work to agents, and only the decisions come back to you.

- Ranked first on Product Hunt on November 26, 2026, with a reported score of 376 (rank and score as reported on the launch page).
- Free and open source under Apache-2.0. Self hosted. No paid tier and no hosted version.
- Runs on your own Claude or ChatGPT subscription. Hub provides no model access.
- Watch the episode: https://youtu.be/jhSPyb8oqWE

## Install in one line

macOS 13.5+ or Linux (glibc 2.28+):

```sh
curl -fsSL https://icons8.github.io/hub/install.sh | bash
```

Windows 10/11, in PowerShell:

```powershell
irm https://icons8.github.io/hub/install.ps1 | iex
```

Nothing has to be installed first and no administrator rights are needed, except on Linux without git (there the line uses the system package manager with sudo, so install git first or use an account with sudo). The line puts Hub's own environment (Python, Node, gh, ffmpeg, and git on macOS and Windows), the Hub code, its service and the `hub` command into your user folder, then opens the page at http://127.0.0.1:8787/.

Do not install as root, and do not clone the repository unless you are changing the core. If the command times out, run it again: it continues.

## First session checklist

1. Open the page and click "Sign in to Claude" on the band above the page. To work on Codex instead, choose "Sign in to Codex": the hub restarts on Codex and installs its CLI by itself.
2. Know the rule before you start: one hub runs one engine. Every session of a hub, the coordinator included, runs Claude Code or every session runs Codex. For the other engine, add a second hub (`hub instance new <name> --engine codex`), each with its own folders, service and page.
3. Set a password for the page (Settings) before anyone else can reach the machine. Until you do, the local page and file browser have no sign in.
4. Write the hub a first, small task in plain words. Watch the feed and the Now tab.
5. Connect Telegram when you want your phone as a full client: questions arrive as cards with buttons, a reply routes back to the session that asked, and `/status` shows who works on what.

## How dispatch works

- You talk to one coordinator session in plain words. It decides who does what, starts the right session and brings the result back.
- Permanent agents have their own area, tools and notes. Short lived executors work on one task in their own git worktree, report, and go to sleep with their conversation intact.
- Every session lands in one feed. The Now tab narrows it to what waits for you, what is new, and what is working.
- A question from a session becomes a card with buttons, on the page and in Telegram. A message to a busy session waits for the end of its turn; an urgent one gets through mid turn.
- Under the hood, an event journal is the single source of truth, the core (`bin/hubd`) is its only writer, and state is a pure function of the journal. A watchdog notices loops, silent sessions and repeated errors; restarts wait until everyone is free and bring fallen sessions back with their conversation.
- Extras built in: a scheduler for recurring jobs, project files in the browser, voice input, a separate browser per session, an away mode with a summary when you are back, and a Usage page with plan limits, forecasts and spend.
- Secrets stay sealed: Settings, Secrets stores passwords and API keys encrypted (AES-256-GCM). An agent uses one through `hub secret fill` or `hub secret exec` and never sees the value; every use is a line in the feed.
- Several hubs can run side by side on one computer, and hubs of different people can be linked by invitation so their agents write to each other, encrypted. It is off until you make or accept an invitation.

## A real workflow

1. Morning goal, written once: "Clear the twelve small fixes in the backlog, split the billing refactor into steps, keep production untouched."
2. The coordinator hands the fixes to executors, each in its own git worktree, and the refactor steps to a permanent agent.
3. You answer three decision cards from your phone: merge order, a naming choice, one risky migration.
4. Results land in the feed. You review diffs and merge. Nothing needed you except the decisions.

The team's own reported results (from their Product Hunt launch post, not independently verified): 546 merged merge requests since September 20, 2026, rewriting their legacy backend, and a booking system in Pinamar, Argentina where patients book at a public hospital and 6 health centers, with 86 doctors and real patients since late October 2026.

## Running Hub on a server

To try Hub away from your own computer, give it a virtual machine: Ubuntu 22.04 or 24.04, or Debian 12 or 13, x86_64, 8 GB of memory (3.5 GB minimum), 20 GB of disk, and your SSH key. Paste the server install line into the user data field when you create it:

```sh
#!/bin/bash
curl -fsSL https://icons8.github.io/hub/install.sh | bash -s -- --server --yes
```

Nothing listens on the server's public address. Reach the page through an SSH tunnel or Tailscale, and use the Telegram bot from your phone. Before you delete the machine, sign out (`sudo -iu hub claude auth logout`): the cloud provider can read the disk and its snapshots, sign in tokens included.

## Gotchas and limits

- Experimental, and honest about it. Hub calls itself an experiment at the top of its readme.
- Sessions run with permission prompts off (Claude Code `bypassPermissions`; Codex: never ask, no sandbox). An agent acts without asking you per action.
- Hub is not a sandbox. It adds order, not walls, and it will not stop a session that is talked into something by text it read. Treat Hub like a terminal on your machine and read SECURITY.md in the repo before running it anywhere that matters.
- Many parallel sessions consume your Claude or ChatGPT allowance much faster. Start with a handful of sessions, watch the Usage page, then scale.
- A conversation does not move between engines: a Claude Code session cannot continue as a Codex session.
- On a Codex hub, session texts go to OpenAI, and your personal `~/.codex` rules and settings do not reach hub sessions (the hub keeps Codex in its own folder).
- Updates arrive in the background about every 6 hours and apply at the next restart; a hub that does not come up on a new version rolls back by itself.

## Who it is for

- Developers already running several Claude Code or Codex sessions in parallel and losing time triaging which session needs them.
- Solo builders pushing several repositories at once.
- Not for you yet if you run one session at a time: you do not need a dispatcher for a single chat.

## Links

- Episode: https://youtu.be/jhSPyb8oqWE
- Hub on GitHub: https://github.com/icons8/hub
- Hub site and install: https://usehub.dev
- Product Hunt launch: https://www.producthunt.com/products/hub-20
- Security notes: https://github.com/icons8/hub/blob/main/SECURITY.md
