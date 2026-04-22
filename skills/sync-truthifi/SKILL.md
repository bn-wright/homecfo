---
name: sync-truthifi
description: Refresh homeCFO memory files from a registered Truthifi MCP server. Primary triggers are explicit ("sync from Truthifi", "pull latest from Truthifi", "refresh my Truthifi data"). **Also trigger this skill — NOT `sync-local` — on any generic refresh/balance/money question ("how are we doing?", "refresh my data", "am I on track?", "what's my spending look like?") when the user's `Data source` field (in `FINANCE.md` or `MEMORY.md`) equals `truthifi`**, even if the user doesn't say the word "Truthifi." The `Data source` field is the router; honor it. If the user is on the BYO CSV path (`Data source = csv` or absent), use `sync-local` instead. Do not run both skills in the same session — they write to the same memory files and one will silently lose.
---

# Sync Truthifi

Pulls fresh financial data from the Truthifi MCP server and updates the user's memory files. The user does not need to know MCP tool names — this skill wraps them into one workflow.

If the user is NOT on the Truthifi path (`Data source = csv` or no Truthifi MCP server is registered), hand off to `sync-local` instead.

## Prerequisites — discover Truthifi's tools (do not hardcode)

Truthifi's MCP tools are registered in Claude Code under a prefix the user chose when they ran their `claude mcp add ...` command. In practice the prefix is often a random UUID (e.g. `mcp__a1b2c3d4-xxxx-xxxx-xxxx-xxxxxxxxxxxx__get_accounts` — yours will be different, check `claude mcp list`), not the string `truthifi`. The tool **suffixes** — the logical names — are stable; the prefix is not.

**Step 1: Enumerate Truthifi-like tools in this session.** Look across the available MCP tools for names ending in any of these suffixes:

- `get_accounts`
- `get_dated_holdings`
- `get_composition`
- `get_market_cap_allocation`
- `get_budget_flows` or `get_investment_transactions`
- `get_findings`

If you find none, STOP. Tell the user:

> "I don't see Truthifi tools in this session. The MCP server isn't connected. Run `claude mcp list` to check — you should see a Truthifi-like entry. If it's missing, follow Truthifi's setup docs to register it (start at truthifi.com). If it's listed but shows an error, `claude mcp list` without extra flags usually surfaces it."

Do NOT try to continue, fall back to CSV, or invent data. The skill has nothing to sync from.

**Step 2: Remember the prefix for this session** and use it consistently — e.g. if the session exposes `mcp__<UUID>__get_accounts`, use `mcp__<UUID>__*` for every call below.

## Tool call conventions

Every Truthifi read tool requires an **`include`** array listing the outer-level fields you want back (nested fields are not filterable — you get the nested structure of any outer field you include). Without `include`, the call errors. Pass the full set of outer fields for each tool unless you have a reason to narrow it. The tool schemas in Claude's environment list the valid values per call.

Tools that take a **`dateRange`** (`get_dated_holdings`, `get_composition`, `get_budget_flows`, `get_investment_transactions`) want ISO dates `{from, to}`. Tools that take a single **`date`** (`get_market_cap_allocation`) want one ISO date that has a holdings snapshot — if there's no snapshot on that exact day, the call errors with "No matching holdings history records found."

## Rate-limit reality — design around it, not despite it

Truthifi's free tier caps daily MCP tool calls. See [`docs/integrations/truthifi.md`](../../docs/integrations/truthifi.md) for the canonical numbers and current rules — that page is the source of truth for free-tier limits and this skill links back to it rather than restating them so they can't drift.

The important thing for this skill: **the free-tier budget is small enough that burning it all on one "refresh everything" run means every follow-up question in the same day gets rate-limited and fails.** So the skill is designed around a hard rule: a normal sync pulls balances ONLY (1 tool call). Everything else is fetched on-demand by the question that actually needs it, or explicitly opt-in via "full refresh".

Three modes, in order of how often you should use each. (**These A/B/C labels are internal shorthand for this skill — do NOT say "Mode A" or "Mode C" to the user.** Talk to them in plain English: "quick balance sync", "answer one question", "full refresh".)

### Mode A — Default sync (1 call). Use this unless the user asks otherwise

Call only `*get_accounts`. Update balances in memory. Done. This is the right answer for "sync from Truthifi" or for any routine money question that just needs current balances. It costs 1 of the user's daily calls and leaves the rest for whatever comes next.

### Mode B — Targeted follow-up (1 call each). For specific questions

When the user asks a question after the default sync and it needs data `get_accounts` didn't provide, call only the tool that answers that question:

