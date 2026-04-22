# homeCFO

**A FIRE skill pack for Claude Code. Project your FI date, reframe spending anxiety against the long-term picture, and run honest quarterly reviews — all against files on your disk.**

Your data stays on your disk. Claude Code reads the files, runs the math, and tells you honestly whether something matters. Anthropic sees the conversation contents (that's how Claude works); nobody else does, unless you opt into a hosted aggregator. See [SECURITY.md](SECURITY.md) for the full privacy story.

---

## The one skill this repo exists for: `finance-perspective`

If you're like me and get stuck obsessing over purchases before you make them, this is the skill that reframes the spiral with logical perspective. When you're spinning about a $10k couch or a 4% market dip, it computes three numbers: one month of portfolio drift, your locked-in annual savings, and the FI-date impact in days. Then it gives a verdict — *noise*, *absorbable*, or *worth attention* — with the math shown. No moralizing. When the math clears a purchase, consider it a proverbial *treat yourself*.

This is the skill I couldn't find anywhere else and the reason this repo exists. [See the skill →](skills/finance-perspective/SKILL.md)

---

## Get started in 5 minutes (Lite)

1. Copy [`FINANCE.template.md`](FINANCE.template.md) into any folder and rename it `FINANCE.md`
2. Fill in the "ABOUT YOU" section at the top
3. Drop your transaction CSV in the same folder — or [connect Truthifi](docs/integrations/truthifi.md) if you'd rather skip exports
4. Open Claude Code in that folder and ask:

> *"How are we doing financially?"*
> *"Am I on track to retire at 50?"*
> *"I just spent $10,000 on a couch. Does that matter?"*

One self-contained file replaces the skills + templates dance. Most people should stop here.

---

## What else is in the pack

- **[`fi-date-projection`](skills/fi-date-projection/SKILL.md)** — recalculates your FI date with three sensitivity bands (5% / 7% / 9% return), so you see the cone of uncertainty instead of a single false-precision number
- **[`sync-local`](skills/sync-local/SKILL.md)** — ingests CSV or JSON exports (Empower, Mint, Monarch, raw bank) with flexible column mapping
- **[`sync-truthifi`](skills/sync-truthifi/SKILL.md)** *(v0.2)* — refreshes memory files from the [Truthifi](https://truthifi.com) MCP server instead of local CSVs, for users who've connected one
- **Memory templates** — Markdown you fill in once with household profile, investments, retirement targets, spending baseline
- **Quarterly-review workflow** — see the [example walkthrough](examples/quarterly-review-walkthrough.md) of a 10-minute FIRE check-in

---

## Bringing in transaction data

Two paths, same downstream skills:

| Path | Setup | Privacy posture | Best for |
|---|---|---|---|
| **[Truthifi MCP](docs/integrations/truthifi.md)** | ~10 min | Data reaches Truthifi + Anthropic | Don't want to export CSVs; already OK with hosted aggregators |
| **[BYO CSV](docs/integrations/README.md)** | ~2 min | Stays on your disk (Anthropic still sees conversations) | Want zero third-party data sharing beyond Anthropic itself |

Set the `Data source` field in your `FINANCE.md` (Lite) or `MEMORY.md` (Full) to `csv` or `truthifi` — the sync skills route on it. Mixing is fine: use Truthifi for current balances, keep old CSVs for history.

> **Note on scrapers.** homeCFO doesn't ship or recommend one — most US bank TOS prohibit automated portal browsing. If you maintain one privately, `sync-local` will still read its JSON output.

---

## Full setup (10 min, for power users)

Install skills globally and keep memory in a private data directory. Worth it if you want different memory for different households, plan to layer on more specialized skills over time, or find yourself copying `FINANCE.md` between folders.

```bash
# 1. Clone
git clone https://github.com/bn-wright/homecfo.git && cd homecfo

# 2. Create a private data directory OUTSIDE the repo
mkdir ~/finance-data

# 3. Copy templates into your private directory
cp memory-templates/*.template.md ~/finance-data/

# 4. Rename and fill in
cd ~/finance-data
for f in *.template.md; do mv "$f" "${f/.template/}"; done
# Windows PowerShell:
#   Get-ChildItem *.template.md | Rename-Item -NewName { $_.Name -replace '\.template','' }

# 5. Install skills
cp -r <homecfo-path>/skills/* ~/.claude/skills/

# 6. Tell Claude where memory lives
cat >> ~/.claude/CLAUDE.md <<'EOF'
## Family Finance
When asked about finances, spending, budget, investments, or retirement,
load `~/finance-data/MEMORY.md` first. That file indexes the rest.
EOF
```

---

## Who this is (and isn't) for

**For:** tech workers and FIRE-curious households who already think in systems, anyone who's outgrown Mint/Monarch and wants something more analytical, Claude Code users tired of re-explaining their finances every session, households where the gap between "sloppy" and "operated" finances compounds into real money.

**Not for:** people who want a pretty mobile app (try [Copilot Money](https://copilot.money)), people who want done-for-you budgeting ([YNAB](https://ynab.com)), people without 30 minutes to fill in memory files, people looking for licensed financial advice (talk to a CFP).

---

## Philosophy

You wouldn't run a company by glancing at QuickBooks once a month. You'd have a CFO who knows the books thoroughly, tells you when something matters, and doesn't bother you when it doesn't. That's what this repo is. Long version: [PHILOSOPHY.md](PHILOSOPHY.md).

---

## Roadmap

- **v0.1** — initial release (`finance-perspective`, `fi-date-projection`, CSV ingestion, memory templates)
- **v0.2** *(shipped 2026-04)* — Truthifi MCP integration; `update-financials` → `sync-local` rename to pair with `sync-truthifi`
- **Future** (demand-driven): more ingestion paths, more skills, more templates (equity grants, rentals, business income), cross-platform Quickstart

The hard line: anything that sends data to a third party other than Anthropic is opt-in, documented, and never the default.

---

<details>
<summary><strong>Prior art — other Claude + finance projects</strong></summary>

Snapshot as of April 2026; the space moves fast, check for yourself.

- **[Show HN: Use Codex/Claude Code as your personal financial assistant](https://news.ycombinator.com/item?id=47232547)** — same shape (coding-agent + local files). homeCFO is a packaged skill set rather than free-form prompts, and FIRE-specific rather than general budgeting.
- **[Claude-Budget-Workspace-Template](https://github.com/danielrosehill/Claude-Budget-Workspace-Template)** — closest sibling. homeCFO ships skills (YAML frontmatter, Anthropic conventions) instead of agents/slash-commands, and focuses on FI-date projection + spending-perspective reframing.
- **[charlie-cfo-skill](https://github.com/EveryInc/charlie-cfo-skill)** — **startup** CFO workflows (cash mgmt, unit economics). homeCFO is a **household** CFO — FI dates, not burn rates.
- **[NumbyAI](https://news.ycombinator.com/item?id=47377333)** — fully local LLM via Ollama. Cleaner privacy story than homeCFO, at the cost of reasoning quality. If zero-cloud matters more than frontier reasoning, start there.
- **[anthropics/financial-services-plugins](https://github.com/anthropics/financial-services-plugins)** — institutional finance (investment banking, equity research). Different audience entirely; homeCFO is for a household, not a desk.

The wedge: nobody else ships a **spending-anxiety-reframing skill** or **FI-date projection with sensitivity bands** as first-class features.

</details>

<details>
<summary><strong>Repository layout</strong></summary>

```text
homecfo/
├── README.md · PHILOSOPHY.md · SECURITY.md · CONTRIBUTING.md · LICENSE
├── FINANCE.template.md             # Lite: one-file setup
├── skills/
│   ├── finance-perspective/
│   ├── fi-date-projection/
│   ├── sync-local/                 # BYO CSV/JSON
│   └── sync-truthifi/              # Truthifi MCP (v0.2+)
├── memory-templates/               # Full: Markdown templates for your data dir
│   ├── MEMORY.template.md
│   ├── user_profile.template.md
│   ├── investments.template.md
│   ├── retirement_snapshot.template.md
│   └── spending_kb.template.md
├── docs/integrations/              # Ingestion-path setup guides
└── examples/
    └── quarterly-review-walkthrough.md
```

</details>

---

## Contributing

Bug reports, new skills, better memory templates, additional MCP integrations — all welcome. See [CONTRIBUTING.md](CONTRIBUTING.md). The hard rule: **no PR may include real financial data**, even sanitized. Templates only.

## License

MIT. Use it, fork it, build a paid product on top of it.

**This is not financial advice.** Output from Claude Code or any LLM is not a substitute for advice from a licensed financial advisor, CPA, or tax attorney.
