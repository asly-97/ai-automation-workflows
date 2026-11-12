# Does Devin remember between sessions?

The practical guide behind the Agentic Pod episode **"Does Devin Remember Between Sessions? What Carries Over and What Starts Blank"**.

Watch: https://youtu.be/7B98vZHzhDs
Read the full post: https://vilix.ai/blog/does-devin-remember-between-sessions-honest-breakdown/

Short answer: the session itself does not carry over. Each Devin session runs in its own isolated VM, and when the session ends or sleeps, the conversation and the reasoning end with it. What does carry over is curated: Knowledge entries, Memory notes organized by Dreaming, and snapshots. Those keep notes and machines, not the full story of the work.

## How a Devin session works

- You give Devin a ticket. It plans, codes, tests, and opens a pull request on its own.
- While the session is alive, it keeps responding to pull request comments on that work. That is live context, not saved memory.
- Once the session ends or sleeps, that thread is done. The next session starts with no memory of the work, the decisions, or the fixes that already failed.

## What actually carries over

| Piece | What it holds | What it misses |
|---|---|---|
| Knowledge entries | A trigger description matched to your current work, plus a short body with the fact or rule. Scoped to one repo, the organization, or the enterprise. Start a prompt with `!` and the entry name to drop an entry into a prompt | Only fires when the trigger matches. It is a note card, not a diary |
| Memory notes | Preferences, corrections, and project lessons saved across sessions, kept organized by Dreaming | The full conversation. Notes keep the lesson, not why the agent tried 3 fixes and dropped 2 |
| Snapshots | The machine: repo checkout, dependencies, and env vars | The mind: why the code changed the way it did |

## The Monday to Tuesday problem

1. Monday: Devin finishes a ticket, tries 3 fixes, drops 2 for good reasons, and opens a clean pull request.
2. Tuesday: a new session starts in a fresh VM. It never met Monday Devin.
3. The same dead ends look brand new, so the next session can repeat work that was already ruled out.
4. A scheduled Devin session is worse: it wakes up blind, with zero recollection, unless Knowledge, Memory notes, or a hand built playbook hands it context first. A playbook helps, but you build it and keep it current by hand.
5. Devin Desktop splits the picture: Cascade Memories do persist (some auto generated, some user rules at global, workspace, and system level), while the Devin Local agent does not persist memories between sessions at all. Users are told to move critical memories into skills.

## The fix: give Devin a memory with Vilix AI

Vilix AI is the shared memory layer that sits next to Devin. Devin keeps doing the coding. Vilix AI carries the past. It is cloud hosted, so there is no database to run, and the same memory works over MCP for Devin, a review chat, and every other connected tool.

The session loop:

1. **At session start:** Devin calls `get_context` in Vilix AI. That loads the full project history: every saved conversation, the decisions, the failures, and the fixes that already failed.
2. **Do the work:** plan and code against that history instead of starting blank, and skip the approaches that already failed, because **failures are saved too**.
3. **At session end:** Devin calls `save_turn` in Vilix AI, with a **source label** on every save, for example `devin: repo-name`. The full conversation history is saved, not just a few facts.
4. **Newest wins:** if Monday and Tuesday disagree, the newest save is the one that counts.
5. **Projects** keep one job's memory with that job. **Project rules** and **user rules** ride along on every read, so every session works under the same instructions.
6. **Skills** store reusable procedures the next session picks up, and the **agent inbox** lets you leave a message that a scheduled session reads on its next run.

## Setup

1. Create a Vilix AI account at https://vilix.ai (free forever plan, no credit card; the 7-day Pro trial also needs no credit card).
2. Connect Vilix AI to Devin and to the other tools you use: https://vilix.ai/get-started (under two minutes per tool, over MCP).
3. Add the session loop to your Devin workflow: `get_context` as the first step of every session, `save_turn` as the last step before the session ends.
4. Give each repo or client its own project in Vilix AI, and put standing instructions in project rules and user rules.
5. Agencies: use **one Vilix AI account per client**, so memories never mix.

## Copy-paste prompts

**Start of session:**

```
Call Vilix AI get_context first: "Devin session start: full project history, decisions, failures, fixes already ruled out". Use that history. Do not retry an approach that already failed unless the code has changed since.
```

**End of session:**

```
Call Vilix AI save_turn with source "devin: <repo>". Save: what the ticket asked, what you changed, which approaches you tried and dropped and why, test results, and any failure with the exact error.
```

**Save a lesson for every future session:**

```
Save to Vilix AI: when this repo's tests fail on a fresh checkout, run the setup step first; the failure is a missing generated file, not a code bug.
```

## FAQ

**Does Devin remember between sessions?**
The session does not. Knowledge entries, Memory notes, and snapshots carry over, but the conversation, the reasoning, and the failed attempts end with the session's VM.

**What is the difference between Knowledge and Memory in Devin?**
Knowledge entries are curated cards: a trigger plus a short body, pulled in when the trigger matches or when you invoke one with a `!` macro. Memory notes are saved preferences, corrections, and project lessons, organized by Dreaming. Both are note level, not the full conversation.

**Do snapshots save my work?**
Snapshots save the machine state: repo checkout, dependencies, and env vars. They do not save why the code changed.

**Does Vilix AI replace Devin Knowledge or Memory?**
No. Keep them for curated cards and notes. Vilix AI carries the full history underneath, and it follows you into your other tools too, not just Devin.

**Can I export or delete what Vilix AI stores?**
Yes. Export everything or delete it anytime.

## Links

- Episode: https://youtu.be/7B98vZHzhDs
- Blog post: https://vilix.ai/blog/does-devin-remember-between-sessions-honest-breakdown/
- Vilix AI: https://vilix.ai
- Connect a tool: https://vilix.ai/get-started
