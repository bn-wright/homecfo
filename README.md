# homeCFO

**A FIRE skill pack for Claude Code. Project your FI date, reframe spending anxiety against the long-term picture, and run honest quarterly reviews — all against files on your disk.**

Your data is stored on your machine — no SaaS account, no third-party aggregator, no server-side database holding your numbers between sessions. Your Claude Code session reads the files, runs the math, and tells you honestly whether something matters.

When you ask Claude a question, the relevant file contents are transmitted to Anthropic as part of that conversation (same as any Claude Code session) and are subject to Anthropic's retention and training policies. See [SECURITY.md](SECURITY.md) for the honest version of the privacy story.

---

## What You Get

- **The headliner skill — [`finance-perspective`](skills/finance-perspective/SKILL.md).** When you're spinning about a $10k couch or a 4% market dip, it computes three numbers: one month of portfolio drift, your locked-in annual savings, and the FI-date impact in days. Then it gives a verdict — *noise*, *absorbable*, or *worth attention* — with the math shown. No moralizing, no "have you considered YNAB." This is the skill I couldn't find anywhere else and the reason this repo exists.
- **[`fi-date-projection`](skills/fi-date-projection/SKILL.md)** — recalculates your financial-independence date from current portfolio, contribution rate, and target. Three sensitivity bands (5% / 7% / 9% return) every time, so you see the cone of uncertainty instead of a single false-precision number.
- **[`update-financials`](skills/update-financials/SKILL.md)** — ingests CSV exports (Empower, Mint, Monarch, raw bank) or JSON from your own scraper. Flexible column mapping, no hard-coded schema, asks before guessing.
- **Memory templates** — sanitized Markdown files you fill in once with your household's profile, investments, retirement targets, and spending baseline. Claude reads them every session.
- **A quarterly-review workflow** — see [`examples/quarterly-review-walkthrough.md`](examples/quarterly-review-walkthrough.md) for what a 10-minute FIRE check-in looks like end-to-end.

## What You Don't Get

- A web app, a SaaS subscription, or a server storing your transactions between sessions
- Affiliate links, robo-advisor upsells, or "premium tier" walls
- A third-party aggregator holding your credentials or scraping on your behalf

## How this differs from other Claude + finance projects

There's real prior art here — this is a snapshot as of April 2026; the space is moving fast, so check for yourself. If you're shopping around, look at these first:

- **[Show HN: Use Codex/Claude Code as your personal financial assistant](https://news.ycombinator.com/item?id=47232547)** — same shape (coding-agent CLI + local files). homeCFO is a packaged skill set rather than a free-form prompt project, and is FIRE-specific rather than general budgeting.
- **[Claude-Budget-Workspace-Template](https://github.com/danielrosehill/Claude-Budget-Workspace-Template)** — household-budget workspace for Claude Code, closest sibling. homeCFO ships skills (with `SKILL.md` + YAML frontmatter per Anthropic's conventions) instead of agents/slash-commands, and focuses on FI-date projection and spending-perspective reframing rather than general budget management.
- **[charlie-cfo-skill](https://github.com/EveryInc/charlie-cfo-skill)** — Claude skill for **startup** CFO workflows (cash mgmt, unit economics, fundraising). homeCFO is a **household** CFO — FI dates, not burn rates.
- **[NumbyAI](https://news.ycombinator.com/item?id=47377333)** — runs a fully local LLM (Ollama) on your machine; nothing goes to the cloud. That's a cleaner privacy story than homeCFO. The tradeoff: you're reasoning with an open-weights local model, not Claude. If zero-cloud matters more than reasoning quality, start there.
- **[anthropics/financial-services-plugins](https://github.com/anthropics/financial-services-plugins)** — Anthropic's official skills for institutional finance (investment banking, equity research, wealth management). Different audience entirely; homeCFO is for a household, not a desk.

The wedge: I couldn't find anyone else shipping a **spending-anxiety-reframing skill** or **FI-date projection** with sensitivity bands as first-class features. That's what this repo bets on.

## Two ways in

**Lite (5 minutes, no install) — start here:** copy [`FINANCE.template.md`](FINANCE.template.md) into any folder, rename it `FINANCE.md`, fill in the "ABOUT YOU" section at the top, drop your transaction CSV in the same folder, and open Claude Code in that folder. That's it. One self-contained file replaces the skills + templates dance. Most people should stop here.

**Full (10 minutes, for power users):** install the three skills into your global Claude config, set up five separate memory templates in a private data directory, point Claude at them via `~/.claude/CLAUDE.md`. More setup, more power — best if you want different memory for different households, more specialized skills over time, or you find yourself copying `FINANCE.md` between multiple folders.

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

The hard line stays the same: anything that sends data to a **third party other than Anthropic** (aggregators, analytics, affiliate trackers) is opt-in, documented, and never the default. Claude itself is always in the loop — that's the whole tool.

## Security

Your data is stored locally — no SaaS, no server-side database, no third-party aggregator. The `.gitignore` blocks all data files from ever entering Git, and pre-commit hooks scan for leaked account numbers. Note that when you talk to Claude, file contents are transmitted to Anthropic for the conversation to work, subject to Anthropic's retention and training policies. See [SECURITY.md](SECURITY.md) for the full threat model and the honest version of what "local-first" does and doesn't buy you.

## Philosophy

You wouldn't run a company by glancing at QuickBooks once a month. You'd have a CFO who knows the books thoroughly, tells you when something matters, and doesn't bother you when it doesn't. That's what this repo is.

Read [PHILOSOPHY.md](PHILOSOPHY.md) for the long version.

## Contributing

Bug reports, scraper requests, new skills, better memory templates — all welcome. See [CONTRIBUTING.md](CONTRIBUTING.md). The hard rule: **no PR may include real financial data**, even sanitized. Templates only.

## License

MIT. Use it, fork it, build a paid product on top of it. If you build something cool with it, I'd love to hear about it.

**This is not financial advice.** Output from Claude Code or any LLM is not a substitute for advice from a licensed financial advisor, CPA, or tax attorney.
