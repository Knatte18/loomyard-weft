---
round: 1
focus:
  - "internal/websterengine/recordbatch.go, recoverbatch.go, waiting.go, state.go: does a waiting record leave the recovery count unchanged on both the fork and the recovery path, and is only the one spawn that replaces a waiting record exempt from the exhaustion check and the count?"
  - "internal/websterengine/report.go and digest.go: does ParseReport require a non-empty need with NEEDS_PARENT and refuse a need on any other status, and does a record written before the new need key still read unchanged?"
  - "internal/websterengine/runlevel.go and internal/webstercli/run.go: does the ErrBatchDeadEnd refusal fire only for a terminal dead batch with RecoveryRetry false or an exhausted terminal batch with no unrendered amendment, and does it name its way forward and have a test reaching it?"
  - "internal/webstercli/recoverbatch.go and recordbatch.go: does a dead or stuck envelope carry recovery_session and name it in its message, does recover-batch exit 0 on a waiting digest, and does record-batch on a waiting batch refuse naming the need and begin-batch?"
  - "internal/shedengine/run.go, status.go and shedadapters/webster.go: is a Stuck blocking at a row with no OnStuck recorded budget-exempt so that an identical repeat folds with repeats 1, without changing any budget comparison or folding across a goto or reroute?"
  - "contracts/stencils/webster master and implementer bodies: does the wording map the shared FAILED-and-the-orch-runs-it clause to NEEDS_PARENT for a webster implementer, tell Master to message its parent once and wait, and drop or weaken no other rule?"
  - "doc.go files (websterengine, shedengine, shedadapters) and PATTERN.md: do the docs land with the behavior change, and does the change description claim only what the diff shows?"
  - "Tests: does each acceptance criterion in the decision record have a covering test, and does any new test duplicate an existing one's coverage?"
---

Seed focus drawn from the board entry's design choice and the decision record's scope and acceptance criteria.
