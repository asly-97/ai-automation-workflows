# Fin Remembers the Chat. Vilix AI Remembers the Work

Intercom Fin is genuinely strong inside one conversation. It tracks long exchanges, runs multi-step procedures like processing payments or updating accounts, and passes the complete conversation to a human on handoff. What it does not do is carry a customer's story forward across conversations, and the scheduled agents around it (nightly sweeps, refund checks, churn-risk flags, survey follow-ups) never see what Fin learned at all.

This guide is the technical companion to Agentic Pod EP65, "Fin Remembers the Chat. Vilix AI Remembers the Work". It covers where Fin's memory actually ends, why your scheduled agents wake up blank, and how to give every agent one shared memory: [Vilix AI](https://vilix.ai).

- Watch the episode: https://youtu.be/cBHi-7xzBjU
- Read the full post: https://vilix.ai/blog/fin-remembers-the-conversation-your-scheduled-agents-wake-up-blank/

## The problem

Fin's memory is real but bounded:

- Inside the chat: Fin tracks details across long exchanges, executes multi-step procedures, and hands off the complete conversation.
- Learning is controlled: when Fin resolves conversations, it can propose new knowledge snippets for your team to review. The queue is capped at roughly 100 items, and anything left unreviewed expires after about four weeks. Nothing a customer said becomes Fin's knowledge until a human approves it.
- Retrieval is not remembering: when Fin needs a fact, it fetches one from help centers, internal articles, public URLs, PDFs, connected tools like Notion or Confluence, and live CRM data. That only works for what is already written down somewhere.
- Across conversations: Fin has no automatic, per-customer memory that survives the end of a conversation. A customer who argued a billing charge in March and returns in June is a fresh face. The promises made, the edge cases uncovered, and the tone the last thread ended on do not ride along automatically.

Even Intercom's own framing gives it away: the argument for one agent owning the whole customer lifecycle is that separate agents "won't have shared memory and goals." Shared memory is the aspiration, not the current state.

## The blind spot: your scheduled agents never meet Fin

Consider the follow-up workflow. A customer gets a refund approved on Monday. On Wednesday, a scheduled agent is supposed to check whether the refund actually landed and message the customer either way. To do that job, the agent needs to know the refund was approved, the amount, the promised timeline, and the account it went to. Fin knows all of that. The scheduled agent knows none of it.

So the agent wakes up blank and does the only thing it can: re-read the thread, re-derive the facts, and hope it parsed the outcome correctly. Multiply that by every scheduled run across your stack, and you pay for the same comprehension work over and over. Worse, the agent will sometimes invent the missing pieces: following up on an issue Fin already closed, or asking the customer a question that was answered three days ago.

This is the core pain of scheduled agents everywhere: competent inside the run, amnesiac between runs. Fin's memory stays inside the conversation window. Your automations live outside it. Nobody connects the two.

## Every pain, and the Vilix AI fix

| Pain | How Vilix AI fixes it |
| --- | --- |
| A returning customer is a fresh face to Fin | Full conversation history lives in Vilix AI, outside the chat window, so the next conversation or agent starts from the real exchange |
| Scheduled agents re-read threads and re-derive facts on every run | At the start of a run, the agent calls `get_context` in Vilix AI and loads the saved outcome instead of reconstructing it |
| Promises and open items get lost between the chat and the follow-up | At the end of every conversation, the outcome is saved with `save_turn` in Vilix AI, with a source label: who the customer is, what happened, what was promised, what stays open |
| The follow-up agent invents missing pieces or re-asks answered questions | The agent reads the actual conversation from Vilix AI, not a summary of a summary, before it acts |
| An exact ticket number or order ID must match | Vilix AI combines semantic and keyword retrieval, so agents find what they meant and still match exact strings |
| Standing promises and team rules get re-explained | Projects, project rules and user rules live in Vilix AI and apply to every connected tool |
| Repeat support workflows are rebuilt per tool | Repeat workflows become skills in Vilix AI; agents can also leave each other notes in the agent inbox |
| Agencies run support automations for several clients | One Vilix AI account per client keeps each client's memory separate and portable |
| You want out | Export everything in a portable format or delete it anytime; Vilix AI is cloud hosted with zero infrastructure for you to run |

## Setup

1. Create a free Vilix AI account at https://vilix.ai/get-started (free forever plan; 7 day Pro trial, no credit card).
2. Connect Fin's surrounding stack to the same Vilix AI account over MCP: your scheduled agents (n8n, Make, custom jobs) and the AI tools your team uses (ChatGPT, Claude, Claude Code, Codex, Cursor, or any MCP client), following the per-tool steps on the get-started page.
3. Add a write step at the end of every support conversation: save the outcome (customer, what happened, what was promised, what stays open) to Vilix AI with `save_turn` and a source label like `intercom-fin`.
4. Add a read step at the start of every scheduled run: call `get_context` in Vilix AI for that customer before the agent does anything else.
5. Put standing promises and escalation rules in a Vilix AI project as project rules, so every agent follows the same rules.

## Prompts

End-of-conversation write (run by your automation after Fin closes a thread):

```
Save this support outcome to Vilix AI with save_turn, source "intercom-fin":
customer, issue, what was decided, what was promised (with amounts and dates),
and what is still open. Keep the full exchange, not just a summary.
```

Start-of-run read (first step of every scheduled agent):

```
Before you act, call get_context in Vilix AI for this customer.
Use the saved outcomes: what was promised, what is still open, and the tone
of the last conversation. Do not ask the customer anything that was already
answered in a saved conversation.
```

Refund check follow-up:

```
Call get_context in Vilix AI for this customer and find the refund approval:
amount, promised timeline, and destination account. Check whether the refund
landed. Message the customer either way, referencing the original promise.
If no approval is saved, do nothing and flag the thread for a human.
```

## FAQ

**Does Fin remember customers between conversations?**
Not automatically. What persists between two separate chats is the reviewed knowledge base plus live lookups, not a running picture of the customer. That is the gap Vilix AI fills.

**Is retrieval the same as memory?**
No. Retrieval is looking something up in the moment, and it only works for what is written down somewhere already. Memory is the outcome of the last conversation being there before the next run starts.

**Do I need to replace Fin?**
No. Keep Fin for the chat. The pattern in this guide connects the chat to the agents around it through Vilix AI, so the whole operation acts like one system.

**What does Vilix AI cost to start?**
The free plan is free forever. Full Pro can be tried for 7 days with no credit card required.

**Can I get my data out of Vilix AI?**
Yes. Export everything in a portable format whenever you want, delete individual memories, or wipe the account instantly.

## Links

- Vilix AI: https://vilix.ai
- Get started: https://vilix.ai/get-started
- Full post: https://vilix.ai/blog/fin-remembers-the-conversation-your-scheduled-agents-wake-up-blank/
- Episode: https://youtu.be/cBHi-7xzBjU
