# Clay column definitions (as built)

Workbook: **HeroDevs NES - Account Prioritization**. Main table: `2 · Accounts (scored)`.

Formula, lookup and people-search columns cost 0 credits. Clay formulas are single JavaScript *expressions*: `const` and `if` statements return nothing, so every rule below is written as ternaries or a small arrow function.

## Group 2 · ICP gate (cheap, runs first)

| Column | Type | Cost | Definition |
|---|---|---|---|
| Enrich company | Clay enrichment (by Domain) | 0.5/row | name, size, industry, description, LinkedIn |
| Business Model & Reason | Use AI, Claude Sonnet, auto-run off | 0.7/row | PRODUCT_SOFTWARE / SOFTWARE_ENABLED_ENTERPRISE / IT_SERVICES_OR_STAFFING / PROFESSIONAL_SERVICES / UNKNOWN, from name, industry and description |
| Suppression Lookup | Look up single row in `0 · Suppression`, Domain = Domain | 0 | `{record: …}` or `null` |
| Suppressed | formula | 0 | `{{Suppression Lookup}}?.record ? "YES" : "NO"` |
| Entity Check | formula | 0 | below |
| ICP Gate | formula | 0 | below |
| ICP Rationale | formula | 0 | below |
| Evidence Age (days) | formula | 0 | `{{Intake Job Posted}} ? Math.round((new Date({{Research Date}}) - new Date({{Intake Job Posted}})) / 86400000) : ""` |

### Entity Check
```js
((a, b) => !b ? "NOT_FOUND" : a === b ? "MATCH" : (a.includes(b) || b.includes(a)) ? "PARTIAL" : "MISMATCH")(
  String({{Company}} || "").toLowerCase().replace(/[^a-z0-9]/g, "").replace(/(incorporated|inc|llc|ltd|gmbh|corporation|corp|plc|group|ag|se)$/, ""),
  String({{Name}} || "").toLowerCase().replace(/[^a-z0-9]/g, "").replace(/(incorporated|inc|llc|ltd|gmbh|corporation|corp|plc|group|ag|se)$/, ""))
```
PARTIAL catches parent vs subsidiary records ("dormakaba" enriched as "dormakaba Americas", "Autodesk" as "Autodesk Construction").

### ICP Gate
```js
["IT_SERVICES_OR_STAFFING", "PROFESSIONAL_SERVICES"].includes({{Business Model}}) ? "FAIL"
: (["MISMATCH", "NOT_FOUND"].includes({{Entity Check}}) || !{{Business Model}} || {{Business Model}} === "UNKNOWN" || !(Number({{Employee Count}}) >= 11)) ? "REVIEW"
: "PASS"
```

### ICP Rationale
```js
{{ICP Gate}} === "FAIL" ? "FAIL: classified " + {{Business Model}} + "; AngularJS work in its job posts likely belongs to client estates, so not a buyer."
: "Model " + ({{Business Model}} || "UNKNOWN (no description)") + " · employees " + ({{Employee Count}} || "not found") + " · entity " + {{Entity Check}}
  + ({{ICP Gate}} === "REVIEW" ? " → verify before spending research credits" : "")
```

## Group 3 · Signal research (Claygent, gated)

| Column | Type | Cost | Definition |
|---|---|---|---|
| EOL Tech Evidence | Use AI → Web research (Claygent), Clay Argon, auto-run off | 3/row | prompt and output schema in [claygent_prompt.md](claygent_prompt.md) |
| S1 Level, S2 Category, S3 Category, Contradiction, S1 Corroboration URL | fields mapped out of the Claygent output | 0 | |

Run condition, generated in Clay from a plain-English description:
```js
{{ICP Gate}}?.toLowerCase() !== "fail" && {{Suppressed}}?.toLowerCase() === "no" && !!{{S1 Evidence Quote (human-verified)}}
```
14 of 26 rows met it.

## Group 6 · QA (the human layer)

| Column | Type | Definition |
|---|---|---|
| S1 QA Override | text, typed by a person | empty unless QA disagrees with the agent |
| QA Note (human) | text | what was checked and why |
| S1 Final | formula | `{{S1 QA Override}} \|\| {{S1 Level}} \|\| ""` |
| Evidence Notes | formula | S2 and S3 evidence sentences with their URLs, from the Claygent output |
| Committee Domain | formula | `{{Tier}} === "HIGH" && !String({{QA Note (human)}} \|\| "").startsWith("QA hold") ? {{Domain}} : ""`, which feeds Find People |

## Group 4 · Score, tier & confidence (formulas)

### S1 Score (0-50): EOL exposure, with staleness built in
```js
!{{S1 Final}} ? "" : ((age, b) => b === 0 ? 0 : age > 365 ? 10 : age > 120 ? Math.min(b, 25) : b)(
  Number({{Evidence Age (days)}}),
  [50, 40, 25, 20, 0, 0][["EXACT_CORROBORATED", "EXACT_SINGLE", "EXPERIENCE_ONLY", "AMBIGUOUS_VERSION", "PLATFORM_FALSE_POSITIVE", "NONE"].indexOf({{S1 Final}})] || 0)
```

### S2 Score (0-30): migration timing
```js
!{{S2 Category}} ? "" : [30, 20, 10, 0, 0][["ACTIVE", "LIKELY", "WEAK", "COMPLETED", "NONE"].indexOf({{S2 Category}})] || 0
```

### S3 Score (0-20): security and compliance pressure
```js
!{{S3 Category}} ? "" : [20, 15, 10, 5, 0][["ATTESTED", "REGULATED", "ENTERPRISE_OR_GOV_BUYERS", "GENERAL", "NONE"].indexOf({{S3 Category}})] || 0
```

