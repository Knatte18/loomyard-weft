---
verdict: CONTINUE
rationale: "F1 is a design finding at MEDIUM (waitshadow.go:132, the shadow's fact refresh never re-reads the interrupt and API-error markers at later ticks), so the latest round carries a gating finding; F2 and F3 are consistency LOW and not gating."
---

The latest review carries one gating finding: F1, class `design`, severity MEDIUM.
No finding was relabeled, and there is no `## Focus departures` section to ratify or reject.
F2 and F3 are `consistency` at LOW and do not gate on their own.
This is the first round, so there is no prior ledger and no recurring key.
