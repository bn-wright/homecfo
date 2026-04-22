<!--
Pre-parsed transaction data and category baselines.
The `sync-local` skill (renamed from `update-financials` in v0.2) appends to this file when it ingests new exports.
Keep raw transaction CSVs/JSONs OUT of the repo (they are gitignored anyway).
-->

# Spending Knowledge Base — last updated {{YYYY-MM-DD}}

## Category taxonomy

These are the categories Claude uses to bucket transactions. Add or rename as needed, but don't fork — keeping the list short keeps analysis useful.

- **Housing** — mortgage/rent, property tax, HOA, home repair
- **Utilities** — electric, gas, water, internet, phone
- **Groceries** — supermarket, butcher, farmer's market
- **Dining** — restaurants, coffee, food delivery
- **Transportation** — gas, parking, transit, auto maintenance, ride-share
- **Insurance** — health, auto, home, life, umbrella
- **Healthcare** — copays, prescriptions, dental, vision
- **Childcare/Education** — daycare, tuition, lessons, school supplies
- **Subscriptions** — streaming, software, gym, memberships
- **Discretionary** — shopping, entertainment, hobbies, gifts
- **Travel** — flights, hotels, rentals
- **Taxes** — quarterly estimates, tax-prep fees, IRS payments
- **Income** — net deposits (paycheck, rental, refunds)
- **Transfers** — between own accounts (exclude from spending)
- **Investments** — contributions to brokerage/IRA/401k

## Monthly category norms ({{YYYY}} baseline)

| Category | Avg monthly | Notes |
|----------|-------------|-------|
| Housing  | $X,XXX | |
| Utilities | $XXX | |
| Groceries | $X,XXX | |
| Dining | $XXX | |
| Transportation | $XXX | |
| Insurance | $XXX | |
| Childcare/Edu | $X,XXX | |
| Subscriptions | $XXX | |
| Discretionary | $X,XXX | |
| Travel | $XXX | bursty, avg over 12mo |
| Taxes | $X,XXX | bursty, see annual |

**Baseline monthly burn (excluding investments):** ${{X,XXX}}

## YTD summary ({{YYYY}})

- **Months covered:** {{N}}
- **Transactions ingested:** {{N}}
- **Total inflows:** ${{XXX,XXX}}
- **Total outflows:** ${{XXX,XXX}}
- **Net savings:** ${{XX,XXX}}
- **Notable anomalies this year:** {{e.g., "April: large IRS payment", "August: HVAC replacement"}}

## Reconciliation rules
<!-- These rules prevent Claude from miscounting income/spending -->
- Computershare deposits → ESPP payroll return, NOT income
- Concur reimbursements → expense reimbursement, NOT income
- Transfers between own accounts → excluded from spending
- {{Add household-specific exclusions here}}

## Notes

- Last sync source: {{path/to/transactions_YYYY.json}}
- Last sync date: {{YYYY-MM-DD}}
