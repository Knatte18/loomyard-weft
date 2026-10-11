---
round: 3
focus:
  - "In `_lyx/discussion/decision-record.md` D3 and D6: `internal/shedengine/goto.go` `Goto` clears `pause_requested`, `pause_before` and `pause_after` on every goto, and `lyx loom pause` with no flag only sets `pause_requested` on a halted run. Settle whether any ordering lets the arbiter's goto, or the Go re-launch, erase an orch or operator pause set while the arbiter was live, and whether D6's state check and the acceptance criteria run before anything clears those fields."
  - "In `_lyx/discussion/decision-record.md` D6's resume bullet, the support-log Question ledger and Notes for the plan writer: `internal/loomcli/resume.go` `resumeFromRunState` writes no status file, and `stepLocked` in `internal/shedengine/run.go` short-circuits only `StateDone`. Settle whether any text still claims the re-launch performs an un-pause that `lyx loom resume` performs."
  - "In `_lyx/discussion/decision-record.md` D1 and D7: the applied mark is written before the run steps, and D7 hands on an applied attempt whose stop key is unchanged. Settle whether the refusal's stated reason matches the cases that reach it, including a re-launch that ends before stepping, and whether a bound on the parent's wakes is stated."
  - "In `_lyx/discussion/decision-record.md` D2: the Publish-halt exclusion reads the Publish row's verify-failure record. Settle whether D2's list of what Go checks matches every input the decision reads."
---

The focus targets the round-2 design gaps around the arbiter's goto and the re-launch, plus the smaller wording gaps beside them.
