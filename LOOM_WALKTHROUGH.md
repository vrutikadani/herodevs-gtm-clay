# Loom walkthrough (5–7 minutes)

These are talking points for recording in your own voice. They are not a script to read word for word. Say it the way you would explain it to a colleague.

## Before you record (5 minutes)

- [ ] Open the Clay workbook in Chrome on the **Overview** tab, with the side panels closed
- [ ] Open a second tab on `2 · Accounts (scored)` → view **Seller view (scored, key columns)**
- [ ] Open a third tab on this GitHub repo's README
- [ ] Have the Geographic Solutions job post (LinkedIn job 4271347979) ready to open
- [ ] Loom: screen + camera bubble, 1080p, desktop notifications off, close Slack and email
- [ ] Do one 30-second test recording to check the audio

## 0:00–0:40 · The problem (screen: GitHub README, top)

- HeroDevs sells patched drop-in support for end-of-life frameworks. The buyer is someone with AngularJS or Vue 2 in production, a rewrite they don't want to rush, and an auditor or customer asking about CVEs.
- So the question I gave Clay was not "who fits a firmographic profile". It was **"who still runs EOL code they own, and is anyone forcing the issue?"**

## 0:40–1:40 · Architecture (screen: Clay Overview)

- Walk the four tables left to right: suppression list, job intake, scored accounts, buyer committee.
- The main idea: **cheap and deterministic first, expensive only where it's earned.** The gate costs about a credit per row. Research costs three, and only runs on rows that pass.
- "The whole build used 82 of 1,000 trial credits."

## 1:40–2:40 · Intake and the gate (screen: Accounts, groups 1–2)

- Find Jobs is free, but the records have no description text, so I read each post and quoted the line. Show the `S1 Evidence Quote` column.
- About three quarters of AngularJS job posts are staffing and consulting firms. Their AngularJS belongs to their clients.
- **First failure:** gating on LinkedIn industry would have dropped Geographic Solutions, which is now my #1 account, because it is labelled IT Services. So the AI classifies the business model and the formula decides. Show `Business Model` → `ICP Gate`.

## 2:40–4:00 · Signals and scoring (screen: Seller view)

- Three signals: EOL exposure 50, migration timing 30, security pressure 20.
- Claygent returns **categories and URLs, not numbers**. The formulas turn them into points. Show one formula briefly.
- Read across one row out loud. **H-E-B:** their own post says "strategic AngularJS-to-Angular modernization", so S2 is ACTIVE, and they pass PCI DSS audits. 90, HIGH.
- Point out that **confidence is a separate column**. Five of six HIGH accounts are MEDIUM confidence, because they have one source.
- Guardrails: customers excluded (the three controls), stale evidence capped, contradictions sent to REVIEW (Medirion, where my own domain input was wrong).

## 4:00–5:15 · The hardest failure (screen: QA columns, then FAILURE_AND_LEARNINGS.md)

- The agent said three accounts were corroborated. I opened every URL. One was a 404 and one was a job-board mirror of the same requisition.
- **Two of three high-confidence calls were wrong, and every automated check would have passed them.**
- The fix is a human override column wired into the score, plus a rule that every HIGH account gets its sources opened before anyone maps buyers.
- (Optional, 15 seconds) Mention that the agent also cited an individual engineer's LinkedIn profile, and that the prompt now forbids personal profiles.

## 5:15–6:15 · Buyer committee (screen: table 3, **Committee Role column only, scroll so names are off screen**)

- Top five HIGH accounts, minus Elective, which is on hold because its size data conflicts.
- Title ladders map titles to economic buyer, champion, evaluator and security. It's a deterministic regex, and anything unclear becomes REVIEW.
- **No emails or phones.** Qualification isn't done yet. Show the gap per account: for example, H-E-B has no economic buyer in the results, and the right first call there is the app owner, not the CTO.

## 6:15–7:00 · What I'd do next (screen: README, Limitations)

- The weights are my judgement. Here is how I'd test them: confirm the estate on first calls, keep a holdout, refit at about 100 outcomes.
- The biggest blind spot: companies that aren't hiring are invisible to job-post intake. Public code and tech fingerprints are the next sources.
- Close: "The part I care about is that every number in that table can be explained, and the places where the AI was wrong are visible, not hidden."

## Don't

- Don't show the buyer-committee names or LinkedIn URLs on screen.
- Don't claim the weights are proven, or that a HIGH tier means intent.
- Don't over-explain Clay's UI. Show the columns and move on.
