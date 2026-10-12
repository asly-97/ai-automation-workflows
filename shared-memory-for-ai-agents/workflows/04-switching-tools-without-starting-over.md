# Workflow 4: Switching Tools Without Starting Over

**For:** builders who start in Lovable, Bolt, Replit or v0 and move to Cursor or Claude Code when the app grows. Also anyone moving between ChatGPT, Claude and Grok.

**Pains it removes:** re-describing the app from scratch in every new chat, losing what was already tried, and burning credits on fixes that already failed.

## Step by step

1. **Connect Vilix AI where you build today** (Lovable: Connectors → + → MCP server → OAuth) and in ChatGPT or Claude, where you talk through ideas ([setup guide](../setup/README.md)).
2. **Create the app's project** with what matters:
   ```
   Create a Vilix AI project "Client Intake" with project rules:
   Built in Lovable with Supabase auth. Clients log in with magic links.
   Two paying clients use it. Never change the intake form fields without asking.
   ```
3. **Save attempts as you go.** When a prompt fixes something or breaks something, ask: *"Save to Vilix AI what we just tried and what happened."*
4. **Switch tools.** Connect Cursor or Claude Code ([setup](../setup/README.md#claude-code)), open the exported repo and say:
   ```
   Load my Client Intake project from Vilix AI and explain the app to me before we change anything.
   ```
   The new tool already knows the app, the decisions, the auth setup, and the fixes that didn't work.
5. **Keep going in plain language.** Ask ChatGPT on your phone *"what did Claude Code change today?"* and it answers from the same memory.
