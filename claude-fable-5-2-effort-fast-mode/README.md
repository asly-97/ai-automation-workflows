# What is Claude Fable 5.2, and how do you use it well?

The practical guide behind the Agentic Pod episode **"What Is Claude Fable 5.2? Effort, Fast Mode and the Real Cost per Task"**.

Watch: https://youtu.be/UrRIgPdk9CI

Anthropic released Claude Fable 5.2 and Claude Mythos 5.2 on November 9, 2026. They are the same underlying model at two safeguard levels: Fable is the public one, Mythos is only for verified organizations. Fable 5.2 is up to 30% faster than 5.1, and its classifiers block about 55% fewer requests. This guide covers the pricing, the 5 effort levels with the real cost per task, fast mode, the migration gotchas that return 400 errors, and a workflow that keeps your bill sane.

## The numbers

| | Claude Fable 5.2 |
|---|---|
| Input price | $8 per million tokens (down 20% from 5.1 at $10) |
| Output price | $40 per million tokens (5.1 was $50) |
| Batch price | $4 input, $20 output per million tokens |
| Cache reads | $0.20 per million tokens |
| Fast mode price | $16 input, $80 output per million tokens |
| Context window | 1,000,000 tokens in |
| Max output | 128,000 tokens |
| Model ID | `claude-fable-5-2` |

Where you can use it: the Claude API, Amazon Bedrock, Google Cloud, Microsoft Foundry, and paid Claude apps. Claude Code needs version 2.1.321 or later. Fast mode lives in Claude Code, the Claude Platform, Google Cloud, and Microsoft Foundry.

## Fast mode: speed is a choice, not a new model

Fast mode makes generation up to 2.5x faster at double the price: $16 per million input tokens and $80 output. Same model, same answers, a higher serving rate. Turn it on when wall clock time is worth more than the tokens, for example a long run blocking a release. Leave it off for batch shaped work where nobody is waiting.

## Effort levels: the real cost per task

Effort is how hard the model thinks before it answers. There are 5 levels: low, medium, high, xhigh, and max. Watch the defaults: the API defaults to high, while Claude Code and the Claude apps default to medium. Same model, different default brainpower depending on the door you walk in.

FrontierCode 2.0 results (132 pull request tasks, vendor and system card numbers):

| Effort | Score | Cost per task |
|---|---|---|
| low | 34.2% | $2.10 |
| medium | 37.6% | $2.48 |
| high | 45.3% | $6.72 |
| xhigh | 49.1% | $14.19 |
| max | 45.7% | $44.70 |

Read the last row twice: max scores lower than xhigh (45.7% vs 49.1%) and costs more than 3 times as much per task. The max button is a trap until your own numbers prove otherwise. One more anchor: high effort roughly matches Opus 5.5 at max effort, for about half the cost per task.

## Migration gotchas (these return 400 errors)

1. **Adaptive thinking cannot be disabled.** Fable 5.2 thinks adaptively, period. Remove any code that tries to turn thinking off.
2. **Forced tool choice breaks.** If your code forces `tool_choice` to `any` or `tool`, the request returns a 400 error. Let the model choose, or restructure the call.
3. **Sampling settings break.** `temperature`, `top_p`, and `top_k` also return 400. Omit them.
4. **30 day data retention is required.** There is no zero data retention on this model unless Anthropic authorizes it.
5. **Claude Code version:** update to 2.1.321 or later before selecting Fable.
6. **Paid plan note:** during the launch promo week (November 9 to 16), Fable was included on paid plans. After that, Fable draws on usage credits on Pro.

## A workflow that keeps the bill sane

1. Keep your current model as the daily driver for ordinary coding and knowledge work. Anthropic itself says to start with Opus 5.5 for most of that work.
2. Route only your longest, hardest runs to Fable, at high effort first.
3. Measure cost per finished task, not cost per token. A cheap level that needs 3 retries is not cheap.
4. Use xhigh only where you measured a real gain on your own tasks. Never by vibes.
5. Skip max until your own benchmark shows it beating xhigh. On FrontierCode 2.0 it does not.
6. Turn fast mode on only when a person or a pipeline is actually waiting on the answer.

## FAQ

**What is the difference between Fable 5.2 and Mythos 5.2?**
Same underlying model, different safeguard level. Fable is public. Mythos is only for verified organizations.

**How much does Claude Fable 5.2 cost?**
$8 per million input tokens and $40 output. Batch is $4 and $20. Cache reads are $0.20. Fast mode doubles the base price to $16 and $80.

**Which effort level should I use?**
High is the sweet spot for hard work: 45.3% on FrontierCode 2.0 at $6.72 per task. Medium is fine for routine work. Xhigh is worth it only where you measured a gain. Max scored below xhigh at $44.70 per task, so skip it by default.

**What breaks when I migrate to Fable 5.2?**
Code that disables thinking, forces tool_choice, or sends temperature, top_p or top_k. All of those return 400 errors. You also need 30 day data retention, and Claude Code 2.1.321 or later.

**Should Fable replace my current model?**
For most coding and knowledge work, no. Use Fable as the specialist for the longest, hardest runs, and keep your current model as the daily driver.

## Links

- Episode: https://youtu.be/UrRIgPdk9CI
- What developers get with Fable 5.2 and Mythos 5.2: https://mer.vin/news/claude-fable-5-2-and-mythos-5-2-what-developers-get/
- Fable 5.2 for agentic coding: https://blog.sandbase.ai/claude-fable-5-2-agentic-coding-try-today-2026/
