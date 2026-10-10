---
round: 3
focus:
  - "In `_lyx/discussion/decision-record.md`, D2, D4 and D11 say each child step dies with the `--until-stop` loop and that teardown kills the loop by pid file. Settle whether a stated mechanism ties the in-flight child's life to the loop's on Linux and Windows, whether D11's bound covers a step already running, and whether the matching acceptance row can pass as designed."
  - "In D2, D5 and D9, settle what a waiter does when the loop lock is free and no current envelope exists, and whether a later invocation can spawn a fresh loop over a status file still reading `running`, re-invoking a half-run producer without the `interrupt_policy` check."
  - "In D3, settle what `--before` and `--after` do when a different target is already recorded, and what bounds `--clear` against a condition another actor set; confirm the Decision states the rule and an acceptance row covers it."
  - "Check that the ownership class D3 and the Constraints row assign to the two condition fields matches who may write and clear them, as defined in `internal/shedengine/status.go`."
  - "Check that the empty-key-dropping wording in Scope, D1, D7 and the acceptance criteria describes one scope, and that the interrupted stop's synthesized error form and its `envelope_path` in D1 and D7 match the closed short error key set in `internal/shedverbs/step.go` and the location keys D7 promises."
---

The first two entries are the design-class questions that gated round 2; the rest are bounded consistency and attack-surface checks.
