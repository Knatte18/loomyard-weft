---
round: 1
focus:
  - "internal/gatecli/prebuild.go and gateslot.Pool.PrebuiltDir: settle whether the content-addressed store is safe under concurrent gate runs (staging file, rename, refresh of modification time, pruning of an entry another run still uses) and on Windows, where a rename over an open binary can fail."
  - "internal/gatecli/run.go: settle whether dropping LYX_TRACE_ID from go test's environment leaves any consumer of the variable inside a test run broken, and whether logger.TraceIDEnv is the only name used."
  - "internal/loomshed/verifygate.go and publishfailure.go: settle whether the failing-test steps run before the told or derived command and the package pass after it in both the round and whole-diff form, and whether any other caller of publishFailureSteps still assumes the old single return."
  - "Scope against the board entry: settle whether each of the four asks (integration-only verify, cached re-run on an unchanged tree, failed test first, benchmark record) is made by the diff, and whether the diff changes anything the entry does not license."
  - "Docs landing in the same diff: settle whether doc.go of the touched packages, PATTERN-test-speed.md and docs/benchmarks/gate-test-xdr.md match the behavior, and whether claims in _lyx/landing/summary.md bear out against the diff."
  - "Test economy: settle whether the new prebuild_test.go and the reworked gatecli subtests duplicate each other's coverage, and whether any added behavior has no test."
---

Seed from the board entry and the change description; no round has been judged.
