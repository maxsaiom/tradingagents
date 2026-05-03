# Using TradingAgents via Claude Code (no API key)

This document describes an alternative way to interact with the TradingAgents
workflow without configuring an LLM provider API key. It is **not a feature of
the framework itself** — the framework still requires a provider key when run
through `tradingagents` CLI or `TradingAgentsGraph().propagate()`.

Instead, this is a usage pattern for Claude Code users who want the
TradingAgents *flow* (analysts → researchers → trader → risk team → portfolio
manager) using Claude Code's session as the reasoning engine, with this repo
as the data layer.

## When to use this mode

| Use case | Recommendation |
| --- | --- |
| Production / batch runs / persistent memory log | Run `tradingagents` CLI with a provider key (OpenAI, Anthropic, DeepSeek, ...) |
| Quick exploration in a chat session, no extra billing | Use the Claude Code mode described here |
| Customising prompts / agent behaviour interactively | Use the Claude Code mode described here |

## What you get / what you don't

What works:

- Real market data via `yfinance` (already in dependencies)
- The same agent roles and report structure as the framework
- Output in any language, including Thai (ask Claude to reply in Thai)
- Free-form follow-up questions on the analysis

What you lose vs. running the framework directly:

- No persistent decision log at `~/.tradingagents/memory/trading_memory.md`
- No LangGraph checkpoint resume
- No structured-output Pydantic schemas / 5-tier rating consistency
- No reflection loop with realised return vs SPY across runs

If you need any of those, switch to the API-key path.

## How to invoke from the chat

After cloning the repo and running `pip install .`, simply ask in the Claude
Code chat. Examples:

```
วิเคราะห์ NVDA วันที่ล่าสุด รายงานเป็นภาษาไทย
Analyse TSLA for 2025-12-01 with all four analysts and a Bull/Bear debate
Compare AAPL vs MSFT for this week, focus on fundamentals
Re-run the NVDA analysis but argue the bear case more aggressively
```

Claude Code will:

1. Pull market data, news, and indicators via `yfinance` (and optionally
   Alpha Vantage if `ALPHA_VANTAGE_API_KEY` is set in the environment).
2. Walk through the TradingAgents roles in order, producing a section per
   role: Market Analyst, Social/Sentiment Analyst, News Analyst,
   Fundamentals Analyst, Bull Researcher, Bear Researcher, Research
   Manager, Trader, Risk team (Aggressive / Neutral / Conservative),
   Portfolio Manager.
3. Emit a final decision: BUY / HOLD / SELL with rationale.

## Suggested prompt patterns

- **Single ticker, single date**: `วิเคราะห์ <TICKER> วันที่ <YYYY-MM-DD>`
- **Latest data**: `วิเคราะห์ <TICKER> วันล่าสุด`
- **Comparison**: `เปรียบเทียบ <TICKER_A> กับ <TICKER_B>`
- **Specific role only**: `ทำ News Analyst report ของ <TICKER>` /
  `ให้ Bull กับ Bear debate <TICKER>`
- **Re-run with different stance**: `วิเคราะห์อีกครั้งโดยให้น้ำหนัก downside มากขึ้น`

## Switching to the framework path later

When you are ready to use the real framework (with persistent memory and
checkpointing), set a provider key and run the CLI:

```bash
cp .env.example .env
# Edit .env, add e.g. OPENAI_API_KEY=...
tradingagents analyze --checkpoint
```

See the main [README.md](../README.md) for the full provider list and config
options.
