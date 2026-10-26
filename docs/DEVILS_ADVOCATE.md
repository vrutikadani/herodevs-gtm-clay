# Devil's advocate review

Each objection is one a skeptical HeroDevs GTM leader could raise. The answer is either what the build does about it, or an admission that it does nothing.

**1. "Job posts are a lagging, noisy proxy. The companies most stuck on AngularJS may not be hiring at all."**
Agreed, and this is the biggest blind spot. Job-post intake only sees companies that are hiring. A frozen AngularJS app with no open roles is invisible to it. *Not solved here.* The next sources would be public code (package manifests on company GitHub orgs), passive front-end fingerprinting from public pages via a tech-stack provider, and CVE-driven lists. Each needs its own false-positive handling.

**2. "26 accounts is not a model, it's a list."**
True. The build shows the mechanism (gate → gated research → deterministic score → QA). It does not show predictive power. That needs the outcomes described in [MEASUREMENT_PLAN.md](MEASUREMENT_PLAN.md).

**3. "Your weights are made up."**
Yes. 50/30/20 is a stated judgement, and [SCORING.md](SCORING.md) says so. The sensitivity check shows the HIGH set does not change under three alternative weightings, because the S1 level drives the tier. That makes the result robust to the weights, but not correct.

**4. "S3 doesn't discriminate."**
Correct. 10 of 14 researched accounts were ATTESTED. Most mid-size software companies hold SOC 2 or ISO 27001 now. S3 should be split into "attestation that covers the app" and "company-wide attestation", or weighted less.

**5. "Scores cluster at 80–90, so the ranking inside HIGH means little."**
Correct. Five of six HIGH accounts sit at 80–90. Inside the tier, a seller should sort by **confidence** first. The buyer-committee selection already does this indirectly, because it skips the QA hold.

**6. "The human evidence read doesn't scale."**
At about 250 company rows per 1,000 posts, a person reading each post is the bottleneck. Credits are not. Options: let Claygent extract the quote from the job URL, with a person reviewing only rows that would reach HIGH; or buy a jobs source that includes description text.

**7. "Claygent gave different answers on different runs, and invented a corroborating URL. Why trust any of it?"**
Don't trust it blindly. The design assumes the agent is wrong sometimes:

- It returns categories, not points.
- A person supplies the key quote.
- Every HIGH account is QA'd.
- Overrides are first-class columns.

Double-running S1 for tier-changing rows is the next step.

**8. "The suppression list misses most customers."**
Yes. HeroDevs says it has 800+ clients, and the public list has 28. In production the suppression lookup would point at the CRM. The mechanism (a lookup in a separate table, then EXCLUDED before scoring) carries over unchanged.

**9. "H-E-B is a 54,000-person grocer. Is that really an NES buyer?"**
Maybe. The evidence is unusually direct: their own post says *"strategic AngularJS-to-Angular modernization"*, and they pass annual PCI DSS audits. But the committee is hard to reach. The right first call is the owner of the named app, not the CTO. The tier says "strong signal". It does not say "easy deal".

**10. "Entity resolution is shaky."**
Agreed. dormakaba was enriched as dormakaba Americas, Autodesk as Autodesk Construction, Sundayy as a different company, and Elective's size looks wrong. The Entity Check formula flags PARTIAL and MISMATCH, but it compares names, not corporate hierarchies. A real build would resolve the parent company from the CRM account hierarchy.

**11. "Experience-only might be underrated."**
Possibly. "AngularJS or Angular 2" in Geographic Solutions' posts was read both ways by the agent. A company that lists AngularJS as required experience probably has some of it. At 25 points these accounts land in MEDIUM, not LOW, which seems right.

**12. "Why not just buy intent data?"**
Intent shows interest in a topic, not exposure to EOL software. Buy it for *timing* later, as an S2 input, once S1 has shown the account has the problem. It does not replace S1.
