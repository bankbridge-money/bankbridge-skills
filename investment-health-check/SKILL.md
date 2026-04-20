---
name: Investment health check
description: Portfolio snapshot across connected brokerage accounts — total value, gain/loss by position, concentration warnings, dividend run-rate. Use when the user asks about their investments, portfolio, holdings, gains, or dividends.
---

# Investment health check

You have access to BankBridge MCP tools. The user wants a clear view of their investment positions and how they're performing.

## When to use

Trigger on:
- "how's my portfolio"
- "what are my holdings"
- "am I up or down this year"
- "what are my biggest positions"
- "how much have I received in dividends"
- "am I too concentrated in any stock"

## Process

1. **Confirm there's an investment account connected.** Call `list_accounts`. If none has `type: "investment"`, tell the user:

   > "I don't see a brokerage connected. If you have one with Fidelity / Schwab / Vanguard / Robinhood / etc., add it via `connect_bank` or the dashboard and I'll run this again."

   Stop here.

2. **Pull holdings + recent activity:**
   - `list_holdings` → current positions with gain/loss
   - `list_investment_transactions` with start_date 365 days ago → buys/sells/dividends for the year
   - Optionally filter `subtype: "dividend"` for a clean dividends cut

3. **Compute key numbers:**
   - **Total value** = sum of `value` across holdings
   - **Total gain/loss (dollar)** = sum of `gain_loss` for positions with `cost_basis != null`. Flag any positions without cost basis — common for long-held positions the brokerage doesn't report.
   - **Total gain/loss (%)** = gain/loss $ / total cost basis $, when denominator is non-zero
   - **Concentration** = each position's `value / total_value`. Flag anything > 20% as concentration risk.
   - **YTD dividends** = sum of absolute `amount` for all transactions where `subtype == "dividend"` and `date` is in current calendar year
   - **Run rate** = (YTD dividends / days elapsed YTD) × 365

4. **Compose the report:**

```
# Investment health check — <today>

## Portfolio at a glance
- Total value: $XXX,XXX
- Total gain/loss: $XX,XXX (+X.XX%)  ← color or "+" for gain, "-" for loss
- Cost basis: $XX,XXX (coverage: X of N positions)

## Positions
Table: ticker | shares | value | gain/loss $ | gain/loss % | % of portfolio
Sort by value descending.

## Concentration
If any position > 20% of portfolio:
  "<ticker> is XX% of your portfolio. Concentration risk — consider whether that's intentional."
Otherwise:
  "Biggest position: <ticker> at XX%. No single stock over 20%."

## Dividends
- YTD received: $X,XXX
- Run-rate projection: $X,XXX / year
- Top 3 payers: <ticker $X, ticker $X, ticker $X>

## Activity last 90 days
Any buys / sells / fees — brief summary.
```

## Style

- Lead with the number (value), then the trend (gain/loss), then the concerns (concentration).
- Don't offer financial advice ("you should rebalance") unless asked.
- If the user asks about a specific ticker, use `get_merchant_history` with the ticker name — though note that investment transactions and merchant history are separate surfaces. Stick to list_investment_transactions filtered by security_id when possible.

## Guardrails

- Do NOT make tax claims ("this is a long-term capital gain") — you don't know acquisition dates at lot level.
- Do NOT predict returns.
- Flag positions with cost_basis = null clearly. Don't include them in percentage-based totals.
