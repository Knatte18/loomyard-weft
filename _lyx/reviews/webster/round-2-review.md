---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/websterengine/recoverbatch.go:261
    summary: recover-batch's uncommitted-paths refusal has a spec row but no test reaches it
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/websterengine/runlevel.go:385
    summary: Run's overview-frame refusal is reachable only by a file race, so no test reaches it
  - id: F3
    severity: LOW
    class: consistency
    location: internal/websterengine/rebaseline.go:182
    summary: the index-only read refusal tells the operator to re-run rebaseline without --card, which an unnamed changed card refuses
  - id: F4
    severity: LOW
    class: scope
    location: internal/batcher/config_test.go:364
    summary: TestMigrateConfig pins the orientation insert index only for startup_context first, so a last-drop-index regression passes
---

Range: `_lyx/loom/rework/` does not exist, so the range is `git merge-base main HEAD..HEAD` (`product.parent` is `main`).
Round-1 files were not read before this review was saved.

### [BLOCKING] recover-batch's uncommitted-paths refusal is untested

**Location:** `internal/websterengine/recoverbatch.go:261` (`recoverSpawn`, the `uncommittedPathsBlock` error), `uncommittedPathsBlock` at `recoverbatch.go:~183`.
**Issue:** The branch adds a refusal: when `UncommittedPaths` or `loadRunWrites` fails, `recoverSpawn` returns `webster: list the worktree's uncommitted paths for batch ...; way forward: transient, re-run \`lyx webster recover-batch NN\``.
The spec row exists (`contracts/specs/refusal-spec.md`, "recover-batch: uncommitted paths unreadable"), and its wording matches the message.
No test reaches it: `recoverbatch_git_test.go` has `TestRecoverBatch_UncommittedPathsPrompt` for the success path only, and a grep of the webster tests finds no failing `DirtyPaths` or write-evidence read against `recover-batch`.
`PATTERN-refusal-way-forward` requires that a test reaches every new refusal, and `Geometry.git()` takes a fake, so the failure is injectable.
**Fix:** Add a row or subtest that makes `DirtyPaths` fail during `recoverSpawn` and asserts the refusal names `recover-batch NN` and `transient`, and that no strand was started and no state was mutated.

### [BLOCKING] Run's overview-frame refusal is not reachable by any test

**Location:** `internal/websterengine/runlevel.go:385`.
**Issue:** `Run` parses the plan (`planparser.ParsePlan`, which reads `00-overview.md` and fails on a missing Card Index), then re-reads the same file in `overviewFrameHash`.
The only way the second read fails after the first succeeded is the file changing between the two reads, so no test can reach `way forward: transient, re-run \`lyx webster run\`` here.
A grep of `runlevel_test.go` and the webstercli tests finds none.
The refusal-spec row "overview frame unreadable" lists `run`, so the row claims a refusal that is untested, which `PATTERN-refusal-way-forward` forbids.
Only the `rebaseline` and restamp paths are covered, by `TestRebaseline_OverviewFrameUnreadable`.
**Fix:** Either derive the frame hash from the bytes `ParsePlan` already read, which removes the second read and the refusal, and drop `run` from the spec row; or give `overviewFrameHash` an injectable read so a test fails it after the parse and asserts the message.
The first removes a branch no caller can act on, so it is the simpler choice.

### [LOW] Index-only read refusal drops --card from the re-run

**Location:** `internal/websterengine/rebaseline.go:182`; spec row "overview frame unreadable".
**Issue:** The refusal ends `transient, re-run \`lyx webster rebaseline\``.
It is reached while `Rebaseline` loops over changed files, with `deps.Cards` already supplied.
A bare re-run omits the `--card NN` the operator passed, so when card files also changed the re-run refuses with "rebaseline unnamed card changed", a detour rather than a dead end.
Every other rebaseline way forward in the spec and in this file names `--card NN`.
**Fix:** Word it as re-running the same command with the same `--card` flags (for example "re-run `lyx webster rebaseline --card NN` with the cards you named"), and update the spec row and `TestRebaseline_OverviewFrameUnreadable`'s expected suffix to match.

### [LOW] TestMigrateConfig does not pin the lowest-index rule on both orders

**Location:** `internal/batcher/config_test.go:364`; implementation `internal/batcher/config.go:145-159`.
**Issue:** The focus question settles as follows.
With both retired weights present, `MigrateConfig` inserts `orientation` at the lowest index either drop held, which is correct: it drops `master_base` first, then `startup_context`, and the minimum of the two indices is valid in the final slice for both file orders.
The test covers only `startup_context` before a trailing `master_base`.
A regression to "insert at the last drop's index" passes that row, because there the last drop is also the lowest.
With `master_base` first and `startup_context` after a custom key (`[master_base, custom, startup_context]`), the correct result is orientation first, and the last-drop variant puts it after `custom`; no row distinguishes them.
**Fix:** Add one row to `TestMigrateConfig` with `master_base` first, a custom key, then `startup_context`, expecting `orientation` where `master_base` stood.

## Settled focus questions with no finding

- `internal/webstercli/reset.go`, the uncommitted-path read after the archive: it is a warning in the success envelope, not a refusal, so it needs no row.
  The message names `git status` in the task worktree, which matches the state (the reset is done), and `reset_test.go:436` reaches it.
- `internal/websterengine/merriambase.go`: `merriamBaseError` has its own spec row, "Merriam start base unreadable", whose trigger lists exactly the files read, and `merriambase_test.go` reaches it through the absent-overview subtest.
  The one helper builds every variant, so one subtest is enough.
- `internal/websterengine/rebaseline.go` index-only read: has the "overview frame unreadable" row and is reached by `TestRebaseline_OverviewFrameUnreadable`; the way-forward wording is F3.
- `internal/webstercli/run.go` help text: the reworded clause reads `...it accepts; --fresh archives the stale`, with `--fresh` mid-line after a semicolon.
  Nothing in it can be read as a flag.

## Prior-round check

Read after this review was saved.
The round-1 fixes hold: the reset's uncommitted read is a warning, the spec has the three transient rows, `MigrateConfig` takes the lowest index, and the help text uses a semicolon.
Round 1's F1 asked for a test reaching each new path; the fixer added them for the reset warning and for `Rebaseline`, and deferred the `Run` one for lack of a seam.
F2 above re-evaluates that deferral, since `PATTERN-refusal-way-forward` has no exception for a missing seam and the second read can be removed instead.
F1 above is a path round 1's F1 named under the same rule that no round-1 fix reached.
