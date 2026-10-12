# Run Record Format (what agents save with save_turn)

A consistent format makes every past run searchable by meaning and easy for the next agent to act on.

```
RESULT: success | partial | failed
JOB: <one line, same wording every run of this job>
WHAT I DID: <steps>
WHAT FAILED: <exact error text, endpoint, settings, attempt count>
WHAT WORKED: <the fix or approach that succeeded>
DECISIONS: <what was decided and why>
NEXT RUN SHOULD: <one or two concrete instructions>
```

Example (a failed run that saves the next one):

```
RESULT: failed
JOB: sync new Shopify orders to the CRM
WHAT I DID: called GET /admin/api/2026-07/orders.json with created_at_min = last run
WHAT FAILED: 429 Too Many Requests after 40 calls/sec; retries made it worse
WHAT WORKED: nothing yet
DECISIONS: stop retrying in a loop; it burns the rate limit
NEXT RUN SHOULD: page with limit=250 and wait 500 ms between calls
```

The next run's `get_context("sync new Shopify orders to the CRM")` returns this record first, so it goes straight to paging with a delay instead of hitting the same 429.

Tips:
- Keep `JOB:` wording identical across runs of the same job so search finds them together.
- Put exact error strings in `WHAT FAILED`; `search_keyword` finds them by exact text later.
- Set `source` to the agent's name so you can filter by agent.
