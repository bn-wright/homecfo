---
name: sync-local
description: Refresh homeCFO memory files from local CSV or JSON files the user maintains on their own disk (Empower exports, Mint/Monarch/bank exports, hand-maintained JSON). Use when the user says "update my financials", "refresh my data", "pull the latest transactions", "I just downloaded a new export", or asks about recent transactions/balances that may not be in memory yet — AND their data source is local files. **Routing rule: before running, check the user's `Data source` field (in `MEMORY.md` or `FINANCE.md`). If it's `truthifi`, hand off to `sync-truthifi` instead — that skill pulls from the Truthifi MCP server and supersedes this one for those users.** Do not run both skills in the same session — they write conflicting data to the same memory files.
---

# Sync Local (formerly `update-financials`)

This skill ingests fresh financial data from local files and updates the user's memory files. It does NOT log in to or scrape any service directly. The user produces the export some other way (downloaded statement, structured JSON they maintain themselves) and drops it in their data directory; this skill reads it.

> **Naming note.** This skill was renamed from `update-financials` to `sync-local` in v0.2 to pair cleanly with `sync-truthifi`. Both read the user's declared `Data source`; one pulls from files on disk, one pulls from Truthifi's MCP server. If you see older docs referring to `update-financials`, it's the same skill.

## Routing: check `Data source` first

Before doing anything, look at the user's memory for a `Data source` field:

- **Lite path:** near the top of `FINANCE.md`, inside the `ABOUT YOU` section.
- **Full path:** at the top of `MEMORY.md` (the index file loaded first).

If the field exists and equals `truthifi`, STOP. Tell the user:

> "You're on the Truthifi path (Data source = truthifi). Use `sync-truthifi` to refresh from the Truthifi MCP server. I won't run `sync-local` here — it would write conflicting data to the same memory files."

Then hand off. Only run this skill if `Data source` is `csv`, absent, or the user has explicitly overridden it this session ("just read the CSV in this folder even though I'm set up for Truthifi").

## When to use

- "Update my financials" (when CSV/JSON is the data source)
- "Refresh the data"
- "What did I spend yesterday?" (when transactions aren't in memory yet, and Truthifi isn't configured)
- After the user mentions a fresh export they just dropped in the folder

## When NOT to use

- The user's `Data source = truthifi` → hand off to `sync-truthifi`
- `sync-truthifi` already ran in this session → stop, don't double-write
- The user is asking a question that doesn't need fresh data — just answer from memory

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

The skill must handle both JSON and CSV. Detect by file extension and use the matching path.

### JSON path

Expected shape (or compatible — be flexible):

```json
{ "transactions": [ { "date": "YYYY-MM-DD", "amount": -12.34, "description": "...", "category": "...", "account": "..." }, ... ] }
```

Or a bare array of transactions. If the shape is unfamiliar, read the first record and adapt.

### CSV path (Empower export and lookalikes)

Empower's "Download Transactions" CSV typically has columns like:

```csv
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

```markdown
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

- ❌ Logging in to or scraping a website directly from this skill (out of scope; most institutions' TOS prohibit it)
- ❌ Sending data to any external API
- ❌ Running this skill AND `sync-truthifi` in the same session — they will both write to the same memory files and one will silently lose. Pick the one matching the user's `Data source`.
- ❌ Ignoring the `Data source` field and defaulting to local-file ingestion for a Truthifi user
- ❌ Rewriting memory files in a way that loses prior structure
- ✅ Read `Data source` → if `csv`, read local files → update local files → tell the user what changed
