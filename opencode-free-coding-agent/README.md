# OpenCode 2.3: the free coding agent that runs any model

The practical guide behind the Agentic Pod episode **"OpenCode 2.3: The Free Coding Agent That Runs Any Model"**.

Watch: https://youtu.be/wONPsgjeqXQ

OpenCode shipped version 2.3.0 on November 13, 2026, and the open source terminal coding agent now sits at 226,858 GitHub stars with more than 30,000 forks. It is model agnostic: the same agent and the same workflow run on free built in models, a local Ollama model, or a paid frontier provider with your own key. This guide covers installing it, starting free safely, running a real task, going local, connecting paid providers, and the gotchas new users hit.

Sources: https://opencode.ai/docs and the hands-on tutorial at https://aifrontierpost.com/articles/opencode-ai-coding-agent-tutorial/

## What OpenCode is

- An open source terminal coding agent: it reads your files, writes code, and runs commands.
- Model agnostic: more than 30,000 forks, and the agent takes free models, your own provider keys, or a model on your own machine.
- Swapping models does not change the workflow: same commands, same session.
- 226,858 GitHub stars, about 15,000 of them gained in a single month.

## Install

Pick one:

```bash
npm install -g opencode-ai
```

```bash
curl -fsSL https://opencode.ai/install | bash
```

```bash
brew install opencode
```

Then check the version and list the models:

```bash
opencode --version   # should print 2.3.0
opencode models      # lists the models on every connected provider
```

On November 14, 2026, `opencode models` listed 12 free models on the built in `opencode` provider. No account, no API key, and no card are needed to start.

## Start free, pick safely

The free roster changes without notice, and the data terms differ per model. Read the terms line for a model before you paste real code into it.

- Start with GLM-5.3-Flash Free or DeepSeek V4.1 Flash Free. Both are zero retention, so your code is not kept or used for training, and both passed independent coding tests in October 2026.
- Other free models keep data for 30 days, and some may use your prompts for training.

## A real task pattern

Give the agent a precise job, and ask for the tests in the same prompt:

1. Name the exact files to create or change.
2. State the exact expected values and behavior.
3. Ask for a test file in the same prompt.
4. Run the tests yourself afterwards. The test runner is the only opinion that counts.

In the hands-on demo behind this episode, the agent wrote a Fibonacci function and a pytest file with 5 tests, and all 5 passed. The pattern, not the toy example, is the point: exact files, exact values, tests included, then you verify.

## Run it local with Ollama

- If Ollama is already running, OpenCode finds it on its own, with no config.
- Pull an 8B class model or larger, for example `qwen3:8b`. Tool use needs that class of model.
- Give Ollama at least 64k of context.
- The same commands then run with no data leaving your machine.

## Paid providers and OpenCode Go

- Inside the app, type `/connect` and pick your provider, or run `opencode auth login` in the terminal.
- Your keys stay in your local `auth.json` or in environment variables.
- OpenCode Go is the hosted option at 10 dollars a month if you would rather not manage keys.

## Gotchas

- The free quota is shared per network. A `FreeUsageLimitError` means the shared limit was hit, not that your project broke. Wait and retry, switch model, or go local.
- Tiny models fail at tool use. In testing, a 0.6B model talked about the task and never called a tool. Use 8B class models or larger for real editing.
- Keep your keys local. In October 2026 OpenCode confirmed a breach that exposed provider keys stored in some Console workspaces. Keys in local `auth.json` or environment variables were not the problem; keys pasted into hosted workspaces were.
- Use the right agent. OpenCode ships a build agent with full access and a read only plan agent for exploring a codebase safely. Run `/init` to write an AGENTS.md file with your project rules so every session starts with them.

## FAQ

**What is OpenCode?**
An open source terminal coding agent that works with any model: free built in models, your own provider keys, or a local Ollama model, all with the same workflow.

**Is OpenCode really free to start?**
Yes. Install it, run `opencode models`, and pick one of the free models on the built in opencode provider. No account and no API key are needed. The free quota is shared per network, so heavy use can hit a limit.

**Which free model should I start with?**
GLM-5.3-Flash Free or DeepSeek V4.1 Flash Free. Both are zero retention and both passed independent coding tests in October 2026.

**Can OpenCode run fully local?**
Yes. Run Ollama, pull an 8B class model or larger, give it at least 64k of context, and OpenCode auto discovers it. No data leaves your machine.

**How do I use my own paid provider?**
Type `/connect` in the app or run `opencode auth login`, and keep the key in your local auth.json or environment variables.

## Links

- Episode: https://youtu.be/wONPsgjeqXQ
- OpenCode docs: https://opencode.ai/docs
- Hands-on tutorial: https://aifrontierpost.com/articles/opencode-ai-coding-agent-tutorial/
