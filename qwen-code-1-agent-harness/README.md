# Qwen Code 1.0: the CLI became an agent harness you can embed

The practical guide behind the Agentic Pod episode **"Qwen Code 1.0: The CLI Became an Agent Harness You Can Embed"**.

Watch: https://youtu.be/CIhLZr1oRD0

Alibaba released Qwen Code 1.0 on November 11, 2026. It merged the agent harness capabilities from Qoder into Qwen Code and open sourced them together under Apache 2.0. What used to be a terminal coding assistant is now an open source agent harness you can embed inside your own software. This guide covers what changed in 1.0, what an agent harness is, how to try it step by step, a real workflow for terminal and embedded use, and the gotchas.

Sources: https://qwenlm.github.io/qwen-code-docs/

## What changed in 1.0

- The Qoder agent harness now lives inside Qwen Code. The harness capabilities from Alibaba's Qoder tool were merged in and open sourced together with Qwen Code.
- The license is Apache 2.0. The software is free and open source.
- Qwen Code is no longer only a terminal coding assistant. The same tool is now also a harness you can embed inside your own software, so you can build your own agent product on top of it.
- The design starts with an in process, in memory event bus, growing toward a distributed, multi node, multi session setup. A unified event model decouples reasoning from tool execution, so you can watch, route, and build on the events an agent produces.
- 1.0 is a Preview branch. The stable branch is retained, so you can keep the stable build while you test the preview.

## What an agent harness is

An agent harness is the machinery around the model. It plans the steps, calls the tools, and keeps the agent on track across a long task. The model does the reasoning; the harness handles the loop around it: read the context, plan, act with a tool, check the result, and adjust. Without a harness you write all of that planning and tool plumbing yourself. With an embeddable harness, your own app gets the agent loop and you bring the model.

## How to try it, step by step

1. Install Qwen Code. There is a standalone install script for Linux and Mac, or use npm with the qwen code package.
2. Open your project folder, then run the qwen command.
3. On first launch, connect a model provider. You can connect Qwen, an OpenAI compatible provider, Anthropic, Gemini, or a local provider. You pick the brain; the harness stays the same.
4. Give it a real task. Ask it to find why a test fails, fix the code, and run the test again.
5. Optional: there is also a VS Code extension in beta, if the terminal is not your home.

## A real workflow: terminal and embedded

### Terminal: fix a failing test in a messy repo

1. Point Qwen Code at the repo and say, in plain words: fix the failing checkout test, and do not break the other tests.
2. It reads the code first. Then it plans the steps, edits one file at a time, runs the tests, and adjusts when a test still fails.
3. That loop, plan, edit, test, adjust, is the harness keeping the agent on track across the whole task instead of stopping after one guess.

### Embedded: build your own agent product

1. Instead of writing the planning and tool plumbing yourself, embed the harness inside your own app.
2. Hook into the event model: the unified events decouple reasoning from tool execution, so your app can watch and route what the agent is doing.
3. Add your own tools for your domain.
4. Connect the model provider your product will use. Your app gets the agent loop, and you bring the model.

## Gotchas and limits

- Preview branch. The 1.0 release is a Preview branch while the stable branch is retained. Preview means interfaces can still change, so keep the stable build around.
- The software is free, the model is not. Qwen Code is free and open source under Apache 2.0, but model usage is billed by your provider. Your provider bills you for the tokens you use.
- The free tier ended. The free Qwen OAuth tier ended in April 2026. If you remember running it for free, that door closed months ago.
- Multi provider means multi setup. Qwen, OpenAI compatible providers, Anthropic, Gemini, and local providers all work, but each one needs its own connection and its own billing.
- Nothing forces a switch today. If you just want a coding assistant, try the preview first. The embeddable harness is the interesting part if you build agents yourself.

## FAQ

**What is Qwen Code 1.0?**
The November 11, 2026 release of Alibaba's Qwen Code. It merged the agent harness capabilities from Qoder into Qwen Code and open sourced them together under Apache 2.0, so the terminal coding assistant is now also an embeddable agent harness.

**What is an agent harness?**
The machinery around the model: it plans steps, calls tools, and keeps an agent on track across a long task. Qwen Code 1.0 lets you embed that machinery inside your own software.

**How do I try Qwen Code?**
Install it with the standalone install script for Linux and Mac, or with npm. Run qwen in your project folder, connect a provider (Qwen, an OpenAI compatible provider, Anthropic, Gemini, or a local provider), and give it a real task. A VS Code extension is also available in beta.

**Is Qwen Code free?**
The software is free and open source under Apache 2.0. Model usage is billed by your provider, and the free Qwen OAuth tier ended in April 2026.

**Is Qwen Code 1.0 stable?**
1.0 is a Preview branch. The stable branch is retained, so you can test the preview without giving up the stable build.

## Links

- Episode: https://youtu.be/CIhLZr1oRD0
- Qwen Code documentation: https://qwenlm.github.io/qwen-code-docs/
