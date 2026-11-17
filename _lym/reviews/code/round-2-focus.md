---
round: 2
focus:
  - "internal/gatecli/redundantmutants.go `runMutants` and internal/testcover/mutantverdict.go `JudgeMutants`: settle whether a candidate whose own run of a mutant was unjudged (timed out or capped) is listed as unconfirmed only when no covering test killed that mutant, and whether the covering runs for such a mutant are queued."
  - "internal/gatecli/redundantmutants.go, the covering-run queue and the `timedOut` bookkeeping around the rerun: settle whether a covering test that timed out on a mutant as a candidate beside other runs is rerun, whether its stop is cleared before the rerun, and whether a test reaches that path."
  - "internal/testcover/runner.go `testOutput`: settle whether a test pins stamping a result line split across writes and skipping an indented subtest result line before the top-level result line."
  - "internal/gateslot/doc.go and internal/gateslot/template.yaml: settle whether any text still says a mutant run past the test bound or the memory cap counts as a kill in `lym gate redundant`."
  - "internal/gatecli/redundant.go, the early return for an empty contained set: settle whether a test asserts the answer's `unconfirmed` key."
---

Each entry names a place and a question; none states an outcome.
