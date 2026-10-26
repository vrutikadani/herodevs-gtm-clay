# ICP

## What HeroDevs sells, and why anyone buys it

Open-source frameworks reach end of life (EOL). The code keeps running, but security patches stop. Then auditors, customer security questionnaires and internal CVE SLAs start flagging it.

HeroDevs **Never-Ending Support (NES)** is a drop-in, patched replacement for the EOL version, with a 14-day CVE SLA, priced per committer. The flagship is AngularJS 1.x (EOL December 2021). Vue 2 (EOL December 2023) comes next. The catalog also covers Java, .NET, Python, PHP, Drupal, Node and others.

The buying reason in HeroDevs' public case studies is nearly always the same. *We have to stay secure and compliant. A rewrite takes 6–24 months, and we will not freeze the roadmap or rush the migration.*

Sources: herodevs.com home, the NES pages, the NES for AngularJS page, and the Keelvar and Statista case studies, all read on 2026-10-26.

## ICP definition used in this build

| Dimension | In | Out |
|---|---|---|
| Business model | Product software companies, and software-enabled enterprises that build and run their own customer-facing or business-critical apps | IT services, staffing and consultancies (their AngularJS belongs to client estates); professional services |
| Tech evidence | AngularJS 1.x or Vue 2 in the company's **own** estate | AngularJS that belongs to a vendor platform the company configures (e.g. ServiceNow Service Portal) |
| Pressure | External security or compliance pressure: regulated sector, attestations, enterprise or government buyers | none required, but it scores lower |
| Size | ≥ 11 employees passes; under 11 or unknown goes to REVIEW | — |
| Relationship | Not a known HeroDevs customer | Anything on the suppression list → EXCLUDED |

## What the research changed from the starting hypothesis

1. **Size matters less than expected.** Public customers range from a ~100-person SaaS company (Keelvar) to the Fortune 100. What matters is a long-lived app that someone must keep secure. Small companies stay in, at lower priority.
2. **Staffing and IT-services firms dominate the data.** About 75% of AngularJS job posts in the intake came from them or from ServiceNow roles. They are not buyers, so the gate removes them before any money is spent.
3. **LinkedIn industry labels are not a usable gate.** Geographic Solutions (workforce software for 40+ US states) and Engage People (loyalty platform) are both labelled "IT Services and IT Consulting". A gate on industry would have dropped two of the best accounts. See [FAILURE_AND_LEARNINGS.md](FAILURE_AND_LEARNINGS.md).
4. **Platform false positives are real.** A Schwarz Digits ServiceNow role lists AngularJS because ServiceNow's portal uses it. The company does not own that code, so S1 = 0.

## The gate

The gate runs before any paid research. AI interprets, the formula decides:

- `Business Model` (AI, 0.7 credits) reads name, industry and description and returns one of five classes.
- `ICP Gate` (formula, free): FAIL for services and staffing; REVIEW for an entity mismatch, an unknown business model, or fewer than 11 or unknown employees; otherwise PASS.
- `Suppressed` (lookup against the customer list, free) is applied in the tier, not the gate, so the reason stays visible.

**Result on 26 accounts:** 11 PASS, 4 REVIEW, 8 FAIL, plus 3 suppression controls. The gate does not use the 1–100 score.

## Suppression

There are 28 public HeroDevs customers, taken from the case studies and the logo walls on the home, NES and AngularJS pages ([clay/suppression_list.csv](../clay/suppression_list.csv)). Three of them (Sisense, Statista, Keelvar) were deliberately added to the account table as controls, and all three came out EXCLUDED.

**Caveat:** "not on the list" only means there is no *public* evidence of a relationship. HeroDevs claims 800+ clients, so a real deployment would check the CRM instead.
