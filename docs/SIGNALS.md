# Signals

Three custom signals, each mapped to one part of the buying reason.

| Signal | Question it answers | Max | Source |
|---|---|---|---|
| **S1 · EOL / legacy tech exposure** | Does this company run AngularJS 1.x or Vue 2 in its *own* code? | 50 | A human-verified job-post quote, plus Claygent corroboration |
| **S2 · Modernization / migration timing** | Is a migration happening now, which is when NES is easiest to justify? | 30 | Claygent: job posts, engineering pages, roadmap language |
| **S3 · Security / compliance pressure** | Does someone outside engineering force the issue? | 20 | Claygent: trust centers, attestations, licenses, sector |

S1 carries the most weight because without the EOL framework there is nothing to sell. S2 and S3 decide *when* and *how hard* to push.

## S1 levels

| Level | Meaning | Points |
|---|---|---|
| EXACT_CORROBORATED | The quote places AngularJS or Vue 2 in the employer's own estate, and an independent second source confirms it | 50 |
| EXACT_SINGLE | The same, but only one source | 40 |
| EXPERIENCE_ONLY | Listed only as candidate experience, a nice-to-have or an "or" alternative | 25 |
| AMBIGUOUS_VERSION | "Angular" or "Vue" with no version given | 20 |
| PLATFORM_FALSE_POSITIVE | AngularJS belongs to a vendor platform (ServiceNow) | 0 |
| NONE | | 0 |

Staleness is part of S1. Evidence from 121–365 days ago is capped at 25 points, and anything older than 365 days scores 10.

## S2 categories

| Category | Points |
|---|---|
| ACTIVE: migration off the EOL framework is underway now | 30 |
| LIKELY: a modernization program or new-stack hiring next to the legacy app | 20 |
| WEAK: only generic "modernize" language | 10 |
| COMPLETED: the migration is already finished, so the window has passed | 0 |
| NONE | 0 |

## S3 categories

| Category | Points |
|---|---|
| ATTESTED: holds or must meet SOC 2, ISO 27001, PCI DSS, HIPAA, FedRAMP/StateRAMP, IRAP | 20 |
| REGULATED: regulated sector, but no attestation found | 15 |
| ENTERPRISE_OR_GOV_BUYERS: sells to buyers who run security reviews | 10 |
| GENERAL | 5 |
| NONE | 0 |

## Where the evidence comes from, and why it is built this way

1. **Intake.** Clay Find Jobs with keyword "AngularJS" in the US and UK found 480 posts; the trial shows 50 of them. Targeted Vue 2 and non-US searches were added on top. Job-posting intake is free in Clay.
2. **A human reads the post.** Find Jobs records hold a title, location, date and URL, but **no description text**, so a keyword hit is not evidence. Every S1 quote in the table was read on the live posting and copied word for word.
3. **Claygent corroborates.** The agent receives the quote and is asked to classify and find independent support. It returns *categories and URLs*, never points ([prompt](../clay/claygent_prompt.md)).
4. **Formulas score** ([formulas](../clay/formulas.md)).
5. **A human QA's every HIGH account.** The `S1 QA Override` column feeds `S1 Final`, so a correction re-scores the row automatically.

## What a signal does *not* prove

A job post that names AngularJS shows that someone at the company works with it. It does not show:

- how much of the estate runs on it
- whether that code is in production
- whether there is budget
- whether anyone has noticed the risk

Every `Why This Score` cell ends with that sentence for this reason.

## Passive research only

No scanning, no probing, no logins. The prompt forbids it, and every source in the table is a public web page. Personal profiles of employees are excluded as sources (see [FAILURE_AND_LEARNINGS.md](FAILURE_AND_LEARNINGS.md)).
