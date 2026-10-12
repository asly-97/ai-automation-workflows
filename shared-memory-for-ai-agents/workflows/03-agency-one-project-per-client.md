# Workflow 3: Agency, One Brain Per Client

**For:** automation freelancers and small studios building with n8n, Make, Cursor, Claude Code and Lovable for several clients.

**Pains it removes:** mixing up client contexts, rebuilding the same automations for every client, and the client workflow that breaks six months later with no record of why it was built that way.

## Structure

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

## Step by step

1. Connect your tools ([setup guide](../setup/README.md)): ChatGPT for scoping, Claude for proposals, Cursor or Claude Code for builds.
2. For every new client, run the onboarding skill ([example](../config/skill-examples.md)):
   ```
   Run my Vilix AI skill "new-client-onboarding" for Acme Coffee.
   ```
   It creates the project, the client's rules and the tasks.
3. Start every build session by naming the client: *"Working on Acme Coffee: the Shopify support bot."* `get_context` loads that client's rules and history.
4. End every build session with a save: what was built, why, which API settings, what failed.
5. When you solve something reusable, save it as a skill: *"Save this as a Vilix AI skill called shopify-support-bot."* Next client, it's one command.

## Six months later

The client changes their Shopify API key and the sync breaks. From whatever tool you have open:

```
Why did we build the Acme Coffee Shopify sync this way, and what settings does it use?
```

Vilix AI returns the original build session: the rate-limit decision (2 calls/sec, paging with limit=250), the webhook setup, and the error you hit on day one and how you fixed it. Fix time: minutes.

## Monthly reports in one prompt

```
Write the October report for Acme Coffee: what we built, what changed, what's next.
```

Every session for that client, from every tool, is already in Vilix AI.
