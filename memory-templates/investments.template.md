<!--
Current investment portfolio: holdings, allocation, net worth.
Refresh this file via the `update-financials` skill or by hand quarterly.
-->

# Investments — as of {{YYYY-MM-DD}}

## Headline numbers

- **Net worth:** ${{X,XXX,XXX}}
- **Investable assets (excl. real estate equity):** ${{X,XXX,XXX}}
- **Real estate equity:** ${{XXX,XXX}}
- **Cash/short-term:** ${{XX,XXX}}

## Asset allocation

| Asset class | Value | % |
|-------------|-------|---|
| US equity | ${{XXX,XXX}} | XX% |
| International equity | ${{XXX,XXX}} | XX% |
| Bonds / fixed income | ${{XXX,XXX}} | XX% |
| Cash | ${{XX,XXX}} | XX% |
| Alternatives (REITs, crypto, etc.) | ${{XX,XXX}} | XX% |

**Target allocation:** {{e.g., 80/15/5 equity/bonds/cash}}
**Drift from target:** {{describe any rebalancing needed}}

## Holdings by account

### {{Account 1, e.g., Taxable brokerage}}
| Symbol | Shares | Value | Notes |
|--------|--------|-------|-------|
| VTI    | XXX    | $XX,XXX | |
| VXUS   | XXX    | $XX,XXX | |

### {{Account 2, e.g., 401(k)}}
| Symbol | Shares | Value | Notes |
|--------|--------|-------|-------|
| ...    | ...    | ...     | |

(Repeat per account)

## Concentrated positions
<!-- Anything over ~5% of investable assets, esp. employer stock -->
- {{TICKER}}: $XXX,XXX ({{X}}% of portfolio) — {{notes, e.g., "ESPP, vesting RSUs"}}

## Recent moves
<!-- Last quarter's notable trades, contributions, withdrawals -->
- {{YYYY-MM-DD}}: {{action}}

## Notes for Claude
- Update headline numbers if the underlying holding values shift more than ~3%
- Treat the {{X}} fund as effectively cash for projection purposes
- {{Any other quirks specific to your portfolio}}
