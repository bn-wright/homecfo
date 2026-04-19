# Security & Privacy

## The promise

Your financial data never leaves your machine. This repo contains no data — only templates, skills, and documentation. The whole design is built around keeping your numbers on your local disk and out of Git.

## Threat model

The realistic risks when running personal finance tools locally:

1. **Accidentally committing real data to Git.** You fill out a template, forget to move it outside the repo, and push.
2. **Leaked credentials.** An API key, session cookie, or password ends up in a config file.
3. **Leaked account identifiers.** Statements, transaction exports, or "ending in 1234" strings slip into commits.
4. **Oversharing with Claude.** You paste raw bank statements into a conversation that syncs to Anthropic servers.

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
- **Don't paste raw statements into Claude.** Let the `update-financials` skill read from your local JSON files instead; Claude sees only the summarized output.
- **Review your `~/.claude/settings.json`** before sharing. If you added project-specific env vars or paths, strip anything personal before committing to a public dotfiles repo.
- **Rotate session cookies** if a scraper profile directory ends up somewhere public.

## Reporting a vulnerability

If you find a security issue in the repo — a regex gap in `.gitleaks.toml`, a template that asks for data it shouldn't, a skill that could exfiltrate files — please open a private security advisory on GitHub rather than a public issue. For anything cryptographic or credential-related, email the maintainer listed in `CODEOWNERS` (if present) or the address on the GitHub profile.

## What this repo does NOT protect against

- Malware on your machine. If your laptop is compromised, local files are compromised.
- Cloud-synced home directories. If `~/finance-data/` is inside a OneDrive/Dropbox/iCloud folder, your data is in that cloud. That's your choice, not this repo's.
- Your own mistakes in Claude Code conversations. Transcripts may be logged by Anthropic per their policy. Read it.
- Unencrypted disk. Turn on FileVault / BitLocker.
