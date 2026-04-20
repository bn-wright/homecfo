<!-- Handoff note for Brian. Delete this file before pushing the repo public. -->

# Status — 2026-04-19

## What's done

v0.1.0 is committed locally and tagged. Six logical commits:

```
8609920 ci: issue/PR templates and pre-commit + no-real-data CI workflow
faf6e8f docs(examples): quarterly-review walkthrough
bf2e23d feat(templates): MEMORY index + user_profile, investments, retirement_snapshot, spending_kb
623c4e0 feat(skills): finance-perspective, update-financials, fi-date-projection
a572d6a docs: README, PHILOSOPHY, SECURITY, CONTRIBUTING, CHANGELOG
4caec98 chore: repo scaffolding (gitignore, license, pre-commit, gitleaks)
```

Repo contents:

- **Top-level docs**: README, PHILOSOPHY, SECURITY, CONTRIBUTING, CHANGELOG, LICENSE
- **3 skills**: finance-perspective, update-financials, fi-date-projection (sanitized — no real numbers)
- **5 memory templates**: MEMORY, user_profile, investments, retirement_snapshot, spending_kb (all `.template.md` with `{{PLACEHOLDER}}` syntax)
- **1 example**: quarterly-review walkthrough using fictional "Sam Example"
- **GitHub config**: 3 issue templates, PR template, CI workflow (pre-commit + no-real-data sanity check)
- **Hygiene**: `.gitignore`, `.gitleaks.toml` (custom regexes for account numbers/SSN/routing), `.pre-commit-config.yaml`, `.markdownlint.json`

Commit author was set to `Brian Wright <brian@homecfo.local>`. If you want to change the email before pushing public, use:

```bash
git filter-repo --email-callback 'return email.replace(b"brian@homecfo.local", b"YOUR_PUBLIC_EMAIL")'
```

## What I deliberately did NOT do

1. **Did not push to GitHub.** No remote configured. You'll need to:
   - Create the GitHub repo (public)
   - `git remote add origin git@github.com:YOUR_USERNAME/homeCFO.git`
   - `git push -u origin main && git push --tags`

2. **Did not install pre-commit hooks locally.** I didn't want to mutate your Python env. To enable:
   ```bash
   cd /c/Users/Brian/code/homeCFO
   pip install pre-commit
   pre-commit install
   pre-commit run --all-files   # baseline check
   ```

3. **Did not include the Empower scraper.** Out of scope for v0.1 per our earlier decision (legal grey area). Keep that in a private repo.

4. **Did not modify your real memory files** in `C:\Users\Brian\FamilyFinance\memory\`. All work was in `C:\Users\Brian\code\homeCFO\` only. Your live data is untouched.

5. **Did not delete this STATUS.md.** It's not committed (run `git status` — you'll see it untracked). Delete before going public.

## Permission allowlist I'm running under

You added the snippet to `~/.claude/settings.json`. Working as expected — no prompts during this session. The `defaultMode: acceptEdits` saved a lot of round-trips on the file writes.

## Open questions for you when you're back

1. **Repo name**: confirm `homeCFO` (camelCase) or prefer `home-cfo`/`homecfo`? Current commits use `homeCFO`.
2. **GitHub username/org**: I left `YOUR_USERNAME` placeholder in README's clone command. Want me to find/replace once you decide?
3. **Public email for commits**: see filter-repo command above if you don't want `brian@homecfo.local` in the public history.
4. **First issues**: want me to draft 3-5 starter issues (e.g., "add Linux Quickstart", "add equity_grants template", "add tax-loss-harvest skill") so the repo doesn't look dead on day one?
5. **Launch post**: want a draft HN/Show HN/blog announcement?

## Suggested next moves

In rough priority order:

1. Read the README and PHILOSOPHY end-to-end and tell me what to sharpen
2. Decide repo name + push to GitHub
3. Run `pre-commit install` and `pre-commit run --all-files` to catch any lint
4. Tweak skill descriptions if any feel off
5. Delete this STATUS.md, then announce

## Time spent

Roughly 25 minutes of tool calls after you left. The bulk was: writing the three skills (each is ~70 lines of careful prose, not boilerplate), writing the five templates (each thoughtful enough to be useful, not just shape), and the philosophy/security docs.

Repo is ready to go public the moment you decide on the GitHub username and the email-rewrite question.
