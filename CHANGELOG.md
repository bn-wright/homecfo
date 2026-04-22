# Changelog

All notable changes to homeCFO are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), versioning follows [SemVer](https://semver.org/).

## [Unreleased]

### Added
- **Truthifi MCP integration (v0.2 track)** — the most-requested v0.2 feature, shipped as the first hosted-aggregator ingestion path:
  - `skills/sync-truthifi/SKILL.md` — new skill that wraps Truthifi MCP tools (`get_accounts`, `get_dated_holdings`, `get_composition`, `get_budget_flows`, `get_findings`, etc.) into one refresh workflow. Users never need to learn MCP tool names.
  - `docs/integrations/truthifi.md` — step-by-step setup guide written for non-technical users: privacy tradeoff up front, copy-paste MCP install command, test prompt, troubleshooting, and a permission-allowlist example to suppress prompts on the common tools.
  - `docs/integrations/README.md` — ingestion-path picker (BYO CSV vs BYO scraper vs Truthifi MCP) with a decision tree and a privacy comparison table.
  - `FINANCE.template.md` (Lite path) — now supports Truthifi too. New "Option A / Option B" setup instructions at the top, a `Data source` field in ABOUT YOU so the user declares their choice (`csv` or `truthifi`), and an inline Skill 4 (Truthifi MCP sync) that instructs Claude to pull from MCP tools when `Data source = "truthifi"`. The "never send to external service" rule was reworded to exclude user-opted-in MCP servers (Truthifi) — the prior wording contradicted the new capability.
  - `SECURITY.md` now contains the canonical statement of the Truthifi privacy tradeoff (three data destinations: disk, Anthropic, Truthifi). Integration docs, the Lite template, and the README all link back to it rather than restating the framing — single source of truth.
  - README updated with an "Option A / Option B" split under "Bringing in transaction data" so Truthifi is discoverable as the easy path.

### Changed (pre-merge refactor of the v0.2 track)
- Removed speculative `claude mcp add truthifi ...` install commands from `docs/integrations/truthifi.md`. The setup guide now points users to Truthifi's own dashboard for the current install command, URL, and auth format — those are Truthifi's to define and change more often than this repo updates.
- `skills/sync-truthifi/SKILL.md` now discovers Truthifi's MCP tools by **suffix** (`*get_accounts`, `*get_dated_holdings`, …) instead of hardcoding the `mcp__truthifi__*` prefix. The prefix is user-configurable at `claude mcp add` time, so hardcoding it silently broke the skill for anyone who picked a different name. Same fix applied to the inline Skill 4 in `FINANCE.template.md`.
- Removed the unimplemented `"both"` option from the Lite template's `Data source` field. The prior wording documented a "Truthifi primary, CSV fallback" behavior that was never actually built; `"csv"` and `"truthifi"` are the two supported values, no silent fallback between them.
- Reworded the "What Lite means" section of `FINANCE.template.md` to be honest that Option B requires a one-time MCP install — the prior wording implied Option B was as zero-install as Option A, which wasn't accurate.
- `FINANCE.template.md` — single-file "Lite" version of homeCFO. Drop one MD into a folder with a CSV, open Claude Code, done. No skill install, no separate templates. Embeds perspective math, FI projection, and CSV ingestion inline.
- README "Two ways in" section presenting Lite vs Full setup paths
- README "Bringing in transaction data" section explaining accepted file formats
- `update-financials` skill now explicitly handles CSV input (Empower-style and lookalikes) with flexible column mapping

### Changed
- README Roadmap reflects that Truthifi MCP has shipped in v0.2; Plaid adapter and generic CSV importer deferred to v0.3+
- `update-financials` skill description and "Required inputs" updated to acknowledge both JSON and CSV paths
- Repo layout in README now includes `skills/sync-truthifi/` and `docs/integrations/`

## [0.1.0] - 2026-04-19

Initial public release.

### Added

- `README.md` with 10-minute Quickstart and repo layout
- `PHILOSOPHY.md` — the operator-mindset essay and operating loop
- `SECURITY.md` — threat model and defenses (gitignore, pre-commit, templates-only)
- `CONTRIBUTING.md` — contribution rules; the "no real data" hard rule
- `LICENSE` — MIT, with explicit "not financial advice" disclaimer
- Three Claude skills:
  - `finance-perspective` — frames short-term concerns against long-term goals
  - `update-financials` — summarizes diffs after refreshing local memory files
  - `fi-date-projection` — recalculates the financial-independence date
- Five memory templates:
  - `MEMORY.template.md` — index file Claude loads first
  - `user_profile.template.md` — household identity, income, accounts
  - `investments.template.md` — holdings, allocation, net worth
  - `retirement_snapshot.template.md` — FI target and current trajectory
  - `spending_kb.template.md` — transaction baseline and category norms
- One example walkthrough: `examples/quarterly-review-walkthrough.md`
- Pre-commit pipeline: gitleaks + custom account-number regexes + large-file blocks + markdown lint
- GitHub config: issue templates (bug/feature/skill), PR template, CI workflow that runs pre-commit on every push
