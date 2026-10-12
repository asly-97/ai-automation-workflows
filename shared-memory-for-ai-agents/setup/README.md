# Connect Vilix AI to Every AI Tool

Step-by-step setup for ChatGPT, Claude, Claude Code, Codex, Cursor, OpenClaw, Hermes, GitHub Copilot, Windsurf, Grok, Manus, Gemini and more. Connect each tool once and they all share one memory.

**MCP endpoint (all tools):**

```
https://api.vilix.ai/mcp
```

**Two ways to sign in:**

| Method | Used by | How |
|---|---|---|
| OAuth (one-time sign in) | ChatGPT, Claude, Claude Code, Codex, Cursor, GitHub Copilot, Windsurf, Grok, Gemini Spark, Lovable, Devin | The tool opens a browser, you sign in to Vilix AI and approve access |
| Personal API key | OpenClaw, Hermes, Manus, Gemini CLI, Cline, Continue, Muse, any server-side agent | In Vilix AI open **Agents → Create API key**, copy it, and send it as `Authorization: Bearer YOUR_VILIX_API_KEY`. Treat this key like a password |

After connecting, check that the tool can see `get_context` and `save_turn`.

## Contents

- [ChatGPT](#chatgpt)
- [Claude (web, desktop, mobile)](#claude-web-desktop-mobile)
- [Claude Code](#claude-code)
- [Codex (CLI, app, IDE)](#codex-cli-app-ide)
- [Cursor](#cursor)
- [OpenClaw](#openclaw)
- [Hermes](#hermes)
- [GitHub Copilot (VS Code and CLI)](#github-copilot-vs-code-and-cli)
- [Windsurf](#windsurf)
- [Grok and Grok Build](#grok-and-grok-build)
- [Manus](#manus)
- [Gemini Spark and Gemini CLI](#gemini-spark-and-gemini-cli)
- [Devin, Lovable, Cline, Continue, Muse](#devin-lovable-cline-continue-muse)
- [Any other MCP tool](#any-other-mcp-tool)
- [Make it automatic](#make-it-automatic)

---

## ChatGPT

Plans: ChatGPT Plus, Pro, Business, Enterprise or Edu.

1. Go to **Settings → Security and login → Developer mode** and turn it on.
2. Open [chatgpt.com/plugins](https://chatgpt.com/plugins) and select **+** to create an app.
3. Name it **Vilix AI**, paste `https://api.vilix.ai/mcp`, and choose **OAuth**.
4. Select **Create**, sign in to Vilix AI and approve access. Your app appears under **Drafts**.
5. In a chat, select **+ → Developer mode → Vilix AI**.
6. To make it automatic, paste the Vilix AI instructions into **Settings → Personalization → Custom instructions** (see [Make it automatic](#make-it-automatic)).

## Claude (web, desktop, mobile)

Works in the browser, desktop app and the phone app.

1. Open [claude.ai/customize/connectors](https://claude.ai/customize/connectors): **Customize → Connectors → + → Add custom connector**.
2. Name it **Vilix AI**, paste `https://api.vilix.ai/mcp`, then **Add**.
3. Select **Connect**, sign in and approve access.
4. In a chat, enable **Vilix AI** under **+ → Connectors**.
5. Instructions go in **your initials → Settings → Instructions for Claude**, or in a Claude project's instructions.

Team and Enterprise: an admin adds it once under **Organization settings → Connectors → Add → Custom → Web**.

## Claude Code

```bash
claude mcp add --transport http --scope user vilix https://api.vilix.ai/mcp
```

1. Start `claude`, run `/mcp`, select **vilix**, complete the browser sign-in and approve access.
2. Put the Vilix AI instructions in `~/.claude/CLAUDE.md` (all projects) or the project's `CLAUDE.md`.
3. For headless or scheduled runs (`claude -p "..."`), put the agent template from [`../prompts/agent-run-template.md`](../prompts/agent-run-template.md) at the top of the prompt.

## Codex (CLI, app, IDE)

**CLI:**

```bash
codex mcp add vilix --url https://api.vilix.ai/mcp
codex mcp login vilix
```

Type `/mcp` inside Codex to check the server.

**Codex app:** **Settings → MCP servers → Add server**. Name it **Vilix AI**, choose **Streamable HTTP**, paste the URL, **Save → Restart → Authenticate**.

**IDE extension:** **gear menu → MCP servers → Add server**, same fields, then **Restart extension**.

Instructions go in `~/.codex/AGENTS.md` or a project `AGENTS.md`.

## Cursor

1. Open `~/.cursor/mcp.json` (create it if missing) and merge in:
   ```json
   {
     "mcpServers": {
       "vilix": { "url": "https://api.vilix.ai/mcp" }
     }
   }
   ```
2. Restart Cursor. **Customize → MCPs → Vilix AI**: enable it and sign in.
3. Use **Agent mode** in Chat.
4. Instructions go in **Customize → Rules** as a User Rule.

Cursor CLI:

```bash
agent mcp login vilix
agent mcp list
```

## OpenClaw

Uses a personal API key (Vilix AI → **Agents → Create API key**).

```bash
openclaw mcp add vilix \
  --url https://api.vilix.ai/mcp \
  --transport streamable-http \
  --header "Authorization=Bearer YOUR_VILIX_API_KEY"

openclaw mcp doctor vilix --probe
```

Control UI alternative: **Settings → MCP → Configured servers → Add server**, name it `vilix`, choose **Streamable HTTP**, paste the URL, add the header `Authorization: Bearer YOUR_VILIX_API_KEY`, then **Save & Publish** and run the doctor check.

Put the agent rules in the agent's `AGENTS.md`. For a heartbeat agent, use [`../prompts/heartbeat-agent.md`](../prompts/heartbeat-agent.md).

## Hermes

Uses a personal API key.

```bash
hermes mcp add vilix --url https://api.vilix.ai/mcp --auth header
hermes mcp test vilix
```

The key prompt is masked. Or configure it in `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  vilix:
    url: https://api.vilix.ai/mcp
    headers:
      Authorization: Bearer YOUR_VILIX_API_KEY
```

## GitHub Copilot (VS Code and CLI)

**VS Code:** Command Palette → **MCP: Add Server → HTTP**. Paste the URL, name it `vilix`, select **Global**. Then **MCP: List Servers → vilix → Start**, confirm trust, sign in, and enable the Vilix AI tools in **Agent mode**. Instructions go in `.github/copilot-instructions.md`.

**Copilot CLI:**

```bash
copilot mcp add --transport http vilix https://api.vilix.ai/mcp
```

Then `/mcp auth vilix` and `/mcp list`. Instructions go in `AGENTS.md`.

## Windsurf

1. **Devin Settings → Cascade → MCP Servers**, open `~/.codeium/windsurf/mcp_config.json`.
2. Merge in (note the key is `serverUrl`):
   ```json
   {
     "mcpServers": {
       "vilix": { "serverUrl": "https://api.vilix.ai/mcp" }
     }
   }
   ```
3. Refresh the server list, sign in with OAuth, and enable the Vilix AI tools in Cascade.

Enterprise: an admin enables MCP, and any allowlist includes the server ID `vilix`.

## Grok and Grok Build

**Grok:** open [grok.com/connectors](https://grok.com/connectors): **Connectors → New Connector → Custom**, paste the URL, sign in and approve access. Grok Business/Enterprise: an admin first adds it at **console.x.ai → your team → Grok Business → Connectors → + Add Connector → Other**.

**Grok Build:**

```bash
grok mcp add --transport http vilix https://api.vilix.ai/mcp
grok mcp doctor vilix
```

Then `/mcps → vilix → i` to authenticate.

## Manus

**Settings → Integrations → Custom MCP Servers → Add Server**. Name it **Vilix AI**, paste the URL, choose **Bearer/API key** and enter your key (or add `Authorization: Bearer YOUR_VILIX_API_KEY`). Select **Test Connection**.

## Gemini Spark and Gemini CLI

**Gemini Spark** (Google AI Pro or Ultra, English, US, personal Google account):
1. Turn on **Keep Activity** in Gemini.
2. **Settings & help → Connected Apps → Custom apps for Spark**, paste the URL, **Next**.
3. Sign in and approve access. In Spark, type `@` and select **Vilix AI**. Confirm memory saves when asked.

**Gemini CLI** (Gemini Code Assist Standard/Enterprise or paid API access), personal API key:

```bash
gemini mcp add --scope user --transport http \
  --header "Authorization: Bearer YOUR_VILIX_API_KEY" \
  vilix https://api.vilix.ai/mcp
```

Then `/mcp list`. Instructions go in `~/.gemini/GEMINI.md`.

## Devin, Lovable, Cline, Continue, Muse

**Devin:**
```bash
devin mcp add --scope user vilix https://api.vilix.ai/mcp
devin mcp login vilix
devin mcp list
```

**Lovable:** **Connectors → + → MCP server**, paste the URL, choose **OAuth**, then **Add & authorize**.

**Cline:** run the `cline mcp` wizard (or `cline config mcp`), or add JSON with `type: "streamableHttp"`, the URL, an `Authorization: Bearer YOUR_VILIX_API_KEY` header, and `autoApprove: []`.

**Continue:** put `VILIX_API_KEY=...` in `~/.continue/.env`, then create `.continue/mcpServers/vilix.yaml`:

```yaml
name: Vilix AI
version: 1.0.0
schema: v1
mcpServers:
  - name: Vilix AI
    type: streamable-http
    url: https://api.vilix.ai/mcp
    requestOptions:
      headers:
        Authorization: Bearer ${{ secrets.VILIX_API_KEY }}
```

**Muse:** in chat, say *"Connect Vilix AI so you have my context."* and paste your API key into the secure link Muse sends.

## Any other MCP tool

Use OAuth if the tool supports it, or the API key as `Authorization: Bearer YOUR_VILIX_API_KEY` with transport **Streamable HTTP**. Then confirm the tool lists `get_context` and `save_turn`.

## Make it automatic

Ask any connected tool:

```
Set up Vilix AI to load and save context automatically.
```

It calls `get_setup_instructions` and returns the exact instruction block for that tool and where to paste it (custom instructions, CLAUDE.md, AGENTS.md, User Rules...). The pattern it installs:

```
get_context  →  answer  →  save_turn  →  return the answer
```

For agents and scheduled jobs, use [`../prompts/agent-run-template.md`](../prompts/agent-run-template.md).

## Check it works

1. In tool A: `Remember in Vilix AI: Northstar is my invoicing SaaS. We use Next.js and Stripe. The pricing page comes first.`
2. In tool B: `What am I building, and what comes first?`
3. Tool B answers from Vilix AI and can tell you which tool saved it.
