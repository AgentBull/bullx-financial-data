# BullX Financial Data Skill

BullX Financial Data is an agent skill for routing supported A-share financial-data and market-simulator tasks to the BullX Financial Data MCP server.

The public repository contains only reusable skill instructions and MCP configuration templates. It must not contain a real Terminal endpoint, API key, filled `Authorization` header, private deployment URL, or user-specific config.

## What This Skill Does

Use this skill when an agent needs BullX-supported data such as:

- A-share symbol resolution, stock names, pinyin, and fuzzy lookup.
- Point-in-time A-share stock and index universes for screening, validation, and backtest inputs.
- Trading calendars, latest tradable dates, and market session checks.
- Daily bars, latest daily snapshots, historical A-share stock minute OHLCV, and supported period aggregation.
- Market breadth, market temperature, advance/decline counts, and intraday trading-activity proxies.
- Realtime A-share stock quote snapshots.
- Factor snapshots, valuation, liquidity, capitalization, and moneyflow.
- Index profiles, daily bars, constituents, weights, and quotes.
- ETF/LOF/exchange-traded fund quotes, single-fund minute/daily bars, batch ETF minute/daily OHLCV, public mutual fund profiles, and fund holdings.
- Macro or industry-economics indicator resolution and time series.
- Financial statements, indicators, fundamentals, shareholders, and risk/compliance records.
- Listed-company announcements, filings, and announcement details.
- BullX parameter dictionaries and industry code decoding.
- Market-simulator account discovery, current holdings, holding-change events, daily performance, private-account creation, and target-weight rebalancing.

The current reference manifest is `2026-07-15.3` with 37 registered public `bullx_*` tools. `tools/list` is API-key-scope-aware: a financial-data-only key sees 31 tools, a key with `market_simulator:read` sees 35, and a key with both simulator read and write scopes sees all 37. If a runtime differs from the count expected for its scopes, trust the runtime surface for that session and report the mismatch.

The skill does not replace generic web search, news research, unsupported markets, real brokerage accounts or order execution, order book/tick data, real-time moneyflow, northbound flow, realtime quote semantics for historical OHLCV tools, ETF/fund active-universe lookup, or announcement full-text extraction. Use the MCP only for data and simulator operations covered by the visible `bullx_*` tools.

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
3. `tools/list` returns the public `bullx_*` tools allowed by the API key: 31 for financial data only, 35 with simulator read, or 37 with simulator read and write.
4. `resources/list` and `prompts/list` may return empty lists; financial data is exposed through tools, not through resources or reusable MCP prompts.
5. A low-risk read-only probe, such as the China A-share trading calendar, succeeds.

If simulator tools are not visible, first check the key's simulator read/write scopes. For other visibility failures, check the local endpoint, API key, protocol headers, and whether the agent process actually received the environment variables.
