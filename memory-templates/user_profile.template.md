<!--
Household identity, income, accounts, and recurring expenses.
Fill in real values, then keep this file outside the homeCFO repo.
Replace every {{PLACEHOLDER}} with your real value or delete the section.
-->

# User Profile

## Identity
- **Primary:** {{YOUR_NAME}}, age {{AGE}}, {{LOCATION}}
- **Spouse/partner:** {{NAME}}, age {{AGE}} (if applicable)
- **Dependents:** {{N_KIDS}} children, ages {{AGES}}
- **Filing status:** {{married_jointly | single | head_of_household}}

## Household income (annual, gross)
- Primary salary: ${{XXX,XXX}}
- Spouse salary: ${{XXX,XXX}}
- Bonus / variable comp (expected): ${{XX,XXX}}
- RSU vest value (expected this year): ${{XX,XXX}}
- Rental net income: ${{XX,XXX}}
- Other: ${{X,XXX}}

**Total expected gross income (this year):** ${{XXX,XXX}}

## Accounts (bank/brokerage, NOT account numbers)

| Institution | Account type | Purpose | Approx. balance |
|-------------|--------------|---------|-----------------|
| {{Bank A}}  | Checking     | Daily ops | ${{X,XXX}} |
| {{Bank A}}  | Savings      | Emergency fund | ${{XX,XXX}} |
| {{Broker B}} | Taxable brokerage | Long-term | ${{XXX,XXX}} |
| {{Broker B}} | Roth IRA — self | Retirement | ${{XX,XXX}} |
| {{Broker B}} | Roth IRA — spouse | Retirement | ${{XX,XXX}} |
| {{401k provider}} | 401(k) | Retirement | ${{XXX,XXX}} |

> **Note:** never put real account numbers, routing numbers, or "Ending in 1234" strings here. The pre-commit hooks will block those, but it's better not to type them at all.

## Properties (if applicable)

### {{Address city/state — e.g., "Primary residence, Austin TX"}}
- Estimated value: ${{XXX,XXX}}
- Mortgage balance: ${{XXX,XXX}}
- Rate: {{X.XX}}%
- Monthly P&I: ${{X,XXX}}
- Other monthly (taxes, insurance, HOA): ${{XXX}}

### {{Rental — city/state}} (repeat per property)
- Estimated value / mortgage / rate / payment / monthly rent / net cash flow

## Recurring monthly expenses (baseline, in dollars)

| Category | Amount | Notes |
|----------|--------|-------|
| Mortgage/rent | $X,XXX | |
| Utilities | $XXX | |
| Groceries | $X,XXX | |
| Insurance (health/auto/home) | $XXX | |
| Childcare/school | $X,XXX | |
| Transportation (gas, maint.) | $XXX | |
| Subscriptions | $XXX | |
| Discretionary (avg) | $X,XXX | |

**Baseline monthly burn:** ${{X,XXX}}

## Hard rules to never break
<!-- Things Claude should never get wrong about your household. -->
- Do NOT count {{X}} as net-new income (e.g., ESPP payroll return, expense reimbursements)
- Locked-in savings target: ${{XX,XXX}}/yr via payroll deductions
- Monthly cash flow buffer: ~${{XXX}}–${{X,XXX}}
- {{Any other invariants — e.g., "max contributions to 401k each year, no exceptions"}}
