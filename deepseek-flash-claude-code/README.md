# Can you run Claude Code on DeepSeek's Flash model?

A practical guide to DeepSeek's Anthropic-compatible API: what the `deepseek-flash` model name now points to, the exact environment variables DeepSeek publishes for running Claude Code on it, a safe first prompt, and the gotchas to check before you trust it with real work. Companion to the Agentic Pod episode "Can You Run Claude Code on DeepSeek's Flash Model?".


Everything below comes from DeepSeek's own API docs (linked at the end). The docs page did not state prices, so this guide does not quote any. Check DeepSeek's pricing page the day you start.

## What changed

- `deepseek-flash` is now served by **DeepSeek-V4.1-Flash-1017**. The calling method stays the same.
- The older names `deepseek-v4-flash` and `deepseek-v4-flash-vision-exp` still work, but the models behind them are retired. Requests are routed to the new Flash model and billed at the Flash price.
- `deepseek-v4-pro` is the larger model.
- Two compatible endpoints: OpenAI format at `https://api.deepseek.com` and Anthropic format at `https://api.deepseek.com/anthropic`. An API key is required.
- DeepSeek lists these agents as supported: Claude Code, Codex, OpenCode, OpenClaw, Hermes, Reasonix, WorkBuddy/CodeBuddy and Qoder. **DeepSeek Harness** is in developer preview.
- A new Search/Fetch API powered by DeepSeek's own search engine is available.

## Setup: Claude Code on DeepSeek, step by step

1. Install Node.js 18 or later, then install Claude Code and check it:

```bash
npm install -g @anthropic-ai/claude-code
claude --version
```

2. Create an API key on the DeepSeek Platform, then set these variables (Linux and macOS). This is the block DeepSeek publishes, copied as shown, including the `[1m]` suffixes:

```bash
export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
export ANTHROPIC_AUTH_TOKEN=<your DeepSeek API Key>
export ANTHROPIC_MODEL=deepseek-flash[1m]
export ANTHROPIC_DEFAULT_OPUS_MODEL=deepseek-flash[1m]
export ANTHROPIC_DEFAULT_SONNET_MODEL=deepseek-flash[1m]
export ANTHROPIC_DEFAULT_HAIKU_MODEL=deepseek-flash
export CLAUDE_CODE_SUBAGENT_MODEL=deepseek-flash
export CLAUDE_CODE_EFFORT_LEVEL=max
export CLAUDE_CODE_AUTO_COMPACT_WINDOW=786432
```

3. Start Claude Code in your project:

```bash
cd /path/to/my-project
claude
```

PowerShell users use the same names with `$env:` syntax, for example `$env:ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic"`.

To undo it, unset the variables (or open a new shell) and Claude Code goes back to its normal setup.

## A safe first prompt

Read-only, so nothing in your repo changes:

> Read this repo and list the three riskiest functions, with one test idea each. Don't change any files.

## Gotchas and limits

- **Model name mapping.** On the Anthropic endpoint, names starting with `claude-opus` map to `deepseek-v4-pro` and are billed at the Pro price. Names starting with `claude-haiku` or `claude-sonnet`, and other `claude*` names, map to `deepseek-flash`. A model name that does not start with `claude` returns a 400 error.
- **Ignored fields.** `container`, `mcp_servers`, `service_tier`, `top_k`, `anthropic-version`, and `cache_control`, `citations` and `is_error` inside message content are ignored. Since `mcp_servers` is ignored in the API request, test how your own MCP setup behaves before relying on it.
- **Unsupported content types.** `document`, `search_result`, `redacted_thinking`, `code_execution_tool_result`, `mcp_tool_use`, `mcp_tool_result` and `container_upload`.
- **Supported.** `tools`, `tool_choice`, `thinking` (with `budget_tokens` ignored), `output_config` with `effort`, `system`, `stream`, `temperature` from 0.0 to 2.0, and image sources.
- **Web search costs extra.** When the model decides to search, Claude Code calls the search tool through DeepSeek's API. That adds model token cost for summarising results plus Search API billing.

## Run your own test

DeepSeek says V4.1 Flash beats its previous Pro model in its own testing. Treat that as their claim and test it yourself:

1. Pick one real task you already finished, such as a bug you fixed last week.
2. Run it on your current setup and on DeepSeek Flash.
3. Compare the diffs and the bills.

## Who it is for

- Good fit: routine, token-heavy work such as test generation, refactors and repo questions, and anyone who wants a second provider as a fallback.
- Test first if: your project depends on MCP servers passed through the API, or on document inputs.

## Sources

- DeepSeek, release notes with the `deepseek-flash` change: https://api-docs.deepseek.com/news/news1017
- DeepSeek, Anthropic API guide (endpoint, model mapping, supported and ignored fields): https://api-docs.deepseek.com/guides/anthropic_api
- DeepSeek, Integrate with Claude Code: https://api-docs.deepseek.com/quick_start/agent_integrations/claude_code
