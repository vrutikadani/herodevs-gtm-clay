# Failures and learnings

Each entry below actually happened during this build, in the order it was found.

## The hardest one: corroboration that looked verified and wasn't

**What happened.** Claygent marked three accounts EXACT_CORROBORATED with a second-source URL each. Every automated check passed: the field was filled, the URL was well formed, and the agent reported its own confidence. In the first scoring pass, those three accounts were the only ones with HIGH confidence.

Then I opened every cited URL for every HIGH account (the QA pass):

| Account | Agent's second source | What it actually was |
|---|---|---|
| Geographic Solutions | A second Geographic Solutions job posting on Paylocity | ✅ Real, first-party and independent |
| Dataiku | `dataiku.com/engineering` | ❌ **A 404 page.** The AngularJS evidence was real (Dataiku's own Greenhouse posting lists it for several teams), but that is the same requisition as the intake, so it is not a second source |
| DATAMYTE | A posting on a job aggregator | ❌ **A mirror of what looks like the same requisition.** It is not independent |

**Impact.** **2 of the 3 HIGH-confidence calls were wrong.** Scores would have stayed in HIGH either way (90 → 80), so the tier did not change. What changed was the confidence label, and confidence is exactly what a seller uses to decide whether to call.

**Why it is the hardest.** Bad AI output usually looks bad: empty, malformed or obviously off topic. This looked fine. The URL existed in the response, the category matched the definition, and the agent's reasoning was fluent. A schema check, a "URL not empty" check, or another LLM asked "is this corroborated?" would most likely all have passed it.

**Fix (built):**

1. A human `S1 QA Override` column feeds `S1 Final`, which feeds the score. Overrides re-score the row automatically and appear as "(QA override)" in `Why This Score`.
2. A `QA Note (human)` column next to the score says what was checked.
3. A rule: **every HIGH account gets its cited URLs opened by a person before buyer mapping.** It takes a few minutes per account, which is cheap compared with a wasted sequence.

**Not built yet (what I would do next):** a free formula that marks corroboration as "same source" when the second URL is a job board and the job title matches the intake. An HTTP status check (passive, a single GET) would catch 404s, but I left it out to keep the build strictly passive.

## Failure 2: the industry-label gate would have thrown away the best accounts

**What happened.** The first gate design was "exclude IT Services and IT Consulting", because ~75% of AngularJS job posts come from staffing and consulting firms. Then enrichment came back with Geographic Solutions (workforce software for 40+ US states, now the #1 account) and Engage People (loyalty platform) both labelled **IT Services and IT Consulting**. Schwarz Digits got the same label too.

**Fix.** The AI interprets and the formula decides. A 0.7-credit Claude call classifies the business model from the company description. The deterministic gate reads that class, not LinkedIn's label. Geographic Solutions → PRODUCT_SOFTWARE → PASS. Virtusa, TCS, Ztek, Photon and Quevera → IT_SERVICES_OR_STAFFING → FAIL.

**Learning.** Firmographic labels are fine as a *feature* and dangerous as a *gate*.

## Failure 3: the research agent cited a private individual

**What happened.** On the first H-E-B run, Claygent "corroborated" AngularJS by citing an individual engineer's LinkedIn profile. That breaks the brief's privacy rule. It is also weak evidence, because one person's CV says nothing about what the company runs today.

**Fix.** The prompt now says: *"Never cite or rely on personal profiles of individual employees … do not name individuals in any field."* H-E-B was re-run, the corroboration correctly dropped, and S1 went from EXACT_CORROBORATED to EXACT_SINGLE (50 → 40). Cost: 3 credits.

## Failure 4: the same row, two different answers

Geographic Solutions came back EXPERIENCE_ONLY on the first run and EXACT_CORROBORATED on the second. The prompt also changed between the two runs (the source rules were added), so this is not a clean test. Still, the line between "experience with AngularJS" and "our stack is AngularJS" is a judgement, and the model can land on either side of it.

**Mitigation in the build:** the human quote is an input, HIGH accounts get QA, and the agent returns categories, so a disagreement is visible as a different category, not as a silently different number. **What I would add:** run the S1 classification twice and send any disagreement to REVIEW. It is cheap at 3 credits per row, and only for rows whose tier would change.

## Failure 5: my own input was wrong, and the contradiction field caught it

I imported Medirion with `medirion.com`. Claygent reported a contradiction: *medirion.com is a parked domain; the company lives on medirion.eu.* The Tier formula sent the row to REVIEW instead of scoring it. The guardrail worked on a mistake I made.

## Smaller ones

- **The JSON schema did not save on the first try.** Text pasted programmatically into Clay's schema editor showed on screen but never reached its state. The first two rows came back as one unstructured string, and "Generate from prompt" fixed it. Learning: check the first output's *shape*, not only its content.
- **Clay formulas are expressions.** A formula with `const` and `if` silently returns `None`. Everything had to be rewritten as ternaries.
- **The trial shows 50 visible intake rows.** Coverage depends on targeted searches, not the full 480.
