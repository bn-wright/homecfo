# Ingestion paths

homeCFO is data-agnostic — you pick how your transactions and balances get into memory. Three paths, ordered by setup effort:

| Path | Setup time | Privacy | Best for |
|---|---|---|---|
| **[Truthifi MCP](truthifi.md)** | ~10 min | Data leaves your machine to Truthifi (hosted aggregator) | Non-technical users, households that already use an aggregator, people who don't want to run a scraper |
| **Bring-your-own CSV** | ~2 min | Stays on your machine | Anyone whose bank/brokerage offers CSV export and who doesn't mind doing it quarterly |
| **Bring-your-own scraper** | ~30 min + Python | Stays on your machine | Power users who want fresh data on demand and are comfortable with Selenium/Playwright |

## Which should I pick?

**Pick Truthifi MCP if:**

- You want to just ask Claude "how are we doing?" without first exporting anything
- You're already comfortable with your data going to an aggregator (Empower, Monarch, Copilot, Mint all do this too)
- You don't want to write or run any code

**Pick BYO CSV if:**

- You want zero third-party data sharing beyond Anthropic itself
- Once-a-quarter updates are fine
- You already download statements

**Pick BYO scraper if:**

- You want daily-fresh data, zero third-party sharing
- You're comfortable fixing a scraper when a bank changes their login page
- You already have one or are happy to write one

You can mix and match — e.g. use Truthifi for balances but keep a local CSV for transactions. homeCFO reads whatever is present.

## Privacy note — all paths

All three paths send the data Claude reads into the conversation to Anthropic — that's how Claude works, and it's not avoidable with this tool. The difference is **what else** happens to the data: BYO CSV/scraper keeps everything else on your disk; Truthifi adds a hosted aggregator as a third destination.

**Canonical framing is in [SECURITY.md](../../SECURITY.md).** The "If you opt into a hosted MCP aggregator" section there is the source of truth for the Truthifi tradeoff; this page just points at it.
