---
name: update-financials
description: Refresh the user's financial memory files from a local data source (transaction export, scraper output, or CSV) and summarize what changed. Use whenever the user says "update financials", "refresh my data", "sync my accounts", "pull the latest transactions", or asks about recent transactions/balances that may not be in memory yet.
---

# Update Financials

This skill ingests fresh financial data from local files and updates the user's memory files. It does NOT scrape any service directly — that's a tool the user runs separately (and keeps in their own private repo). This skill reads the output.

## When to use

- "Update my financials"
- "Refresh the data"
- "What did I spend yesterday?" (when transactions aren't in memory yet)
- "Sync the latest from [bank/brokerage]"
- After the user mentions running their own scraper/export

## What this skill does

1. **Locate the latest data files.** Look in the user's data directory (path is in their CLAUDE.md) for files matching:
   - `transactions_YYYY.json` or `transactions_*.csv`
   - `accounts_latest.json` or `accounts_*.json`
   - `holdings_*.json`
2. **Diff against current memory.** Compare new data to what's in `spending_kb.md`, `investments.md`, etc.
3. **Update the memory files.** Append new transactions, refresh balances, recompute totals.
4. **Summarize changes.** Show the user a brief diff: new transactions, balance deltas, any unusual line items.

## Required inputs

- The user's CLAUDE.md must specify their data directory path
- Source files must already exist in that directory (this skill does NOT fetch them)
- Files must be JSON or CSV in a recognizable shape (account list, transaction list, holdings list)

## Output format

```
## Financial update — {{DATE}}

**Sources read:** transactions_2026.json (N tx, last: YYYY-MM-DD), accounts_latest.json

**Memory files updated:**
- spending_kb.md: +N transactions since last sync
- investments.md: net worth $X → $Y (Δ $Z)
- retirement_snapshot.md: FI date {{old}} → {{new}}

**Notable changes:**
- [Anything unusual: large transactions, unexpected balance moves, new accounts]

**Sanity checks:**
- [Account-level reconciliation: do new balances match expected? Flag mismatches]
```

## Conventions

- **Never delete historical data.** Only append or update-in-place.
- **Preserve user annotations.** If the user has commented on a memory file ("// this was a one-time medical bill"), keep those comments when rewriting the file.
- **Categorize new transactions** using the existing taxonomy in `spending_kb.md`. Don't invent new categories without flagging.
- **Flag, don't fix, anomalies.** If a transaction looks miscategorized or duplicated, tell the user — don't silently change it.

## Anti-patterns

- ❌ Scraping a website directly from this skill (out of scope; legal grey area)
- ❌ Sending data to any external API
- ❌ Rewriting memory files in a way that loses prior structure
- ✅ Read local files → update local files → tell the user what changed
