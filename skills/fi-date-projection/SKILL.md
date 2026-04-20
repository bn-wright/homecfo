---
name: fi-date-projection
description: Recalculate the user's financial-independence (FI) date based on current portfolio, contribution rate, and target. Use whenever the user asks "when can I retire", "am I on track", "what's my FI date", "how does X change my retirement timeline", or any question about the trajectory toward financial independence.
---

# FI Date Projection

The user's financial-independence date is the most important number in the household. It compresses a hundred decisions into one fact. This skill keeps it honest and current.

## When to use

- "When can I retire?"
- "Am I on track?"
- "What's my FI date?"
- "How does [decision X] move my timeline?"
- "If I save an extra $Y/month, how much sooner?"
- "If the market returns 5% instead of 7%, what happens?"

Also use as part of the quarterly review (see `examples/quarterly-review-walkthrough.md`).

## What this skill does

1. **Load inputs from memory:**
   - Current portfolio value (from `investments.md`)
   - Annual contribution rate (from `user_profile.md` or `retirement_snapshot.md`)
   - FI target amount (from `retirement_snapshot.md`)
   - Expected real return assumption (default 7% nominal / ~5% real, but use the user's assumption if specified)
2. **Project forward.** Use the standard future-value formula:

   ```text
   FV = P × (1 + r)^t + C × ((1 + r)^t − 1) / r
   ```

   where P = current portfolio, C = annual contribution, r = annual return, t = years.
   Solve for `t` such that `FV ≥ FI target`.
3. **Express the answer as a date.** Add `t` years to today.
4. **Show sensitivity.** Always include at least three scenarios: pessimistic (5%), base (7%), optimistic (9%). Optionally show contribution-rate sensitivity (±$10k/yr).
5. **Update `retirement_snapshot.md`** with the new projection if the user asks you to persist it.

## Output format

```markdown
## FI Projection — {{DATE}}

**Inputs:**
- Current portfolio: $X
- Annual contributions: $Y (locked-in via payroll)
- FI target: $Z
- Assumed real return: R%

**Base case (R%):** FI on YYYY-MM-DD (age N, in M years)

**Sensitivity:**
| Return | FI Date | Age |
|--------|---------|-----|
| 5%     | ...     | ... |
| 7%     | ...     | ... |
| 9%     | ...     | ... |

**What would move this most:**
- [Highest-leverage variable, e.g., "spouse re-entering workforce in 2029 vs 2031 = ~14 months"]
```

## Important framing notes

- **Don't conflate nominal and real returns.** If the FI target is in today's dollars, use real returns. If it's nominal, use nominal. State the assumption explicitly.
- **Don't double-count Social Security or pensions** unless they're already in the FI target.
- **Don't project beyond 25 years** with high precision — the cone of uncertainty is too wide. Flag it: "Projections beyond 2050 are illustrative only."
- **The FI date moves slowly.** Most one-time events change it by days or weeks, not years. Use the `finance-perspective` skill to communicate this when the user is anxious about a small number.

## Anti-patterns

- ❌ Inventing a return assumption without telling the user
- ❌ Hiding the math — always show inputs and at least one sensitivity
- ❌ Treating the projection as a guarantee
- ✅ Honest math, clear assumptions, sensitivity bands
