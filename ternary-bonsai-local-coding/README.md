# Ternary Bonsai 2.5: run a 27B coding model locally in 5.9 GB

PrismML announced Ternary Bonsai 2.5 on November 24, 2026: a 27-billion-parameter coding model stored with ternary weights, so the whole model is 5.9 GB instead of roughly 54 GB at 16-bit precision. It is a free download under the Apache 2.0 license (commercial use allowed), and launch coverage describes llama.cpp and MLX builds that run locally on Mac, iPhone and iPad. It is an upgrade to Ternary Bonsai 2 27B, aimed at agentic coding, multi-turn tool use and long-horizon task execution.

Watch the episode: https://youtu.be/vg8Rz_ANhOM (Agentic Pod EP66)

## What ternary weights are

A weight is one number inside the model, one tiny dial it learned during training. A normal model stores each dial at 16-bit precision. A ternary model stores each dial as one of exactly three values: minus one, zero, or one. Three options work out to about 1.76 bits per weight.

The math is the whole story:

- 27 billion weights at 16 bits each is roughly 54 GB.
- 27 billion weights at about 1.76 bits each is 5.9 GB.
- That is about nine times smaller, small enough to live on a laptop or a phone.

The tradeoff is precision. Three values per dial carry less nuance than 65,536 values, so the interesting question is never "is it smaller" (it is) but "how much coding ability survives the squeeze". That is what the benchmark section below is for.

## Download

1. Get the model files from PrismML's official release channel (the launch coverage links the download). Pick the build that matches your machine: MLX for Apple Silicon Macs, iPhone and iPad, or the llama.cpp build for the widest hardware support.
2. Check the download size before you start: you are looking for about 5.9 GB for the full 27B model. If a file is far bigger, you probably grabbed a full-precision variant.
3. Keep the model file somewhere with fast storage. A local model loads from disk every run, and a slow external drive makes startup drag.

## Run it with llama.cpp

1. Install llama.cpp (on a Mac: `brew install llama.cpp`).
2. Start the model in chat mode:

```bash
llama-cli -m /path/to/ternary-bonsai-2.5.gguf -p "Explain what this function does" -n 512
```

3. For agent-style use, run the llama.cpp server so your tools can talk to it over a local API:

```bash
llama-server -m /path/to/ternary-bonsai-2.5.gguf --port 8080
```

4. Smoke test it: ask it to explain a function from your own repo, then ask a follow-up that depends on the first answer. Multi-turn coherence is the first thing to check.

## Run it with MLX (Mac, iPhone, iPad)

1. Install MLX and the MLX language model tooling (`pip install mlx-lm` on a Mac).
2. Load the MLX build of the model and run a first prompt:

```bash
python -m mlx_lm.generate --model /path/to/ternary-bonsai-2.5-mlx --prompt "Write a function that retries a failed HTTP request three times" --max-tokens 512
```

3. On iPhone or iPad, use an MLX-based chat app build from the launch coverage and load the same model file. Expect short sessions: the win on a phone is quick lookups and small edits, not long agent runs.

## Wire it into a local coding agent workflow

1. Run the model behind a local server (llama-server above, or your MLX app of choice) so your agent tool can reach it at `http://localhost` instead of a cloud API.
2. Point your coding agent or editor integration at that local endpoint. Any tool that accepts a custom local model endpoint works; the model never needs to leave the machine.
3. Start the agent on read-only tasks first: explain this file, find where this error is raised, draft a test for this function.
4. Move to edits in a scratch branch. Let the agent propose diffs, run the project tests yourself, and read every diff before it lands.
5. Only then try a full multi-step task: one real issue, start to finish, with the test suite as the referee.

Why bother: no API meter running while the agent iterates, and your code never leaves the machine. For private repos and client work, that second point is often the deciding one.

## Benchmark it on your own repo (do this before trusting any score)

PrismML claims 75.6 on SWE-Bench Verified, an agentic coding average up from 37.9 to 54.2, and a tool-calling average up from 60.2 to 82.2 versus Bonsai 2. Those are vendor numbers from the announcement. No independent lab has verified the 75.6 yet, and SWE-Bench scores move with the harness, the setup and the number of attempts. Treat the claim as a reason to test, not a reason to switch.

A one-afternoon test that tells you more than any leaderboard:

1. Pick five real issues from your own repo that you already fixed by hand. You know the correct fix and the tests that prove it.
2. Give the model the same issue text you worked from, nothing more.
3. Let it produce a fix, run the project's own tests, and grade pass or fail exactly like SWE-Bench would.
4. Score the tool use separately: across a multi-step task, how often did it read the right file, edit the right place, and recover from a failing test without losing the thread? Small models usually stumble here first.
5. Repeat one issue at a longer context (a big file, a long log) and watch whether quality holds up.

Five issues will not give you a publishable score. They will tell you whether this model survives contact with your codebase, which is the only score that matters for your work.

## Gotchas and limits

- Precision is the price of ternary weights. On fuzzy, ambiguous tasks, a full-precision model still has more room to think. Expect the gap to show up on subtle bugs before it shows up on boilerplate.
- The headline benchmark is unverified. One vendor run is marketing until an independent lab reproduces it under a known harness.
- A 5.9 GB model still needs RAM headroom for context. Long contexts grow the memory footprint well past the file size, especially on phones.
- Speed depends on your hardware. Apple Silicon with MLX is the launch target; older machines will run it, just slower.
- It is one model in a crowded month: Z.ai's GLM-5.4 (a 750B loop transformer) landed November 23, 2026 and Thinking Machines' Inkling 2 Mini landed November 20, 2026. Bonsai's bet is the opposite direction: small enough to own outright.

## Who it is for

- Developers who want a coding agent with no per-token bill and no code leaving the machine.
- Anyone on a laptop or phone who could never fit a 54 GB model but can spare 5.9 GB.
- Teams who need an offline or air-gapped coding assistant and can accept the precision tradeoff.
- Skeptics with a free afternoon: the download is free, the license allows commercial use, and your own repo is the only benchmark that counts.

## Sources

- Tech Featured, "Ternary Bonsai 2.5: PrismML 27B, 5.9 GB, SWE-Bench": https://techfeatured.com/20935/ternary-bonsai-2-5-prismml-27b-5-9-gb-swe-bench
- PrismML announcement via PRNewswire, November 24, 2026, also covered by RuntimeWire.
