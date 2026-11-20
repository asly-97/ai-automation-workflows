# Eclipse Enclave 1.0: your coding agent, sandboxed

The practical guide behind the Agentic Pod episode **"Eclipse Enclave 1.0: Your Coding Agent, Sandboxed"**.

Watch: https://youtu.be/uUp7L1E7qdY

The Eclipse Foundation announced Eclipse Enclave 1.0 on November 19, 2026. Enclave is a free, open source, MIT licensed sandbox that runs each coding agent in its own isolated container, with only the files, network hosts and secrets the developer chooses. Six agents are built in: Claude Code, Codex, OpenCode, GitHub Copilot CLI, Pi, and Eclipse Theia, which can attach to a running session. Amp and Antigravity ship separately in an enclave-extensions repository, and tooling can also be pulled from git repos. It runs on Linux and macOS including arm64, with deb and rpm packages, and a Windows launcher via WSL2. This guide covers how the sandbox works, how to try it step by step, the backends, and the caveats.

## What shipped

- Eclipse Enclave 1.0, announced by the Eclipse Foundation on November 19, 2026. The project registry dates version 1.0.0 to October 31, 2026.
- Free and open source under the MIT license. Download it from enclave.eclipse.dev and run it on your own machine.
- Each agent session runs in its own isolated container. The agent sees the project files you mount, and nothing else on your machine.
- You choose the files, the network hosts the agent may call, and the secrets it may use. Everything else, like your SSH keys and other projects, stays out of reach.

## The six built-in agents

- Claude Code
- Codex
- OpenCode
- GitHub Copilot CLI
- Pi
- Eclipse Theia, which can also attach to a session that is already running, so you can open a window into live work.

More tooling:

- Amp and Antigravity ship separately in the enclave-extensions repository.
- Enclave can also pull tooling straight from git repos, so you are not stuck with the built-in set.

## How to try it, step by step

1. Download Enclave from enclave.eclipse.dev.
2. Install it on Linux or macOS, including arm64 machines. On Linux there are deb and rpm packages. On Windows, use the launcher that runs it through WSL2.
3. Pick a container backend: Docker in rootful mode with buildx, or Podman, where rootless works.
4. Start a session, pick your agent, and mount only the one project it should touch.
5. Allow only the network hosts the task needs, and give it development credentials only.
6. Watch the session, or attach Eclipse Theia to it, and review everything the agent writes before you keep it.

## Backends

- Docker: works in rootful mode, with buildx.
- Podman: rootless works, so it runs without full system rights.
- QEMU microVM (experimental): runs the agent inside a tiny virtual machine. Honest caveat: it currently operates without the Docker gateway network restrictions, so the network filtering you get on the Docker path does not apply there yet. Treat it as experimental.

## HomeShell

- HomeShell is a graphical companion that groups your sessions into projects, so parallel work stays tidy.
- It is a preview. There is no stable release yet, so expect rough edges.

## Gotchas and limits

- A sandbox limits damage. It does not judge code quality. Anything the agent writes still needs your review, same as always.
- Point database credentials at a development schema only, never production. Inside the sandbox, give the agent the spare keys, not the house keys.
- The QEMU microVM backend is experimental and currently runs without the Docker gateway network restrictions.
- HomeShell is a preview with no stable release yet.
- Two dates, both real: the registry dates version 1.0.0 to October 31, 2026, and the public announcement was November 19, 2026.

## FAQ

**What is Eclipse Enclave?**
A free, open source, MIT licensed sandbox from the Eclipse Foundation, announced November 19, 2026, that runs each coding agent in its own isolated container with only the files, network hosts and secrets you choose.

**Which agents does Enclave run?**
Six are built in: Claude Code, Codex, OpenCode, GitHub Copilot CLI, Pi, and Eclipse Theia. Amp and Antigravity ship separately in the enclave-extensions repository, and tooling can also be pulled from git repos.

**What does Enclave cost?**
Nothing. It is free and open source under the MIT license, and you run it on your own machine.

**Does a sandbox make agent code safe to ship?**
No. A sandbox limits the damage an agent can do while it works. It does not judge code quality, so review everything the agent writes, and point database credentials at a development schema only.
