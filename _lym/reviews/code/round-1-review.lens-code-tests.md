```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that would fail if it broke.

**Per behaviour**

1. **`Seed` leaves `current_producer` empty.** `internal/loomshed/seed_test.go:55` (`checkSeedWritesExpectedStatus`) asserts `CurrentProducer == ""`. If `Seed` named a row again, it would fail. Pinned.
2. **A step reads an empty `current_producer` as the first row, calls that row, and persists the next row's name.** `TestStep_EmptyCurrentProducerStartsAtTheFirstRow` (`internal/shedengine/run_routing_test.go`, new) checks four things. `StepResult.Producer` is `Start`, the first producer is called once and the second not at all, the persisted `CurrentProducer` is `Next`, and the history has one `Start` entry. Pinned.
3. **`pause_before` naming the first row fires on an empty seed and persists that row's name.** `TestStep_EmptyCurrentProducerPausesBeforeTheFirstRow` (same file) asserts the paused state, zero producer calls and a persisted `CurrentProducer` of `Start`. This path is distinct from #2, because the pause check sits after the resolution, so it is not redundant. Pinned.
4. **`Routing.ProgressAt("")` reports the first row.** The new row "an empty producer is the start" in `internal/shedengine/progress_test.go:35` pins it. This also covers `shedverbs/human.go:55` and `step.go:266`, which call `ProgressAt`. Pinned.
5. **`RowAt` returns `current` unchanged when `producers` is empty.** No test. `ProgressAt` is not exercised with an empty producer list, and `progressOf` in `step.go` guards that case before it calls. Nothing fails if this branch is dropped, though the code reaching it today is also guarded elsewhere. Severity LOW, class `design`, location `internal/shedengine/run.go:85`. The fix is one row for an empty list, or a direct `RowAt` table test.
6. **Removal of `TestRecipe_SeedAndResumeRowNamesExist` (`internal/loomrecipe/recipe_test.go`).** It guarded that the row `Seed` hard-coded, `Preflight`, exists in a recipe. That behaviour no longer exists, so the deletion drops no coverage of live behaviour. Correct deletion. Nothing in the diff is a replacement for "seed works for every recipe", but #2 covers the mechanism recipe-independently.
7. **Smoke test `TestSmokeBootstrapLifecycle` now asserts `st.State != ""` instead of `CurrentProducer != ""`.** This is a weak assertion. The exact seeded state (`running`) is already pinned by `seed_test.go`, so no gap. NIT at most, and I raise no finding.
8. **`goto` and resume on a status with no producer name.** The diff changes no code in these verbs. `Goto` and `Relaunch` refuse `StateRunning` (`goto.go:107` and `:225`), which is the only state a fresh seed has. `gotoAdmitted` already maps an unmatched current with empty history to the first row. The empty-producer case here is unreachable. There is no new behaviour to pin and no test gap.
9. **The `anomaly.go` comment, the `loomshed.go` comment and the spec and docs edits.** These are prose, so they are exempt.
10. **Crash-resume of an empty-history seed.** The code is unchanged, and `DetectCrashResume` still excludes empty history. Its existing tests are untouched.

**Findings**
- F-tests-1: LOW, `design`, `internal/shedengine/run.go:85`. `RowAt`'s empty-producers branch has no test (see #5).

**Tool calls:** none. The inherited context already held the full diff and the relevant code.
