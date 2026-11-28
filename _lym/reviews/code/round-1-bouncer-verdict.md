---
verdict: CONTINUE
rationale: "Round 1 carries F1, a BLOCKING design finding: a hub reconcile can commit a profiles.yaml that runprofile.Load refuses, which cites PATTERN-config-layers and has no test row."
---

F1 is gating on its class and severity.
It cites `PATTERN-config-layers`, which requires a meaning change to ship a reinterpretation run by `hubreconcile`,
so it stays BLOCKING.

F2 (LOW) and F3 (NIT) are `consistency` findings and do not gate.

No finding was relabelled.
The review carries no `## Focus departures` section.
