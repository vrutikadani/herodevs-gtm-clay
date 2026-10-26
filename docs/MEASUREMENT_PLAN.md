# Measurement plan

The score is a hypothesis. This plan is how it gets tested and recalibrated.

## What "working" means

| Question | Metric | Target to beat |
|---|---|---|
| Is HIGH really an EOL estate? | **Estate confirmation rate:** share of accounts whose first conversation confirms AngularJS 1.x or Vue 2 in production | HIGH ≥ 60%, and clearly above MEDIUM |
| Does tier predict interest? | Positive reply rate and meeting rate by tier | HIGH ≥ 2× MEDIUM |
| Does it predict revenue? | Opportunity rate and win rate by tier (lagging, about 2 quarters) | Monotonic: HIGH > MEDIUM > LOW |
| Is confidence calibrated? | Confirmation rate by Evidence Confidence | HIGH confidence > MEDIUM > LOW |
| Is the gate safe? | **Gate false-negative audit:** each month, a person reads 20 FAIL rows | ≤ 1 real buyer per 20 |
| Is it efficient? | Credits and human minutes per confirmed estate | Down quarter over quarter |

## Instrumentation

1. **Tier at the time of action.** Snapshot `Tier`, `Signal Score`, `Evidence Confidence`, `S1 Final` and `Scored At` into the CRM when an account is worked. Tiers change as evidence ages, so reports must use the tier the seller saw, not today's.
2. **One outcome field per account:** `estate_confirmed` = yes / no / unknown, set by the seller after the first conversation. This is the label everything else depends on.
3. **QA overrides as data.** Each `S1 QA Override` is a free label for agent accuracy. Track agent vs. human agreement by S1 level.
4. **A holdout.** About 10% of HIGH accounts get no tier shown to the seller and are worked with the normal process. Without this, a higher HIGH meeting rate could just mean sellers try harder on HIGH.

## Recalibration

- **At about 100 outcomes:** fit a logistic regression of `estate_confirmed` on S1 level, S2 category, S3 category and evidence age. Replace 50/30/20 with points proportional to the fitted effects, and keep the guardrails as rules.
- **S3 first.** In the build sample, 10 of 14 researched accounts were ATTESTED, so S3 hardly separates accounts. Either weight it less, or split ATTESTED into "attestation covers the app that runs the EOL framework" and "company-wide attestation".
- **Staleness from data.** Replace the 120- and 365-day cut-offs with the evidence age where confirmation rates actually fall.
- **Agent reliability.** If agent-vs-human disagreement on S1 is above ~15%, send EXACT_CORROBORATED to REVIEW until a person confirms it.

## Cadence

| When | What |
|---|---|
| Weekly | New Find Jobs intake (free) → evidence read → gate → research the survivors |
| Monthly | Re-score evidence older than 120 days, run the gate false-negative audit, review the override log |
| Quarterly | Tier-vs-outcome report, holdout comparison, weight refit once n ≥ 100 |

## Guardrails on the measurement itself

- Do not judge the model on reply rates alone. Messaging quality confounds them, so estate confirmation comes first.
- Do not refit on fewer than about 100 labelled outcomes. Below that, stay with the stated judgement weights.
- Report confidence intervals. At n = 30 per tier, a 20-point gap in confirmation rate is noise.
