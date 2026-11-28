---
verdict: CONTINUE
rationale: "Round 1 carries one BLOCKING design finding (F1) and three MEDIUM design findings (F2, F3, F4); no finding was relabelled."
---

F1 says the printed wait includes the shared build and the `--copies` prebuild.
F2 says a rerun tier's line prints a zero wait.
F3 says the summary-line patterns refuse a line written by a binary on the other side of the change.
F4 says no test pins the printed wait on a real tier run.
F5 and F6 are LOW `consistency` findings and do not gate.

The review has no `## Focus departures` section, and none of its findings cites a PATTERN entry.
