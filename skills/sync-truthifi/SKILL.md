---
name: sync-truthifi
description: Refresh homeCFO memory files from a registered Truthifi MCP server. Use when the user says "sync from Truthifi", "pull latest from Truthifi", "refresh my Truthifi data", or asks a money question where Truthifi is the configured data source (check the user's `Data source` field in FINANCE.md or `~/finance-data/MEMORY.md`). If the user is on the BYO CSV path instead, use `update-financials`. Do not run both skills in the same session — they write to the same memory files and one will silently lose.
---

# Sync Truthifi

Pulls fresh financial data from the Truthifi MCP server and updates the user's memory files. The user does not need to know MCP tool names — this skill wraps them into one workflow.

If the user is NOT on the Truthifi path (their `Data source` is `csv` or no Truthifi MCP server is registered), hand off to `update-financials` instead.

## Prerequisites — discover Truthifi's tools (do not hardcode)

Truthifi's MCP tools are registered in Claude Code under a prefix the user chose when they ran their `claude mcp add ...` command (commonly something containing `truthifi`, but it can be anything). The tool **suffixes** — the logical names — are stable; the prefix is not.

**Step 1: Enumerate Truthifi-like tools in this session.** Look across the available MCP tools for names ending in any of these suffixes:

- `get_accounts`
- `get_dated_holdings`
- `get_composition`
- `get_market_cap_allocation`
- `get_budget_flows` or `get_investment_transactions`
- `get_findings`

If you find none, STOP. Tell the user:

> "I don't see Truthifi tools in this session. The MCP server isn't connected. Run `claude mcp list` to check — you should see a Truthifi-like entry with a connected status. If it's missing, follow Truthifi's setup docs to register it (start at truthifi.com)."

Do NOT try to continue, fall back to CSV, or invent data. The skill has nothing to sync from.

**Step 2: Remember the prefix for this session** and use it consistently — e.g. if the session exposes `mcp__truthifi__get_accounts`, use `mcp__truthifi__*` for every call below.

## Step-by-step

### Step 1 — Pull current state

Call the discovered tools in parallel (they're independent). Default date range is last 30 days when unspecified; respect whatever the user asked for.

- `*get_accounts` — current balances by account
- `*get_dated_holdings` — investment positions as of today
- `*get_composition` — asset allocation by class
- `*get_market_cap_allocation` — US vs international, cap bands
- `*get_budget_flows` — recent spending/income (prefer `*get_investment_transactions` for brokerage-only users)
- `*get_findings` — any warnings Truthifi surfaced

### Step 2 — Diff against current memory

Before writing anything, compare what came back to what's in memory:

- `investments.md` — did total net worth move? Which accounts? Surface the deltas.
- `spending_kb.md` — which transactions are new since the last sync? Month-to-date spend?
- `retirement_snapshot.md` — does the new portfolio value change the FI projection materially? Only recompute if the delta is >1%.

### Step 3 — Update memory files

Rewrite only the sections that changed. Preserve:

- User-written comments and annotations
- Historical data (never delete)
- Account nicknames — don't replace "Joint Checking" with Truthifi's canonical name if the user already labeled it
- Memory file headers / frontmatter

For `investments.md` specifically, update the "as of" date and the account balances table. Holdings detail is expensive to re-fetch and usually stable day-to-day; only refresh holdings if the user asks or if it's been more than a week since the last full refresh.

### Step 4 — Handle partial failures honestly

If one of the calls above fails while others succeed, update only the memory sections you got clean data for. Tell the user which ones are stale and why. Do NOT:

- Write empty data over existing memory
- Invent numbers to fill gaps
- Retry aggressively (one retry per failed call is plenty)

### Step 5 — Report what changed

```markdown
## Truthifi sync — {{TODAY}}

**Accounts pulled:** N (of M attempted)
**Memory files updated:** investments.md, spending_kb.md[, retirement_snapshot.md if recomputed]
**Stale / not refreshed this sync:** [list any sections that failed and why]

**Balance changes since last sync ({{DATE}}):**
- [Account]: $X → $Y ({{± $Δ}}, {{± %}})
- [Only show accounts with >$100 or >1% moves — suppress noise]

**New transactions:** N since {{DATE}}
- Top 3 by amount:
  - [Date] [Merchant] [Category] $X

**Notable:**
- [Anything surfaced via *get_findings]
- [Unusual balance moves worth calling out]
- [Accounts that appeared or disappeared]

**Data now current through:** {{latest transaction date}}
```

### Step 6 — Stay ready

The user probably wants to follow up with a money question now that data is fresh. Don't pre-summarize — confirm the sync is done and wait.

## Key rules

- **Do not call write tools.** Tool suffixes ending in `create_*`, `update_*`, or `delete_*` (e.g. `*create_asset_liability`) mutate the user's Truthifi account. Only call them if the user explicitly asks to add or modify a manual entry.
- **Do not count internal transfers as spending or income.** Truthifi usually flags these; skip them in the spending summary.
- **Honor household-specific exclusions.** If the user's CLAUDE.md or memory files declare rules like "skip Computershare deposits" or "skip Concur reimbursements," apply them here — Truthifi doesn't know about them.
- **If Truthifi returns empty data across the board**, their sync is likely down. Stop and tell the user. Don't overwrite existing memory with zeros.
- **Rate limiting.** If multiple tool calls fail with rate-limit errors, back off and tell the user.

## Anti-patterns

- ❌ Hardcoding `mcp__truthifi__*` — the prefix is user-configurable, discover it per session
- ❌ Silently overwriting user annotations in memory files
- ❌ Running this skill AND `update-financials` in the same session (they will write conflicting data; pick one per session)
- ❌ Calling MCP write tools without explicit user direction
- ❌ Reporting every transaction in the summary — surface only what's notable
- ✅ One sync call → one honest diff report (including any partial failures) → wait for the next question
