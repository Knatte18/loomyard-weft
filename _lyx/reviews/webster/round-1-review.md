---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: cmd/lyx/stencilseed_integration_test.go:57
    summary: two new test packages each declare their own thread-safe logger sink and captureLogs helper, duplicating the shared logcapture kit that PATTERN-testkit requires
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/fabriccli/fabric.go:778
    summary: the new reconcile failures for a hub-wide config commit or push failure and a warp-binding commit failure or present-path push failure have no test
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/fabricengine/bolt.go:80
    summary: PushRecorded's retry-also-fails and no-seed-commits-ahead paths are untested although the focus and card 13 name them
  - id: F4
    severity: LOW
    class: design
    location: internal/hubreconcile/walk.go:208
    summary: a rerun with nothing new to commit writes the build stamp while an earlier hub-wide commit is still unpushed, so the stamp-absent guarantee lasts one run only
  - id: F5
    severity: NIT
    class: consistency
    location: internal/reedcli/smoke_switchseed_test.go:36
    summary: comment still cites the one-argument ModeFor and says an unstamped binary is production mode, both false after cards 2 and 3
---

Range reviewed: no live rework round exists (`_lyx/loom/rework/` is absent), so `git diff $(git merge-base main HEAD)..HEAD` over the thirteen card commits; `product.parent` is `main`.
I ran the touched packages and then the whole module under `lyx gate test --tags integration`; both runs exited 0, so the findings below are review findings, not test failures.

### [BLOCKING] Duplicated log-capture helper instead of the shared kit

**Location:** `cmd/lyx/stencilseed_integration_test.go:57`, `internal/hubreconcile/ensure_integration_test.go:160`
**Issue:** `PATTERN-testkit` requires shared test support used by two or more packages to live in one kit under `internal/testkit/<kit>/`, "never duplicated per package".
`internal/testkit/logcapture` already provides exactly this: `Buffer`, a mutex-guarded `Write`/`String`, and `Capture`, which points `logger` at it and restores `os.Stderr` and verbosity in `t.Cleanup`.
The diff adds two identical `logSink` types with identical `Write`/`String` methods and two identical `captureLogs` helpers, one per package, instead of importing the kit.
The kit imports only `internal/logger`, so no import cycle bars either package from using it.
The scan behind `PATTERN-testkit` does not catch this (it covers imports only), so it falls to review.
**Fix:** delete both `logSink` types and both `captureLogs` helpers and call `logcapture.Capture(t)` in the tests that use them.

### [MEDIUM] Hub-config and warp-binding failure paths of `lyx fabric reconcile` are untested

**Location:** `internal/fabriccli/fabric.go:778`, `internal/fabriccli/fabric.go:784`, `internal/fabriccli/fabric.go:831`, `internal/fabriccli/fabric.go:867`
**Issue:** focus question 4 asks whether the verb fails on both a hub-config and a warp-binding commit or push failure, with the failure reaching the exit status.
By reading, the code does this: `firstFailure` is set at all four sites, the envelope is built, and `errWithRecordFields` returns it unless a pair failure takes precedence.
Tests reach only one of the four: `TestRunCLI_ReconcileBackfillFailureFailsTheVerb` has the heal skipped and the warp binding `recorded` with its push failing.
The hub-wide commit-or-push failure after a landed heal (lines 778 and 784) has no test at all.
The old `Reconcile_HubConfigPushFailureIsNonFatal` step was rewritten into a skipped-heal step, so nothing asserts a non-zero exit when the heal landed and its push failed.
The warp-binding commit failure (831) and the `present` path's push failure (867) are also unasserted.
`TestRunCLI_AdoptRemoteWeftScenario` also dropped its `code != 0` check outright and now ignores the exit status, so it pins none of this either.
**Fix:** add rows to the existing reconcile tests, not new top-level tests, for a heal that lands with a rejected push, a warp-binding commit failure, and a `present` push failure, each asserting a non-zero exit with `pairs` and the `*_detail` key still in the envelope.

