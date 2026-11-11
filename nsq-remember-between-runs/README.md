# Does NSQ remember between runs?

The practical guide behind the Agentic Pod episode **"Does NSQ Remember Between Runs? The Next Message Starts Blank"**.

Watch: https://youtu.be/PQ8Ik-aSglQ
Read the full post: https://vilix.ai/blog/does-nsq-remember-between-runs/

Short answer: no. NSQ is a realtime distributed messaging platform. It moves messages reliably, and it never records what a consumer found, decided or drafted. The next message starts blank.

## How NSQ moves a message

- A publisher sends a message to a **topic**. Topics and channels are created on first use.
- Every **channel** on that topic gets its own copy of the message.
- Inside a channel, each message goes to **one** connected consumer. Channels buffer independently.
- A channel holds messages in memory up to `mem-queue-size`, which defaults to **10,000** per topic and channel. Overflow goes to a disk backed queue. If `mem-queue-size` is 0, everything writes to disk. It is still a queue, not a notebook.
- **Ephemeral channels** discard overflow and vanish when the last consumer disconnects.

## Why the next run starts blank

A consumer ends a message one of three ways, and none of them writes a result:

| Ending | What NSQ does | What gets remembered |
|---|---|---|
| `FIN` | Message finished and removed | Nothing |
| `REQ` | Message requeued for another try | Nothing, the body is unchanged |
| Timeout | NSQ auto requeues the message | Nothing, the body is unchanged |

Delivery is **at least once**, so duplicates are possible. There is no result record, no notes field, and no history the next message can read.

## The Monday to Tuesday problem

Picture a scheduled AI consumer on a feedback channel:

1. Monday: it groups a batch of feedback by theme, flags two repeats of a known request, drafts replies, then sends `FIN`.
2. Tuesday: a new batch arrives. The consumer starts from the new messages only and groups everything from scratch.
3. The known request looks brand new, so it can get flagged as new again. Same work, done twice.
4. If Monday's run failed halfway, that failure leaves no trace either. Tuesday cannot learn from it.

The queue remembers the message. It never remembers the finding.

## The fix: give the consumer a memory with Vilix AI

Vilix AI is the shared memory layer that sits next to NSQ. NSQ keeps doing delivery. Vilix AI carries the past. It is cloud hosted, so there is no extra database to run next to `nsqd`, and the same memory works over MCP for the consumer, a review chat, and every other connected tool.

The run loop:

1. **Before the run:** the consumer calls `get_context` in Vilix AI. That loads the saved themes, the known requests, the drafted replies, and why each earlier call was made.
2. **Do the work:** group only what is genuinely new, and check new items against the saved known requests before flagging anything.
3. **After the run:** the consumer calls `save_turn` in Vilix AI, with a **source label** on every save, for example `nsq-consumer: feedback`. The full conversation history is saved, not just a few facts.
4. **Failures are saved too**, so a bad run teaches the next one instead of repeating silently.
5. **Newest wins:** if Monday and Tuesday disagree, the newest save is the one that counts.
6. **Projects** keep one job's memory with that job. **Project rules** and **user rules** ride along on every read, so the consumer always works under the same instructions.
7. **Skills** store reusable procedures the next agent picks up, and the **agent inbox** lets you leave a message that a scheduled agent reads on its next run.

## Setup

1. Create a Vilix AI account at https://vilix.ai (free forever plan, no credit card; the 7-day Pro trial also needs no credit card).
2. Connect Vilix AI to the tool or agent runtime your consumer uses: https://vilix.ai/get-started (under two minutes per tool, over MCP).
3. Add the run loop to the consumer: `get_context` as the first step after taking a message, `save_turn` as the last step before `FIN`.
4. Give each job its own project in Vilix AI, and put standing instructions in project rules and user rules.
5. Agencies: use **one Vilix AI account per client**, so memories never mix.

## Copy-paste prompts

**Start of run:**

```
Call Vilix AI get_context first: "feedback consumer run: saved themes, known requests, drafted replies, open failures". Use that history. Do not flag anything as new until you have checked it against the saved known requests.
```

**End of run:**

```
Call Vilix AI save_turn with source "nsq-consumer: feedback". Save: themes found, new requests, repeats matched to known requests, replies drafted, and any failure with the exact error.
```

**Save a lesson for every future run:**

```
Save to Vilix AI: when a feedback item matches a known request, link it to the existing request instead of opening a new one.
```

## FAQ

**Does NSQ store messages on disk?**
Yes, past the in-memory limit a channel writes to a disk backed queue. That is still queued messages waiting for delivery, not a record of results.

**Can I put the results back into NSQ as a new message?**
You can, but then every consumer has to parse result messages out of the queue, dedupe them, and rebuild history by hand. A memory layer does that job properly, with search, projects and rules.

**Does Vilix AI replace NSQ?**
No. Keep NSQ for delivery. Vilix AI only carries what NSQ was never built to carry: results, decisions, failures and rules.

**Can I export or delete what Vilix AI stores?**
Yes. Export everything or delete it anytime.

## Links

- Episode: https://youtu.be/PQ8Ik-aSglQ
- Blog post: https://vilix.ai/blog/does-nsq-remember-between-runs/
- Vilix AI: https://vilix.ai
- Connect a tool: https://vilix.ai/get-started
- NSQ: https://nsq.io
