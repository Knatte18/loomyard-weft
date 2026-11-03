```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (code-tests)**

1. **Child-recipe record** (`ChildRecipeFile`, `ChildRecipeRel`, `WriteChildRecipe`, `ReadChildRecipe`): pinned by `TestChildRecipe` (`internal/shedrun/childrecipe_test.go:15-34`). It covers the path, the relative form, empty-and-nil before a write, and the round trip with a replace.
2. **Status commit carries the child-recipe record** (`battenRunCommitPaths`): pinned by `TestBattenRunCommitPaths` (`commitstatus_test.go:180-186`).
3. **Seed-Child's `WriteSeed` seam writes the child recipe after the child seed** (`wire.go:1417`): **none.** The grep finds no test that reaches this closure. The behaviours with no row are:
   - the record holds the recipe the child was seeded with;
   - the record is written only after `shedrun.WriteSeed` succeeds;
   - a record write failure propagates out of the seam.
   Only `shedrun.WriteChildRecipe` is tested directly. Removing line 1417 would break no test, and `ClearMotifRun` would then never clear in production.
4. **`clearMotifRun` judgement**: `TestClearMotifRun_RemovesOnlyADoneMotifRun` (`cleanup_test.go:542-612`) pins these rows:
   - a done motif run is removed, committed and pushed;
   - a done darn run stays;
   - a done run with no record stays;
   - a running motif run stays;
   - a slug with no status stays;
   - a held run lock stays.
5. **`clearMotifRun` paths with no row:**
   - **A records commit failure is returned** (`cleanup.go`, the `case err != nil: return false, err` branch). There is no commit-failing stub row, and it is the path `TestCreate_StepFailureNamesTheRerun` fakes at the `beamcli` seam.
   - **A scratch-empty failure only warns and `cleared` is still true.** No row.
   - **A push-spawn failure only warns and `cleared` is still true.** No row.
   - **Done and motif but not tracked in the records HEAD:** it commits nothing, spawns nothing and reports `cleared` true. The test's stubs make every status tracked (`stubs.committed[StatusRel]`), so this untracked branch has no row.
   - **An undecodable status:** `judgeStatus` reports not done, so it returns false with a nil error. No row.
   - **A `ReadChildRecipe` error:** it returns an error. No row.
   - **`ClearMotifRun` outside the prime or a beam:** it returns the `ResolveBattenPrime` refusal. No row in this diff, though `ResolveBattenPrime` has its own tests elsewhere.
6. **`createSteps` clearing guard** (`found && Done && !PairPresent`): pinned by three `TestCreate` rows:
   - `DoneMotifRunWithItsPairGoneIsClearedBeforeThePawlRunStarts` (calls `owner`, `clear`, `start`; `MotifRunCleared` and `RunStarted` set);
   - `DoneRunOfAnotherRecipeIsLeftForTheStartToRefuse` (`clear` called, nothing cleared);
   - `DoneRunWhosePairIsStillHereIsNotOfferedForClearing` (no `clear` call).
   Whether a not-found or running run skips `clear` rests on the existing rows' `wantCalls` lists, which I did not re-read. They would fail if the guard ignored `found` or `state`.
7. **A clearing error is the verb's error naming the re-run:** pinned by `TestCreate_StepFailureNamesTheRerun/MotifRunClearFailure`.
8. **The `motif_run_cleared` envelope key:** no test pins the JSON tag. `TestCreate` asserts the `MotifRunCleared` struct field only. This is LOW; the key is an operator-visible output shape the change description names.
9. **`hubCreateDeps`'s `ClearMotifRun` closure** (`deps.go`): none, a one-line pass-through, and I raise no finding.

**Findings**

- **MEDIUM, `design`**, `internal/battencli/wire.go:1417`: the child-recipe write in the `WriteSeed` seam is untested. It is the only producer of the record that `clearMotifRun` depends on, so deleting it passes every row while the feature goes dead. Evidence: the grep shows no test touching this closure, and `wire.go:1410-1417` shows the write.
- **LOW, `design`**, `internal/battencli/cleanup.go` (`clearMotifRun`, the commit-error branch and the two warn-only branches): the error-versus-warn split behind `cleared`, which the focus names, has no row.
  - **Commit failure:** returns `(false, err)`.
  - **Scratch-empty failure:** warns and returns true.
  - **Push-spawn failure:** warns and returns true.
  - **Untracked done motif run:** removed with no commit and no spawn.
  They fit as rows in `TestClearMotifRun_RemovesOnlyADoneMotifRun` through the existing stubs, not as new tests. Evidence: `cleanup.go` `clearMotifRun` lines 150-170 and `cleanup_test.go:546-612`.
- **LOW, `consistency`**, `internal/beamcli/create_test.go:300-343`: no assertion on the `motif_run_cleared` JSON key.

**Test economy:** I found no duplicate. `TestClearMotifRun_RemovesOnlyADoneMotifRun` and `TestChildRecipe` each cover a behaviour no other test covers. The new `TestCreate` rows are rows, not new functions. The `TestBattenRunCommitPaths` addition is an assertion in the existing test.

**Tool calls**
1. A `grep` over `internal/**/_test.go` for the child-recipe and clear symbols: my context did not show whether any test reaches the `wire.go` seam or asserts the `motif_run_cleared` key.
