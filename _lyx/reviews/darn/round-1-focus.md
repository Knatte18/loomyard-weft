---
round: 1
focus:
  - "internal/gatecli/prebuild.go: the content-addressed store, its prune and `refreshStored` republish; settle whether a concurrent gate run can lose or half-write a binary it is about to use, and whether the path stays free of a `.lyx` literal."
  - "internal/gatecli/run.go and internal/logger/trace.go: the `LYX_TRACE_ID` strip from go test's environment; settle whether any caller still needs the variable inside the test process, and whether the `logger.TraceIDEnv` literals are replaced everywhere the diff claims."
  - "internal/loomshed/verifygate.go and publishfailure.go: failing-tests-first prefix for both verify forms; settle whether test names are quoted and anchored correctly in the command and whether the round and whole-diff forms behave alike."
  - "internal/impactset/derive.go: the impacted-set command now runs the integration tier alone; settle whether vet under each tag and the build step still cover every tag the old command ran."
  - "The change description against the diff: the claim that no test file is excluded by the `integration` tag, the retention and prune statement, and the `shedbuild` test fixture change."
  - "Docs landing in the same diff: `PATTERN-test-speed.md`, `doc.go` of gatecli, gateslot, impactset, loomshed and languageconfig, and the `CLAUDE.md` push rule agree with the code; check test economy of the new `prebuild_test.go` rows against existing gatecli tests."
---

Seed focus drawn from the board entry's three asks: one tier, reused passed results, failed test first.
