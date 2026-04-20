# Contributing to homeCFO

Thanks for considering a contribution. The repo is small on purpose, but new skills, better templates, and bug reports are all welcome.

## The hard rule

**No PR may contain real financial data.** Not your data, not example data, not "anonymized" data that's still recognizably someone's. Templates use placeholders. Examples use round, obviously-fake numbers (`$100,000 portfolio`, `Brian Example`, etc.).

If you're unsure whether something counts as "real data," it does. Use placeholders.

## What's in scope

- New Claude skills under `skills/` that match the repo's philosophy (insight-oriented, operator-mindset, no daily-checking pattern)
- Improvements to existing skills (clearer prompts, better edge-case handling)
- Memory templates for situations not yet covered (e.g., `business_income.template.md`, `equity_grants.template.md`)
- Documentation: clarifying the philosophy, fixing typos, expanding the Quickstart for other OSes
- Pre-commit / `.gitleaks.toml` improvements that catch more leak patterns

## What's out of scope

- Anything that sends data to a third party
- Anything that requires an account, API key, or paid service to function
- "Premium" tiers, affiliate links, referral codes
- Scrapers for specific banks/brokerages (legal grey area — keep those in your own private repo)
- Specific investment recommendations or "this asset is good" content (this is a tooling repo, not advice)

## Workflow

1. **Open an issue first** for non-trivial changes. A new skill or a template overhaul deserves a quick discussion before you write 500 lines.
2. **Fork and branch.** Branch names: `feat/<short-name>`, `fix/<short-name>`, `docs/<short-name>`.
3. **Install pre-commit hooks** before your first commit:

   ```bash
   pip install pre-commit
   pre-commit install
   ```

4. **Make the change.** Keep PRs focused — one skill or one fix per PR.
5. **Update the changelog.** Add an entry to `CHANGELOG.md` under `## [Unreleased]`.
6. **Open the PR.** Use the PR template. Tick the "no real financial data" checkbox honestly.

## Skill conventions

A new skill lives in `skills/<skill-name>/SKILL.md` with YAML frontmatter:

```yaml
---
name: skill-name
description: One sentence on when this triggers. Be specific about user phrases.
---
```

Body sections to include:

- **When to use** — concrete trigger phrases and contexts
- **What this skill does** — the actual workflow
- **Inputs expected** — which memory files it reads
- **Output format** — what the user sees back

Keep skills under ~300 lines. If yours is longer, split bundled references into `skills/<name>/references/`.

## Template conventions

Templates live in `memory-templates/<name>.template.md`. They:

- Use `{{PLACEHOLDER}}` for fields the user fills in
- Include realistic-looking but fake examples in comments
- Open with a short comment block explaining what the file is for and which skills read it

## Code of conduct

Be civil, be specific, assume good faith. No harassment, no discrimination. The maintainers reserve the right to close PRs or block users who violate this.

## Licensing

By contributing, you agree your contribution is licensed under the MIT License (same as the rest of the repo).
