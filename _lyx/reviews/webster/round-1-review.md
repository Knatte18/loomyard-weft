---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/webstercli/reset.go:202
    summary: New error paths in the diff name no way forward and have no spec row, PATTERN-refusal-way-forward (reset after-archive uncommitted read, Run overview frame hash read)
  - id: F2
    severity: LOW
    class: consistency
    location: internal/websterengine/merriambase.go:68
    summary: New transient refusals (Merriam start base read, recovery uncommitted-path listing, rebaseline overview frame read) have no refusal-spec row of their own
  - id: F3
    severity: LOW
    class: design
    location: internal/batcher/config.go:150
    summary: MigrateConfig inserts orientation at the index of the first key dropped, not the lowest, so a weights map with startup_context before master_base gets it misplaced, or corrupted when master_base is the last key
  - id: F4
    severity: NIT
    class: consistency
    location: internal/webstercli/run.go:50
    summary: Reworded run long help reads "for an edit it accepts -- --fresh archives", a dash separator that now looks like a flag
---

Range reviewed: `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`, no rework round exists), 24 commits, one per card.
`go build`, `go vet` (default and `integration` tags), `go test ./...`, `go test -tags integration ./...` and `lyx loom lint-comments --base <merge-base>` all pass.
Every card's contract is implemented as written; the findings below are what the plan did not say but `PATTERN.md` binds.

### [BLOCKING] New error paths name no way forward and have no spec row

