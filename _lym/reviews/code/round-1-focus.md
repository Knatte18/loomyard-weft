---
round: 1
focus:
  - "internal/shuttleengine/stepstop.go: does the orphan check read only run records (a run reading running whose waiting process is gone), with no dependence on status error text, and can it stop a run whose waiter is still alive or whose record is mid-write?"
  - "internal/shedverbs/loop.go and loopwait.go: is the removed error-text recognition fully gone with no caller left, and does each loop start reconcile before the first child on every path, including a re-step after an uninterrupted stop?"
  - "internal/loomcli/looparm.go, cli.go, arm.go: is the new wiring (scratch dir, friction dir, nil location) correct on failure paths, and are errors from reconciliation handled rather than swallowed?"
  - "Docs in internal/shedverbs/doc.go and internal/shuttleengine/doc.go: do they match the behavior the diff leaves?"
  - "Tests: TestSpecFor_ScratchAndFrictionDir is listed as not judged, kept; check whether it needs a testtiming:keep marker, and check the new loop, loopwait and stop tests against their covering tests."
---

Seed focus for a decision-record-only review. The board entry asks to reconcile from run records, not error text.