### [MEDIUM] Push retry: second failure and no-seed-commits paths are untested

**Location:** `internal/fabricengine/bolt.go:80`, `internal/fabricengine/boltpullfirst_integration_test.go:354`
**Issue:** focus question 1 asks whether the board is clean when the retry also fails.
By reading it is: `PushRecorded` retries `pushWeftAt` once, never again, and a failed retry leaves the board at upstream plus the replayed commits because `pushWithRebaseRetry` aborts its own rebase.
`TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict` covers only a successful drop-then-push and a refused drop (replay conflict).
No test drives a drop that succeeds followed by a second push failure, nor a conflict with no seed commits ahead (`len(seed) == 0` at line 120), although card 13 states both outcomes ("a second failure is returned, never retried again").
The gate-ordering half of focus question 1 holds: `dropSeedCommits` calls `checkPathRequest` first, `.CherryPick(` and `.ResetKeep(` are banned tokens elsewhere in `fabricengine`, and `TestBoardDropRequest_RefusesForeignBoard` pins the ownership refusal.
**Fix:** add two rows to that test's subtests, a seed commit dropped and then a second push rejected (assert the error, a clean tree, no mid-rebase state and one `commits_dropped` entry), and a rebase conflict with only non-seed commits ahead (assert the original error and an unchanged tip).

### [LOW] Unpushed hub-wide commit does not keep the stamp absent beyond one run

**Location:** `internal/hubreconcile/walk.go:208`
**Issue:** `reconcileHubWide` reports `skipped` after a landed commit whose push failed, so that run leaves the stamp absent.
On the next run `pullFirst` finds an unmoved upstream and returns no skip, the reconcile has nothing new to write, `res.Committed` is false and the function returns `false, nil` without pushing, so `ensure` writes the stamp while the earlier hub-wide commit is still unpushed.
`doc.go` says a hub-wide commit whose push failed leaves the stamp absent, which holds for one run only; the test covers the moved-upstream rerun, not this one.
The commit is not lost, since the next `lyx board sync` pushes it, so the impact is small.
**Fix:** either push when the board holds unpushed commits even though nothing new was committed, or reword `doc.go` and the Info line to say the stamp stays absent only for the run that failed to push.

### [NIT] Stale comment in the reed switch smoke test

**Location:** `internal/reedcli/smoke_switchseed_test.go:36`
**Issue:** the comment says `stencilstore.ModeFor(buildinfo.IsDev())` returns `ModeDev` and that "an unstamped binary is production mode and never emits the dev-refusal warn".
`ModeFor` now takes three arguments and an unstamped binary maps to `ModeUnstamped`, which warns.
The assertion (a dev-stamped binary, empty stderr) is unaffected.
**Fix:** reword the comment to the three-way `ModeFor` and drop the production-mode claim.

### Focus questions answered

- Cards 7 and 13: the drop runs only behind the destruction gate; the retry leaves the board clean on a second failure (F3 covers the missing test).
- Cards 3 and 4: `StencilSource.Older` orders by ancestry when the worktree holds both commits and by commit time otherwise, `RecordedOlder` only for a strict ancestor or a strictly earlier time; `ModeFor` yields `ModeProduction` only for a production-stamped, clean binary, and a dirty or unstamped one gets `ModeUnstamped`. Not a defect.
- Cards 7, 8, 10 and 11: each skip Warn is logged once in `Bolt.skip`, called from `pullFirst`; the pre-run, `hubreconcile` and `runReconcile` read the returned skip and write nothing. The callers' own Info lines are not repeats of the Warn.
- Card 12: gc is skipped while the probe reports another pair's session or fails, with the keys still written, and the day-old prune test keeps the younger object. Not a defect.
- Test economy: no new test duplicates an existing one beyond F1's helpers; the gaps are F2 and F3.
