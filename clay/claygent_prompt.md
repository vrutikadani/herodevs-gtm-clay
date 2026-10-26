# Claygent prompt: `EOL Tech Evidence`

- Use case: Web research (Claygent)
- Model: Clay Argon, 3 credits per row
- Auto-run: off
- Run condition: ICP Gate ≠ FAIL, Suppressed = NO, and a human-verified quote exists
- Output: JSON Schema, generated from the prompt in Clay

In the prompt below, `[Column]` marks a column token inserted with `/` in Clay.

```text
Company: [Company] ([Domain])
Human-verified quote from this company's own job post (posted [Intake Job Posted], title [Intake Job Title]): [S1 Evidence Quote (human-verified)]

Context: HeroDevs sells Never-Ending Support, security-patched drop-in replacements for end-of-life
open-source frameworks (AngularJS 1.x, EOL Dec 2021; Vue 2, EOL Dec 2023). Use passive public research
only: read public web pages; never scan, probe or log in to any system. Only use evidence you can cite
with a public URL. If you find nothing, answer NONE. Do not guess.

Source rules: cite only company-owned pages, job postings, product or security documentation, public
code repositories, press releases or reputable news. Never cite or rely on personal profiles of
individual employees (for example LinkedIn /in/ pages or personal blogs), and do not name individuals
in any field.

s1_level (EOL tech in the company's OWN product or internal apps):
  EXACT_CORROBORATED = the quote places AngularJS or Vue 2 in the employer's own estate AND you found a
    second independent public source (another job post, engineering blog, GitHub, docs) confirming it
  EXACT_SINGLE = the quote places it in the employer's estate but there is no second source
  EXPERIENCE_ONLY = listed only as candidate experience, a nice-to-have or an 'or' alternative
  AMBIGUOUS_VERSION = 'Angular' or 'Vue' without a version
  PLATFORM_FALSE_POSITIVE = AngularJS belongs to a vendor platform the employer configures
    (e.g. ServiceNow Service Portal)
  NONE
s1_corroboration_url: URL of the second source, or NONE.

s2_category (migration timing):
  ACTIVE = public evidence that a migration off the EOL framework is underway now
  LIKELY = a modernization program or new-stack hiring alongside the legacy app, without an explicit
    migration of that framework
  WEAK = only generic 'modernize' language
  COMPLETED = public evidence that the migration already finished
  NONE
s2_evidence: one short sentence, quoting the source where possible. s2_url: source URL or NONE.

s3_category (security/compliance pressure):
  ATTESTED = the company publicly holds or must meet SOC 2, ISO 27001, PCI DSS, HIPAA,
    FedRAMP/StateRAMP, IRAP or similar for its products
  REGULATED = operates in a regulated sector (financial services, healthcare, government, critical
    infrastructure) but no attestation was found
  ENTERPRISE_OR_GOV_BUYERS = sells to enterprises or public-sector buyers who run security reviews
  GENERAL = none of these
  NONE = could not determine
s3_evidence: one short sentence. s3_url: source URL or NONE.

contradiction: YES if any source contradicts the quote (e.g. the app was already rewritten, or the
domain belongs to a different organisation), else NO. contradiction_note: one short sentence or NONE.
```

## Design choices

- **The agent returns categories, never numbers.** The formulas in [formulas.md](formulas.md) turn categories into points. Changing a weight is then a formula edit, and no re-run (and no re-spend) is needed.
- **The human quote goes in as an input.** The agent is asked to *corroborate* evidence a person has already read, not to discover it. Job records from Clay Find Jobs carry no description text, so a keyword hit alone proves nothing.
- **The source rules came from a real failure.** The first H-E-B run cited an individual engineer's LinkedIn profile as corroboration. See [FAILURE_AND_LEARNINGS.md](../docs/FAILURE_AND_LEARNINGS.md).
