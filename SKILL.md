---
name: bullx-financial-data
description: Route financial-data tasks to the BullX Financial Data MCP server and call the correct bullx_* tools. Use when a task involves A-share stock lookup, Chinese market trading calendars, daily bars or OHLCV, latest daily snapshots, market breadth, intraday activity proxies, realtime quotes, factor snapshots, valuation, liquidity, moneyflow, fundamentals, financial statements, financial indicators, shareholders, risk/compliance, index data, ETF/fund data, macroeconomic series, company announcements, filings, BullX parameter dictionaries, industry code decoding, or validating data from financial-data-mcp. Trigger on BullX financial data, bullx-financial-data, financial-data-mcp, bullx_* tools, A股行情, 交易日历, 日线, 市场宽度, 盘中活跃度, 因子, 指数, 基金, ETF, 宏观, 财务三表, 公告, 股东, 风险合规, or similar market-data queries that should use the configured MCP instead of ad hoc scraping.
---

# BullX Financial Data

## Overview

Use this skill to recognize financial data requests that belong on BullX Financial Data MCP and to select the right `bullx_*` tool quickly. Treat the MCP as the preferred source for supported BullX financial data instead of web search, scraping, or hand-written local queries.

The usual server name is `bullx-financial-data`. In some Codex runtimes it may expose tools under a namespace such as `mcp__bullx`, but the tool names remain `bullx_*`.

## Routing Rules

Use BullX MCP when the task asks for:

- Stock identity resolution: ticker, stock name, pinyin, fuzzy A-share lookup.
- Market calendar or freshness: trading day, holiday, latest tradable date, market session status.
- Prices and volumes: historical daily bars, latest N daily bars, weekly/monthly/quarterly/yearly aggregation, OHLCV, adjusted or unadjusted prices.
- Market state: A-share market breadth, market temperature, advance/decline counts, session-aware intraday snapshots, and trading activity proxies.
- Realtime stock quote: latest price, percent change, and volume for one supported A-share stock.
- Factors: valuation, turnover, liquidity, capitalization, moneyflow, industry-relative strength, factor field semantics.
- Index data: index profile, daily bars, constituents, weights, and latest index quote.
- ETF/fund data: exchange-traded fund quote, intraday bars, daily bars, public mutual fund profile, and fund holdings.
- Macro data: macro or industry-economics indicator resolution and time series.
- Fundamentals: financial statements, financial indicators, per-share metrics, performance forecasts.
- Company structure and risk: basic info, shareholders, risk/compliance records.
- Documents: listed-company announcements, filings, search results, announcement details.
- Dictionaries: BullX parameter code decoding and industry code/name decoding.
- Verification: checking whether financial-data-mcp returns a result, warning, partial range, or stale data.

Do not use BullX MCP as the primary tool for:

- Generic company news that is not an exchange announcement or BullX-supported financial document.
- Non-financial product, UI, code, deployment, or architecture tasks unless they explicitly need BullX financial data.
- ETF/fund symbols through stock market-data tools. Use `bullx_fund_data_*` tools for supported ETF/LOF/exchange-traded fund and public mutual fund requests.
- US/HK price history unless a visible BullX MCP tool explicitly supports that request.
- A-share stock minute bars, order book, tick data, real-time main-fund inflow, northbound flow, or announcement full-text extraction unless a visible BullX MCP tool explicitly supports that request.

## Tool Selection

Prefer these tool intents:

- `bullx_reference_data_resolve_cn_stock`: Resolve A-share symbol, short name, full name, pinyin, or fuzzy query. Use before other stock tools when the symbol is uncertain.
- `bullx_reference_data_decode_param`: Decode BullX parameter-system codes and parameter values.
- `bullx_reference_data_decode_industry`: Decode SW, CSRC, GICS, BullX industry codes, short codes, exact Chinese industry names, hierarchy paths, or child industries.
- `bullx_stock_data_get_stock_basicinfo`: Get listed company profile and stock basic information.
- `bullx_stock_data_get_stock_fundamentals`: Get financial statements, indicators, profitability, per-share metrics, and forecasts.
- `bullx_stock_data_get_stock_equity_holders`: Get shareholders and ownership data.
- `bullx_stock_data_get_stock_risk_compliance`: Get risk, compliance, warning, or regulatory data.
- `bullx_market_data_get_trading_calendar`: Check trading day, market session, previous/latest/next trading day, and calendar range.
- `bullx_market_data_get_stock_history_bars`: Get historical daily bars or supported aggregations with explicit date range and price adjustment.
- `bullx_market_data_get_latest_daily_bars`: Get latest N daily bars or current-day daily snapshot when close data is not available.
- `bullx_market_data_get_market_breadth`: Get session-aware A-share market breadth, turnover, return distribution, and optional BullX market temperature. Intraday rows are partial daily snapshots.
- `bullx_market_data_get_intraday_activity_snapshot`: Get trading-activity proxy by full market, current SW industry mapping, or explicit symbols. This is not realtime moneyflow, main-fund inflow, northbound flow, order book, or tick data.
- `bullx_market_data_get_stock_factor_snapshot`: Get official daily factor snapshot and optional factor semantics.
- `bullx_market_data_get_realtime_quote`: Get one supported A-share stock realtime quote snapshot.
- `bullx_index_data_get_index_basicinfo`: Get public index profile fields and latest constituent summary.
- `bullx_index_data_get_index_daily_bars`: Get index point daily bars or supported period aggregations.
- `bullx_index_data_get_index_constituents`: Get index constituent weight snapshots.
- `bullx_index_data_get_index_quote`: Get the latest index point quote.
- `bullx_fund_data_get_fund_quote`: Get realtime quote fields for exchange-traded funds, ETFs, and LOFs.
- `bullx_fund_data_get_fund_intraday_bars`: Get minute bars for exchange-traded funds, ETFs, and LOFs.
- `bullx_fund_data_get_fund_daily_bars`: Get daily OHLCV bars for exchange-traded funds, ETFs, and LOFs.
- `bullx_fund_data_get_fund_basicinfo`: Get public mutual fund profile, manager, size, benchmark, and NAV sections.
- `bullx_fund_data_get_fund_holdings`: Get public mutual fund top holdings and asset allocation.
- `bullx_macro_data_resolve_indicators`: Resolve a macro or industry-economics query into candidate indicators.
- `bullx_macro_data_get_economic_series`: Get macro or industry-economics time series by explicit indicators or an unambiguous query.
- `bullx_financial_docs_search_company_announcements`: Search company announcements and exchange filings.
- `bullx_financial_docs_get_announcement_detail`: Read a specific announcement detail after search.

