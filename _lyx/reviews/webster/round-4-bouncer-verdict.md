---
verdict: CIRCLING
rationale: "Round 4 carries two BLOCKING findings, F1 and F2, both consistency; F1 maps to the key comment-line-breaks-fixed-column, open in rounds 1, 2 and 3 and still open, and F2 maps to markdown-line-breaks-independent-clauses, open since round 3, so each fix round clears the listed clause form and leaves another."
---

No finding was relabelled, so no departure from the review's classes or severities applies, and the review carries no `## Focus departures` section.

`F1` (Go comments joining clauses at a comma before `so`) is the same key that rounds 1, 2 and 3 filed for other clause forms: fixed-column wrapping, then semicolons and `, and`, now `, so`.
It stays at BLOCKING, so it counts as gating under the recurring-key rule even though its class is `consistency`.
Round 3's ledger recorded one remaining comment, and round 4 lists more than thirty lines across twenty files, so the count did not fall.
The fixes cleared the listed instances and not the rule, which is why the same key keeps reopening.

`F2` (markdown lines joining clauses) opened in round 3 and is open again in round 4 at BLOCKING.

The round-3 design finding `resume-skips-scratch-dir-creation` is resolved: round 4's focus verdicts show `resumeFromRunState` and `runLockHeld` create the scratch directory first, and a `TestResumeVerb` table row covers it.

The next round should fix by rule: scan every added or changed Go comment line and markdown line for a comma or semicolon followed by a clause with its own subject and verb, not only the sites listed.
