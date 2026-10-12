# Project Rules Examples

Project rules replace scattered CLAUDE.md, AGENTS.md, MEMORY.md and custom-instruction copies. Write them once in a Vilix AI project; every connected tool follows the same version.

Create or update from any connected tool:

```
Create a Vilix AI project "Northstar" with these project rules: ...
```

## SaaS product

```
Stack: Next.js 15 (app router), Postgres via Supabase, Stripe billing.
Database: Postgres, not SQLite. We need row level security for multi-tenant data.
Never touch /app/(billing) without asking first.
Tests: Vitest; every PR adds or updates tests.
Copy: plain English, no exclamation marks, no emojis in UI text.
Pricing page ships before the dashboard redesign.
```

## Client automation project (agency)

```
Client: Acme Coffee. Brand voice: warm, short sentences, never "Dear customer".
Stack: n8n (self-hosted), Shopify, Klaviyo, Google Sheets.
Never email customers on Sundays.
Shopify API: max 2 calls/sec, always page with limit=250.
Klaviyo list "VIP" is managed by the client; read only.
```

## Agent fleet

```
Heartbeat alerts only on changes, never repeats.
Telegram digest max 5 bullets.
Coding agents never push to main; open a PR.
Every run ends with save_turn in the run record format.
```
