---
name: Monthly money review
description: Orchestrate a complete monthly financial review using BankBridge — cashflow, top categories, merchants, subscriptions, investment delta, narrative takeaways. Use when the user asks for a monthly check-in, a "how am I doing", or a broad look at their money for a specific month.
---

# Monthly money review

You have access to BankBridge MCP tools. The user wants a complete picture of their finances for a specific month.

## When to use

Use this skill when the user says something like:
- "how am I doing financially this month"
- "give me a monthly review / check-in / rundown"
- "summarize my money for March"
- "where did my money go last month"

## Process

1. **Pick the month.** If the user named one ("March", "2026-03", "last month"), use that. Otherwise default to the most recent complete calendar month. Never use the current in-progress month unless they ask specifically — partial data skews the narrative.

2. **Gather the data.** Call in this order:
   - `get_monthly_cashflow` for the target month → income, expenses, net, top sources, top categories
   - `get_spending_summary` grouped by `category` for the same date range → full category breakdown
   - `get_spending_summary` grouped by `merchant`, limit 10 → top merchants
   - `get_recurring_charges` with date range covering the target month → list of recurring charges active that month
   - `list_holdings` + `list_investment_transactions` filtered to the target month if the user has a brokerage account connected (skip otherwise)

3. **Compose the report** in this exact structure. One page of well-formatted markdown:

```
# Monthly money review — <Month Year>

## Cashflow
- Income: $X,XXX
- Expenses: $X,XXX
- Net: $X,XXX (label: "saved" if positive, "over-spent" if negative)
- Savings rate: X% of income (only if income > 0)

## Top 5 spending categories
| Category | Spend | % of total |
|---|---|---|

## Top 10 merchants
| Merchant | Spend | Visits |
|---|---|---|

## Recurring charges
List every recurring charge with amount, frequency, total monthly impact. Call out the monthly subtotal.

## Investment delta (if any brokerage connected)
- Total portfolio value: $X,XXX
- Gain/loss this month: $X,XXX
- Biggest mover: TICKER +/-X%
- Dividends received: $XXX

## Takeaways
Two to four short, concrete observations. Prioritize things the user might not already know: surprise expenses, category shifts vs. the month before (use get_monthly_cashflow on the prior month if useful for comparison), subscriptions that crept up, a merchant they visited unusually often, anything genuinely actionable.
```

## Style

- Write in the user's voice — second person ("you spent..."), not third ("the user spent...").
- Real numbers from the tools only. Never estimate or fabricate.
- Keep takeaways honest: if things are fine, say so briefly. No fake urgency.
- If BankBridge returns a warning (e.g. REAUTH_REQUIRED, NO_CONNECTIONS), relay it to the user before proceeding or in place of the report.

## Guardrails

- Do not suggest specific subscription cancellations unless the user asks. This skill is a report, not a recommendation engine.
- Do not pattern-match emotionally charged categories (medical, debt collections, gambling). Just report numbers.
- Do not compare against "averages" — you don't have population data. Compare against the user's own prior months only.
