# Changelog

All notable changes to homeCFO are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), versioning follows [SemVer](https://semver.org/).

## [Unreleased] — v0.2 track

### Added
- **Truthifi MCP integration** — the first hosted-aggregator ingestion path, shipped end-to-end:
  - `skills/sync-truthifi/SKILL.md` — wraps Truthifi's MCP tools (`get_accounts`, `get_dated_holdings`, `get_composition`, `get_budget_flows`, `get_findings`, etc.) into one refresh workflow. Discovers tools by **suffix** (`*get_accounts`, …) so the skill works regardless of what prefix the user picked at `claude mcp add` time. Defaults to a single `get_accounts` call (balances only) because Truthifi's free tier caps at 5 calls/day — a "pull everything" sync would burn the daily budget before follow-ups. A targeted one-call mode handles specific questions (spending / allocation / holdings), and an explicit opt-in full-refresh mode warns the user up front that it will exceed the free-tier limit and waits for confirmation. Skill also documents six live-API gotchas (required `include` array, empty-but-valid `{"output": []}` responses, `get_market_cap_allocation` single-date-snapshot quirk, `maxAvailableDateRange` often null, 90-day window for holdings/composition, rate-limit errors are terminal — never retry).
  - `docs/integrations/truthifi.md` — setup guide for non-technical users: privacy tradeoff up front, pointer to Truthifi's own dashboard for the current install command (so we don't drift from their authoritative instructions), test prompt, troubleshooting, and a permission-allowlist example to suppress prompts on the common tools.
  - `docs/integrations/README.md` — ingestion-path picker (BYO CSV vs Truthifi MCP) with a decision tree and a privacy comparison table.
  - `SECURITY.md` — canonical statement of the Truthifi privacy tradeoff (three data destinations: disk, Anthropic, Truthifi). Everything else links back to this rather than restating the framing.
- **`FINANCE.template.md`** — single-file "Lite" version of homeCFO. Drop one MD into a folder with a CSV (or connect Truthifi), open Claude Code, done. No skill install, no separate templates. Supports both paths via an "Option A / Option B" split and a `Data source` field in ABOUT YOU so the user declares their choice (`csv` or `truthifi`); an inline Skill 4 pulls from Truthifi MCP tools when `Data source = "truthifi"`. The "never send to external service" rule was reworded to exclude user-opted-in MCP servers (Truthifi) — the prior wording contradicted the new capability.
- README: "Two ways in" (Lite vs Full) and "Bringing in transaction data" sections; `sync-truthifi` listed alongside `sync-local` in "What You Get"; repo layout includes `skills/sync-truthifi/` and `docs/integrations/`.
- `examples/quarterly-review-walkthrough.md` — calls out the CSV assumption up front, includes a Truthifi-path mini-example, and cross-references the `Data source` field in `MEMORY.template.md`.

### Changed
- **`update-financials` skill renamed to `sync-local`** to pair cleanly with `sync-truthifi`. Same behavior; the rename makes the two ingestion paths symmetric in the skill listing. `sync-local` now also explicitly handles CSV input (Empower-style and lookalikes) with flexible column mapping, and its description documents the routing rule: check the `Data source` field first — if it's `truthifi`, hand off to `sync-truthifi` instead of running.
- `sync-local` and `sync-truthifi` cross-reference each other and share a reciprocal "don't run both in the same session — they write to the same memory files" anti-pattern. Previously only `sync-truthifi` carried the warning, so a user invoking the local path first could still collide.
- `CONTRIBUTING.md` acknowledges user-opted-in MCP servers as an in-scope ingestion pattern, and clarifies that scrapers against logged-in bank portals remain out of scope.
- README Roadmap reflects that Truthifi MCP has shipped in v0.2.

### Removed
- **Bring-your-own-scraper as a recommended/documented path.** Earlier drafts pitched a Selenium/Playwright path against bank portals as a third ingestion option. Most US bank/brokerage TOS prohibit automated browsing of customer portals; we don't want to push users toward a category of tool that can get accounts locked, and we don't want to maintain one either. `sync-local` still reads JSON if someone maintains a scraper privately, but the pattern isn't documented, endorsed, or supported here. The `CONTRIBUTING.md` "out of scope" list and the `SECURITY.md` threat model were updated to match.
- The unimplemented `"both"` option from the Lite template's `Data source` field. The prior wording documented a "Truthifi primary, CSV fallback" behavior that was never actually built; `"csv"` and `"truthifi"` are the two supported values, no silent fallback between them.
- Speculative hardcoded `claude mcp add truthifi ...` install command from `docs/integrations/truthifi.md`. The setup guide now points users to Truthifi's own dashboard for the current install command — those are Truthifi's to define and change more often than this repo updates.

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
  - `update-financials` — summarizes diffs after refreshing local memory files (renamed to `sync-local` in v0.2)
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
