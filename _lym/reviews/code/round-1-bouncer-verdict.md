---
verdict: CONTINUE
rationale: "Round 1 carries two gating design findings at MEDIUM (F1 notice receiver resolution unchanged in internal/hibernatecli/deps.go, F2 untested notice-file branches in internal/hibernateengine/watch.go), so the decision rule gives CONTINUE; no BLOCKING finding, no relabel, no Focus departures section."
---

The decision rule makes a `design` finding at MEDIUM or worse gating.
F1 and F2 are both `design` at MEDIUM and stay as the reviewer labelled them; their entries show no mislabel.
F3 and F4 are LOW and do not gate on their own.

No key recurs across rounds and none was open earlier, so nothing is carried from a prior ledger.
