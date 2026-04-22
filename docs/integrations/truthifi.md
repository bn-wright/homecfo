# Truthifi MCP setup

**What this is:** the easiest way to get your financial data into homeCFO, if you're OK with a hosted aggregator. You sign up for [Truthifi](https://truthifi.com), connect your accounts to them once (the way you would with Empower, Monarch, etc.), then install their MCP server into Claude Code. After that, Claude can ask Truthifi for your current balances, holdings, and transactions every time you start a conversation — no CSV downloads, no scrapers.

**Who this is NOT for:** anyone whose hard requirement is "my financial data never leaves my machine except to Anthropic." Truthifi is a hosted service; by design, they have your data. If that's not acceptable to you, use the [BYO CSV or BYO scraper path](README.md) instead.

---

## Before you start — the honest privacy picture

When you use this integration, your data goes to **three** places:

1. **Truthifi** — holds your account credentials and syncs balances/transactions on your behalf. Subject to [their privacy policy](https://truthifi.com/privacy).
2. **Anthropic** — when you ask Claude a question, the data Claude fetches from Truthifi during that conversation is transmitted to Anthropic to process your request. Subject to [Anthropic's retention and training policies](https://claude.com/legal).
3. **Your machine** — homeCFO's memory files (`investments.md`, `spending_kb.md`, etc.) are updated with the results. These stay on your disk.

That's a real tradeoff. You're getting "no scraping, no CSV exports, no login for you" in exchange for "one more service has your data." The tradeoff is the same one you already made with Empower/Monarch/Copilot — Truthifi is just more focused on the MCP/Claude use case.

**If that's acceptable, read on.** Otherwise, close this doc and use CSV export instead.

---

## Setup — 10 minutes

### Step 1 — Sign up for Truthifi (5 min)

1. Go to [truthifi.com](https://truthifi.com) and create an account.
2. Link your financial accounts — banks, brokerages, 401(k), HSA, mortgage. Same flow as any aggregator.
3. Wait for the initial sync to complete (usually a few minutes; some institutions take overnight).
4. On your Truthifi account page, find the MCP connection details. You'll need the **MCP URL** (looks like `https://mcp.truthifi.com/<your-key>`) or the **install command** they provide.

> **If Truthifi's UI has changed:** their docs at [truthifi.com/docs](https://truthifi.com/docs) are the source of truth for the exact install step. The instructions below were current as of April 2026.

### Step 2 — Install the MCP into Claude Code (2 min)

Open a terminal and run the install command Truthifi provides. It'll look something like:

```bash
claude mcp add truthifi --url https://mcp.truthifi.com/<your-key>
```

Or if Truthifi publishes their server as an npm package:

```bash
claude mcp add truthifi -- npx -y @truthifi/mcp-server
```

(Use whatever Truthifi's current docs say — these are illustrative.)

Verify it installed:

```bash
claude mcp list
```

You should see `truthifi` in the list with a green ✓ or equivalent "connected" status. If it shows an error, see **Troubleshooting** below.

### Step 3 — Install the homeCFO sync skill (1 min)

If you've already done the [Full setup](../../README.md#quickstart--full-setup-10-minutes), the `sync-truthifi` skill is part of the skills bundle — nothing more to do.

If you're on the [Lite path](../../README.md#two-ways-in) with just `FINANCE.md`, you don't need the skill — just ask Claude things like *"pull my latest from Truthifi and tell me how we're doing"* and Claude will use the MCP tools directly. The skill just standardizes the workflow for repeat use.

### Step 4 — Test it (2 min)

Open Claude Code in your homeCFO data folder and ask:

> *"Sync my data from Truthifi and tell me what changed."*

Claude should:

1. Call Truthifi MCP tools to pull accounts, balances, holdings, and recent transactions
2. Update your `investments.md` and `spending_kb.md` memory files
3. Show you a summary — balance deltas, new transactions, anything notable

If that works, you're done. Going forward, anything you'd normally say to homeCFO ("how are we doing?", "am I on track?", "I just spent $10k on a couch — does that matter?") will work against live Truthifi data.

---

## What the `sync-truthifi` skill does

The skill wraps the MCP tool calls into one workflow so you don't have to remember tool names. Specifically, it:

1. Calls `get_accounts` for current balances
2. Calls `get_dated_holdings` for investment positions as of today
3. Calls `get_composition` + `get_market_cap_allocation` for asset allocation
4. Calls `get_budget_flows` (or `get_investment_transactions` for brokerage) for recent activity
5. Calls `get_findings` to surface any warnings Truthifi has flagged (fees, concentration risk, etc.)
6. Writes the results into your memory files using the same format as `update-financials`
7. Prints a summary of what changed

You can always call the MCP tools directly if you want — e.g. *"what's my asset allocation from Truthifi?"* will work without the skill. The skill just makes "refresh everything" a one-liner.

---

## Troubleshooting

**`claude mcp list` shows Truthifi as disconnected.**
Usually means the URL or auth key is wrong. Re-run the install command with the exact value from your Truthifi dashboard. If the dashboard has rotated your key, you'll need to `claude mcp remove truthifi` and add it again with the new URL.

**Claude says "I don't see a Truthifi tool."**
The MCP server is installed but not exposing tools to this session. Try restarting Claude Code (`Ctrl+C`, then `claude` again). If that doesn't fix it, run `claude mcp list --verbose` to see the server's actual status.

**Balances from Truthifi don't match my bank.**
Truthifi has a sync lag like any aggregator — typically a few hours, sometimes longer for 401(k) / HSA providers. If a balance looks stale, check Truthifi's dashboard first to see if their sync succeeded. If Truthifi is also stale, it's an upstream issue, not a homeCFO issue.

**A specific account isn't showing up.**
Each institution has to be connected to Truthifi first. homeCFO only sees what Truthifi has. Log into Truthifi's dashboard and confirm the account is linked.

**Too many Claude permission prompts when the skill runs.**
Add the Truthifi tools you use most to your project `.claude/settings.json` allow list. Example:

```json
{
  "permissions": {
    "allow": [
      "mcp__truthifi__get_accounts",
      "mcp__truthifi__get_dated_holdings",
      "mcp__truthifi__get_composition",
      "mcp__truthifi__get_budget_flows",
      "mcp__truthifi__get_findings"
    ]
  }
}
```

(The exact tool names depend on how Truthifi namespaces them — check `claude mcp list --verbose` for the canonical names.)

---

## Removing the integration

If you decide Truthifi isn't for you:

```bash
claude mcp remove truthifi
```

Then optionally delete your account on truthifi.com so they unlink from your institutions. Your homeCFO memory files are unaffected — they're just Markdown on your disk.
