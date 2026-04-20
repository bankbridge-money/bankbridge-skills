---
name: Budget builder
description: Draft a realistic monthly budget from the user's actual last-3-months spending, split into fixed vs variable, with room for savings. Use when the user asks for a budget, wants to set spending limits, or needs help knowing what realistic targets look like.
---

# Budget builder

You have access to BankBridge MCP tools. The user wants a budget drafted from their real numbers, not someone else's template.

## When to use

Trigger on:
- "make me a budget"
- "what should I budget for food / groceries / etc"
- "help me figure out realistic limits"
- "I need to save more — where can I cut"

## Process

1. **Pull three months of real data.**
   - `get_monthly_cashflow` for each of the last 3 complete calendar months (skip current in-progress month)
   - `get_spending_summary` grouped by `category` for each month
   - `get_recurring_charges` with start_date 6 months ago, min_occurrences: 3 (catches quarterly and biannual)

2. **Establish a realistic income baseline.**
   - Average monthly income across the 3 months.
   - If income varies by > 25%, call out the variability — a budget based on averages is fragile for freelancers.
   - Flag any one-off income (tax refund, bonus, gift) and don't include in the baseline. Use `get_monthly_cashflow` top_income_sources to distinguish.

3. **Classify spending: fixed vs variable.**
   - **Fixed**: rent/mortgage, utilities, insurance, subscriptions, loan payments, gym. From `get_recurring_charges` + RENT_AND_UTILITIES + LOAN_PAYMENTS categories.
   - **Variable**: everything else — food, dining, transportation, entertainment, retail.

4. **Calculate target ranges.**
   - For each variable category: median across 3 months = **realistic baseline**. 10% below median = **tight**. The 75th percentile = **comfortable**.
   - Don't suggest anything below 60% of the 3-month minimum for that category — the user has real costs.

5. **Compose the output:**

```
# Budget draft — based on <Month1>, <Month2>, <Month3>

## Income baseline
- Monthly income (average): $X,XXX
- Variability: <low / moderate / high — with % spread>
- Excluded one-offs: <list, if any>

## Fixed monthly costs — $XXX
Flat-rate items you can't easily cut without a life change.
| Item | Amount |
|---|---|

## Variable targets
For each of top 6 variable categories:
| Category | Last 3mo avg | Tight | Realistic | Comfortable |
|---|---|---|---|---|

## Savings / headroom
- Income:              $X,XXX
- Fixed:               -$XXX
- Variable (realistic): -$XXX
- Remaining:           $XXX  ← this is what you can save, invest, or keep in checking as buffer

## Tradeoffs
Two or three short observations:
- "Your groceries have gone from $X to $Y — not a budget problem, just a data point."
- "Dining is X% of variable spend. If you want to shift to savings, this is the biggest lever."
- "You're running a $X/mo surplus already — the budget mostly just formalizes it."

## Next step
Offer one (1) thing:
- If they have a savings account connected, suggest a target transfer amount.
- If not, suggest opening one.
- Don't overwhelm with 5 recommendations.
```

## Style

- Treat the user like an adult. Don't shame spending categories.
- Use medians not means where possible — one bad month shouldn't skew targets.
- Give ranges, not a single number. Life varies.

## Guardrails

- Never recommend eliminating a category entirely ("stop eating out") — unrealistic and condescending.
- If the 3-month data shows the user consistently spending more than they earn, flag it clearly and suggest they look at the income side first, not try to cut $50/mo from entertainment.
- Do not recommend specific financial products (Roth IRA, HYSA, ETFs). Stay in budgeting.
