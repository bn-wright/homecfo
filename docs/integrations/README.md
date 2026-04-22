# Ingestion paths

homeCFO is data-agnostic — you pick how your transactions and balances get into memory. Two supported paths:

| Path | Setup time | Privacy | Best for |
|---|---|---|---|
| **[Truthifi MCP](truthifi.md)** | ~10 min | Data leaves your machine to Truthifi (hosted aggregator) | Non-technical users, households that already use an aggregator, people who don't want to fuss with CSV exports |
| **Bring-your-own CSV** | ~2 min | Stays on your machine | Anyone whose bank/brokerage offers CSV export and who doesn't mind doing it quarterly |

## Which should I pick?

**Pick Truthifi MCP if:**

- You want to just ask Claude "how are we doing?" without first exporting anything
- You're already comfortable with your data going to an aggregator (Empower, Monarch, Copilot, Mint all do this too)
- You don't want to write or run any code

**Pick BYO CSV if:**

- You want zero third-party data sharing beyond Anthropic itself
- Once-a-quarter updates are fine
- You already download statements

You can mix and match — e.g. use Truthifi for balances but keep a local CSV for older transactions. homeCFO reads whatever is present.

> **A note on scrapers.** Earlier drafts of this doc recommended a "bring-your-own scraper" path (Selenium/Playwright against bank portals). That was removed: most US bank/brokerage TOS prohibit automated browsing of their customer portals, and we don't want to push users toward a category of tool that can get accounts locked. If you already maintain one in your own private repo for personal use, the `update-financials` skill will still read its JSON output — but the path isn't documented or recommended here.

## Privacy note — both paths

Both paths send the data Claude reads into the conversation to Anthropic — that's how Claude works, and it's not avoidable with this tool. The difference is **what else** happens to the data: BYO CSV keeps everything else on your disk; Truthifi adds a hosted aggregator as a third destination.

**Canonical framing is in [SECURITY.md](../../SECURITY.md).** The "If you opt into a hosted MCP aggregator" section there is the source of truth for the Truthifi tradeoff; this page just points at it.
