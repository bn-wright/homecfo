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
   - `transactions_YYYY.json` or `transactions_*.csv` (or `.tsv`, `.xlsx`)
   - `accounts_latest.json` or `accounts_*.json`/`accounts_*.csv`
   - `holdings_*.json` or `holdings_*.csv`
2. **Detect format and parse accordingly** (see "Input formats" below).
3. **Diff against current memory.** Compare new data to what's in `spending_kb.md`, `investments.md`, etc.
4. **Update the memory files.** Append new transactions, refresh balances, recompute totals.
5. **Summarize changes.** Show the user a brief diff: new transactions, balance deltas, any unusual line items.

## Input formats

The skill must handle both JSON (from a custom scraper) and CSV (from a manual export). Detect by file extension and use the matching path.

### JSON path

Expected shape (or compatible — be flexible):

```json
{ "transactions": [ { "date": "YYYY-MM-DD", "amount": -12.34, "description": "...", "category": "...", "account": "..." }, ... ] }
```

Or a bare array of transactions. If the shape is unfamiliar, read the first record and adapt.

### CSV path (Empower export and lookalikes)

Empower's "Download Transactions" CSV typically has columns like:

```
Date, Account, Description, Category, Tags, Amount
2026-04-15, Chase Checking, "STARBUCKS #1234", Dining, , -6.85
```

Other institutions use variants — Mint-style, Monarch, raw bank exports. **Don't hard-code column names**. Instead:

1. Read the first row as headers
2. Map flexibly using these synonyms:
   - **Date**: `Date`, `Transaction Date`, `Posted Date`, `Trans Date`
   - **Amount**: `Amount`, `Debit`/`Credit` pair (combine: credit positive, debit negative)
   - **Description**: `Description`, `Merchant`, `Payee`, `Name`, `Memo`
   - **Category**: `Category`, `Tags`, `Classification`
   - **Account**: `Account`, `Account Name`, `Source Account`
3. If a required field (date or amount) can't be mapped, **stop and ask the user** — don't guess.
4. Sign convention: in the normalized output, **negative = money out, positive = money in**. Convert if the source uses a different convention (e.g., all-positive with separate Debit/Credit columns, or Empower's "Amount" column which is already signed).

After parsing, treat the result identically to the JSON path.

### Tips for CSV-only users

- If `accounts_latest.csv` is missing (most CSV exports don't include balances), skip the balance-refresh step and tell the user. They can paste current balances into `investments.md` manually for the next quarterly review.
- Empower's CSV doesn't include investment holdings — only cash transactions. For holdings, the user will need to update `investments.md` by hand or via a separate export.
- Save a normalized JSON copy alongside the CSV (`transactions_2026.json`) so subsequent runs are fast and the JSON path can be used.

## Required inputs

- The user's CLAUDE.md must specify their data directory path
- At least one source file (CSV or JSON) must exist in that directory
- This skill does NOT fetch data from any service

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
