# Truthifi MCP setup

**What this is:** the hands-off way to get your financial data into homeCFO, if you're OK with a hosted aggregator. You sign up for [Truthifi](https://truthifi.com), connect your accounts to them once (the way you would with Empower, Monarch, etc.), then register their MCP server with Claude Code. After that, Claude can ask Truthifi for your current balances, holdings, and transactions every time you start a conversation — no quarterly CSV downloads.

**Who this is NOT for:** anyone whose hard requirement is "my financial data never leaves my machine except to Anthropic." Truthifi is a hosted service; by design, they have your data. If that's not acceptable to you, use the [CSV path](README.md) instead.

> **Status note (April 2026):** this integration doc and the `sync-truthifi` skill were written against Truthifi's MCP tool surface as observed in a Claude Code session. If you hit a snag — a command that doesn't exist, a tool that returns something unexpected — please [open an issue](https://github.com/bn-wright/homecfo/issues) so we can correct it. The skill is designed to discover Truthifi's tools dynamically, so small name changes should not break it.

---

## Before you start — the honest privacy picture

When you use this integration, your data touches **three** places: your disk, Anthropic, and Truthifi. That's one more destination than the CSV path, in exchange for setup convenience.

**The canonical statement of this tradeoff lives in [`SECURITY.md`](../../SECURITY.md) → "If you opt into a hosted MCP aggregator".** Read that section before continuing. If the framing here and the framing there ever drift apart, SECURITY.md wins.

**If this tradeoff is acceptable, continue. Otherwise, use the CSV path.**

---

## Setup — ~10 minutes

### Step 1 — Sign up for Truthifi

1. Go to [truthifi.com](https://truthifi.com) and create an account.
2. Link your financial accounts — banks, brokerages, 401(k), HSA, mortgage. Same flow as any aggregator.
3. Wait for the initial sync to complete (usually minutes; some institutions take longer, especially 401(k) / HSA providers).
4. On your Truthifi account page, find their **MCP installation instructions**. That's the canonical source for how to register their MCP server with Claude Code, and it will reflect whatever command + URL + auth format they currently use.

### Step 2 — Register Truthifi's MCP server with Claude Code

Follow Truthifi's own docs for this. At time of writing they expose a `claude mcp add` command you can copy-paste from your dashboard. The exact command, URL, and auth format are Truthifi's to define, and change more often than this repo updates — so trust their page, not us.

After you run their install command, verify it registered:

```bash
claude mcp list
```

You should see a Truthifi-like entry with a connected status. If it shows an error, see **Troubleshooting** below.

### Step 3 — Install the homeCFO sync skill (Full setup only)

If you've done the [Full setup](../../README.md#quickstart--full-setup-10-minutes), the `sync-truthifi` skill is already in `skills/` — nothing more to do.

If you're on the [Lite path](../../README.md#two-ways-in) with just `FINANCE.md`, the sync logic is built into the file itself (Skill 4 inside `FINANCE.template.md`). Set the `Data source` field in ABOUT YOU to `"truthifi"` and you're good.

### Step 4 — Test it

Open Claude Code in your homeCFO data folder (or the folder with your `FINANCE.md`) and ask:

> *"Sync my data from Truthifi and tell me what changed."*

Claude should:

1. Enumerate the Truthifi MCP tools available in your session (the skill discovers them by name suffix — it doesn't hardcode a prefix)
2. Call them in parallel to pull accounts, balances, holdings, asset allocation, recent activity, and findings
3. Update your memory files (or compute an inline answer if you're on Lite)
4. Show you a summary — balance deltas, new transactions, anything notable

If that works, you're done. Going forward, anything you'd normally say to homeCFO ("how are we doing?", "am I on track?", "I just spent $10k on a couch — does that matter?") will work against live Truthifi data.

---

## What the `sync-truthifi` skill does

The skill wraps the MCP tool calls into one workflow so you don't have to remember tool names. It:

1. Discovers Truthifi's tools in the current session (matches by suffix like `*get_accounts`, not by hardcoded prefix — so it keeps working if you registered the server under a different name)
2. Calls the read tools in parallel: `*get_accounts`, `*get_dated_holdings`, `*get_composition`, `*get_market_cap_allocation`, `*get_budget_flows` (or `*get_investment_transactions` for brokerage-only), `*get_findings`
3. Never calls write tools (`*create_*`, `*update_*`, `*delete_*`) unless you explicitly ask
4. Writes results into memory files, preserving your comments, annotations, and account nicknames
5. Handles partial failures honestly — if one call fails, it updates what it can and tells you what's stale

You can always call the MCP tools directly if you want — e.g. *"what's my asset allocation from Truthifi?"* works without the skill. The skill just makes "refresh everything" a one-liner.

---

## Troubleshooting

**`claude mcp list` shows Truthifi as disconnected.**
Usually the URL or auth key is wrong. Re-run Truthifi's install command with the exact value from your dashboard. If they rotated your key, `claude mcp remove <name>` and re-add.

**Claude says "I don't see Truthifi tools in this session."**
The MCP server is registered but not exposing tools. Restart Claude Code (`Ctrl+C`, then `claude` again). If that doesn't fix it, `claude mcp list --verbose` shows the server's actual status.

**Balances from Truthifi don't match my bank.**
Truthifi has a sync lag like any aggregator — hours for most institutions, longer for 401(k) / HSA providers. If a balance looks stale, check Truthifi's dashboard first. If Truthifi is also stale, it's an upstream issue, not a homeCFO issue.

**A specific account isn't showing up.**
Each institution has to be connected to Truthifi first. homeCFO only sees what Truthifi has. Confirm the account is linked in Truthifi's dashboard.

**Too many Claude permission prompts when the skill runs.**
Add the Truthifi tools you use most to your project `.claude/settings.json` allow list. The exact names depend on how you registered the server — run `claude mcp list --verbose` to see the canonical names, then add entries like:

```json
{
  "permissions": {
    "allow": [
      "mcp__<your-prefix>__get_accounts",
      "mcp__<your-prefix>__get_dated_holdings",
      "mcp__<your-prefix>__get_composition",
      "mcp__<your-prefix>__get_budget_flows",
      "mcp__<your-prefix>__get_findings"
    ]
  }
}
```

**The skill claims a tool "wasn't found" even though `claude mcp list` shows Truthifi connected.**
Either the tool is called something different on Truthifi's side than this doc assumes, or the MCP server didn't finish handshaking. Open an issue on the homeCFO repo with the output of `claude mcp list --verbose` so we can update the expected suffixes.

---

## Removing the integration

```bash
claude mcp remove <your-truthifi-registration-name>
```

Then optionally delete your account on truthifi.com to unlink from your institutions. Your homeCFO memory files are unaffected — they're just Markdown on your disk.