| Question shape | Call |
|---|---|
| "What did I spend this week / on groceries / MTD?" | `*get_budget_flows` (or `*get_investment_transactions` for brokerage-only), 30-day window |
| "What's my asset allocation?" / "Am I over on stocks?" | `*get_composition`, 90-day window |
| "US vs international?" / "Am I too concentrated in mega-cap?" | `*get_market_cap_allocation`, today's date (retry earlier if no snapshot) |
| "Show me my holdings / positions." | `*get_dated_holdings`, 90-day window |
| "Any warnings / does Truthifi flag anything?" | `*get_findings` |

Each follow-up costs 1 of the user's daily calls. Always **narrate remaining budget** in the sync report (Step 5 template below includes this), and if you're issuing a call that brings the user close to or past their free-tier limit, call it out first in plain English: *"This will use 1 of your remaining 2 Truthifi calls today — after this you'll have 1 left before tomorrow's reset at midnight ET. Still want me to pull it?"*

### Mode C — Full refresh (up to 6 calls). Only when the user asks for it

Trigger phrases: *"do a full refresh from Truthifi"*, *"quarterly review"*, *"refresh everything"*, *"pull all my Truthifi data."*

**Before issuing Mode C, tell the user explicitly:** *"A full refresh will use up to 6 Truthifi tool calls, which exceeds the free-tier daily limit of 5. Expect `get_findings` (lowest priority) to be rate-limited. If you're on a paid plan, disregard. Proceed?"* — and actually wait for confirmation.

If they confirm, call all six in priority order (most important first) in a single parallel batch:

