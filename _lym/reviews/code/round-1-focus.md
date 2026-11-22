---
round: 1
focus:
  - "internal/gatecli/reports.go, run.go: concurrency of the report chain with the last tier. Settle whether every path (tier failure, lint failure, halt, cancel, early return, panic) stops and waits for the chain, with no leaked goroutine or process and no report written when the call halts."
  - "internal/gatecli/reports.go: held output lines and the judged reports. Settle whether output order and report content match the former sequential tail, and whether a report written beside a running tier can read a stale or partial tree or race with the tier's own files."
  - "internal/gatecli/memoryguard.go: withoutForeign and runningCommands holds/add/remove. Settle whether the guard still protects the report steps' own test binaries, and whether the shared command registry is race-free and always released."
  - "internal/gatecli/run.go: where the steps start (beside the first run of the highest tier, after every earlier run passed) and where red-first stays. Settle whether the start condition and the red-first placement match the Goal, including a plan with a single tier or a gate key with no report steps."
  - "Scope against the decision record: the Goal asks for the reports to run beside the tier 2 tests, or only on the last gate, and drops the before-and-after timing. Settle whether internal/gateslot/template.yaml and tiers.go, doc.go and the commit message claim only what the diff does, and whether the change description makes any timing claim without figures."
  - "Test economy: reportSteps.any, reportStepsOf and lastTierStart are reached by no mapped test. Settle whether each is covered through TestReportChain or another test, and whether the new tests assert independent literals rather than recomputing the code's own result."
---

Seed focus from the decision record's Goal and the diff stat. The mutation report judged no mutants, so no survivor is a lead.
