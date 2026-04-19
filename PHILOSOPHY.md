# Philosophy

## The Operator Mindset

A growth-stage startup doesn't run its finances by occasionally glancing at the bank balance. It has a CFO. The CFO knows the runway down to the day, knows which expenses move the needle and which are noise, knows when a number is an emergency and when it's just normal variance.

Most households — including high-earning ones — run their finances the opposite way. They log into a banking app once a week, feel a vague sense of unease or relief, and close the tab. Big decisions get made on instinct. Small decisions get agonized over. Long-term goals stay fuzzy abstractions.

**homeCFO is the argument that a household with $1M+ in net worth should be operated, not vibed.**

## Why Now

Three things became true in the last 18 months that made this approach possible:

1. **LLMs got good enough to be a competent analyst.** Claude can read a hundred pages of memory files, understand a household's full financial picture, and give context-aware answers in seconds. This wasn't possible in 2023.
2. **Local-first tooling matured.** Claude Code, MCPs, and skills give you a real workflow that runs on your machine without sending data to anyone.
3. **The generic finance-app market commoditized.** Every dollar feature has been built ten times. The remaining differentiation is *insight*, and insight requires context the apps don't have.

## The Operating Loop

| Cadence | Activity | Time |
|---------|----------|------|
| **Daily** | Nothing. Stop checking. Your savings are on autopilot. | 0 min |
| **Weekly** | Nothing. Really. | 0 min |
| **Monthly** | One question to Claude: *"Are we on pace this month?"* | 5 min |
| **Quarterly** | Structured review: net worth, FI projection, allocation drift, lessons | 30 min |
| **Annually** | Tax prep, contribution limits review, big-picture trajectory check | 2 hr |
| **Ad hoc** | When you make a big decision, ask Claude what it actually costs against your long-term goal | 2 min |

That's the entire system. Roughly **40 hours per year**. Compare that to the way most households run finances: zero structured time + constant low-grade anxiety.

## The Three Skills That Matter Most

Everything else is implementation detail. The repo's value lives in three operator habits:

### 1. Long-term framing for short-term concerns

Most financial worries are noise relative to the long-term picture. A $500 overspend feels significant in the moment. Against a portfolio that earns $11,000/month from compounding, it's invisible. The `finance-perspective` skill exists to do this math out loud, every time, so you stop wasting energy on noise.

This is the skill that probably matters most. The honest answer to "is this a big deal?" is usually "no, here's why, in numbers" — but you need the numbers in front of you to feel it.

### 2. Memory as the source of truth

Your financial situation isn't a row in a database. It's a story: who you are, what you earn, what you own, what you owe, what you're saving for, what rules you've decided to live by. That story lives in Markdown files Claude reads at the start of every session. When the story changes, you update the files. That's the entire knowledge management system.

This works because LLMs are fluent in unstructured text. Trying to fit a household's finances into a normalized schema is the wrong abstraction.

### 3. Honest math on the FI date

Most households don't actually know how close they are to financial independence. They have a vague target ("$3M? maybe?") and no projection that updates with reality. The `fi-date-projection` skill closes the loop: feed it your current portfolio and contribution rate, get a date back, see how each decision moves it.

When you can see the FI date move in response to choices, you make different choices.

## Anti-Patterns This Repo Rejects

- **Daily portfolio checking.** It's noise. Your portfolio doesn't care if you watch it. Watching it makes you trade more, which makes you poorer.
- **Spreadsheet maximalism.** A spreadsheet that's always out of date is worse than no spreadsheet. Memory files are easier to keep current because they're just words.
- **Optimizing micro-expenses.** Whether you spend $4 or $6 on coffee will not move your retirement date by a single day. Whether your spouse re-enters the workforce in 2029 vs 2031 might shift it by years. Spend your attention proportionally.
- **Confusing income for wealth.** A high salary is the input. The output is whether your savings rate is locked in. The repo encourages you to set up payroll-deduction savings and then ignore the inflows.
- **Treating personal finance as identity.** This isn't about being a "saver" or a "spender." It's about whether the math works.

## What Operating Your Family's Finances Looks Like (After 6 Months)

You'll notice you check accounts less, not more. You'll have one or two strong opinions about what's actually limiting your timeline (it's almost never the latte). You'll be able to answer "how are we doing?" in 30 seconds instead of avoiding the question. Big decisions will feel less heavy because you'll know exactly what they cost.

That's the entire promise.

## Inspirations & Related Reading

- *Die With Zero* — Bill Perkins. The argument that optimizing for "as much as possible" is the wrong objective.
- *The Psychology of Money* — Morgan Housel. Why behavior matters more than math, and how to design systems that survive your own behavior.
- *Just Keep Buying* — Nick Maggiulli. The case for consistent automated investing as the dominant strategy.
- The FIRE community generally (Mr. Money Mustache, ChooseFI) — for the muscle of thinking in years and decades.

This repo is what happens when you take those ideas seriously and ask "what would it look like to actually operate against them, with an LLM as the analyst?"
