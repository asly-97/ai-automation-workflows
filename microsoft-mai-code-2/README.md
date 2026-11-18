# Microsoft MAI Code 2: near-frontier coding at 5% of the cost?

The practical guide behind the Agentic Pod episode **"Microsoft MAI Code 2: Near-Frontier Coding at 5% of the Cost?"**.

Watch: https://youtu.be/rln4NOry2LI

Microsoft launched MAI Code 2 and MAI Code 2 Flash at Ignite on November 17, 2026, and both are generally available now in GitHub Copilot. Microsoft claims Code 2 reaches 98% of GPT-6.1 Sol performance at 40% of the cost, and 96% of Claude Opus 5.5 performance at 5% of the cost, on selected coding evaluations at maximum reasoning effort. The MAI cost math uses a 50% promotional price, and Microsoft says the models are not yet tuned for harder problems, including very long running agent work. This guide covers what shipped, the headline numbers with the full fine print, how to try it in GitHub Copilot step by step, a real workflow, and the gotchas.

Sources: https://bravenewcoin.com/insights/microsoft-mai-code-2-ai-price-war and https://www.epcgroup.net/insights/microsoft-ignite-2026-day-1-announcements

## What shipped

- MAI Code 2 is Microsoft's own coding model, built for everyday engineering work: debugging, changes across multiple files, and user interface work.
- MAI Code 2 Flash is the smaller, faster model, aimed at high volume jobs: explanations, targeted edits, and routine fixes.
- Both models are generally available now inside GitHub Copilot. There is no waitlist and no new tool to install; they sit in the Copilot model picker next to the models you already use.
- Code 2 Flash is also available in a limited, private preview in Microsoft Foundry, for teams that build on Foundry.

## The numbers, with the fine print

Microsoft's own comparison, as reported at the Ignite launch:

- Code 2 reaches 98% of GPT-6.1 Sol performance at 40% of the cost.
- Code 2 reaches 96% of Claude Opus 5.5 performance at 5% of the cost.

Now the fine print, which matters more than the headline:

- The scores are averaged over selected coding evaluations (DeepSWE 1.1, TerminalBench 2.1, SWE-Atlas, FrontierCode 1.1), run at maximum reasoning effort.
- Three of the tests ran on Microsoft's internal platform, through the GitHub Copilot CLI, without subagents.
- The figures are vendor reported and have not been independently reproduced.
- The MAI cost math uses a 50% promotional price. The comparison is real on paper, but it is priced at half price.
- A relative score on selected tests does not mean the model finishes that share of your real tasks. Your repo is the only test that counts.

## How to try it in GitHub Copilot, step by step

1. Open GitHub Copilot, in your editor or in the Copilot CLI.
2. Open the model picker, the same picker where you choose between models today.
3. Pick MAI Code 2 for a bigger job, like debugging across files. Pick MAI Code 2 Flash for a quick job, like an explanation or a targeted edit.
4. Give it a real task from your own work, not a toy prompt. For example: explain this function and fix the failing test next to it.
5. Compare honestly. Run the same task with your usual model, then check the result and the bill. A cheaper model that needs three retries can cost more than a pricey model that nails it first try.
6. Optional: if your team builds on Microsoft Foundry, Code 2 Flash is there in limited preview.

## A real workflow: split the day by job size

1. The pile of small fixes goes to Flash. Explain this code, make this targeted edit, fix this routine bug. Fast, cheap, high volume work.
2. The bug report goes to Code 2. Debugging across files is exactly what Microsoft built it for: it reads the repo, follows the change across files, and works on the fix.
3. The one truly hard problem keeps the expensive frontier model. You split the day by job size, not by loyalty to one model.

## Gotchas and limits

- Not yet tuned for harder problems. Microsoft says so itself: that includes fundamental science and very long running agent work. Overnight autonomous agent runs are not the pitch.
- Promo pricing. The cheap cost math depends on a 50% promotional price. Do not budget a whole year on a launch discount.
- Vendor run benchmarks. Selected tests, maximum reasoning, internal platform. Promising, but not independently reproduced yet.
- Cheap per call is not cheap per task. Retries add up. Measure cost per finished task on your own work.

## FAQ

**What is MAI Code 2?**
Microsoft's own coding model, launched at Ignite on November 17, 2026. It is built for everyday engineering work like debugging, multi file changes, and user interface work, and it is generally available in GitHub Copilot.

**What is MAI Code 2 Flash?**
The smaller, faster model in the same launch, aimed at high volume tasks like explanations, targeted edits, and routine fixes. It is in GitHub Copilot and in limited preview in Microsoft Foundry.

**Should I trust the 98% and 96% numbers?**
They are Microsoft's own figures, averaged over selected coding evaluations at maximum reasoning effort, with three tests on Microsoft's internal platform, and the cost side uses a 50% promotional price. Treat them as a reason to test the models on your own work, not as a settled result.

**How do I try MAI Code 2?**
Open GitHub Copilot, open the model picker, pick Code 2 or Code 2 Flash, and give it a real task from your own work. Compare the result and the bill against your usual model.

**Is MAI Code 2 good for long running agent work?**
Microsoft says the models are not yet tuned for harder problems, including very long running agent work. Keep a frontier model for the hardest, longest jobs.

## Links

- Episode: https://youtu.be/rln4NOry2LI
- Launch report: https://bravenewcoin.com/insights/microsoft-mai-code-2-ai-price-war
- Ignite 2026 Day 1 announcements: https://www.epcgroup.net/insights/microsoft-ignite-2026-day-1-announcements