**Location:** `internal/webstercli/reset.go:202`; `internal/websterengine/runlevel.go:385` (`overviewFrameHash` error returned bare from `Run`).
**Issue:** `PATTERN-refusal-way-forward` requires every refusal reachable through a webster verb to name its way forward and to have a row in `contracts/specs/refusal-spec.md`.
Two paths the diff adds do neither.
`resetCmd` returns `webster: the run record was archived but the uncommitted paths could not be read: <err>` after the reset has already moved the branch, archived the run record and fabric-synced.
The message names no step, and the obvious step is wrong: re-running `lyx webster reset --to start` now refuses with `the run has no state.json`, because the record was just archived.
The envelope is also an error for an operation that completed, so a caller cannot tell the reset happened.
`Run` returns the `overviewFrameHash` error unwrapped (`websterengine: overview frame hash <dir>: read 00-overview.md: ...`), with no `way forward:` clause; the same helper's errors surface through `restampBaseline` the same way.
Neither has a row (`raw I/O` is about a run file's `stat`/`mkdir`/`read`/`write`, and a `git status` read is not one), and no test reaches either.
**Fix:** For the reset, either report the completed reset with an empty or absent `uncommitted` and a warning, or return the error with a way forward that matches the state (the reset is done; `lyx webster run` starts the new run, and `uncommitted` can be read with `git status`).
Give the `overviewFrameHash` errors a `way forward: transient, re-run the verb` clause.
Add a spec row for each in the same commit, and reach each from a test.

### [LOW] New transient refusals without their own spec row

**Location:** `internal/websterengine/merriambase.go:68` (`merriamBaseError`); `internal/websterengine/recoverbatch.go:261`; `internal/websterengine/rebaseline.go` (the `overviewIndexOnly` read failure, `transient, re-run lyx webster rebaseline`).
**Issue:** Card 8 adds a verb-level refusal for an unreadable `CLAUDE.md`, Master stencil, PATTERN directive or `00-overview.md`, card 18 one for a failed uncommitted-path listing in `recover-batch`, and card 19 one for an unreadable overview in `rebaseline`.
Each names its way forward, and `TestMerriamBase` reaches the first, but `refusal-spec.md` has no row for any of them.
Only `raw I/O` (run files) could cover them, and a stencil or git read is not a run file.
The existing `batch the cards of plan` refusal is also ungrouped, so this follows an existing gap, which is why it is LOW and not folded into F1.
**Fix:** Add one `transient` row per refusal, or widen `raw I/O` to name the stencil, `CLAUDE.md`, overview and `git status` reads.

### [LOW] MigrateConfig can misplace or corrupt `orientation` when `startup_context` precedes `master_base`

**Location:** `internal/batcher/config.go:145`
**Issue:** `insertAt` is set from the first retired key found in the loop order `master_base`, `startup_context`, not from the lowest index of the keys dropped.
With `startup_context` listed before `master_base`, the index recorded for `master_base` is two slots too high after `startup_context` is removed, so `orientation` lands after the wrong pair.
If `master_base` is the last key, `insertAt` exceeds the shortened `Content` length but not its capacity, and `weights.Content[:insertAt]` re-exposes the stale tail, duplicating keys in the output.
`TestMigrateConfig` pins each retired key alone but never both together, so nothing catches it.
A profile carrying both is unlikely, since the loader has refused `startup_context` since before this task, which is why the severity is LOW.
**Fix:** Track the minimum index across the drops (or drop in descending index order), and add a row with both keys in the order `startup_context`, then `master_base`.

### [NIT] Run long help splices a dash into `--fresh`

**Location:** `internal/webstercli/run.go:50`
**Issue:** Card 17 rewrote the sentence to `...rebaseline for an edit it accepts -- --fresh archives the stale state`.
The `--` was a separator in the old sentence; next to the new clause it reads as a flag pair.
**Fix:** Close the parenthesis after the way-forward clause and start `--fresh archives...` as its own sentence.

## Answers to the round's focus questions

1. Reset routes (cards 14, 15, 17).
   Every route that follows a `reset --to start` with a second step ends in plain `lyx webster run`, or in the row's `ReentryStep` where `Run` returns it.
   No `--fresh` follows a reset, and `git reset --keep` survives only in `gitrepo` and in `pattern/PATTERN-no-denied-recovery.md`, as a permitted form.
   `websterretiredsteps_test.go` pins both.
   The remaining `run --fresh` texts follow `rm <path>`, `restore-plan` or an unbegun-card rebaseline, as card 17 leaves them.
   Single-step refusals (`reset: no recorded target, last-batch-head` and its siblings) name only `lyx webster reset --to start`, as card 11 words them; not a defect.
2. Pending-findings guard (cards 12, 14).
   `ArchiveRunAfterReset` judges against `HeadSHA`, reads no recorded commit, and refuses only on an uncleared contract file, a differing plan path or a suspect path differing from HEAD.
   `Run --fresh` reaches `archiveRunInPlace` and `checkPendingFindings`, the same functions; there is no duplicate.
3. `ErrInvalid` (cards 1, 5, 6).
   It is set only after a present file has been read (`FillMissing`, `MissingKeys`, `Resolve`, each module's unmarshal and validation); an unreadable file and an absent one stay unmarked, and `TestLoad_Refusals` covers the unreadable case under both loaders.
   `resolvePersistentPreRun` decides the flag with `errors.As` on `configLoadRefusal`, and the Master template has the stop rung.
4. `master_base` (cards 2, 8, 9).
   It is gone from code, the template and the loader (a retired key refuses with `ErrRetiredKey`); it survives only in the migration, the read-only `RetiredMasterBase` field, the refusal text and their tests.
   `orientation` is 31400 in `template.yaml`, the base is computed only when a partition forms, and the calibrate report fits against it without refitting the template.
5. Cards 19, 20, 22, 23, 24 behave as their contracts state, including the archive-only reset, the index-only overview acceptance (`overviewFrameHash` restamped by `restampBaseline`), Plan-Write's archive-before-rotate order and `planCommitPathspec`.
6. Test economy.
   The new top-level tests are the ones the cards name, each for a behavior no existing test covers; `archivereset_test.go` is a separate file because its test needs the external package's `fakeGit`.
   No redundant test and no uncovered added behavior, other than the paths named in F1 and F2.
