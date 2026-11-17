---
round: 1
focus:
  - "Scope against the board entry's three parts: the diff touches the timeout/memory-cap verdict and the per-test seconds, but shows no change that sorts each test's `funcs` and `blocks` before the map is written. Settle whether the map writer in `internal/testcover` already guarantees that order (check where a test's `funcs` and `blocks` are filled and written) or whether the funcs-order part is unmet."
  - "`internal/testcover/mutantverdict.go` and `internal/gatecli/redundant.go`: settle whether a candidate whose confirmation rests on a timed-out or memory-capped run is always listed as unconfirmed, with those mutants, and never confirmed; check the case with a unique kill beside an unjudged run, the case with several covers, and a mutant that both killed and timed out in different covering tests."
  - "`internal/gatecli/redundantmutants.go` and `internal/testcover/runner.go`: settle whether a timeout and a memory-cap end are both detected, from the run's real outcome, and kept apart from a genuine failing test; check that a covering test that timed out is rerun alone like one that killed, and that the rerun's own timeout is handled the same way."
  - "`internal/testcover/runner.go` per-test seconds: settle whether the RUN-line to result-line time is correct for subtests, parallel tests, a test with no RUN line, and output split across reads, and whether the recorded resolution now separates tests as the entry asks."
  - "Test economy: `internal/gatecli/redundant_integration_test.go`, `internal/testcover/mutantverdict_test.go` and `internal/testcover/runner_test.go` changes; settle whether any assertion duplicates a covering test or whether the unjudged-run behavior and the per-test timing lack coverage."
  - "Docs and pattern: `docs/overview.md`, the `doc.go` files and `pattern/PATTERN-test-speed.md` changes; settle whether each states the new behavior accurately against the code and whether any other doc naming the old kill-count behavior was left stale."
---

Seeded from the board entry and the committed range's stat and commit messages.
