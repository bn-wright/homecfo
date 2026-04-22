---
name: sync-truthifi
description: Refresh the user's financial memory files by pulling current accounts, holdings, asset allocation, and transactions from the Truthifi MCP server. Use whenever the user says "sync from Truthifi", "pull latest from Truthifi", "update from Truthifi", "refresh my Truthifi data", or asks a money question when Truthifi is their configured data source. This skill replaces update-financials for users on the Truthifi integration path — do NOT run both against the same memory files in one session.
---

# Sync Truthifi

Pulls fresh financial data from the Truthifi MCP server and updates the user's homeCFO memory files. The user does not need to know MCP tool names — this skill wraps them into one workflow.

## Prerequisites

The user must have the Truthifi MCP server installed. Verify by checking whether any `mcp__truthifi__*` tools (or similarly namespaced — the exact prefix depends on how Truthifi was registered) are available in the session.

**If Truthifi MCP tools aren't available, stop and tell the user:**

> "I don't see Truthifi MCP tools in this session. Either the server isn't installed, or it hasn't connected yet. Check with `claude mcp list` — you should see `truthifi` with a connected status. If it's not there, follow the [Truthifi setup guide](../../docs/integrations/truthifi.md)."

Do NOT try to continue without MCP tools — the skill has nothing to sync from.

## Step-by-step

### Step 1 — Pull current state

Call these Truthifi MCP tools in parallel (they're independent):

- `get_accounts` — current balances by account
- `get_dated_holdings` — investment positions as of today
- `get_composition` — asset allocation by class (stocks / bonds / cash / alternatives)
- `get_market_cap_allocation` — US vs international, large/mid/small cap
- `get_budget_flows` — recent spending/income activity (or `get_investment_transactions` for brokerage-only users)
- `get_findings` — any warnings Truthifi has flagged (high fees, concentration, etc.)

Use whatever date range the user asks for. Default to last 30 days for transactions when unspecified.

### Step 2 — Diff against current memory

Before writing anything, compare what came back to what's already in memory:

- **investments.md**: did total net worth move? Which accounts? Surface the deltas.
- **spending_kb.md**: what transactions are new since the last sync? What's the month-to-date spend?
- **retirement_snapshot.md**: does the new portfolio value change the FI projection materially? (Only recompute if the delta is >1% — small moves are noise.)

### Step 3 — Update memory files

Rewrite only the sections that changed. Preserve:

- User-written comments and annotations
- Historical data (never delete)
- Account nicknames (don't replace "Joint Checking" with Truthifi's canonical name if the user already labeled it)
- The memory file header/frontmatter

For `investments.md` specifically, update the "as of" date in the header and the account balances table. Holdings detail is expensive to re-fetch and usually stable day-to-day; only refresh holdings if the user asks or if it's been more than a week since the last full refresh.

### Step 4 — Report what changed

Output format:

```markdown
## Truthifi sync — {{TODAY}}

**Accounts pulled:** N
**Memory files updated:** investments.md, spending_kb.md[, retirement_snapshot.md if recomputed]

**Balance changes since last sync ({{DATE}}):**
- [Account]: $X → $Y ({{± $Δ}}, {{± %}})
- [Only show accounts with >$100 or >1% moves — suppress noise]

**New transactions:** N since {{DATE}}
- Top 3 by amount:
  - [Date] [Merchant] [Category] $X
  - ...

**Notable:**
- [Anything Truthifi flagged via get_findings]
- [Any unusual balance moves the user should know about]
- [If an account disappeared or a new one appeared]

**Data now current through:** {{latest transaction date}}
```

### Step 5 — Stay ready

The user probably wants to follow up with a money question now that data is fresh. Don't prematurely summarize everything — just confirm the sync is done and wait.

## Key rules

- **Do not create accounts via MCP.** `create_asset_liability` is a write operation — only call it if the user explicitly asks to add a manual asset (e.g. "add our home equity to Truthifi as $680k").
- **Do not count Truthifi's "transfers" between the user's own accounts as income or spending.** Truthifi usually marks these; skip them in the spending summary.
- **Honor Brian's existing rules** (if configured in CLAUDE.md): skip Computershare deposits, skip Concur reimbursements, etc. Truthifi doesn't know about these household-specific exclusions — the skill is the enforcement layer.
- **If Truthifi returns empty data** (e.g. their sync is down), stop and tell the user. Don't write empty memory files over the existing ones.
- **Rate limiting.** If multiple Truthifi tool calls fail in a row with rate-limit errors, back off and tell the user — don't spam.

## Anti-patterns

- ❌ Silently overwriting user annotations in memory files
- ❌ Running this skill AND `update-financials` in the same session (they'll write conflicting data)
- ❌ Calling MCP write tools (`create_asset_liability`, etc.) without explicit user direction
- ❌ Reporting every transaction in the summary — surface only what's notable
- ✅ One sync call → one diff report → wait for the next question
