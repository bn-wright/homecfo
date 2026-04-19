# Example: Quarterly Review Walkthrough

This is what a 30-minute quarterly review looks like with homeCFO. The user, "Sam Example," is a fictional household with round numbers — none of this is real data.

## The setup

- **Sam Example**, 38, software engineer, married, 2 kids
- Household income: $300k gross
- Investable assets: $1.0M
- Target: retire at 50 with $2.5M (today's dollars)
- Saves $50k/yr via payroll deduction
- Memory files at `~/finance-data/`

## The conversation

> **Sam:** It's the start of Q3. Run me through the quarterly review.

Claude loads `MEMORY.md`, then `user_profile.md`, `investments.md`, `retirement_snapshot.md`, and `spending_kb.md`.

> **Claude:**
>
> ## Q2 Review — 2026-07-01
>
> ### Net worth change
> - End of Q1: $1,000,000
> - End of Q2: $1,047,500 (+$47,500, +4.75%)
> - Source of change: contributions $12,500 + market gain $35,000
>
> ### Allocation drift
> | Class | Target | Actual | Drift |
> |-------|--------|--------|-------|
> | US equity | 60% | 64% | +4 |
> | Intl equity | 20% | 18% | -2 |
> | Bonds | 15% | 13% | -2 |
> | Cash | 5% | 5% | 0 |
>
> Recommendation: rebalance ~$40k from US equity into intl + bonds at next contribution.
>
> ### Spending vs baseline
> - Baseline monthly burn: $11,000
> - Q2 actual avg: $11,400 (+3.6%)
> - Driver: April vacation (+$2,800), otherwise within norms
>
> ### FI date update
> - Last quarter projection: 2038-09 (age 50, ~12.2 yrs away)
> - This quarter: 2038-08 (age 50, ~12.1 yrs away)
> - Net change: pulled in by ~1 month due to Q2 market gain
>
> ### What to discuss
> 1. Allocation rebalance (above) — small but worth doing
> 2. Vacation overshoot is noise, no action needed
> 3. Cash position is at target; no excess to deploy
>
> ### Anything I should know about Q3?

> **Sam:** We're considering a $30k bathroom remodel. Worth it against the timeline?

Claude invokes `finance-perspective`:

> **Verdict:** Absorbable.
>
> **The math:**
> - Concern: $30,000 (likely from cash, not portfolio sale)
> - Monthly portfolio drift: ~$6,100 (concern is ~5 months of drift)
> - Locked-in savings: $50k/yr (concern is 60% of one year's contributions, but you're not skipping savings)
> - FI date impact: ~3 months
>
> **Why:** A $30k cash purchase pushes the FI date out by ~3 months. That's real but not severe — you'd retire in October instead of August 2038. If the bathroom is a multi-decade quality-of-life upgrade, the math supports it.
>
> **Action:** Decide on quality-of-life grounds, not financial. The numbers work either way.

> **Sam:** Done. Update the snapshot.

Claude updates `retirement_snapshot.md` with the new projection and notes the planned bathroom expense.

## What just happened

In ~10 minutes of conversation, Sam:

- Got a clean Q2 close
- Identified a small rebalancing action
- Made a $30k decision with eyes open about the cost
- Updated the snapshot for next quarter

No spreadsheet was opened. No app was checked. The memory files now reflect the current state.

## Your version

Your numbers will be different. Your skills will trigger differently. But the cadence is the same: 30 minutes, four times a year, and the household runs.
