# Buyer committee

## Which accounts

The committee covers the **top 5 HIGH accounts not on QA hold**, selected by a formula (`Committee Domain`):

```
Tier = HIGH  AND  QA Note does not start with "QA hold"
```

Those 5 are Geographic Solutions, H-E-B, dormakaba, Dataiku and DATAMYTE. Elective is HIGH but held, because enrichment says 2–10 employees and its LinkedIn page says 11–50. That should be resolved before anyone spends time on contacts.

## Who buys NES

The roles come from HeroDevs' own case studies:

- A **CTO** owns the trade-off between migrating now and staying supported (Keelvar, Statista).
- An **architect or senior frontend engineer** evaluates the drop-in and does the implementation work.
- **Security or compliance** is usually the trigger: SOC 2, HIPAA, IRAP, or customer questionnaires.

## Title ladders (deterministic)

| Role | Ladder, matched in order | Why they matter for NES |
|---|---|---|
| OUT_OF_SCOPE (checked first) | loss prevention, "in test", material compliance, electrical, data engineering, infrastructure, cloud, SRE | Titles that match "security" or "manager" but do not own app code |
| SECURITY_COMPLIANCE | CISO, chief information security, information security, security officer, head of security, GRC, compliance | Owns CVE exceptions and audit findings; often the trigger |
| ECONOMIC_BUYER | CTO, chief technology, VP of engineering, vice president … engineering, head of engineering | Owns the migrate-vs-support decision and the budget |
| CHAMPION | director of engineering, engineering director, software department director, engineering manager | Feels the freeze-the-roadmap pain directly |
| TECHNICAL_EVALUATOR | architect, principal, staff, team lead, senior software engineer | Validates the drop-in and does the work |
| PROCUREMENT | procurement, purchasing, sourcing, vendor management | Only at large enterprises, and only once a deal exists |
| REVIEW | anything else | A person decides |

The formula is in [clay/formulas.md](../clay/formulas.md). Order matters: exclusions are checked first, then security, then the most senior titles.

## Result

Clay Find People, scoped to the 5 domains, used 10 title keywords with "similar to" matching and a limit of 10 per company. It returned 38 people at 0 credits.

| Account | Economic buyer | Champion | Tech evaluator | Security / compliance | Out of scope | Gap |
|---|---|---|---|---|---|---|
| Geographic Solutions | — | — | 7 | 1 (CISO) | 0 | No economic buyer or champion found |
| H-E-B | — | 3 | 2 | 1 (CISO) | 4 | No economic buyer in results (54k employees; the app owner sits in a digital unit) |
| dormakaba (Americas) | 1 (VP Eng) | 3 | 0 | 1 (Compliance) | 2 | Entity: the Americas subsidiary, not the group |
| Dataiku | 1 (CTO) | 3 | 3 | 0 | 3 | No security contact found |
| DATAMYTE | 0 | 1 | 2 | 0 | 0 | Small company; the economic buyer is likely the CEO or a founder |

## What this does **not** do

- **No emails or phones were enriched.** Qualification is not finished. Five of the six HIGH accounts have MEDIUM confidence, and work-email enrichment would cost about 0.6–1.1 credits a person for people who may never be contacted.
- **No contact data leaves Clay.** This repository contains role counts, not names. The people table exists only in the private workbook. The shared template copies configuration, not rows.
- **"Similar to" title search is fuzzy.** It returned an "Area Loss Prevention & Security Manager" and a "Software Engineer in Test". The OUT_OF_SCOPE ladder exists because of those two.

## Next step per gap

1. **Geographic Solutions:** a targeted search for "VP / Director, Software Development".
2. **H-E-B:** find the owner of the Own Brand Product Systems app named in the job post. The CTO of a 54,000-person retailer is the wrong first call.
3. **dormakaba:** confirm whether the AngularJS estate belongs to the Americas unit or the group.
4. **DATAMYTE:** check the founder or CEO title once a champion confirms the estate.
