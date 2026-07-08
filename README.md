# BullX Financial Data Skill

BullX Financial Data is an agent skill for routing supported A-share financial-data tasks to the BullX Financial Data MCP server.

The public repository contains only reusable skill instructions and MCP configuration templates. It must not contain a real Terminal endpoint, API key, filled `Authorization` header, private deployment URL, or user-specific config.

## What This Skill Does

Use this skill when an agent needs BullX-supported data such as:

- A-share symbol resolution, stock names, pinyin, and fuzzy lookup.
- Trading calendars, latest tradable dates, and market session checks.
- Daily bars, latest daily snapshots, OHLCV, and supported period aggregation.
- Market breadth, market temperature, advance/decline counts, and intraday trading-activity proxies.
- Realtime A-share stock quote snapshots.
- Factor snapshots, valuation, liquidity, capitalization, and moneyflow.
- Index profiles, daily bars, constituents, weights, and quotes.
- ETF/LOF/exchange-traded fund quotes, minute bars, daily bars, public mutual fund profiles, and fund holdings.
- Macro or industry-economics indicator resolution and time series.
- Financial statements, indicators, fundamentals, shareholders, and risk/compliance records.
- Listed-company announcements, filings, and announcement details.
- BullX parameter dictionaries and industry code decoding.

The skill does not replace generic web search, news research, unsupported markets, A-share stock minute bars, order book/tick data, real-time moneyflow, northbound flow, or announcement full-text extraction. Use the MCP only for data covered by the visible `bullx_*` tools.

## Public Files

- `SKILL.md`: Core routing rules and tool-selection guidance for agents.
- `llm.txt`: Agent-facing installation and verification instructions. Terminal-generated prompts should point agents here.
- `agents/openai.yaml`: OpenAI/Codex metadata. It intentionally declares the MCP dependency without a concrete URL.
- `agents/codex.mcp.template.toml`: Codex MCP config template.
- `agents/claude-code.mcp.template.json`: Claude Code HTTP MCP config template.
- `agents/kimi.mcp.template.json`: Kimi CLI MCP config template.
- `agents/openclaw.mcp.template.json5`: OpenClaw MCP config template.

## Installation Model

Installation is deliberately split into two layers:

1. Install or enable this public skill so the agent understands when and how to use BullX tools.
2. Use Terminal's Financial Data MCP API Key page to generate private instance values and fill the local MCP config.

Terminal should provide:

```text
process.env.BULLX_FINANCIAL_DATA_MCP_ENDPOINT="<terminal-origin>/api/v1/financial-data/mcp"
process.env.BULLX_FINANCIAL_DATA_MCP_API_KEY="<user-api-key>"
```

Use these values only in the local agent runtime, local secret store, or local MCP config. Do not commit filled templates.

For Codex CLI, prefer registering the endpoint with `codex mcp add --url <endpoint> --bearer-token-env-var BULLX_FINANCIAL_DATA_MCP_API_KEY`.
The URL should be the concrete Terminal URL for that environment. The API key should stay in an environment variable or local secret store, and the shell or launcher that starts Codex must actually export that variable.
Do not manually add `MCP-Protocol-Version` in Codex `http_headers`; Codex sends MCP transport headers during negotiation, and a manual value can be merged into a duplicate header that breaks `tools/list`.
If an existing Codex config has `http_headers = { "MCP-Protocol-Version" = ... }` for this server, remove that header and restart Codex discovery.

## Verification

After installation, restart or refresh the target agent's MCP discovery and verify:

1. The MCP server is registered as `bullx-financial-data`.
2. The agent can call `initialize` or `tools/list`.
3. `tools/list` returns public `bullx_*` tools.
4. `resources/list` and `prompts/list` may return empty lists; financial data is exposed through tools, not through resources or reusable MCP prompts.
5. A low-risk read-only probe, such as the China A-share trading calendar, succeeds.

If tools are not visible, check the local endpoint, API key, protocol headers, and whether the agent process actually received the environment variables.
