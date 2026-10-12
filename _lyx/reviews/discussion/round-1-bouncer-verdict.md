---
verdict: CONTINUE
rationale: "Round 1 carries two BLOCKING design findings (F1 landing-to-run mapping rests on a conditional checkpoint commit, F2 tolerant readers loosen strict live gate decodes with no bound) and design findings at MEDIUM (F3, F4, F6, F8); no relabels applied."
---

Round 1 is the first round, so there is no prior ledger and no recurring key.
Gating findings at MEDIUM or worse: F1 and F2 (BLOCKING, design), F3, F4, F6, F8 (MEDIUM, design).
F5 and F9 (decision) and F7 (consistency) are not gating by class, and F10 to F13 are LOW or NIT.
No finding was relabeled, and the review carries no `## Focus departures` section.
No PATTERN entry is cited by a finding, so the constraint list changes no weight here.
