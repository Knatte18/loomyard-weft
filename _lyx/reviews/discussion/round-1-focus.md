---
round: 1
focus:
  - "D3 + D5 (decision-record.md): settle when the CLI layer loads logger.yaml relative to the arm check, and whether that load's own logging or D5's Warn record can arm a sink in a process that otherwise never armed, contradicting D3's 'a process that never armed a sink sweeps nothing' and the acceptance criterion on it."
  - "Goal vs D1-D4 (decision-record.md): settle whether the Goal's 'must still exist when the driver reads it' is guaranteed by the decisions or only made likely by rank and count, and whether an acceptance criterion pins the Goal's own driver-read scenario or only the D2 ranking unit case."
  - "D1 rationale vs D2 rationale (decision-record.md): settle whether trace-id grouping helps the Webster-step scenario at all, given D2 describes that step's children as arming without an inherited trace id, so each child forms its own group."
  - "D6 (decision-record.md): settle whether D6 carries a decision the plan writer needs beyond Scope's out-of-scope line on arm triggers and level policy, or whether its rationale is deliberation about the rejected suppression option that belongs in support-log.md's Rejected alternatives."
  - "Open risks and Auto-mode assumptions (decision-record.md): settle whether each entry is something the plan writer acts on; check the Windows proc.IsAlive note ('noted for completeness') and the unreproduced operator-surface cause, which appears in both sections and in the support log's Q8."
---

The first entry is the likeliest real gap.
Moving the sweep to exit (D3) and adding a config load with a Warn fallback (D5) interact with the arm triggers the task leaves unchanged,
and the decision record never says whether the load runs in every process or only in one whose sink already armed.

The second and third entries test whether the design actually closes the failure it was written for.
The fourth and fifth are leanness checks.
Apply the completeness-before-leanness test before any relocation finding:
D6's "arm triggers unchanged" and the Open risks' reconcile note may carry constraints the plan writer needs.

Checked against the rubric and not promoted to focus:
the Notes for the plan writer section is optional, and its function and file names are a head start the stencil allows there.
Rejected alternatives appear only in support-log.md, as the contract requires.
Call-site enumeration, for example every `NotifyExit` caller or every consumer of the trace filename grammar, belongs to Plan-Write.
