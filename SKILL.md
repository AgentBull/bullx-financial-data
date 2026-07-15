---
name: bullx-financial-data
description: Route BullX financial-data and market-simulator requests to the configured MCP and choose the correct bullx_* tool. Use for A-share lookup and universes, trading calendars, daily or minute OHLCV, market breadth, realtime quotes, factors, fundamentals, financial statements, shareholders, risk/compliance, index, ETF/fund, macro, announcements, dictionaries, simulator discovery, current holdings, holding-change history, performance, private simulator creation, or target-weight rebalancing. Trigger on BullX financial data, bullx-financial-data, financial-data-mcp, bullx_* tools, A股行情, 股票池, active universe, 交易日历, 日线, 分钟线, OHLCV, 市场宽度, 因子, 指数, 基金, ETF, 宏观, 财务三表, 公告, 股东, 风险合规, 模拟盘, 模拟盘持仓, 持仓变化, 收益曲线, 建仓, 调仓, or similar requests that should use BullX MCP instead of scraping.
---

# BullX Financial Data

## Overview

Use this skill to recognize financial data requests that belong on BullX Financial Data MCP and to select the right `bullx_*` tool quickly. Treat the MCP as the preferred source for supported BullX financial data instead of web search, scraping, or hand-written local queries.

The usual server name is `bullx-financial-data`. In some Codex runtimes it may expose tools under a namespace such as `mcp__bullx`, but the tool names remain `bullx_*`.

Current reference implementation: manifest version `2026-07-15.3`, 37 registered public tools. `tools/list` is scope-filtered: financial-data-only keys see 31 tools, keys with `market_simulator:read` see 35, and keys with simulator read plus write see all 37. If the configured runtime differs from the count expected for its scopes, trust the visible runtime tools for that session and report the mismatch instead of inventing unavailable calls.

## Routing Rules

Use BullX MCP when the task asks for:

- Stock identity resolution: ticker, stock name, pinyin, fuzzy A-share lookup.
- Active universe: point-in-time A-share stock or equity-index universes for screening, validation, backtest inputs, index-constituent research, and data-quality checks.
- Market calendar or freshness: trading day, holiday, latest tradable date, market session status.
- Prices and volumes: historical daily bars, latest N daily bars, historical A-share stock minute OHLCV, weekly/monthly/quarterly/yearly aggregation, adjusted or unadjusted prices.
- Market state: A-share market breadth, market temperature, advance/decline counts, session-aware intraday snapshots, and trading activity proxies.
- Realtime stock quote: latest price, percent change, and volume for one supported A-share stock.
- Factors: valuation, turnover, liquidity, capitalization, moneyflow, industry-relative strength, factor field semantics.
- Index data: index profile, daily bars, constituents, weights, and latest index quote.
- ETF/fund data: exchange-traded fund quote, single-fund intraday/daily bars, batch ETF minute/daily OHLCV, public mutual fund profile, and fund holdings.
- Macro data: macro or industry-economics indicator resolution and time series.
- Fundamentals: financial statements, financial indicators, per-share metrics, performance forecasts.
- Company structure and risk: basic info, shareholders, risk/compliance records.
- Documents: listed-company announcements, filings, search results, announcement details.
- Dictionaries: BullX parameter code decoding and industry code/name decoding.
- Market simulator reads: discover accessible simulator accounts, inspect current holdings and cash, replay holding-change events, and read daily performance.
- Market simulator writes: create a private simulator from initial cash and target weights, or rebalance the API-key user's private simulator to target weights.
- Verification: checking whether financial-data-mcp returns a result, warning, partial range, or stale data.

Do not use BullX MCP as the primary tool for:

- Generic company news that is not an exchange announcement or BullX-supported financial document.
- Non-financial product, UI, code, deployment, or architecture tasks unless they explicitly need BullX financial data.
- ETF/fund symbols through stock market-data tools. Use `bullx_fund_data_*` tools for supported ETF/LOF/exchange-traded fund and public mutual fund requests.
- US/HK price history unless a visible BullX MCP tool explicitly supports that request.
- Order book, tick data, real-time main-fund inflow, northbound flow, ETF/fund active universe, or announcement full-text extraction unless a visible BullX MCP tool explicitly supports that request.
- Same-day historical OHLCV as a realtime source. Quant-backed OHLCV data is overnight batch-updated and may return `NO_RESULTS` or temporary upstream errors for the server-local current date.
- Real brokerage positions or order execution. BullX market simulators are Terminal simulation accounts, not broker accounts.
- Reading another user's private simulator, or mutating public/ITBT simulators. Simulator access is derived from the API key and enforced by the server.

