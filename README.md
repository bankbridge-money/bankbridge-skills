# BankBridge Skills

Claude Agent Skills that wrap [BankBridge](https://bankbridge.money) — the MCP server that gives Claude read-only access to your bank accounts, transactions, and investment holdings.

Skills here are broader, more autonomous capabilities than the [19 slash-command prompts](https://github.com/bankbridge-money/bankbridge-plugin) shipped with the plugin. Each skill orchestrates multiple BankBridge tools to accomplish a higher-level goal.

## Included skills

| Skill | What it does |
|-------|--------------|
| [`monthly-money-review`](./monthly-money-review/SKILL.md) | Complete monthly financial review — cashflow, categories, merchants, subscriptions, investment delta, narrative takeaways. |
| [`subscription-auditor`](./subscription-auditor/SKILL.md) | Find every recurring charge, flag price creeps, surface duplicates and zombie subs, suggest cancel candidates. |
| [`investment-health-check`](./investment-health-check/SKILL.md) | Portfolio value, gain/loss by position, concentration risks, dividend income run-rate. |
| [`fraud-detective`](./fraud-detective/SKILL.md) | Surface unusual charges, duplicate transactions, unfamiliar merchants, off-hours card use. |
| [`budget-builder`](./budget-builder/SKILL.md) | Draft a realistic monthly budget from your actual last-3-months spending, split into fixed vs variable. |
| [`tax-prep-assistant`](./tax-prep-assistant/SKILL.md) | Export a year of transactions grouped by category for bookkeeping or tax filing. |

## Requirements

Each skill requires BankBridge to be connected as an MCP server in the Claude host you're using. Get started at [bankbridge.money](https://bankbridge.money).

## How to use these

- **Claude Code:** `/plugin marketplace add bankbridge-money/bankbridge-skills` then install individual skills, or point Claude at specific SKILL.md files with `@skills/...`.
- **Claude Desktop / claude.ai:** skills ship as prompt templates. Copy the body of any SKILL.md into a new conversation to activate.
- **Any MCP client:** the skill bodies are plain-text prompts; paste them in.

## Privacy

BankBridge stores no transaction, balance, or account data. Every tool call live-fetches from your bank via trusted banking rails — the skills in this repo inherit that property. Nothing here caches or persists your financial data.

## Related

- **BankBridge MCP server:** [bankbridge.money](https://bankbridge.money)
- **Claude Code plugin (19 slash commands):** [bankbridge-money/bankbridge-plugin](https://github.com/bankbridge-money/bankbridge-plugin)
- **Support:** [support@bankbridge.money](mailto:support@bankbridge.money)

## License

MIT. Use freely in your own workflows, tools, and plugins.
