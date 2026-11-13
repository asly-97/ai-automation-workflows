# Does Amazon Kinesis remember between runs?

The practical guide behind the Agentic Pod episode **"Does Amazon Kinesis Remember Between Runs? What a Record Keeps and What It Loses"**.

Watch: https://youtu.be/aOTiPRm-TLM

Read the full post: https://vilix.ai/blog/does-amazon-kinesis-remember-between-runs/

## The problem

A Kinesis record can sit in its stream for 24 hours by default, or up to 8760 hours, a full 365 days, and the next run still starts knowing nothing about the work already done on it. Kinesis Data Streams remembers the record perfectly. It remembers the work not at all. A scheduled automation that reads the stream every night cannot tell which records it already investigated, what it decided, or whether its action succeeded, so on replay it can act twice, for example emailing the same customer twice.

## What a Kinesis record actually keeps

- A sequence number, a partition key, and an immutable data blob. Nobody edits a record after it lands.
- Records live inside shards. A stream is a set of shards, and each shard holds an ordered sequence of records.
- The partition key decides the shard. The same key stays in order inside one shard. Different shards have no combined order.
- Reading never deletes. Several consumers can read the same record independently, and any consumer can replay it inside the retention window.
- Retention is 24 hours by default, extendable to 8760 hours (365 days) for extra cost. Once retention passes, the record is gone, even if nobody ever read it.

## Every pain, and the Vilix AI fix

| Pain | Why it happens | The Vilix AI fix |
|---|---|---|
| The record survives, the work on it does not | A record holds bytes only: no note of which run handled it, what was checked, or what was decided | At run start the workflow calls get_context in Vilix AI and loads which records were already investigated and every decision already made |
| Checkpoints live outside the stream | Consumers pull records and track their own position. With the Kinesis Client Library, checkpoints live in a DynamoDB lease table. That tells you where you stopped reading, not what you did | Vilix AI saves the second half: the decisions, the full reasoning, and the failures, with save_turn at the end of every run |
| Replay is not memory of work | Replay returns the exact starting record byte for byte, with no record of whether the action succeeded | Failures are saved in Vilix AI too, so the next run skips what already failed instead of repeating it. Newest save wins |
| A second workflow starts blind | n8n, Make, Zapier and custom agents each see the stream, not each other's work | The same Vilix AI memory is shared over MCP, so every workflow reads the same past. One account per client for agencies, so memories never mix |
| Standing rules get lost between runs | Stream tooling has nowhere to keep project rules or repeat workflows | Projects and project rules, plus user rules, live in Vilix AI. Skills hold the repeat workflows, and the agent inbox holds notes for the next agent |

Vilix AI is cloud hosted, so there is no database or server to run. It has a free forever plan and a 7 day Pro trial with no credit card, and you can export everything or delete it anytime: https://vilix.ai

## Setup: the two-call loop on every run

1. Connect Vilix AI to your workflow tool over MCP: https://vilix.ai/get-started (under two minutes per tool).
2. At the start of every run, before the workflow reads the stream, call get_context to load the investigated records, the decisions, and the failures.
3. Do the work: read the Kinesis stream from your checkpoint as usual (your own consumer, the Kinesis Client Library with its DynamoDB lease table, or enhanced fan out for a dedicated read path).
4. At the end of the run, call save_turn with a source label on every save, recording which records were handled, what was decided, and whether each action succeeded.
5. Keep standing rules in Vilix AI projects and project rules, and repeat workflows as skills, so the next run and the next tool follow the same rules.

## Prompts

Run start:

```
Before reading the stream, call get_context in Vilix AI. List the Kinesis records already investigated, the decision made for each one, and any failed actions. Skip every record that already has a completed action. Work only on records with no completed action.
```

Run end:

```
Call save_turn in Vilix AI with a source label for this workflow. Save: each record handled (sequence number and partition key), what was checked, the decision, and whether the action succeeded or failed. Save failures too, with the reason.
```

Second workflow handoff:

```
Call get_context in Vilix AI and read what the first workflow saved about this stream. Do not repeat any action it already completed. Continue from its decisions and save your own results back with save_turn when you finish.
```

## FAQ

**Does Amazon Kinesis remember between runs?**
The stream remembers the record, inside its retention window. It does not remember the work: which run handled a record, what was decided, or whether the action succeeded.

**How long does Kinesis keep a record?**
24 hours by default, extendable up to 8760 hours (365 days) for extra cost. After retention passes the record is deleted, read or not.

**Where do Kinesis checkpoints live?**
Outside the stream. Consumers track their own position, and with the Kinesis Client Library the checkpoint lives in a DynamoDB lease table. Enhanced fan out gives a consumer a dedicated read path, and the checkpoint still lives outside.

**Why is replay not enough for scheduled automations?**
Replay hands back the exact starting record with nothing attached. A nightly run that cannot see which records were already investigated can act twice on the same record.

**How does Vilix AI fix it?**
get_context at run start loads the full history of investigated records, decisions and failures. save_turn at run end saves the new work back with source labels, newest wins, shared over MCP with every connected workflow.

## Links

- Episode: https://youtu.be/aOTiPRm-TLM
- Full post: https://vilix.ai/blog/does-amazon-kinesis-remember-between-runs/
- Vilix AI: https://vilix.ai
- Connect Vilix AI: https://vilix.ai/get-started
