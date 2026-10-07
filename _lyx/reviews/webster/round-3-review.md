---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/loomcli/resume.go:47
    summary: lyx loom resume reads the status lock and probes the run lock without creating their ephemeral directory, so it fails with a raw lock-open error on a worktree whose scratch directory is absent
  - id: F2
    severity: LOW
    class: consistency
    location: internal/loomcli/reviewstatus_test.go:140
    summary: TestShuttleWaiting doc comment joins three independent clauses on one line after commas, against PATTERN-comment-line-breaks
  - id: F3
    severity: LOW
    class: consistency
    location: docs/overview.md:472
    summary: changed markdown lines join independent clauses with a semicolon or a comma plus and on one line, against the repo's semantic-line-break rule
---

Range: `product.parent` is `main` and no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
`go build ./...`, `go vet ./...`, `go vet -tags llm ./...`, `go vet -tags tmux ./...`, `go test ./...` and `go test -tags tmux ./internal/reedengine/... ./internal/reedcli/...` all pass on HEAD.
No card carries a `Rename` or `Delete` group, so the per-card mechanical check has no AST-script or `assert-no-callers` target; each card's `Edit`/`Create` list matches the files the diff touches.

### [MEDIUM] `lyx loom resume` assumes the run's scratch directory exists

**Location:** `internal/loomcli/resume.go:47` (the status read) and `internal/loomcli/resume.go:121` (`runLockHeld`)
**Issue:** The status file is durable under `_lyx/shed/<run>/`, but `status.json.lock` and `run.lock` live in the ephemeral `shedrun.ScratchDir` (`internal/shedrun/paths.go`).
`state.ReadJSONStrict` "does not create missing parent directories" and `lock.TryAcquireWriteLock` opens with `O_CREATE` but never creates a parent.
`resumeFromRunState` calls both on the scratch directory with no `MkdirAll` before them.
On a worktree where the status file exists and `.lyx/shed/<run>/` does not (a pair re-created from the branch), the verb refuses with a raw `acquire read lock: open …: no such file or directory` and names no way forward.
The sibling verbs already guard this: `approve.go` creates `filepath.Dir(StatusLockPath)` before its read, and `arm.go`'s `loomPreStepUnmarked` carries a comment explaining that the run-lock probe must create the directory first.
Batten's `runLockHeld` in `internal/battencli/wire.go` does the same.
`TestResumeVerb` always builds the scratch directory, so no row reaches this case.
**Fix:** Create `filepath.Dir(c.shedPaths.StatusLockPath)` before the status read in `resumeFromRunState`, and `filepath.Dir(c.shedPaths.LockPath)` before the probe in `runLockHeld`, as `arm.go` does.
Add a row to the `TestResumeVerb` table, not a new test, whose fixture has a status file and no scratch directory, and assert the verb answers the run's state instead of a lock-open error.

### [LOW] `TestShuttleWaiting` comment shares a line between independent clauses

**Location:** `internal/loomcli/reviewstatus_test.go:140`
**Issue:** The comment reads "asserts a live shuttle marker is reported when no verify runs, a live verify marker is reported ahead of it, and a dead shuttle marker falls through to the review note" on one line.
Each of the three has its own subject and verb, which `PATTERN-comment-line-breaks` requires a break between.
Round 2's F1 broke the same shape in `TestDriverStrandFrom`, so this is the one instance left in the added or changed Go comments.
**Fix:** Break the comment after each comma, one clause per line, as `TestDriverStrandFrom`'s comment does.

### [LOW] Changed markdown lines keep independent clauses on one line

**Location:** `docs/overview.md:472`, `docs/overview.md:473`, `pattern/PATTERN-completion-signal.md:15`, `pattern/PATTERN-completion-signal.md:17`, `contracts/specs/loom-status-spec.md:76`, `contracts/specs/loom-status-spec.md:83`, `contracts/stencils/orch/orch-template-role.md:18`
**Issue:** CLAUDE.md requires semantic line breaks in every `.md` file, one sentence per line and a break at a clause boundary with its own subject and verb.
The lines this branch added or changed carry such boundaries:
- `docs/overview.md:472` has two semicolons before "the reason, … is what batten logs" and "`lyx batten pause <slug>` stops the wait".
- `docs/overview.md:473` has "; a retiring strand is left to `lyx loom start`".
- `pattern/PATTERN-completion-signal.md:15` has "; a gated run's strand carries none, …", and `:17` has ", and the relaunch types no prompt".
- `loom-status-spec.md:76` has ", and the first one with a live pid wins", and `:83` has ", and the attempt clause of a verify is dropped".
- `orch-template-role.md:18` has ", and you take it".

The `PATTERN.md` entry lines are one line each by that file's own format and are not part of this finding.
**Fix:** Break each of those lines at the named boundary, keeping the wording; touch no other line.
`contracts/stencils/claims_test.go` matches stencil phrases, so run `go test ./contracts/stencils/...` after the stencil edit.

## Focus directive verdicts

- **Comment line breaks:** one remaining instance, F2.
  Every other added or changed Go comment line that carries a semicolon or a comma plus "and", "but" or "or" joins a compound predicate, a list, or a parallel "that" complement of one verb ("asserts X, and that Y"), so it needs no break.
  No added comment line holds two sentences.
- **Test parallelism in `internal/reedengine`:** not a defect.
  `TestAddStrandUnless_NamedStrandSkips`, `TestAddStrandUnless_NoNamedStrandAdds`, their subtests and `TestWaitMarkSegmentExpandsOnARealTmux` all call `t.Parallel`.
  The other added tests without it (`internal/battenshed/innerrun_halted_test.go`, `internal/shuttleengine/waitmark_test.go`) name the process-global state they touch in their file headers.
- **Coverage of added behavior:** not a defect.
  `reedDriverDirectory.DriverRow` is reached by `TestReedDriverDirectory_DriverRow`, which covers the first driver row among others, none, and a passed-through directory error.
  The `ChildRunLockHeld` closure's body is `runLockHeld`, which `TestRunLockHeld` drives over a real lock file for an absent directory, a held lock, and a released probe, and `TestWire_BuildsLazilyForANonexistentTaskWorktree` pins the closure as wired.
