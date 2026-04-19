---
name: finance-perspective
description: Apply long-term financial perspective to short-term money concerns. Use whenever the user asks whether something "matters," "is a big deal," "should I worry about it," or asks about the long-term impact of a short-term financial event — overspending, a market dip, a surprise expense, a missed savings contribution, a one-time large purchase, or any moment where they sound anxious about a number that may be small relative to their long-term picture.
---

# Finance Perspective

The job of this skill is to do honest, quantitative reframing — never dismissive, never falsely reassuring. If something is actually a problem, say so. If it's noise relative to the long-term picture, show the math that makes it noise.

## When to use

Trigger phrases include but aren't limited to:

- "Is this a big deal?"
- "Does this matter?"
- "Should I be worried about [X]?"
- "How much will this hurt our retirement?"
- "I just spent $X on Y. What does that do to our timeline?"
- "The market dropped Z%. How bad is that?"

Also trigger when the user *sounds* anxious about a specific number, even if they don't ask the question directly. ("Ugh, we spent $4k on the vacation.")

## What this skill does

1. **Load the relevant memory files first.** The household's profile, investments, and retirement snapshot live in their data directory (path is in the user's CLAUDE.md). Read them before responding.
2. **Quantify the concern in three dimensions:**
   - **Vs. monthly portfolio drift.** A portfolio of $P at expected return r has monthly drift of `P × r / 12`. Compare the concern to one month's drift. (e.g., $2M × 7% / 12 ≈ $11,700/month.)
   - **Vs. annual locked-in savings.** If their savings are on payroll-deduction autopilot, the concern competes with one-time discretionary money, not with the savings rate.
   - **Vs. FI date sensitivity.** Roughly: how many days/weeks does this move the financial-independence date? Most one-time events move it by hours or days. Frame it that way.
3. **Give a verdict.** One of: "noise — here's why," "real but absorbable — here's why," "this matters — here's what to do."
4. **Do not moralize.** The user is not a bad person for spending money. The math is the math.

## Required reading

Before answering, load whichever of these exist in the user's memory directory:

- `MEMORY.md` (index)
- `investments.md` or `investments_*.md` (current portfolio)
- `retirement_snapshot.md` (FI target and timeline)
- `user_profile.md` (income, recurring expenses)

The user's CLAUDE.md tells you the absolute path.

## Output format

Keep it short. Aim for under 200 words unless the user asks for depth.

```
**Verdict:** [Noise / Absorbable / Worth attention]

**The math:**
- Concern: $X
- Monthly portfolio drift: $Y (concern is Z% of one month)
- Locked-in annual savings: $A (concern is B% of one year)
- FI date impact: ~N days (current FI date: YYYY-MM-DD)

**Why:** [1-2 sentence honest interpretation]

**Action:** [None / Specific small adjustment / Real change to consider]
```

## Examples

**Example 1 — overspend:**

> User: "We spent $2,000 on a couch. Bad?"
>
> Response:
> **Verdict:** Noise.
>
> **The math:**
> - Concern: $2,000
> - Monthly portfolio drift: ~$11,700 (concern is 17% of one month)
> - Locked-in savings: $60k/yr (concern is 3.3% of one year)
> - FI date impact: ~6 days
>
> **Why:** A one-time furniture purchase moves your FI date by less than a week. Your savings rate is unchanged because it's payroll-deducted.
>
> **Action:** None. Enjoy the couch.

**Example 2 — real concern:**

> User: "I'm thinking of taking a year off unpaid. Bad?"
>
> Response:
> **Verdict:** Worth attention.
>
> **The math:**
> - Lost contributions: $60k
> - Lost compounding on those contributions over 5 yrs: ~$24k
> - FI date impact: ~14 months
>
> **Why:** Skipping a year of savings isn't noise — it removes a contribution that would have compounded for the rest of your working life.
>
> **Action:** If the year off is worth it for non-financial reasons, take it knowing the cost. If not, consider a partial sabbatical or moving the timing later when contributions matter less.

## Anti-patterns

- ❌ "Don't worry about it!" — vague reassurance with no numbers
- ❌ "You should cut back on X to save Y" — moralizing
- ❌ Re-deriving the FI date from scratch when `retirement_snapshot.md` already has it
- ✅ Concrete math, brief verdict, no judgment
