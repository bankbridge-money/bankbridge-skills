---
name: Subscription auditor
description: Surface every recurring charge across the user's accounts, detect price creeps and duplicates, and help them decide what to cancel. Use when the user asks about subscriptions, recurring charges, "what am I paying for", or wants to cut monthly spend.
---

# Subscription auditor

You have access to BankBridge MCP tools. The user wants a thorough look at everything billing them on a recurring basis, plus actionable follow-ups.

## When to use

Trigger on:
- "find my subscriptions"
- "what am I paying for"
- "what can I cancel"
- "are any subscriptions going up in price"
- "do I have any zombie subscriptions"
- generic "I'm trying to cut spending"

## Process

1. **Pull the corpus.** Call `get_recurring_charges` with `start_date` 12 months ago (not the default 6) and `min_occurrences: 2`. The wider window catches yearly subscriptions like domain renewals, accounting software, insurance bumps.

2. **Enrich each charge with full history.** For any charge flagged as recurring OR whose amount looks non-trivial (> $5/mo), call `get_merchant_history` with `start_date` 12 months ago. Store: first_charge, last_charge, charge_count, average, min, max.

3. **Classify each charge:**
   - **Active recurring** — charged in the last 45 days on its detected cadence. Normal.
   - **Price creep** — last charge > first charge by more than 10%. Calculate: "Netflix: $11.99 → $15.99 (+33% in 8 months)".
   - **Duplicate** — two different merchant_names whose amounts + cadence + timing make them likely the same charge (Plaid occasionally splits one sub across two merchant labels). Flag for user to verify.
   - **Zombie** — detected as recurring, but no charge in the last 90 days. Likely already cancelled; confirm before including in any cancel list.
   - **Forgettable** — < $20/mo, no activity signals from the user that we can derive. Flag gently: "might be worth reviewing."

4. **Compose the output** in this structure:

```
# Subscription audit — <today's date>

## Active subscriptions (MM/mo total: $XXX)
Table: merchant | amount | cadence | last charge | notes

## Price creeps
Merchants whose price went up. Show old → new + % change + time span.

## Possible duplicates
If any — ask the user to confirm whether each pair is actually the same sub.

## Maybe-cancelled (no recent activity)
Subscriptions we thought were active but haven't charged in 90+ days.

## Suggestions (only if the user asked)
If and only if the user explicitly asked "what should I cancel" or similar,
pick 2-3 candidates based on:
  - Smallest utility (forgettable tier)
  - Biggest price creep
  - Possible duplicate with a competitor
Do NOT recommend cancellation of anything the user has not flagged.
```

## Style

- Numbers only, no guesses. If you don't have data, say so.
- Don't be judgmental about categories. Adult services, gambling, food delivery — just report, no editorializing.
- If the user has BankBridge memory entries marked "keep" or "intentional" (e.g., they explicitly mentioned keeping a sub), don't re-surface them for cancellation.

## Guardrails

- Do not suggest cancelling utilities, insurance, rent, or mortgage automatically — these are rarely optional.
- For price creeps, provide the numbers; don't jump to "you should fight this" unless asked.
- Zombie detection: ask the user before including anything that last-charged less than 180 days ago. Quarterly and semi-annual subs look like zombies in a 90-day window.
