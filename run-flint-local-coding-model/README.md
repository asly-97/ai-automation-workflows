# How do you run Recursive's Flint coding model on your laptop?

The practical guide behind the Agentic Pod episode **"Can You Run a Free AI Coding Model on Your Laptop? Recursive Flint"**: what Flint is, how to run it with Ollama, what hardware to plan for, a small-steps prompt workflow, and the honest limits.

## What Flint is

- Released October 21, 2026 by Recursive (CEO Richard Socher, formerly chief scientist at Salesforce).
- Free, open weights under the Apache 2.0 license.
- A mixture of experts model: about 40 billion parameters in total (39.8B on the Ollama page), about 2.5 billion active per token.
- Recursive says a small team built it from scratch in under four months, with its Eureka Machine research system contributing to nearly every stage under human supervision.
- Company-reported speed: about 131 tokens per second on a MacBook Pro with an Apple M4 Pro chip, and over 150 tokens per second on an Nvidia RTX Spark. Similar models on the same MacBook were reported at roughly 58 to 69.

## Run it with Ollama

```
ollama run flint:40b
```

- Download size on the Ollama page: 23 GB. Plan memory accordingly; the Wired reviewer tested on a Mac with 48 GB.
- Context length on the Ollama page: 262K tokens.
- Other builds are on Hugging Face from RecursiveSI: an INT4 build for Apple Silicon (https://huggingface.co/RecursiveSI/flint-int4-gguf) and an NVFP4 GGUF build for other GPUs (https://huggingface.co/RecursiveSI/flint-nvfp4-gguf). Check each model card for the runtime it supports.
- Sources disagree on exact memory requirements and on GPU support for the default Ollama tag, so check the Ollama library page for your hardware before pulling 23 GB: https://ollama.com/library/flint:40b

## A workflow that fits a small local model

Recursive says Flint needs more explicit instructions than the large paid coding assistants, so split work into small, checkable steps:

1. "Write only the tests for a function that slugifies titles. Stop after the tests."
2. "Now write the function so those tests pass."
3. "Explain what could break."

Each step is easy to review, and because the model is local and free you can redo a step as many times as you like. Your code stays on your machine.

Ollama serves local models on your own machine, so editors and tools that can talk to a local Ollama model can use it. Check your tool's docs.

## Gotchas

- The benchmark results are Recursive's own: it says Flint beat other models under 50 billion parameters on SWE-bench Verified, and across six tests only Alibaba's Qwen3.8 27B scored higher. No absolute scores were published in the coverage we read.
- Token speed measures how fast the model writes, not how well it codes.
- The early Wired review covered small projects (a to-do app and a web research tool). Large codebases and complex engineering are unproven.

## Who it is for

Developers with a high-memory Mac or a strong GPU, people who care about privacy or want to avoid usage fees, and anyone who wants to experiment. If you need the best result on a large codebase, keep a paid tool open next to it.

## Sources

- https://www.techtimes.com/articles/329391/20261022/recursive-launches-flint-ai-free-coding-model-runs-laptops.htm
- https://www.analyticsinsight.net/news/recursive-flint-ai-free-40-billion-coding-model-runs-offline
- https://ollama.com/library/flint:40b
