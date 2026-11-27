```yaml
lens: code-affected-set
outcome: reported
```

Lens task: code-affected-set. For each changed surface, I checked what reads or describes it and whether the diff leaves any of it wrong.

**Per change: the set found, covered or not**

- **`shuttleengine.RerunMark` and `GateEntry.Rerun` removed.** Covered: the callers in `gatefixer/rowgate.go` (`fixerEntry`, `waitOnly`, `spawning`, `spawnAndJudgeAgain`), the `gatefixer/doc.go` line, the `rowgate_test.go` rows, the `shedrecipe/entries_gate.go` doc and code, and `entries_gate_test.go`. Not covered: the doc comment of `spawnAndJudgeAgain` (F1).
- **`verifytree.WithReachTurn`, `ReachPending`, `ReachListedPath` removed.** Covered: `verifytree/doc.go`, `weaverengine/geometry.go` and `strand.go`, `loomcli/wiring.go` and `wiring_test.go`, `envkit` `nilLegal`, `shedrecipe` `Env.VerifyReach`. `go vet` passes on these packages. `WriterOnly` stays live through `landingshed/describereview.go:246`, so the rowgate row on it still tests a real producer. Covered.
- **Rubric markers `redundancy_report`, `reach_report`, `mutation_report` removed.** Covered: the stencil `loom-rubric-code-review.md`, `contracts/stencils/rubric_test.go`, `shedadapters/rubric.go` and `bouncer.go` with their tests, `shedrecipe` `Env`, `entries_bouncer.go`, `entries_burler.go` with their tests, the `loomcli/doc.go` paragraph and the note files. The repo-wide search finds no other stencil or spec naming them. Covered.
- **Config keys `redundancy_*`, `reach_budget_min`, `mutation_*` and `GateKey.Reports` removed.** Covered: `gateslot` config struct, spec categories, bounds, `decodeGates`, `Gate()`, template, doc and `TestLoadConfig`; `gatecli/keynames.go`; the `loomcli` test fixtures that string-match template lines (`landingdeps_verify_test.go`, `wiring_darn_test.go`); `docs/overview.md`. Covered.
- **Verbs `gate reach`, `gate mutate`, `gate redundant` and `test --map` removed.** Covered: `cli.go` help and flags, `detachfront.go` `words()`, the `cli_test` and `detachfront_test` rows, `gatecli/doc.go`, README, overview, PATTERN text. The repo-wide search outside `docs/research` finds no remaining mention. Covered.
- **Process-cap and memory-guard carve-out, `guardOwnTests`, `startMemoryGuard`.** Covered: `tierrun.go`, `run.go`, `cli.go` fields, the doc paragraphs in `gatecli/doc.go`, `PATTERN-daemon-wakeups.md`, the `memory_quit_mb` template comment.
  - `proc.MemoryCapCoversTree` lost its last production caller (`redundantmutants.go:565` at base). The allowlist entry fits the background's rule that a function reading its package's own state takes an entry naming its tests (`pattern/PATTERN-test-only-code.md`, Rule bullet 5), and a `Loosened-Check` trailer names it. Covered.
- **The gateslot doc's list of non-tier holders emptied** (`gateslot/doc.go` hunk at lines 43–51). Not covered: `PATTERN-gate-slot.md` still points at that list (F3).
- **`PATTERN-test-speed` rewrite.** `PATTERN.md` and the background file are both updated, and `PATTERN-coverage-map-reader` is removed together with its enforcing test `cmd/lym/coveragemapreader_test.go`. One new sentence is wrong (F2).
- **`testcover` cut.** `Toolchain` lost `Dir`, `Ctx` and `BuildTimeout`, `GuardKilled` was removed, `JudgeAgainst` and `JudgeMap` went, and the `SpawnVerdict` JSON tags were dropped. The only consumer, `cmd/testtiming/redundancy.go:314,331`, uses `Go`, `Env`, `TestTimeout`, `MemoryLimit` and `Grace` only, and never marshals `SpawnVerdict`. `testcover/doc.go` and the `docs/overview.md` line are updated. Covered.
- **`testreach` cut, with `Module` and `FileSet` moved into `changedtests.go`.** Callers are `gatecli/gomodule.go` and `redfirst.go`; `testreach/doc.go` and the overview line are updated. Covered.
- **`lymdirs.TestCoverDirName` removed.** No reader is left. Covered.
- **Commit trailers.** Both `Loosened-Check` trailers on `3026c4337` name the check and the guard's measured report. No `Pattern-Departure` or `Root-Cause` trailers. Nothing to flag.
- **Declared PATTERN changes.** The directive declares none, so nothing is raised.

**Findings**

- **F1.** LOW, consistency, `internal/gatefixer/rowgate.go:205`.
  - **Summary:** the doc comment of `spawnAndJudgeAgain` still says "inner run under the entry's re-run mark", but the mark is removed and the function now calls `w.waitThenRun(store, rec, inner, true, true)` directly (line 226).
  - **Evidence:** `rowgate.go:204-226`; `shuttleengine/gate.go` diff removing `RerunMark`.
  - **Fix:** drop the clause ", inner run under the entry's re-run mark".
- **F2.** LOW, consistency, `pattern/PATTERN-test-speed.md:222`.
  - **Summary:** the new bullet says `cmd/testtiming -redundancy` "takes a tier's `env` and a gate slot as a tier run does". testtiming reads no tier's `env` from `gate.yaml`: it sets `LYM_TEST_TIER` from its tags and its go flags through `gateslot.HolderEnv`, and every mode, not only `-redundancy`, holds one slot.
  - **Evidence:** `cmd/testtiming/resources.go:130-141` (`withGateSlot` returns `HolderEnv(os.Environ(), lease, tier0Parallel)`); `cmd/testtiming/main.go:10-13,30-32`.
  - **Note:** the base sentence it replaces described the gate's own redundancy runs, not testtiming.
  - **Fix:** say that `cmd/testtiming`, in every mode, holds one gate slot through `gateslot.HolderEnv` and sets `LYM_TEST_TIER` from its tags. Keep the coverage-binary sentence.
- **F3.** LOW, consistency, `internal/gateslot/doc.go:43-45`, with the stale reader at `pattern/PATTERN-gate-slot.md:15`.
  - **Summary:** the diff removed every holder the gateslot doc named (the redundancy step, `lym gate redundant`, `lym gate mutate`, the mutation step). The doc now names no non-tier holder, while `PATTERN-gate-slot.md` still places a gate build or test "in one of the non-tier holders `internal/gateslot`'s doc lists". `cmd/testtiming` is a remaining holder (`resources.go:147-163`, `pool.Acquire` with `HolderEnv`) and is not listed.
  - **Evidence:** `gateslot/doc.go:43-45` as left; `PATTERN-gate-slot.md:15` in the diff's context lines; `cmd/testtiming/resources.go:147-163`.
  - **Fix:** name the remaining non-tier holders, at least `cmd/testtiming`, in the gateslot doc.

**Tool calls**

1. `grep` for `WriterOnly` producers and leftover reach-turn text, and `weaverengine/strand.go` 120–150: needed to know whether `WriterOnly` and the batch gate doc still had live meaning after the reach turn left.
2. `grep` for remaining map text, plus `proc/doc.go` 50–65 and the range's trailers: needed the repo-wide map wording and the `proc` probe's doc for the allowlist change.
3. `pattern/PATTERN-test-only-code.md`: needed to judge whether the allowlist entries for `MemoryCapCoversTree` fit the rule.
4. `gateslot/doc.go` 30–50: needed to see what non-tier holder list survives the hunk, for `PATTERN-gate-slot`'s reference.
