# Skill Examples

A skill is a reusable procedure saved once in Vilix AI and loaded by any agent in any tool with `get_skill`. Agents check `list_skills` before non-trivial tasks.

Create one from any connected tool:

```
Save this as a Vilix AI skill called "release-notes": ...
```

## release-notes

```
When: after a release is tagged.
1. get_context("changes since last release for <project>").
2. Group changes: New, Improved, Fixed. User-facing language only.
3. Max 8 bullets. Link the PRs.
4. Save the result with save_turn, source "release-notes".
```

## new-client-onboarding (agency)

```
When: a new automation client signs.
1. create_project("<client>") with project rules: brand voice, stack, API limits, do-not-do list.
2. create_task for each workflow in the scope, priority from the contract.
3. Clone the matching workflow skills (lead-gen-n8n, shopify-support-bot).
4. save_turn a kickoff summary with source "onboarding".
```

## debug-failed-run

```
When: an agent run reports RESULT: failed.
1. search_keyword(<exact error string>) across all sources.
2. If a past run recorded WHAT WORKED for this error, apply it.
3. If not, try one new approach, then save_turn with the full run record.
```
