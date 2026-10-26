# Scoring

```
Signal Score (1–100) = S1 (0–50) + S2 (0–30) + S3 (0–20)        floor of 1 for any scored row
```

A score is only computed for rows that passed the gate, are not suppressed, and have a human-verified quote.

## Thresholds

| Tier | Rule |
|---|---|
| **HIGH** | Score 75–100 and S1 ≥ 20 |
| **MEDIUM** | Score 50–74, or a score ≥ 75 capped because S1 < 20 |
| **LOW** | Score 1–49, or no verified EOL evidence |
| **REVIEW** | Contradictory evidence, or the enriched record is a different company |
| **EXCLUDED** | Public HeroDevs customer, or failed the ICP gate |

## Guardrails

| Requirement | How it is enforced |
|---|---|
| S1 < 20 caps the tier at Medium or lower | Written into the Tier formula. With 50/30/20 weights it can never fire, because S2 + S3 max out at 50 and reaching 75 needs S1 ≥ 25. It stays so the rule survives a weight change. |
| Existing customer → EXCLUDE | Suppression lookup → `Suppressed = YES` → EXCLUDED, before anything else is checked |
| Stale evidence lowers confidence | Job post > 120 days old: S1 capped at 25, and confidence cannot be HIGH. Older than 365 days: S1 = 10 and confidence LOW |
| Contradictory evidence → REVIEW | `Contradiction = YES` → Tier REVIEW. Medirion: the imported domain turned out to be a parked page |
| Completed migration lowers S2 | COMPLETED = 0 points. The window has closed, even though the account may still be worth knowing |

## Evidence Confidence is not part of the score

The score says how strong the opportunity looks. Confidence says how much to trust that reading.

| Confidence | Rule |
|---|---|
| HIGH | EXACT_CORROBORATED, ≤ 120 days old, entity MATCH, no contradiction |
| LOW | Contradiction, entity mismatch or not found, > 365 days old, or S1 weaker than EXACT |
| MEDIUM | everything else |

This is why a row can be **HIGH tier with MEDIUM confidence**: strong-looking signals with only one source behind them. Five of the six HIGH accounts are exactly that, which is the honest state of the evidence.

## Results (2026-10-26, after QA)

| # | Account | S1 | S2 | S3 | Score | Tier | Confidence |
|---|---|---|---|---|---|---|---|
| 1 | Geographic Solutions | 50 EXACT_CORROBORATED | 20 LIKELY | 20 ATTESTED | 90 | HIGH | HIGH |
| 2 | H-E-B | 40 EXACT_SINGLE | 30 ACTIVE | 20 ATTESTED | 90 | HIGH | MEDIUM |
| 3 | dormakaba | 50 EXACT_CORROBORATED | 20 LIKELY | 20 ATTESTED | 90 | HIGH | MEDIUM (entity PARTIAL) |
| 4 | Elective | 40 EXACT_SINGLE | 30 ACTIVE | 15 REGULATED | 85 | HIGH | MEDIUM (QA hold: size) |
| 5 | Dataiku | 40 EXACT_SINGLE (QA) | 20 LIKELY | 20 ATTESTED | 80 | HIGH | MEDIUM |
| 6 | DATAMYTE | 40 EXACT_SINGLE (QA) | 20 LIKELY | 20 ATTESTED | 80 | HIGH | MEDIUM |
| 7 | NCR Atleos | 25 EXPERIENCE_ONLY | 20 | 20 | 65 | MEDIUM | LOW |
| 8 | Dover Fueling Solutions | 25 | 10 | 20 | 55 | MEDIUM | LOW |
| 9 | APCON | 25 | 20 | 10 | 55 | MEDIUM | LOW |
| 10 | Medirion | 25 | 10 | 15 | 50 | REVIEW | LOW |
| 11–13 | Engage People, Autodesk, Transport for NSW | 25 | 0 | 20 | 45 | LOW | LOW |
| 14 | Schwarz Digits | 0 PLATFORM_FALSE_POSITIVE | 0 | 10 | 10 | LOW | LOW |
| — | Sundayy | not researched (enriched to a different company) | | | | LOW | — |
| — | 8 IT services, staffing and professional services firms | ICP gate FAIL | | | | EXCLUDED | — |
| — | Sisense, Statista, Keelvar (controls) | public customers | | | | EXCLUDED | — |

All rows, with evidence quotes: [sample/public-safe-sample.csv](../sample/public-safe-sample.csv).

## The weights are a judgement, not a measurement

50/30/20 encodes one belief: without the EOL framework there is nothing to sell (S1), and timing and pressure decide when to push (S2, S3). Nothing in this build measured that. Here is how much the conclusions depend on it:

| Weights (S1/S2/S3) | HIGH set | What changes |
|---|---|---|
| 50/30/20 (built) | the 6 above | — |
| 60/25/15 | same 6 | order within HIGH changes |
| 40/40/20 | same 6 | H-E-B moves to the top |
| 62.5/37.5/0 (S3 dropped) | same 6 | Dover drops from MEDIUM to LOW |

The HIGH set is stable because S1 levels drive the tiers. The weak spot is **S3: 10 of 14 researched accounts came back ATTESTED**. In this sample S3 barely separates accounts, and it acts more like a constant. The fix is in [MEASUREMENT_PLAN.md](MEASUREMENT_PLAN.md): fit the weights to meeting and opportunity outcomes once there are enough of them.

## Why This Score never overclaims

Every scored row ends with: *"Shows public evidence of EOL tech; does not prove production use, budget or buying intent."* The text is built by a formula from the structured fields, so it cannot say more than the fields do.
