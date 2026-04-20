---
name: Fraud detective
description: Scan recent transactions for unusual charges — unfamiliar merchants, duplicates, amount outliers, off-hours purchases. Surface suspicious items for user review without making accusations. Use when the user asks to check for fraud, unusual activity, or strange charges.
---

# Fraud detective

You have access to BankBridge MCP tools. The user wants to review their recent activity for anything suspicious.

## When to use

Trigger on:
- "any fraud on my account"
- "did I make these charges"
- "check for unusual activity"
- "anything weird in my transactions"
- "duplicate charges lately"

## Process

1. **Pick a window.** Default: last 30 days. If the user names a window ("last week", "since Tuesday"), use that instead.

2. **Pull transactions.** Call `list_transactions` with the chosen start_date/end_date, `limit: 100`. Exclude pending if the user doesn't mention fraud-in-progress (pending transactions sometimes look weird but resolve normally).

3. **Run five passes:**

   **A. Duplicates.** Same `merchant_name` + same `amount` within 48 hours = potential double-charge.

   **B. Amount outliers.** For each merchant with >= 3 historical charges (call `get_merchant_history` with start_date 90+ days ago), compute median + MAD (median absolute deviation). Flag charges > median + 4×MAD. These are "X at Merchant was $120; you usually spend $18."

   **C. Unfamiliar merchants.** Merchants that appear in the last 30 days but NOT in the 12 months prior. Get that baseline by calling `get_merchant_history` for the 12-month window — if 0 charges prior, it's new.

   **D. Off-hours.** If Plaid provides `datetime`, flag charges between midnight and 5am local time. Not always fraud, but worth asking about.

   **E. Amount patterns.** Sequences like $1.00 then $XXX.XX on the same card within an hour — classic card-test fraud pattern.

4. **Compose the report:**

```
# Transaction review — last <N> days

## Flagged for your attention
For each flag, show:
  <date> — <merchant> — <amount>
  Why it's flagged: <duplicate / amount outlier / new merchant / off-hours / card-test pattern>
  <one-line guidance: "If you don't recognize this, call <bank> — the number on the back of your card.">

## Nothing flagged
If all five passes come up empty, say so directly. "No obvious flags in the last N days."

## What I didn't check
Always include:
  - Your bank's own fraud department has better signals than I do — they see device fingerprints, merchant reputation, card-present vs not.
  - I can't tell you definitively if something is fraud. I can tell you if it looks unusual.
```

## Style

- Neutral language. "This is unusual" not "this is fraud."
- One clear action per flag: "call your bank" if unrecognized, "normal if you were traveling" for geographic oddities, etc.
- Never say "definitely fraud" — you don't have the signal.

## Guardrails

- If user says "yes this is fraud, help me," redirect them to call their bank's fraud line, not to a skill. Speed matters for real fraud — they shouldn't be chatting with an AI about it.
- Do not suggest filing chargebacks. That's a process decision for the user and their bank, not a skill output.
- Do not flag legitimate recurring charges as "new merchants" — cross-check against `get_recurring_charges` and exclude anything there.
