# What is Meta Muse Ember, and how do you try it?

The practical guide behind the Agentic Pod episode **"What Is Meta Muse Ember? The Tiny Model Built for Agent Tasks"**.

Watch: https://youtu.be/tT9CMKiMucI

Muse Ember is the smallest, cheapest model in Meta Superintelligence Labs' Muse family, launched November 9, 2026. It is built for the repetitive work inside agent runs: tool calls, routing, information extraction, long document processing, and coding subtasks like reading files, running tests and making targeted changes.

## The numbers

| | Muse Ember |
|---|---|
| Input price | $0.10 per million tokens |
| Cached input | $0.004 per million tokens |
| Output price | $0.50 per million tokens |
| Context window | Up to 1,000,000 tokens in |
| Max output | Up to 512,000 tokens out |
| Model ID (Meta Model API) | `muse-ember-1.0` |

## Benchmarks, including the honest loss

- **Intelligence Index 36**, matching Claude Haiku at about one third of the cost per task.
- Also reported on AutomationBench, Terminal-Bench and the Coding Agent Index.
- **The loss:** FrontierCode 27.2 vs Haiku 34.9. On the hardest frontier coding work, the bigger model still wins.
- It is **not open weights**. You use it through Meta's API and partner clouds.

## How to try it

1. **Meta Model API:** sign up for the Meta Model API, pick model `muse-ember-1.0`, and call it through the Responses API shape.
2. **Muse Code:** Ember is available inside Meta's Muse Code tooling for coding subtasks.
3. **Cloud partners:** also available through Google Cloud and Oracle Cloud.

A safe first job: point Ember at a folder of support tickets or documents and ask for structured extraction (category, summary, action) as JSON. Check 20 results by hand before you trust the pipeline.

## A real workflow: big model plans, Ember does the legwork

1. Your main model plans the task and breaks it into small, checkable steps.
2. Ember runs the volume steps: reading files, extracting fields, routing requests, running tests, making small targeted edits.
3. The main model reviews Ember's outputs and handles anything ambiguous or high stakes.
4. Keep a human check on the first batch of every new job type. Cheap and wrong is still wrong.

## Gotchas

- Not open weights, so no local or self-hosted runs.
- It is tuned for repetitive subtasks, not frontier reasoning. Route hard problems to a bigger model.
- Benchmarks are launch numbers. Test it on your own workload before switching traffic.
- Long context is not the same as good long-context judgment. Spot check summaries of very long documents.

## FAQ

**What is Muse Ember for?**
The busywork inside agent runs: tool calls, routing, extraction, long document processing and small coding subtasks, at very low cost per task.

**How much does Muse Ember cost?**
$0.10 per million input tokens, $0.004 cached input, $0.50 per million output tokens.

**Is Muse Ember open weights?**
No. It is API only, through the Meta Model API, Muse Code, Google Cloud and Oracle Cloud.

**Should Ember replace my main model?**
No. Use it under a planner model: Ember does the volume, the bigger model does the thinking and the checking.

## Links

- Episode: https://youtu.be/tT9CMKiMucI
- Meta Superintelligence Labs Muse Ember announcement, November 9, 2026: https://lapaasvoice.com
