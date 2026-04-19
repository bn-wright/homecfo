# Changelog

All notable changes to homeCFO are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), versioning follows [SemVer](https://semver.org/).

## [Unreleased]

### Added
- `FINANCE.template.md` — single-file "Lite" version of homeCFO. Drop one MD into a folder with a CSV, open Claude Code, done. No skill install, no separate templates. Embeds perspective math, FI projection, and CSV ingestion inline.
- README "Two ways in" section presenting Lite vs Full setup paths
- README "Bringing in transaction data" section explaining accepted file formats
- `update-financials` skill now explicitly handles CSV input (Empower-style and lookalikes) with flexible column mapping
- README Roadmap section flagging v0.2 ingestion paths (Truthifi MCP, Plaid, generic CSV importer)

### Changed
- `update-financials` skill description and "Required inputs" updated to acknowledge both JSON and CSV paths

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