1. `*get_accounts` — essential.
2. `*get_budget_flows` (or `*get_investment_transactions`) — essential for spending.
3. `*get_dated_holdings` — net-worth / investment summary.
4. `*get_composition` — asset allocation.
5. `*get_market_cap_allocation` — cap bands (can error on exact-date match; that's fine, mark stale).
6. `*get_findings` — Truthifi's own commentary; often rate-limited on free tier.

On rate-limit error mid-batch, stop issuing new calls. Report what landed.

## Step-by-step (for default sync and full refresh; targeted follow-up is single-call and self-evident)

### Step 1 — Pull current state

Per the mode rules above. Default is balances-only (1 call). Only go to full refresh if the user explicitly asked and you've confirmed the call-count with them.

**Date-window defaults (when used):**

- Transactions (`get_budget_flows` / `get_investment_transactions`): **30 days.**
- Holdings / composition: **90 days.** Truthifi snapshots on its own cadence; 30 days is often empty.
- Market cap allocation: **today.** If "No matching holdings history records found," retry each day back for up to 7 days, then mark stale.

Respect any window the user explicitly asked for ("show me last week").

### Step 2 — Diff against current memory

Before writing anything, compare what came back to what's in memory:

- `investments.md` — did total net worth move? Which accounts? Surface the deltas.
- `spending_kb.md` — which transactions are new since the last sync? Month-to-date spend? *(Full refresh or transactions-only follow-up.)*
- `retirement_snapshot.md` — does the new portfolio value change the FI projection materially? Only recompute if the delta is >1%.

### Step 3 — Update memory files

Rewrite only the sections you have fresh data for. Preserve:

- User-written comments and annotations
- Historical data (never delete)
- Account nicknames — don't replace "Joint Checking" with Truthifi's canonical name if the user already labeled it
- Memory file headers / frontmatter

For `investments.md` specifically, update the "as of" date and the account balances table. Holdings detail only changes on full refresh (or explicit holdings follow-up); don't touch it otherwise.

**Masked account numbers from Truthifi (e.g. `xxxx6886`) are safe to keep in memory files** — they're masked by design and don't grant access on their own. See `SECURITY.md` for the policy.

### Step 4 — Handle partial failures and empty results honestly

**Distinguish empty-but-valid from error.** `{"output": []}` means the call succeeded and there's no data in the requested window. This is common and expected — for example, investment-only accounts have no budget flows, and holdings snapshots aren't taken daily, so a 30-day window may legitimately have no snapshots. That is NOT a failure.

When you see empty output, don't overwrite existing memory with zeros; keep what was already there and note in the summary that Truthifi had nothing fresh in the window. In the report, say it plainly: *"Holdings: no snapshot in the last 90 days — this isn't a bug, Truthifi just hasn't snapshotted recently. Existing values kept; run a full refresh if you want a wider window."*

**Actual errors** (rate limit, HTTP error, "no matching holdings history records"): update the memory sections you DO have data for, tell the user what's stale, and DO NOT retry — especially not rate-limit errors. The limit resets at midnight ET; retries just burn tomorrow's budget.

### Step 5 — Report what changed

Talk to the user in plain English. Don't say "Mode A" / "Mode B" / "Mode C" — say what you actually did.

```markdown
## Truthifi sync — {{TODAY}}

**What I pulled:** Just balances (quick sync) | {{tool}} only (answering your {{question}}) | Everything (full refresh)
**Truthifi calls used today:** N of your daily limit ({{remaining}} left before midnight ET reset)
**Accounts pulled:** N (of M)
**Memory files updated:** investments.md[, spending_kb.md, retirement_snapshot.md]
**Nothing fresh in this window:** [list anything that returned empty — "holdings: no snapshot in last 90 days" — not an error, just no new data]
**Stale / skipped:** [list anything rate-limited or errored]

**Balance changes since last sync ({{DATE}}):**
- [Account]: $X → $Y ({{± $Δ}}, {{± %}})
- [Only accounts with >$100 or >1% moves — suppress noise]

**(Follow-up / full refresh only) New transactions:** N since {{DATE}}
- Top 3 by amount: [Date] [Merchant] [Category] $X

**(Full refresh only) Notable:**
- [Anything surfaced via *get_findings]
- [Unusual balance moves worth calling out]
- [Accounts that appeared or disappeared]

**Data now current through:** {{latest transaction date or balance date}}
```

### Step 6 — Stay ready

The user probably wants to follow up with a money question now. Don't pre-summarize — confirm the sync is done and wait. If their follow-up needs data beyond balances, that's a targeted follow-up: answer it by calling only the one tool that question requires.

## Gotchas observed against the live Truthifi MCP (April 2026)

1. **`include` is required on every tool.** Pass the full outer-field list unless narrowing intentionally.
2. **Empty holdings/composition on a 30-day window is normal.** Widen to 90 days for those tools. If still empty, report "holdings stale — Truthifi hasn't snapshotted recently," don't assume zero.
3. **`get_market_cap_allocation` needs a single date that HAS a snapshot.** Today may not. Retry earlier dates for up to ~7 days, then mark stale.
4. **`maxAvailableDateRange` may be `null` per account.** Truthifi's docs suggest using it to pick windows; in practice it's often null. Fall back to the defaults above.
5. **Free-tier rate limit (resets midnight ET)** is per-call, not per-session. This is why the default is balances-only. See `docs/integrations/truthifi.md` for the current per-tier caps.
6. **Rate-limit errors are terminal for this sync.** Do NOT retry.

## Key rules

- **Default to balances-only.** One call is the right answer for almost everything. Never burn the free-tier budget on a full refresh unless the user asked for it by name.
- **Do not call write tools.** Suffixes ending in `create_*`, `update_*`, or `delete_*` (e.g. `*create_asset_liability`) mutate the user's Truthifi account. Only call them if the user explicitly asks to add or modify a manual entry.
- **Do not count internal transfers as spending or income.** Truthifi usually flags these; skip them in the spending summary.
- **Honor household-specific exclusions.** If the user's CLAUDE.md or memory files declare rules like "skip Computershare deposits" or "skip Concur reimbursements," apply them here — Truthifi doesn't know about them.
- **If Truthifi returns empty data ACROSS THE BOARD** (every call returns `[]` including `get_accounts`), their sync is likely down. Stop and tell the user. Don't overwrite existing memory with zeros.
- **Speak plain English to the user.** A/B/C is internal shorthand; in user-facing reports, describe what you actually did.

## Anti-patterns

- ❌ Running the full 6-call battery by default — that burns the free-tier day on one sync
- ❌ Hardcoding `mcp__truthifi__*` — the prefix is user-configurable (often a UUID); discover it per session
- ❌ Omitting the required `include` array on a call
- ❌ Treating `{"output": []}` as an error (it usually means "no data in this window" — valid, not broken)
- ❌ Retrying after a rate-limit error (resets at midnight ET; retries only burn more of tomorrow's budget)
- ❌ Silently overwriting user annotations in memory files
- ❌ Saying "Mode A" or "Mode C" in user-facing output — those are internal labels; describe what you actually did ("I just pulled balances" / "I pulled everything")
- ❌ Running this skill AND `sync-local` in the same session (they will write conflicting data; pick one per session based on the user's `Data source`)
- ❌ Calling MCP write tools without explicit user direction
- ❌ Reporting every transaction in the summary — surface only what's notable
- ✅ Balances-only by default → targeted follow-up when a specific question needs more → full refresh only if the user explicitly asked for one and confirmed the call-count cost
