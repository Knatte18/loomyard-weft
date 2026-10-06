# Card 6 — logger-prune

**Edit:**
- `internal/logger#`
- `internal/loggerconfig#`
- `internal/output#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of `internal/logger`, `internal/loggerconfig` and `internal/output`.

`internal/logger` has many `Removable: no` candidates, which are folded or kept, never deleted (Decision 2).
Tests that differ only in level, verbosity or sink input are fold leads; a test that pins an exact log line or JSON envelope shape is a unique assertion under Decision 2.
`sink_callsite_integration_test.go` is the one `integration` file; its measured tests are tier-move leads (Decision 5), and one that needs a built binary or subprocess stays.
Tests that redirect a process-global sink keep the comment naming that state instead of `t.Parallel` (`PATTERN-test-speed`).

**ImpactSummary:** Test files of three logging and output packages only; no production change.

**Commit:** 6: Prune logger and output tests
