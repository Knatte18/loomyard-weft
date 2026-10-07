---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/battenshed/doc.go:20
    summary: Go comments this branch added or changed still join two independent clauses on one line at a comma before so, against PATTERN-comment-line-breaks
  - id: F2
    severity: BLOCKING
    class: consistency
    location: docs/overview.md:477
    summary: Markdown lines this branch added or changed still join independent clauses on one line (overview.md, PATTERN-completion-signal.md, loom-status-spec.md, orch-template-role.md), against the semantic-line-break rule in CLAUDE.md
---

Build, `go vet` and `go test ./...` pass on the branch head, and `GOOS=windows go build` of `verifyrun` and `verifytree` is clean.
The review range is `git diff $(git merge-base main HEAD)..HEAD`: `_lyx/loom/rework/` does not exist, so no live round narrows it.

### [BLOCKING] Go comments still join independent clauses at a comma before "so"

**Location:** `internal/battenshed/doc.go:20,23,28`; `internal/battenshed/innerrun.go:37,49,206,337`; `internal/battenshed/innerrun_halted_test.go:2,288`; `internal/battenshed/innerrun_test.go:457`; `internal/landingshed/deps.go:159,165`; `internal/landingshed/verifygate.go:21`; `internal/loomcli/driverlaunch.go:53`; `internal/loomcli/resume_test.go:47`; `internal/loomcli/reviewstatus.go:78`; `internal/reedengine/doc.go:50,53`; `internal/reedengine/lifecycle.go:693`; `internal/reedengine/windowsize.go:141,148`; `internal/shedverbs/spec.go:83`; `internal/shuttleengine/doc.go:64,151`; `internal/shuttleengine/run.go:360`; `internal/shuttleengine/waitmark.go:28,92,102`; `internal/shuttleengine/waitmark_test.go:2`; `internal/verifyrun/procgroup_windows.go:14`; `internal/websterengine/verifygate.go:254`

**Issue:** `PATTERN-comment-line-breaks` and `CLAUDE.md` require a break at an independent-clause boundary in every new or changed Go comment.
Each listed line carries `, so <subject> <verb>...` (or a colon-then-`, so` form) on a single line, where `so` is the coordinating conjunction and each side has its own subject and verb.
Earlier rounds broke the semicolon and `, and` cases, and left the `, so` ones.

**Fix:** Break each line after the comma, so the clause that starts with `so` is a line of its own.
Leave untouched lines, the compound-predicate lines (`X, Y, and Z` sharing one subject), and the `tmux` argument list quoted in `windowsize.go` as they are.

### [BLOCKING] Markdown lines still join independent clauses on one line

**Location:** `docs/overview.md:347` (the verify-gate parenthesis `...flaky; a run that exceeds...` and `, and a failure that survives goes back...`), `docs/overview.md:477`, `docs/overview.md:479`, `docs/overview.md:480`; `pattern/PATTERN-completion-signal.md:18`; `contracts/specs/loom-status-spec.md:88`; `contracts/stencils/orch/orch-template-role.md:38,40`

**Issue:** The focus asked whether any changed `.md` line still joins independent clauses.
`overview.md:477` joins `which brings ... in place` and `so a later lyx loom resume wakes that driver`, line 479 joins its colon clause and `so a restarted batten spawns the bootstrap again`, line 480 joins `batten records the confirmation ... and spawns nothing` and `so a second driver is never stacked`, and line 347 carries a semicolon and a `, and a failure that survives...` clause inside one long line.
`PATTERN-completion-signal.md:18` joins `Reed cannot know that a gate passed` and `so a finished gated run's strand is still relaunched`.
`loom-status-spec.md:88` joins `It is read from the markers and the round store, never from the status file` and `so the file's schema above is unchanged`.
`orch-template-role.md:38,40` join a clause and an imperative `so run ...` that carries its own verb.
`internal/loomcli/reviewstatus_test.go` has no remaining violation: its comments already break at the clause boundaries.

**Fix:** Break each line at the named boundary, with the `so`/`and` clause starting the next line.

### Focus verdicts

- **Scratch directory before the lock opens (`internal/loomcli/resume.go`):** not a defect.
  `resumeFromRunState` calls `os.MkdirAll(filepath.Dir(c.shedPaths.StatusLockPath), 0o755)` before `state.ReadJSONStrict`, and `runLockHeld` calls `os.MkdirAll(filepath.Dir(c.shedPaths.LockPath), 0o755)` before `lock.TryAcquireWriteLock`.
  A worktree with a status file and no scratch directory therefore gets the run's state, as `approve.go` and `arm.go` do.
- **`TestResumeVerb` coverage:** not a defect.
  The row `running with the scratch directory absent answers from the run's state` points both lock paths at a directory that does not exist, and a `running` status reaches both the status read and the run-lock probe.
  It is a table row, so `PATTERN-test-economy` holds, and removing either `MkdirAll` makes it fail.
- **Per-card mechanical checks and test economy:** no finding raised, from a read of the reed, shuttle, verify and loomcli diffs rather than a re-run of each card's check.
  The build, vet and full suite pass, and I found no new test that duplicates a covering test.
