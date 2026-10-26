---
verdict: CONTINUE
rationale: "Round 1 carries F1, a MEDIUM design finding (rename-leaves-test misses tests in the external <dir>_test package), which is gating; F2, F3 and F4 are LOW/NIT and no finding is relabeled."
---

F1 is gating under the decision rule: class `design` at MEDIUM.
F2 (LOW), F3 (LOW) and F4 (NIT) are `design` but below MEDIUM, so they do not gate; they ride along with the fix.
No finding was relabeled and no focus departures section is present.