## Tool Selection

Prefer these tool intents:

- `bullx_reference_data_resolve_cn_stock`: Resolve A-share symbol, short name, full name, pinyin, or fuzzy query. Use before other stock tools when the symbol is uncertain.
- `bullx_reference_data_decode_param`: Decode BullX parameter-system codes and parameter values.
- `bullx_reference_data_decode_industry`: Decode SW, CSRC, GICS, BullX industry codes, short codes, exact Chinese industry names, hierarchy paths, or child industries.
- `bullx_reference_data_get_active_universe`: Get point-in-time A-share stock or equity-index universe records. Use for screening universe construction, batch symbol validation, backtest denominators, index-universe research, and data-coverage checks. v1 supports `cn_ashare` and `cn_index`; do not use it for fund/ETF universes.
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
- `bullx_market_data_get_stock_minute_ohlcv`: Get historical minute OHLCV bars for one or more Chinese A-share stocks. Prefer completed historical dates before the server-local current date; use realtime quote for latest price or realtime percent change.
- `bullx_index_data_get_index_basicinfo`: Get public index profile fields and latest constituent summary.
- `bullx_index_data_get_index_daily_bars`: Get index point daily bars or supported period aggregations.
- `bullx_index_data_get_index_constituents`: Get index constituent weight snapshots.
- `bullx_index_data_get_index_quote`: Get the latest index point quote.
- `bullx_fund_data_get_fund_quote`: Get realtime quote fields for exchange-traded funds, ETFs, and LOFs.
- `bullx_fund_data_get_fund_intraday_bars`: Get minute bars for one exchange-traded fund, ETF, or LOF. The public shape is stable; the implementation may use Quant first and Wind as fallback, so inspect `FALLBACK_SOURCE_USED`.
- `bullx_fund_data_get_fund_daily_bars`: Get daily OHLCV bars for one exchange-traded fund, ETF, or LOF. The public shape is stable; the implementation may use Quant first and Wind as fallback, so inspect `FALLBACK_SOURCE_USED`.
- `bullx_fund_data_get_etf_minute_ohlcv`: Get historical minute OHLCV bars for one or more Chinese ETFs. Prefer completed historical dates before the server-local current date.
- `bullx_fund_data_get_etf_daily_ohlcv`: Get historical daily OHLCV bars for one or more Chinese ETFs. Prices are forward-adjusted; volume and money are unadjusted. Prefer completed historical dates before the server-local current date.
- `bullx_fund_data_get_fund_basicinfo`: Get public mutual fund profile, manager, size, benchmark, and NAV sections.
- `bullx_fund_data_get_fund_holdings`: Get public mutual fund top holdings and asset allocation.
- `bullx_macro_data_resolve_indicators`: Resolve a macro or industry-economics query into candidate indicators.
- `bullx_macro_data_get_economic_series`: Get macro or industry-economics time series by explicit indicators or an unambiguous query.
- `bullx_market_simulator_list_accounts`: List simulator accounts readable by the API-key user: owned accounts, accounts allowed by the existing public rule, and published ITBT accounts. This discovery call uses stored projection watermarks; use a detail tool when current facts are required.
- `bullx_market_simulator_get_current_holdings`: Synchronously refresh one readable simulator and return current holdings, available quantities, cash, equity, weights, `data_as_of`, and `calculated_at`.
- `bullx_market_simulator_get_holding_changes`: Replay buy, sell, T+1 settlement, cash-dividend, bonus-share, and transfer-share events with before/after quantity, cost basis, available quantity, and cash.
- `bullx_market_simulator_get_performance`: Return the daily equity curve, daily PnL, daily return, cumulative return, and maximum drawdown for one readable simulator.
- `bullx_market_simulator_create_account`: Create a private user-owned simulator and initial portfolio in one batch from `name`, `initial_cash`, zoned `buy_time`, and `initial_positions[{symbol,target_weight,buy_price}]`. The server validates the inputs, derives whole lots, preserves residual cash, and recalculates once.
- `bullx_market_simulator_rebalance_account`: Rebalance the API-key user's own private simulator from `account_id`, zoned `rebalance_time`, and `target_positions[{symbol,target_weight,price}]` covering the complete target portfolio. Use weight `0` to exit. The server derives quantities, sells first, buys second, and recalculates once.
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
9. For active-universe requests, use `bullx_reference_data_get_active_universe` only for A-share stocks or equity indices, and preserve `as_of_date`, filters, pagination, and listing/trading status in analysis.
10. For Quant-backed stock/ETF OHLCV, prefer completed historical dates before the server-local current date. Do not treat same-day `NO_RESULTS` or temporary upstream unavailability as proof that older historical data is unavailable.
11. Inspect `warnings`, `sources`, `meta`, row counts, and quality fields before drawing conclusions.
12. Report the data date, cutoff/session, symbol, adjustment basis, and any missing or partial fields in user-facing answers.

