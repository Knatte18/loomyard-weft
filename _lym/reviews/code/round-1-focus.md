---
round: 1
focus:
  - "internal/runplan and internal/websterengine/runplan.go: does every moved piece take its run, reports, prompts and scratch directories and its state-mutation lease from the caller's Geometry, or does any path still hard-code Webster's directories?"
  - "Every shared refusal in internal/runplan (plan-conflict, plan-partition, post-batch, suspect, await, verify-failure): does its way-forward text come from the caller, and does Webster pass the same text it produced before the move?"
  - "internal/websterengine/state.go and internal/runplan state types: do the embedded run-plan State and the batch-record type keep Webster's persisted JSON/YAML keys and load/save behaviour unchanged, with Webster-only batch-record fields optional?"
  - "internal/runplan post-batch check entry point: does it take a recorded partition mapped onto the current plan, so the later-reference check sees pending cards and only the run's last batch reads as last, and does Webster's mapping stay the same?"
  - "Report type and parser: is Journal an optional field, and does parsing a report without it behave as before?"
  - "Aliases in websterengine: does every name the decision record says moved keep an alias, with no behavioural drift in recordbatch.go, beginbatch.go, verifygatereport.go and strand.go after their large deletions?"
  - "Moved tests and export_test.go: were Webster's existing tests only edited for State nesting and aliasing, not weakened or deleted (fingerprint_test.go shows deletions; check where the coverage went)?"
  - "Scope: changes in loomshed, loomcli, shedadapters, failuretrace, hubgeom, standalonegeom and tools/tokencount against the decision record's Scope list, and any change the record does not license."
  - "Test economy: the two contained candidates in the redundancy report for runlevel_test.go and cli_test.go/parity_test.go, judged by comparing assertions with their covering tests."
---

Seed focus from the decision record's D11 and the diff stat; the arbiter entries concern Weaver's later work and are not subjects here.
