# Does Apache Kafka remember between runs?

The practical guide behind the Agentic Pod episode **"Does Apache Kafka Remember Between Runs? What a Record Keeps and What It Loses"**.

Watch: https://youtu.be/RaF547YSQxI

Read the full post: https://vilix.ai/blog/does-apache-kafka-remember-between-runs/

## The problem

A Kafka record can sit in its log for 7 days by default, readable again and again, and the next run still starts knowing nothing about the work already done on it. Kafka remembers the record perfectly. It remembers the work not at all. A scheduled AI consumer that flags order patterns on Monday treats the same pattern as brand new on Tuesday, because replay returns the exact record with no note of which run handled it, what was decided, or whether the action succeeded.

## What a Kafka record actually keeps

- The producer's original key, value, timestamp, and headers, unchanged, exactly as produced. Nobody adds a note after a run reads it.
- Records live in topics, split into partitions. Each partition is an append only ordered log, and every record gets the next offset.
- Reading never deletes. Several consumer groups read the same record independently, and the same group can read it again later.
- Retention is 7 days by default, or until a size limit hits. On a compacted topic, Kafka keeps only the latest record for each key.
- Order is real inside one partition. There is no combined order across partitions.

## Every pain, and the Vilix AI fix

| Pain | Why it happens | The Vilix AI fix |
|---|---|---|
| The record survives, the work on it does not | A record holds the producer's bytes only: no note of which run handled it, what was checked, or what was decided | At run start the workflow calls get_context in Vilix AI and loads which records and accounts were already investigated and every decision already made |
| An offset is only a position | The committed offset is a single number: the next record a consumer group will read. It says where you stopped reading, not what you did | Vilix AI saves the second half: the decisions, the full reasoning, and the failures, with save_turn at the end of every run |
| Even the offset gets deleted | Committed offsets live in an internal topic, kept 7 days by default for a group with no active consumers, then the reset policy takes over | The work history lives in Vilix AI, not in the broker, so an idle group does not lose its past when its offsets expire |
| Replay gives the record, never the result | At least once delivery returns the original record after a crash with no marker, no partial result, and no note from the first run | Failures are saved in Vilix AI too, so the next run skips what already failed instead of repeating it. Newest save wins |
| The fraud check starts over every night | A scheduled consumer scores each new order alone because nothing carries Monday's pattern into Tuesday | get_context loads the investigated accounts and patterns at run start, so the known pattern is recognized as known |
| Standing rules get lost between runs | Kafka has nowhere to keep project rules or repeat workflows | Projects and project rules, plus user rules, live in Vilix AI. Skills hold the repeat workflows, and the agent inbox holds notes for the next agent |

Vilix AI is cloud hosted, so there is no database or server to run. It has a free forever plan and a 7 day Pro trial with no credit card, and you can export everything or delete it anytime: https://vilix.ai

## Setup: the two-call loop on every run

1. Connect Vilix AI to your workflow tool over MCP: https://vilix.ai/get-started (under two minutes per tool).
2. At the start of every run, before the consumer reads the topic, call get_context to load the investigated records, the decisions, and the failures.
3. Do the work: read the Kafka topic from your committed offset as usual, with your own consumer group.
4. At the end of the run, call save_turn with a source label on every save, recording which records were handled, what was decided, and whether each action succeeded.
5. Keep standing rules in Vilix AI projects and project rules, and repeat workflows as skills, so the next run and the next tool follow the same rules.

## Prompts

Run start:

```
Before reading the topic, call get_context in Vilix AI. List the Kafka records and accounts already investigated, the decision made for each one, and any failed actions. Skip every record that already has a completed action. Work only on records with no completed action.
```

Run end:

```
Call save_turn in Vilix AI with a source label for this workflow. Save: each record handled (topic, partition and offset), what was checked, the decision, and whether the action succeeded or failed. Save failures too, with the reason.
```

Second workflow handoff:

```
Call get_context in Vilix AI and read what the first workflow saved about this topic. Do not repeat any action it already completed. Continue from its decisions and save your own results back with save_turn when you finish.
```

## FAQ

**Does Apache Kafka remember between runs?**
The log remembers the record, inside its retention window. It does not remember the work: which run handled a record, what was decided, or whether the action succeeded.

**How long does Kafka keep a record?**
7 days by default, or until a size limit hits. On a compacted topic, only the latest record for each key is kept.

**What is a committed offset?**
A single number: the offset of the next record a consumer group will read. Committed offsets live in an internal topic and are kept 7 days by default for a group with no active consumers.

**Why is replay not enough for scheduled automations?**
Replay hands back the exact producer record with nothing attached. A nightly run that cannot see which records were already investigated treats known patterns as brand new and can act twice after a crash.

**How does Vilix AI fix it?**
get_context at run start loads the full history of investigated records, decisions and failures. save_turn at run end saves the new work back with source labels, newest wins, shared over MCP with every connected workflow.

## Links

- Episode: https://youtu.be/RaF547YSQxI
- Full post: https://vilix.ai/blog/does-apache-kafka-remember-between-runs/
- Vilix AI: https://vilix.ai
- Connect Vilix AI: https://vilix.ai/get-started