## Market Simulator Rules

1. Never send or invent `userId` or `user_id`. The Bearer API key is the only user identity; simulator tools accept `account_id` when an account must be selected.
2. Treat simulator read access as the union of the key owner's accounts, accounts allowed by Terminal's public-visibility rule, and published ITBT accounts. An inaccessible account returns `NOT_FOUND`; do not infer whether it exists.
3. Require `market_simulator:read` for the four read tools. Require both simulator read and the separately enabled `market_simulator:write` scope for create or rebalance.
4. For fresh account facts, call `list_accounts` only to discover an `account_id`, then call holdings, holding changes, or performance and inspect `data_as_of`, `calculated_at`, pagination, warnings, and access reasons.
5. For morning event-impact analysis, combine current simulator holdings with the existing announcement, risk/compliance, fundamentals, and market-data tools. The MCP supplies facts; the external agent owns scheduling and impact reasoning.
6. Call `create_account` only for an explicit creation request. Supply `name`, numeric `initial_cash`, one zoned `buy_time` such as `2026-06-10T14:30:00+08:00`, and `initial_positions[{symbol,target_weight,buy_price}]` for canonical A-share symbols. Keep weights and prices as JSON numbers. Do not supply lots or quantities; the server validates the time and prices with simulator trading rules, rounds down to whole 100-share lots after fees, and keeps unused cash.
7. Call `rebalance_account` only for an explicit rebalance request on the key user's private simulator. Supply `account_id`, a zoned `rebalance_time`, and `target_positions[{symbol,target_weight,price}]` covering the union of current and target symbols; keep weights and prices as JSON numbers and use `target_weight=0` for exits. Do not calculate order quantities yourself; the server validates the time and prices before applying the sell-first batch.
8. Treat other users' accounts, public accounts, and ITBT accounts as read-only even when they are visible. Creation always produces a private account owned by the API-key user.
9. Treat simulator writes as non-idempotent, like the Terminal web flow. Do not send `expected_plan_hash` or `idempotency_key`; these fields are unsupported. After a timeout or ambiguous error, read back accounts, holding changes, and current holdings before deciding whether to retry because an error does not guarantee that no durable write occurred.
10. On successful writes, inspect `meta.mutation_source`, `account_id`, and `trade_ids`. On write errors, also inspect `meta.mutation_committed` and any durable account/trade IDs before retrying. On all calls, preserve server warnings, sources, and request IDs in the result explanation.

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
3. Run `tools/list` or use tool discovery to confirm the scope-appropriate `bullx_*` tools are visible: 31 for financial data only, 35 with simulator read, or 37 with simulator read and write.
4. Treat empty `resources/list` and `prompts/list` results as normal. The business surface is `tools/list` and `tools/call`.
5. Make a low-risk read-only probe, such as `bullx_market_data_get_trading_calendar` for `cn_ashare`. When simulator read is enabled, also confirm the four `bullx_market_simulator_*` read tools are listed before attempting account access.
6. If tools are still invisible after configuration is correct, restart the Codex session so MCP discovery reloads.
