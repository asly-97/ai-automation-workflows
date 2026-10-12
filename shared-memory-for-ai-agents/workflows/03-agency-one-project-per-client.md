# Workflow 3: Agency, One Brain Per Client

**For:** AI agencies, automation freelancers and small studios that build AI agents and automations for businesses (support agents, lead-gen flows, n8n and Make pipelines, OpenClaw and Hermes agents), and anyone who sells standalone AI agents.

**Pains it removes:** building a memory system from scratch for every client, client contexts mixing together, and the workflow that breaks six months later with no record of why it was built that way.

There are two ways to set this up. Most agencies use **Model A**.

## Model A (recommended): one Vilix AI account per client

Every client gets their **own Vilix AI account**, the same way a business gets its own Gmail or Slack workspace. That account is the long-term memory for everything you build for them.

```
Client: Acme Coffee            Client: Brightline Legal        Client: Nova Fitness
Vilix AI account (Acme)        Vilix AI account (Brightline)   Vilix AI account (Nova)
 ├─ support agent  (OpenClaw)   ├─ intake agent   (Hermes)      ├─ booking agent (n8n + Claude)
 ├─ order sync     (n8n)        ├─ doc parser     (Claude Code) └─ weekly report (ChatGPT task)
 └─ weekly report  (ChatGPT)    └─ follow-ups     (Make)
```

**Who pays:** either you include the Vilix AI plan in your monthly retainer (you create and maintain the account, the client never touches it), or the client pays for their own plan and gives your agents access. Either way, the memory belongs to the client's business, not to your laptop.

**Why agencies like it:**
- **No memory to build.** No Postgres, Redis or vector database to set up per client. The memory layer is already built; you wire it up.
- **Clean separation.** Each client's data lives in its own account, so nothing can bleed between clients.
- **Easy handover.** If the client takes the system in-house, the account (and every decision and run history in it) goes with them.
- **A better retainer.** "Your agents learn from every run and remember every decision" is a concrete thing to sell and maintain month after month.

### Set it up for a new client

1. **Create the client's Vilix AI account** at [vilix.ai](https://vilix.ai) (with the client's email if they own it, or your agency email if you manage it under the retainer).
2. **Create an API key** in that account: **Agents → Create API key**. One key per client account. Store it in your secret manager, never in a prompt.
3. **Connect every agent and automation you build for that client** to `https://api.vilix.ai/mcp` with that client's key as `Authorization: Bearer <CLIENT_VILIX_API_KEY>`:
   - OpenClaw: `openclaw mcp add vilix --url https://api.vilix.ai/mcp --transport streamable-http --header "Authorization=Bearer <CLIENT_VILIX_API_KEY>"`
   - Hermes: `hermes mcp add vilix --url https://api.vilix.ai/mcp --auth header`
   - Any other MCP client: [setup guide](../setup/README.md)
4. **Write the client's rules once** (in that account):
   ```
   Add these Vilix AI rules: brand voice is warm and short. Never email customers on Sundays.
   Shopify API: max 2 calls/sec, always page with limit=250.
   ```
5. **Add the run template** to every agent prompt: [`agent-run-template.md`](../prompts/agent-run-template.md). Use clear sources like `Acme support agent`, `Acme order sync`.
6. **Your own build sessions** (Cursor, Claude Code) for that client can connect to the same account, so the agents and your dev work share one history.

### Selling standalone agents (support, sales, booking)

If you sell a ready-made agent, give each customer their own Vilix AI account and wire the agent to it with that customer's API key. The agent gets long-term memory of every conversation, decision and fix from day one, and you never build or host a memory system yourself.

## Model B: one account, one project per client

Good for solo freelancers who want everything in their own account. Each client is a **project** inside your Vilix AI account, with its own project rules and tasks.

### Structure

```
Vilix AI
├── Project: Acme Coffee      rules: brand voice, Shopify limits, "never email on Sundays"
│     tasks: lead-gen flow, support bot, monthly report
├── Project: Brightline Legal rules: formal tone, no customer data in prompts
│     tasks: intake portal, document parser
├── Project: Nova Fitness     rules: ...
└── Skills (shared across clients)
      lead-gen-n8n, shopify-support-bot, new-client-onboarding, monthly-report
```

### Step by step

1. Connect your tools ([setup guide](../setup/README.md)): ChatGPT for scoping, Claude for proposals, Cursor or Claude Code for builds.
2. For every new client, run the onboarding skill ([example](../config/skill-examples.md)):
   ```
   Run my Vilix AI skill "new-client-onboarding" for Acme Coffee.
   ```
   It creates the project, the client's rules and the tasks.
3. Start every build session by naming the client: *"Working on Acme Coffee: the Shopify support bot."* `get_context` loads that client's rules and history.
4. End every build session with a save: what was built, why, which API settings, what failed.
5. When you solve something reusable, save it as a skill: *"Save this as a Vilix AI skill called shopify-support-bot."* Next client, it's one command.

## Six months later (both models)

The client changes their Shopify API key and the sync breaks. From whatever tool you have open:

```
Why did we build the Acme Coffee Shopify sync this way, and what settings does it use?
```

Vilix AI returns the original build session: the rate-limit decision (2 calls/sec, paging with limit=250), the webhook setup, and the error you hit on day one and how you fixed it. Fix time: minutes.

## Monthly reports in one prompt (both models)

```
Write the October report for Acme Coffee: what we built, what changed, what's next.
```

Every session for that client, from every tool, is already in Vilix AI.
