# AI Visibility Content Pipeline

A daily pipeline that gets a small product cited in AI answers (ChatGPT, Perplexity, Google AI Overviews) by publishing one focused, specific article per day. Agents do the research, drafting, publishing and tracking. I only pick the best draft and leave notes.

## The problem

AI answer engines pull from pages that answer a specific question clearly and with real detail. To show up for the questions people in your niche actually ask, you need a lot of those pages. Writing one a day by hand is not realistic when you're also building the product.

The first version of this pipeline was fully automatic and had two problems:

1. **Duplicates.** Each run started blank. The queue in Notion only had titles, so the agent kept writing the same article from a slightly different angle a few days apart.
2. **No quality gate.** Everything it wrote went live, good or bad.

The current version fixes both: every run reads the full history of past runs before writing, and nothing publishes without my approval.

## Architecture

```
                 +------------------------+
   schedule ---> |  Draft agent (Claude)  | <---- reads ----+
   (morning)     +------------------------+                 |
                   |  pulls next question                   |
                   v                                        |
             +-----------+                          +---------------+
             |  Notion   |                          |   Vilix AI    |
             |  queue    |                          | (memory layer)|
             +-----------+                          +---------------+
                   |                                        ^
                   |  3 drafts sent to me                   |
                   v                                        |
             +-----------+     approve + notes     +-----------------+
             |    Me     | ----------------------> | Publish agent   | -- saves run --+
             +-----------+                         +-----------------+
                                                     |        |
                                                     v        v
                                                 +-------+ +----------+
                                                 | Ghost | | IndexNow |
                                                 +-------+ +----------+

   schedule ---> Ranking agent (hourly) ---> checks each target question ---> logs to memory
   (hourly)
```

## Stack

| Tool | Job in this workflow | Link |
|---|---|---|
| Claude (scheduled tasks) | Runs the draft, publish and ranking agents | [claude.ai](https://claude.ai) |
| Notion | Content queue: target question, status, live URL | [notion.so](https://www.notion.so) |
| Ghost | The blog. Posts are created through the Admin API | [ghost.org](https://ghost.org) |
| Vilix AI | Memory layer over MCP. Every run reads past runs before it writes, and saves itself when it's done | [vilix.ai](https://vilix.ai) |
| IndexNow | Tells Bing and other engines a new URL exists, right after publishing | [indexnow.org](https://www.indexnow.org) |

## How a day runs

1. **Morning trigger.** The draft agent takes the oldest `Queued` row from the Notion queue. Each row is one real question people type into AI tools in my niche.
2. **Read memory first.** Before writing anything, the agent pulls every past run: published titles, angles, target questions, which drafts I picked, which I rejected, and my notes. If the question or angle is too close to something already live, it skips to the next row.
3. **3 drafts.** It writes 3 versions of the article, each with a different angle, and sends them to me.
4. **I review.** I pick one and leave notes if something is off. This is the only manual step.
5. **Publish.** The publish agent applies my notes, creates the post in Ghost, pings IndexNow with the new URL, and writes the live URL and date back to the Notion row.
6. **Save the run.** The full run goes into memory: the question, the 3 angles, which one I picked, why the others lost, my notes, and the live URL. The next morning's drafts start from this.
7. **Track.** An hourly ranking agent searches each target question and logs where the post appears. Future topics lean toward what's working.

## Why the memory layer matters here

Notion holds the queue, but a queue only knows titles. The agent needs the *reasoning* from past runs: which angles were already used, which drafts I rejected and why, what my notes said. That lives in full run history, not in a database row.

Vilix AI is connected to Claude as an MCP server. The prompts call two tools:

- `get_context` at the start of a run, with the target question. It returns relevant past runs.
- `save_turn` at the end of a run, with the full record of what happened.

Because the memory is shared, the ranking agent, the draft agent and the publish agent all read and write the same history. The ranking agent's findings show up in the draft agent's context the next morning without any glue code.

## Setup

### 1. Notion queue

Create a database with the schema in [`config/notion-content-queue.md`](./config/notion-content-queue.md). Fill it with real questions. Good sources: Google's "People also ask", AlsoAsked, the questions your users send you, and threads in your niche.

### 2. Ghost

Create a custom integration in Ghost (Settings > Integrations) to get an Admin API key. The publish agent creates posts with `POST /ghost/api/admin/posts/?source=html`.

### 3. IndexNow

Generate a key, host the key file at the root of your domain, then ping after each publish:

```
GET https://api.indexnow.org/indexnow?url=<LIVE_URL>&key=<YOUR_KEY>
```

A `200` or `202` response means it was accepted.

### 4. Memory

Add Vilix AI as a custom connector in Claude using the MCP endpoint `https://api.vilix.ai/mcp`. The record each run saves is described in [`config/memory-record-format.md`](./config/memory-record-format.md).

### 5. Scheduled agents

Create three scheduled tasks in Claude, one per prompt:

| Agent | Schedule | Prompt |
|---|---|---|
| Draft agent | Every morning | [`prompts/01-morning-drafts.md`](./prompts/01-morning-drafts.md) |
| Publish agent | After approval | [`prompts/02-publish-approved.md`](./prompts/02-publish-approved.md) |
| Ranking agent | Hourly | [`prompts/03-ranking-check.md`](./prompts/03-ranking-check.md) |

## Writing for AI answers: what I've learned

- **One question per article**, answered in the first two sentences. AI engines quote the part that answers the question; don't make them dig.
- **Real specifics.** Exact behavior, limits, versions, steps. Generic filler gets skipped.
- **Target questions, not keywords.** "Does X remember between sessions?" beats "X memory".
- **Keep it current.** Outdated details are the fastest way to lose trust, with readers and with AI engines.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Near-duplicate articles | Agent skipped the memory read | Make `get_context` the first step in the draft prompt, and have it list the past angles it found before writing |
| My notes ignored in later drafts | Notes not saved with the run | Check the publish agent saves the notes field (see the memory record format) |
| Post live but not indexed | IndexNow key file not reachable | Open `https://yourdomain.com/<key>.txt` in a browser; it must return the key |
| Ghost returns 401 | Admin API token expired | Ghost tokens are short-lived JWTs; generate a fresh one per request |
| Ranking agent shows "not ranked" for weeks | Question too competitive, or page not indexed | Check indexing first, then move the question down the queue |
