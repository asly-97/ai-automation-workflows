# Can you run an AI agent locally? NVIDIA Nemotron 3.5 Bolt

A practical guide to NVIDIA's Nemotron 3.5 Bolt: what it is, how a 74B model fits in 24 GB, how to try it step by step with Ollama, LM Studio, llama.cpp or vLLM, what to use it for, and the gotchas. Companion to the Agentic Pod episode "Can You Run an AI Agent Locally? Nemotron 3.5 Bolt Explained".

- 🎧 Watch the episode: [Agentic Pod on YouTube](https://youtu.be/bifhvmz5GG4)

Facts come from launch coverage published October 15 and 16, 2026. Benchmark claims are NVIDIA's own unless noted, so check the model card before relying on a detail.

## What Nemotron 3.5 Bolt is

- **Bolt 74B-A9B 24GB**: 74 billion parameters in total, about 9 billion active per step (a hybrid mixture of experts design, which is what "A9B" refers to).
- **2-bit quantization** squeezes the weights so the model fits in 24 GB of memory.
- Runs on GeForce RTX PCs, DGX Spark, DGX Station and RTX PRO workstations.
- Supported runtimes: llama.cpp, Ollama, LM Studio and vLLM (Unsloth is listed too).
- NVIDIA targets local research, web browsing, coding and debugging.
- **Super VL (120B-A12B)** launched alongside it. That one is the server model for enterprises, not a desk model.
- **Bolt E4B 4GB** is a smaller model for voice assistants and device interfaces. NVIDIA said it follows later in the month, so it was not out yet when this was written.

## How good is it?

- NVIDIA says Bolt ranks first among open models in its class on PinchBench and second on BrowseComp. These are vendor-reported.
- The Decoder cites the Artificial Analysis Intelligence Index (v4.3.2): Bolt scores 21, Super VL scores 26, and the top model on the list scores 58.
- Read that as: a capable local worker, not a replacement for the best cloud models.

## Try it step by step

1. **Check your hardware.** You want a 24 GB GPU (an RTX class card) or a DGX Spark.
2. **Pick a runtime.** If you are new to this, start with Ollama or LM Studio. llama.cpp and vLLM give you more control.
3. **Download the Bolt build** your runtime supports from Hugging Face (look for NVIDIA's Nemotron 3.5 Bolt listing).
4. **Run the local server.** These runtimes can serve a local OpenAI compatible endpoint. Point your agent tool at that address instead of a cloud one.
5. **Give it one small real job.** Summarise a repo, or fix one failing test. Run the same job on your usual cloud model and compare.
6. **Cap the steps** while you test, so a looping agent cannot run away.

## What to use it for

- Routine, low-risk agent work: triaging a folder of notes, drafting commit messages, a first-pass code review.
- Private files and offline work, where nothing should leave your machine.
- Jobs where a miss is cheap and you can check the result.

Send hard problems to a bigger cloud model. Local for the boring work, cloud for the hard thinking.

## Gotchas

- 2-bit quantization trades accuracy for size, so expect misses on hard tasks.
- Long contexts use a lot of memory. Test with your real prompt sizes.
- Vendor benchmarks are not your workload. Run your own test.
- Agents run in loops, so one bad step can repeat. Set a step limit.
- Bolt E4B was not shipped yet at the time of writing.

## Who it is for

- Good fit: people with sensitive files, people working offline, and anyone tired of paying per token for routine agent work.
- Wait if you need top-tier reasoning on every task or you do not have a 24 GB GPU.

## Quick checklist

1. 24 GB GPU or DGX Spark ready.
2. Ollama or LM Studio installed.
3. Bolt build downloaded and served locally.
4. Your agent tool pointed at the local endpoint.
5. One small real job compared against your cloud model.
6. Step limit set.

## Sources

- The Decoder, "Nvidia's Nemotron 3.5 Super VL is an open model built to run enterprise AI agents nonstop" (October 15, 2026): https://the-decoder.com/nvidias-nemotron-3-5-super-vl-is-an-open-model-built-to-run-enterprise-ai-agents-nonstop/
- ITBrief, "Nvidia launches Nemotron 3.5 open models for agents" (October 16, 2026): https://itbrief.com.au/story/nvidia-launches-nemotron-3-5-open-models-for-agents
- TechRepublic, "Nvidia Launches Three Nemotron AI Models, Including Two for Local AI Agents": https://www.techrepublic.com/article/news-nvidia-nemotron-3-5-super-vl-bolt-ai-models/
