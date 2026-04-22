<!--
This is the index file Claude loads first when answering finance questions.
It tells Claude which other files exist and what's in each.
Keep entries one line each. Update this file when you add or rename memory files.
-->

# Memory Index

**Data source:** csv
<!-- Exactly one of: "csv" or "truthifi". Tells skills which ingestion path to
     use. "csv" = local files in your data dir, parsed by `sync-local`.
     "truthifi" = pulled live from the Truthifi MCP server by `sync-truthifi`.
     If you switch paths, update this field — the skills route on it. -->

- [User Profile](user_profile.md) — Identity, household, income, accounts, recurring expenses
- [Investments](investments.md) — Current portfolio: holdings, allocation, net worth as of {{LAST_UPDATED}}
- [Retirement Snapshot](retirement_snapshot.md) — FI target, timeline, runway projections
- [Spending Knowledge Base](spending_kb.md) — Categorized transactions for {{YEARS}}; baseline category norms

<!--
Add additional files as your needs grow. Examples:
- [Equity Grants](equity_grants.md) — RSU vest schedule, ESPP, ISO/NSO grants
- [Rental Properties](rentals.md) — Property-level cash flow, mortgage details
- [401k Plan Details](401k_plan.md) — Contribution sources, mega backdoor Roth
- [Tax Plan](tax_plan.md) — Withholding strategy, estimated payments
-->