## Call Pattern

1. Resolve ambiguous stocks first. Do not guess a canonical symbol when the user gives only a Chinese name or fuzzy code.
2. For date-sensitive data, call `bullx_market_data_get_trading_calendar` when trading-day status, latest official date, or stale gap matters.
3. Pass explicit dates when the user gives them. If omitted, state that the MCP default date was used.
4. Preserve price basis. Default daily bars are forward-adjusted unless the user requests unadjusted/raw prices.
5. Treat current-day daily snapshots as daily snapshots, not minute data.
6. For market-state questions, use market breadth or intraday activity tools instead of looping over many single-stock calls. Treat activity snapshots as turnover/return proxies, not real moneyflow.
7. For `bullx_market_data_get_intraday_activity_snapshot(scope="sw_industry")`, use the current numeric SW industry code such as `27`, `2701`, `270101`, or `sw270101`; do not pass SW industry-index symbols such as `801xxx.SI`.
8. For ETF/LOF/exchange-traded fund requests, use `bullx_fund_data_*` tools. For index requests, use `bullx_index_data_*` tools. Do not force these symbols through stock tools.
9. Inspect `warnings`, `sources`, `meta`, row counts, and quality fields before drawing conclusions.
10. Report the data date, cutoff/session, symbol, adjustment basis, and any missing or partial fields in user-facing answers.

## Configuration Contract

Expected MCP registration:

- Server: `bullx-financial-data`
- Transport: Streamable HTTP
- Endpoint: use the URL generated by Terminal's Financial Data MCP API Key page for the current environment. Local development commonly uses `http://localhost:3000/api/v1/financial-data/mcp`, but shared skill files must not hard-code that URL.
- Auth: `Authorization: Bearer ${BULLX_FINANCIAL_DATA_MCP_API_KEY}`
- Generic HTTP MCP clients that require manual headers: `MCP-Protocol-Version: 2025-11-25` and `Accept: application/json, text/event-stream`

Keep API keys and instance-specific URLs out of repositories, reports, scripts, and shared skill files. The MCP config should reference environment variables such as `BULLX_FINANCIAL_DATA_MCP_ENDPOINT` and `BULLX_FINANCIAL_DATA_MCP_API_KEY` when the target agent supports them.

For Codex CLI, the safest setup is to register the concrete endpoint URL with `codex mcp add --url` and keep only the API key behind `bearer_token_env_var`. Do not manually add `MCP-Protocol-Version` to Codex `http_headers`; Codex sends MCP transport headers during negotiation, and a fixed manual value can be merged into a duplicate protocol header that breaks `tools/list` startup. Do not assume every Codex version expands `${...}` inside the URL field. Also confirm the shell or app launcher that starts Codex actually exports the API key variable; having it only in a project `.env.local` file is not enough.

If the target agent does not expand environment variables inside MCP config headers, keep the generated `Authorization` header in that agent's local secret/header store. Never commit a filled config containing a real endpoint for a private deployment or a real API key.

## Verification

When the MCP is newly configured or a call fails:

1. Check `codex mcp get bullx-financial-data` or the equivalent MCP registration in the current agent.
2. Confirm the API key environment variable is present in the process that starts Codex.
3. Run `tools/list` or use tool discovery to confirm `bullx_*` tools are visible.
4. Treat empty `resources/list` and `prompts/list` results as normal. The business surface is `tools/list` and `tools/call`.
5. Make a low-risk read-only probe, such as `bullx_market_data_get_trading_calendar` for `cn_ashare`.
6. If tools are still invisible after configuration is correct, restart the Codex session so MCP discovery reloads.
