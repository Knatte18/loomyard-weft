---
verdict: CONTINUE
rationale: "Round 1 carries four gating design findings at MEDIUM (F1, F2, F4, F5) and no BLOCKING; no relabel or focus departure was made."
---

The latest review holds ten findings: 0 BLOCKING, 5 MEDIUM, 4 LOW, 1 NIT.
Gating means class `design`, so the gating MEDIUMs are F1 (retype watch reads turn starts from before the watched send), F2 (busy retype uses up the one retype), F4 (fresh-start retype untested) and F5 (busy retry bounds untested).
F3 is a MEDIUM `scope` finding and does not gate on its own, but it is carried as open.
F6 to F8 are LOW `design`, F9 and F10 are `consistency`.
No finding was relabeled, and the review carries no `## Focus departures` section.
No ledger key recurs, and no key was open in an earlier round.
