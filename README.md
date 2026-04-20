# homeCFO

**Run your family's finances the way a startup runs its books — with Claude Code as your CFO.**

Most personal finance tools want you to log in every week and scroll their charts. homeCFO flips the model. Your data lives on your machine. Your Claude Code session knows your full financial picture. You ask questions, get context-aware analysis, and update your memory files as life changes.

---

## What You Get

- **Claude skills** that know your numbers:
  - [`finance-perspective`](skills/finance-perspective/SKILL.md) — frames short-term concerns against long-term goals
  - [`update-financials`](skills/update-financials/SKILL.md) — merges new transaction data into your memory files
  - [`fi-date-projection`](skills/fi-date-projection/SKILL.md) — recalculates your financial-independence date with current portfolio
- **Memory templates** — sanitized Markdown files you fill in once with your household's profile, investments, retirement targets, and spending baseline. Claude reads them every session.
- **A philosophy** — treat your family finances like a company, with quarterly reviews, KPIs, and a CFO on call. See [PHILOSOPHY.md](PHILOSOPHY.md).

## What You Don't Get

- A web app, a SaaS subscription, or a server storing your transactions
- Affiliate links, robo-advisor upsells, or "premium tier" walls
- Any data leaving your machine, ever

## Two ways in

**Lite (5 minutes, no install):** copy [`FINANCE.template.md`](FINANCE.template.md) into any folder, rename it `FINANCE.md`, fill in the "ABOUT YOU" section at the top, drop your transaction CSV in the same folder, and open Claude Code in that folder. That's it. One self-contained file replaces the skills + templates dance.

**Full (10 minutes, what's below):** install the three skills into your global Claude config, set up five separate memory templates in a private data directory, point Claude at them via `~/.claude/CLAUDE.md`. More setup, more power — best if you want different memory for different households or more specialized skills over time.

If you're not sure, start Lite. You can graduate later.

## Quickstart — Full setup (10 Minutes)

```bash
# 1. Clone the repo
git clone https://github.com/bn-wright/homecfo.git
cd homecfo

# 2. Create a private data directory OUTSIDE the repo
mkdir ~/finance-data

# 3. Copy templates into your private directory
cp memory-templates/*.template.md ~/finance-data/

# 4. Rename and fill in each template with your real data
cd ~/finance-data
for f in *.template.md; do mv "$f" "${f/.template/}"; done
# (on Windows PowerShell:
#   Get-ChildItem *.template.md | Rename-Item -NewName { $_.Name -replace '\.template','' })

# 5. Install the Claude skills into your user config
cp -r <homecfo-path>/skills/* ~/.claude/skills/

# 6. Tell Claude where your memory lives
cat >> ~/.claude/CLAUDE.md <<'EOF'
## Family Finance
When asked about finances, spending, budget, investments, or retirement,
load `~/finance-data/MEMORY.md` first. That file indexes the rest.
EOF
```

Now open Claude Code and ask:

- *"How are we doing financially?"*
- *"Am I on track to retire at 50?"*
- *"I just spent $10,000 on a couch. Does that matter?"*

## Bringing in transaction data

Drop any of these into your `~/finance-data/` directory and the `update-financials` skill picks them up:

- `transactions_YYYY.csv` — exported from Empower, Mint, Monarch, or your bank's portal. The skill auto-detects common column-name variants (Date/Transaction Date, Amount/Debit+Credit, Description/Merchant/Payee, etc.). If a column can't be mapped, Claude will ask before guessing.
- `transactions_YYYY.json` — output from a custom scraper, in `{"transactions": [...]}` shape or a bare array.
- `accounts_latest.json` — current account balances. Optional; only if your data source provides them. Most CSV exports don't include balances, in which case you update `investments.md` by hand at quarterly reviews.

The skill normalizes whatever you feed it into a consistent internal shape (negative = money out, positive = money in) before updating memory files.

Claude loads your memory files and gives you a contextual answer.

## Repository Layout

```text
homecfo/
├── README.md                    # You are here
├── PHILOSOPHY.md                # The operator mindset
├── LICENSE                      # MIT — use it, fork it, sell services around it
├── SECURITY.md                  # How your data stays private
├── CONTRIBUTING.md              # How to add a skill or memory template
├── skills/                      # Claude skills (drop into ~/.claude/skills/)
│   ├── finance-perspective/
│   ├── update-financials/
│   └── fi-date-projection/
├── memory-templates/            # Markdown templates (copy into your data dir)
│   ├── MEMORY.template.md       # The index file Claude loads first
│   ├── user_profile.template.md
│   ├── investments.template.md
│   ├── retirement_snapshot.template.md
│   └── spending_kb.template.md
└── examples/
    └── quarterly-review-walkthrough.md
```

## Who This Is For

- Tech workers, HENRYs, and FIRE-curious households who already think in systems
- People who've outgrown Mint/Monarch and want something more analytical
- Anyone already using Claude Code who's tired of re-explaining their finances every session
- Households earning $100k+ where the difference between "sloppy" and "operated" finances compounds into hundreds of thousands of dollars

## Who This Is Not For

- People who want a pretty mobile app — use [Copilot Money](https://copilot.money)
- People who want done-for-you budgeting — use [YNAB](https://ynab.com)
- People without 30 minutes to fill in memory files — this is a tool, not a product
- People looking for licensed financial advice — talk to a CFP

## Roadmap

v0.1 is deliberately small. Likely directions for v0.2+ — order will follow what users actually ask for:

- **Alternative ingestion paths.** v0.1 assumes you bring your own data files (CSV exports, your own scraper, etc.). v0.2 will document optional integrations for users who'd rather wire in an existing aggregator:
  - [Truthifi](https://truthifi.com) via MCP (hosted; data leaves your machine in exchange for no scraping)
  - Plaid-backed services via a thin adapter
  - Generic CSV importer with a column-mapping helper
- **More skills**: `tax-loss-harvest-checker`, `equity-grant-tracker`, `mortgage-vs-invest`, `quarterly-review` (one-shot driver for the example walkthrough)
- **More templates**: `equity_grants`, `rentals`, `tax_plan`, `business_income`
- **Cross-platform Quickstart**: Linux/macOS-native instructions alongside the current Windows + PowerShell paths

The hard line stays the same: anything that sends data off your machine is opt-in, documented, and never the default.

## Security

Your financial data never leaves your machine. The `.gitignore` blocks all data files from ever entering Git. Pre-commit hooks scan for leaked account numbers. See [SECURITY.md](SECURITY.md) for the full threat model.

## Philosophy

You wouldn't run a company by glancing at QuickBooks once a month. You'd have a CFO who knows the books thoroughly, tells you when something matters, and doesn't bother you when it doesn't. That's what this repo is.

Read [PHILOSOPHY.md](PHILOSOPHY.md) for the long version.

## Contributing

Bug reports, scraper requests, new skills, better memory templates — all welcome. See [CONTRIBUTING.md](CONTRIBUTING.md). The hard rule: **no PR may include real financial data**, even sanitized. Templates only.

## License

MIT. Use it, fork it, build a paid product on top of it. If you build something cool with it, I'd love to hear about it.

**This is not financial advice.** Output from Claude Code or any LLM is not a substitute for advice from a licensed financial advisor, CPA, or tax attorney.
