# Security & Privacy

## The design (honest version)

Your financial data is **stored** on your machine — this repo contains no data, only templates, skills, and documentation, and the whole design keeps your numbers on your local disk and out of Git. There is no SaaS backend, no aggregator, no server-side database tied to your account holding balances or transactions between sessions.

What this does NOT mean: that Anthropic never sees your numbers. When you ask Claude a question, the file contents Claude reads are transmitted to Anthropic as part of that conversation — that's how the model has context to answer. Those conversations are subject to Anthropic's retention and (depending on your account settings) training policies. Check [claude.com/legal](https://claude.com/legal) and your account privacy settings for current terms.

So the honest framing is: **your data lives on your disk, you control what ends up in the files Claude reads, and no third party other than Anthropic is in the loop** — *unless you opt into a hosted MCP aggregator like Truthifi*, in which case see the next section. That's a meaningfully different posture than a typical finance SaaS, but it is not "your data never touches a server."

### If you opt into a hosted MCP aggregator (e.g. Truthifi)

homeCFO supports pulling data from [Truthifi](https://truthifi.com) — a hosted aggregator that exposes your accounts over MCP — as an alternative to CSV/scraper ingestion. This is opt-in and off by default. If you turn it on, the honest picture changes: your data now touches **three** places, not two.

| Destination | Role | Governed by |
|---|---|---|
| **Your disk** | Memory files live here between sessions | You |
| **Anthropic** | Processes every Claude conversation, including data pulled in-session | [Anthropic's retention + training policy](https://claude.com/legal) |
| **Truthifi** | Holds the credentials that sync your accounts; serves the data to Claude on your behalf | [Truthifi's privacy policy](https://truthifi.com) |

The tradeoff: you get "no scraping, no CSVs, no login friction" in exchange for "one more service has your data." It's the same tradeoff you already made if you use Empower, Monarch, Copilot, or Mint — Truthifi is just focused on the MCP/Claude use case.

**This is the canonical statement of the privacy tradeoff.** The integration docs and setup guides link back here rather than restating it. If the framing in those docs ever drifts from this section, this section wins.

**If this tradeoff isn't acceptable to you**, use the CSV or scraper path instead — both keep data on your disk, with Anthropic as the only external recipient. See [`docs/integrations/README.md`](docs/integrations/README.md) for the picker.

### If you want zero-cloud AI for finance

homeCFO makes a deliberate tradeoff: it uses Claude, so Anthropic sees your conversations, and in exchange you get frontier-grade reasoning that local open-weights models can't yet match. If that tradeoff is wrong for you, check out **[NumbyAI](https://news.ycombinator.com/item?id=47377333)** — it runs a fully local LLM via Ollama, so nothing leaves your machine. You'll get less capable reasoning, but the privacy posture is genuinely end-to-end local. Pick the constraint that matters more for your situation.

## Threat model

The realistic risks when running personal finance tools locally:

1. **Accidentally committing real data to Git.** You fill out a template, forget to move it outside the repo, and push.
2. **Leaked credentials.** An API key, session cookie, or password ends up in a config file.
3. **Leaked account identifiers.** Statements, transaction exports, or "ending in 1234" strings slip into commits.
4. **Oversharing with Claude.** All Claude conversations transmit their contents to Anthropic for processing. Pasting a raw statement (with account numbers, routing numbers, statement images) puts those identifiers into a conversation log. Categorized transactions and round-number balances are a very different risk profile than raw statements — be deliberate about the difference.
5. **Trusting a hosted aggregator** (only if you opt into Truthifi or similar). You've delegated account read access to a third party. Their breach becomes your breach. Mitigation: read-only credentials where the institution supports it, strong unique password + 2FA on the aggregator account, and drop the integration if you stop using it.

Each is addressed below.

## Defenses in this repo

### 1. `.gitignore` blocks all data files

The repo ships with a `.gitignore` that refuses to track:

- `memory/` — any directory named `memory`
- `transactions_*.json`, `accounts_*.json`, `holdings_*.json`
- `*.csv`, `*.xlsx`, `*.ofx`, `*.qfx` (bank export formats)
- `.env`, `.env.*`, `credentials*`, `*.key`, `*.pem`
- `.chrome-profile/`, `.playwright-profile/` (scraper profiles with session cookies)

If you discover an oversight, open a PR against `.gitignore` — don't commit data and rely on deletion.

### 2. Pre-commit hooks scan every commit

Install them once:

```bash
pip install pre-commit
pre-commit install
```

The configured hooks:

- **gitleaks** — scans for common credential patterns (AWS keys, API tokens, private keys)
- **custom regexes** (`.gitleaks.toml`) — scans for account-number phrases ("Ending in 1234"), SSN patterns, routing numbers
- **detect-private-key** — blocks RSA/OpenSSH/PGP keys
- **check-added-large-files** — blocks any file >500KB (statements, full exports)

A hook failure blocks the commit. If you hit a false positive on a template or example, add the string to the allowlist in `.gitleaks.toml` — don't `--no-verify`.

### 3. Templates, not data

Files in `memory-templates/` end in `.template.md` and contain only placeholders (`{{YOUR_NAME}}`, `$XXX,XXX`). The Quickstart copies them *out* of the repo into your private data directory before you fill them in. That way there's no path back into Git.

### 4. The repo never handles credentials

No skill in this repo accepts a password or API key. The `update-financials` skill (if you use it) calls a scraper you supply locally; the scraper's credentials live in your OS keychain or a gitignored `.env`, never in the repo.

## Recommendations for users

- **Keep your filled-in memory files outside the repo.** The Quickstart puts them in `~/finance-data/` or similar. Don't move them back in "just to edit them."
- **Don't paste raw statements into Claude.** Let the `update-financials` skill read from your local JSON/CSV files instead; that way Claude works from the categorized summaries you control, not statement PDFs with account numbers in the header.
- **Remember conversations are transmitted.** Everything you type to Claude, and every file Claude reads on your behalf during a conversation, goes to Anthropic. That's how the model answers. Don't put anything in a memory file — or a Claude prompt — that you wouldn't be comfortable with Anthropic processing under their retention/training policy.
- **Review your `~/.claude/settings.json`** before sharing. If you added project-specific env vars or paths, strip anything personal before committing to a public dotfiles repo.
- **Rotate session cookies** if a scraper profile directory ends up somewhere public.

## Reporting a vulnerability

If you find a security issue in the repo — a regex gap in `.gitleaks.toml`, a template that asks for data it shouldn't, a skill that could exfiltrate files — please open a private security advisory on GitHub rather than a public issue. For anything cryptographic or credential-related, email the maintainer listed in `CODEOWNERS` (if present) or the address on the GitHub profile.

## What this repo does NOT protect against

- Malware on your machine. If your laptop is compromised, local files are compromised.
- Cloud-synced home directories. If `~/finance-data/` is inside a OneDrive/Dropbox/iCloud folder, your data is in that cloud. That's your choice, not this repo's.
- Anthropic processing your conversations. By design, Claude Code transmits file contents to Anthropic so the model can reason about them. Transcripts are logged per Anthropic's policy. Read it. This repo cannot change that — only your choice of what to put in the files changes it.
- Unencrypted disk. Turn on FileVault / BitLocker.
