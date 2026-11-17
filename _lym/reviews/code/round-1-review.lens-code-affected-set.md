```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, find everything that calls it, reads it or describes it, and flag what the diff left wrong.

**Per change: the affected set and whether it is covered**

1. **`JudgeMutants` takes a third `unjudged` argument; `MutantVerdict.Unjudged` is new.**
   - Callers at `redundantmutants.go:183` and `:191`: covered.
   - Test `mutantverdict_test.go`: covered.
   - Package doc `internal/testcover/doc.go:52-55`: covered.

2. **`runMutants` returns kills and unjudged runs; timeouts and capped runs no longer count as kills; a covering test that timed out reruns alone.**
   - Caller `run`: covered.
   - Test `redundantmutants_guard_test.go:76`: covered.
   - `internal/gatecli/doc.go:1198-1208`: covered.
   - `mutationStats` comment: covered.
   - `runJob` doc: covered.
   - `internal/gateslot/doc.go:180`: **not covered**.
   - `internal/gateslot/template.yaml:6-7`: **not covered**.
   - `docs/research/speed/map-tooling-trial-2026-11-17/report.md:73`: a measurement report, exempt.

3. **The `lym gate redundant` output gains an `unconfirmed` key; refuted entries now come from `Unique` only.**
   - `redundant.go`: the `Long` help and both answer sites are covered.
   - `gatecli/doc.go:1221-1223`: covered.
   - `docs/overview.md:966-967`: covered.
   - `pattern/PATTERN-test-speed.md:315-319`: covered.
   - Integration test: covered.
   - `README.md:78` names no output keys: unaffected.
   - Nothing else reads the key: no stencil or other doc does.

4. **The `Long` help text was trimmed.** It dropped the operator list, and dropped the removable-set rule ("slowest first … every block stays covered and every mutant killed").
   - No test asserts the removed text.
   - The removed rule is still stated in the doc at `gatecli/doc.go` and in `testcover/doc.go:56`. Covered.

5. **`RunResult.Elapsed` is now the time from the `=== RUN` line to the result line; `Toolchain.Now` is new.**
   - Consumers:
     - `testmappackage.go:638` (map `Seconds`, doc at `coveragemap.go:74` says "standalone duration", still true)
     - `redundantmutants.go:236` (`mutantTimeout`, held up by its floor)
     - `internal/gatecli/redundancy.go:564`
     - `cmd/testtiming/redundancy.go:87`
     - the `Elapsed` docs in `verdict.go:24` and `mutantverdict.go:23`

     None claims the binary's own figure. Covered.
   - `testcover/doc.go:16-17`: covered.
   - Runner test: covered.

6. **`pattern/PATTERN-test-speed.md` background edit.** The directive declares no PATTERN changes, so none needs landing. The `PATTERN.md` one-line summary of `PATTERN-test-speed` does not describe kill counting. Covered.

7. **Commit messages.** The range has no `Pattern-Departure:` trailers. Nothing to judge.

**Findings**

1. MEDIUM, class `consistency`: `internal/gateslot/doc.go:180`
   - Summary: the package doc still says "A mutant run past the test bound or the memory cap counts as a kill in `lym gate redundant`." The diff reverses that: such a run is now unjudged, never a kill.
   - Evidence: `internal/gateslot/doc.go:180`, against `internal/gatecli/redundantmutants.go:252-262` (the `record` switch no longer adds a kill for capped or timed-out runs).

2. MEDIUM, class `consistency`: `internal/gateslot/template.yaml:6-7`
   - Summary: these are the operator-facing config comments. For `redundancy_test_timeout_sec` the comment says a mutant run in `lym gate redundant` "then counts as a kill". For `redundancy_test_memory_mb` it says a mutant run "counts as a kill". The diff makes both unjudged.
   - Evidence: `internal/gateslot/template.yaml:6`, `:7`; `internal/gatecli/redundantmutants.go:252-262`.

Tool calls:
1. Grepped the repo for kill, time-bound and memory-cap wording and for the old timing wording. The context only held the earlier grep of three files' kill phrases.
2. Grepped for every file naming `gate redundant`, and for tests asserting the removed help text. Docs outside the manifest and help-text assertions were not in context.
3. Grepped `README.md`, `template.yaml`, `testmap.go` and `PATTERN-daemon-wakeups.md` for redundant-verb wording. The template's config comments were not in context.
4. Grepped the `Seconds`/`Elapsed` field docs that receive `RunResult.Elapsed`. Their wording was not in context.
