# HeroDevs — Signal-Based Account Prioritization in Clay

A Clay workbook that finds companies still running **end-of-life AngularJS or Vue 2** in their own software, rules out the noise cheaply, and ranks what is left with a transparent 1–100 score. Each account gets a tier, a separate confidence rating, a plain-English reason, a seller angle, a next research step and, for the top accounts, a mapped buyer committee.

Built on a Clay free trial for **82.3 of 1,000 credits**.

**Clay template (configuration only, no data):** https://app.clay.com/shared-workbook/share_0tnig3jxrKngEC425Yb

![Workflow](assets/workflow-diagram.svg)

## Result

26 accounts, scored on 2026-10-26:

| Tier | Accounts |
|---|---|
| **HIGH (6)** | Geographic Solutions 90 · H-E-B 90 · dormakaba 90 · Elective 85 · Dataiku 80 · DATAMYTE 80 |
| **MEDIUM (3)** | NCR Atleos 65 · Dover Fueling Solutions 55 · APCON 55 |
| **REVIEW (1)** | Medirion: contradictory evidence (the imported domain was a parked page) |
| **LOW (5)** | Engage People, Autodesk, Transport for NSW (45); Schwarz Digits (10, ServiceNow false positive); Sundayy (wrong entity) |
| **EXCLUDED (11)** | 8 IT-services, staffing and professional-services firms (failed the ICP gate); 3 public HeroDevs customers (suppression controls) |

Only **Geographic Solutions** is HIGH tier *and* HIGH confidence. The other five HIGH accounts have one source each, and the table says so. Each one comes with a QA note.

![Score, tier and confidence](assets/clay-score-tier-confidence.jpg)

## How it works

1. **Intake (free).** Clay Find Jobs: "AngularJS" posts from the last 120 days. Job records have no description text, so **a person reads each post and quotes the evidence**.
2. **ICP gate (1.2 credits/row).** Enrich company, then an AI business-model class, then a suppression lookup and an entity check. The formula returns PASS, REVIEW or FAIL. Staffing and services firms are removed here, before any research spend. → [ICP](docs/ICP.md)
3. **Gated signal research (3 credits/row).** Claygent runs only where the gate did not fail, the account is not a customer, and verified evidence exists: 14 of 26 rows. It returns **categories and URLs, never points**. → [Signals](docs/SIGNALS.md)
4. **Deterministic scoring (free).** S1 EOL exposure 0–50, S2 migration timing 0–30, S3 security pressure 0–20, then tier, guardrails and confidence. → [Scoring](docs/SCORING.md)
5. **Human QA of every HIGH account.** `S1 QA Override` feeds `S1 Final`, which feeds the score. The QA pass changed 2 confidence calls and put 1 account on hold.
6. **Buyer committee for the top 5 (free).** Clay Find People with title ladders and a deterministic role formula. **No emails or phones.** → [Buyer committee](docs/BUYER_COMMITTEE.md)

## The three signals

| | Signal | Points | The question |
|---|---|---|---|
| S1 | EOL / legacy tech exposure | 0–50 | Is AngularJS 1.x or Vue 2 in their *own* code, corroborated and recent? |
| S2 | Modernization / migration timing | 0–30 | Is a migration happening now? |
| S3 | Security / compliance pressure | 0–20 | Does an auditor or a customer force the issue? |

Guardrails: S1 < 20 caps the tier at MEDIUM; customers are EXCLUDED; stale evidence lowers S1 and confidence; contradictions go to REVIEW; a completed migration zeroes S2.

**The weights are a judgement, not a measurement.** A sensitivity check under three other weightings leaves the HIGH set unchanged. [MEASUREMENT_PLAN.md](docs/MEASUREMENT_PLAN.md) describes how to replace the weights with fitted ones.

## The hardest failure

Claygent marked three accounts as *corroborated* by a second source. When I opened the URLs, one was a 404 and another was a job-board mirror of the same requisition. **Two of the three HIGH-confidence calls were wrong**, and every automated check had passed them. The fix is a human override column wired into the score, plus a rule that every HIGH account's sources get opened before buyer mapping. → [FAILURE_AND_LEARNINGS.md](docs/FAILURE_AND_LEARNINGS.md)

## Repository

| Path | Contents |
|---|---|
| [docs/ICP.md](docs/ICP.md) | Research on HeroDevs, the ICP, the gate, suppression |
| [docs/SIGNALS.md](docs/SIGNALS.md) | Signal definitions and evidence rules |
| [docs/SCORING.md](docs/SCORING.md) | Formula, thresholds, guardrails, results, weight sensitivity |
| [docs/BUYER_COMMITTEE.md](docs/BUYER_COMMITTEE.md) | Title ladders, role logic, coverage gaps |
| [docs/CLAY_ARCHITECTURE.md](docs/CLAY_ARCHITECTURE.md) | Tables, column groups, design rules, **credit-efficiency review** |
| [docs/FAILURE_AND_LEARNINGS.md](docs/FAILURE_AND_LEARNINGS.md) | What broke, in order |
| [docs/MEASUREMENT_PLAN.md](docs/MEASUREMENT_PLAN.md) | How to prove or disprove the score |
| [docs/DEVILS_ADVOCATE.md](docs/DEVILS_ADVOCATE.md) | The strongest objections, answered or conceded |
| [clay/formulas.md](clay/formulas.md) | Every formula, copy-pasteable |
| [clay/claygent_prompt.md](clay/claygent_prompt.md) | The research prompt and design notes |
| [clay/suppression_list.csv](clay/suppression_list.csv) | 28 public HeroDevs customers with sources |
| [sample/public-safe-sample.csv](sample/public-safe-sample.csv) | All 26 accounts with evidence, scores and tiers: company-level only |
| [LOOM_WALKTHROUGH.md](LOOM_WALKTHROUGH.md) | Script for the 5–7 minute walkthrough |

## Privacy and safety

- **Passive public research only.** No scanning, probing or logins. Every source is a public web page.
- **No personal contact data in this repository.** The buyer committee is summarized as role counts. Names live only in the private Clay workbook, and no emails or phones were enriched.
- **The research prompt bans personal profiles as sources**, after the agent cited one (see the failure log).
- The Clay template shares configuration, not rows.
- No credentials or API keys are used or stored anywhere in this repository.

## Limitations

The sample is small (26 accounts) and comes from a US/UK-heavy job-post intake. Companies that are not hiring are invisible to it. The suppression list covers public customers only. The weights are untested against outcomes, and S3 hardly separates accounts in this sample. Details are in [DEVILS_ADVOCATE.md](docs/DEVILS_ADVOCATE.md).
