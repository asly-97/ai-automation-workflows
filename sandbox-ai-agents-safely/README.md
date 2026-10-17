# How do you sandbox an AI agent? Perplexity SPACE explained

A practical checklist for running AI agents without giving them your live systems, built around Perplexity's open source SPACE sandbox. Companion to the Agentic Pod episode "How Do You Sandbox an AI Agent? Perplexity SPACE Explained".

- 🎧 Watch the episode: [Agentic Pod on YouTube](https://youtu.be/W97uhhINoq4)

Facts about SPACE come from coverage of Perplexity's October 15, 2026 announcement, checked on 2026-10-17. Speed and security numbers are Perplexity's own claims. Check the official announcement and the repository before relying on a detail.

## What SPACE is

- Name: Sandboxed Platform for Agentic Code Execution. Open sourced on October 15, 2026 under the Apache 2.0 license, with code on GitHub.
- It is the sandbox platform behind Perplexity's Computer agent, and it was used in production before the open source release.
- Built on Firecracker microVMs, the virtualization technology AWS created. Each task gets a fresh microVM, destroyed when the work ends.
- Perplexity reports a median sandbox creation time of about 60 milliseconds, down from 185 milliseconds. No independent benchmark was found.
- Sandboxes can be paused and resumed. Rolling snapshots are kept for up to one week.
- Credentials are injected only at the moment they are needed and are never stored inside the sandbox. Bring-your-own-key encryption is supported.
- In a September 2026 red team exercise, testers made 108 exploit attempts and none produced a sandbox escape. Some network policy issues were found and fixed.
- It runs on Firecracker and Kubernetes, so it is infrastructure for a team rather than a one-click app.

## The six-step checklist for your own agents

1. **One disposable box per task.** Never let an agent run in the same place as your real project. Throw the box away when the task ends.
2. **No keys inside the box.** Hand credentials to a task only when it needs them, and keep them out of the files and the environment the agent can read.
3. **Scoped, short-lived keys.** Use the smallest scope that works (for example read-only on one repository) and let keys expire. Never give an agent your master key.
4. **Test apart from live.** A test agent should not be able to reach production data, even by accident. Use separate accounts, projects and credentials.
5. **Limit the network.** Allow only the hosts the task needs. Even strong sandboxes found network policy issues, so treat egress rules as part of the design.
6. **Snapshot before risky steps, and keep a log.** Snapshot before installs, migrations and deletes so you can roll back. Log every command the agent ran so you can see what happened.

## If you do not run your own sandbox

- Use the sandbox your agent tool already provides, and still apply steps 2 to 6.
- Turn on approvals for any action that sends, deletes or pays.
- Review what the agent can read before you start: environment variables, `.env` files, cloud credentials and browser sessions.

## If you are a platform team

- Read the SPACE repository and the announcement, then test the speed and isolation claims in your own environment.
- Plan for Kubernetes and Firecracker operations before you adopt it.

## Verdict

The design is sound and the ideas travel to any agent setup. Trust the design, and test the numbers yourself.

## Sources

- Perplexity open sources SPACE (CryptoBriefing): https://cryptobriefing.com/perplexity-open-sources-space-agent-sandbox/
- How Perplexity thinks about AI agent sandboxes (The New Stack): https://thenewstack.io/perplexity-space-agent-sandboxes