### Signal Score (1-100)
```js
{{S1 Level}} ? Math.max(1, Number({{S1 Score (0-50)}}) + Number({{S2 Score (0-30)}}) + Number({{S3 Score (0-20)}})) : ""
```

### Tier
The output type is Select, which colours the values: HIGH green, MEDIUM amber, LOW light red, REVIEW violet, EXCLUDED grey.
```js
{{Suppressed}} === "YES" ? "EXCLUDED"
: {{ICP Gate}} === "FAIL" ? "EXCLUDED"
: !{{S1 Level}} ? "LOW"
: ({{Contradiction}} === "YES" || {{Entity Check}} === "MISMATCH") ? "REVIEW"
: Number({{Signal Score (1-100)}}) >= 75 ? (Number({{S1 Score (0-50)}}) < 20 ? "MEDIUM" : "HIGH")
: Number({{Signal Score (1-100)}}) >= 50 ? "MEDIUM" : "LOW"
```
With these weights the S1 < 20 cap can never fire (S2 + S3 max out at 50, so 75 needs S1 ≥ 25). It stays in the formula so the guardrail survives a weight change.

### Evidence Confidence (kept separate from the score)
```js
!{{S1 Final}} ? "" : ({{Contradiction}} === "YES" || ["MISMATCH", "NOT_FOUND"].includes({{Entity Check}}) || Number({{Evidence Age (days)}}) > 365
  || ["EXPERIENCE_ONLY", "AMBIGUOUS_VERSION", "PLATFORM_FALSE_POSITIVE", "NONE"].includes({{S1 Final}})) ? "LOW"
: ({{S1 Final}} === "EXACT_CORROBORATED" && Number({{Evidence Age (days)}}) <= 120 && {{Entity Check}} === "MATCH") ? "HIGH"
: "MEDIUM"
```

### Why This Score
```js
!{{S1 Final}} ? ({{Suppressed}} === "YES" ? "Excluded: public HeroDevs customer (suppression list)."
  : {{ICP Gate}} === "FAIL" ? "Excluded at ICP gate. " + {{ICP Rationale}}
  : "Not scored: no human-verified EOL evidence for this account.")
: "S1 " + {{S1 Score (0-50)}} + "/50 " + {{S1 Final}} + ({{S1 QA Override}} ? " (QA override)" : "")
  + " (job post " + {{Evidence Age (days)}} + "d old) + S2 " + {{S2 Score (0-30)}} + "/30 " + {{S2 Category}}
  + " + S3 " + {{S3 Score (0-20)}} + "/20 " + {{S3 Category}} + " = " + {{Signal Score (1-100)}}
  + ". Shows public evidence of EOL tech; does not prove production use, budget or buying intent."
```

## Group 5 · Seller action

### Seller Angle
```js
{{Tier}} === "EXCLUDED" ? "" : {{Tier}} === "REVIEW" ? "Hold outreach until a person resolves the conflicting evidence."
: {{Tier}} === "LOW" ? "Low priority: no outreach until the evidence improves."
: {{S2 Category}} === "ACTIVE" ? "Migration already underway: NES keeps the legacy app patched (14-day CVE SLA) until cutover, so the team can finish the rewrite without a security gap or a rushed deadline."
: ["ATTESTED", "REGULATED"].includes({{S3 Category}}) ? "Compliance pressure: an end-of-life framework inside an audited estate is a finding waiting to happen; NES gives a supported, patched drop-in now while the migration is planned around the roadmap."
: "Discovery first: confirm which apps still run the EOL framework and who signs off on CVE exceptions before pitching."
```

### Next Research Step
The rule picks the first gap that applies.
```js
{{Tier}} === "EXCLUDED" ? "None: excluded."
: !{{S1 Final}} ? "Find first-party evidence of AngularJS or Vue 2 before spending credits."
: {{Contradiction}} === "YES" ? "Resolve the contradiction manually before outreach."
: {{Entity Check}} !== "MATCH" ? "Confirm which entity owns the app (enriched record: " + {{Name}} + ") before mapping buyers."
: ["EXPERIENCE_ONLY", "AMBIGUOUS_VERSION", "PLATFORM_FALSE_POSITIVE", "NONE"].includes({{S1 Final}}) ? "Find a first-party source that places AngularJS or Vue 2 in their own product (engineering blog, public repo, docs, second job post)."
: {{S1 Final}} === "EXACT_SINGLE" ? "Find a second independent source that corroborates the EOL framework."
: ["NONE", "WEAK"].includes({{S2 Category}}) ? "Look for migration timing: rebuild roles, roadmap posts or 'legacy' language."
: ["GENERAL", "NONE"].includes({{S3 Category}}) ? "Check the trust center for SOC 2, ISO 27001, PCI or public-sector authorizations."
: "Map the buyer committee and confirm who owns the legacy application."
```

## Table 3 · Buyer committee: Committee Role
```js
((t) => /loss prevention|in test|material compliance|electrical|data engineering|infrastructure|cloud|\bsre\b/.test(t) ? "OUT_OF_SCOPE"
: /\bciso\b|chief information security|information security|security officer|head of security|\bgrc\b|compliance/.test(t) ? "SECURITY_COMPLIANCE"
: /\bcto\b|chief technology|chief technical|vp of engineering|vice president.*engineering|head of engineering/.test(t) ? "ECONOMIC_BUYER"
: /director of engineering|engineering director|software department director|engineering manager/.test(t) ? "CHAMPION"
: /architect|principal|staff|team lead|senior software engineer/.test(t) ? "TECHNICAL_EVALUATOR"
: /procurement|purchasing|sourcing|vendor management/.test(t) ? "PROCUREMENT"
: "REVIEW")(String({{Job Title}} || "").toLowerCase())
```
