---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/hubreconcile/walk.go:204
    summary: A reconcile or commit failure after files were written leaves them uncommitted, and a retry then sees them as current and never commits them
  - id: F2
    severity: LOW
    class: scope
    location: internal/preflightshed/preflight.go:109
    summary: reconcileRefusal has an unreachable Module-empty branch and a commit-failure branch no test reaches
  - id: F3
    severity: LOW
    class: scope
    location: internal/loomcli/arm.go:132
    summary: No test reaches the two new lyx loom start and resume reconcile refusals that PATTERN-refusal-way-forward requires a test for
  - id: F4
    severity: NIT
    class: consistency
    location: internal/hubreconcile/walk.go:310
    summary: snapshotConfig has a data-is-nil case that os.ReadFile can never produce
---

Range: `main` is `product.parent` and no `_lyx/loom/rework/` round exists, so the range is `git merge-base main HEAD..HEAD`, which is cards 1 to 21 plus the closing `fix:` commit.
`go build ./...`, `go vet ./...`, `go test ./...` and `go test -tags integration` over the touched packages all pass.

Checked and found sound:

- Every start verb calls `Ensure` before any module config load: `orchcli`'s `Command` pre-run for `start` only, `loomCLI.arm` for `start` and `resume`, `battenCLI.arm` for `run`.
  The `Preflight` row calls it before `preflight.Check`.
- The stamp is written only after a walk that returns no error and no mid-merge skip, and the hub lock is released by a `defer` on every path after acquisition.
- The walk skips an incomplete pair before opening it, and `lyx batten run` reconciles its new pair unconditionally after both a fresh create and an already-present pair.
- `behind` takes ancestry lazily (once per `Source`, only for a source-ahead stencil), carries no production-deploy advice, and a failed or unstamped read degrades to unknown ancestry.
- The `edit_directive` marker is on every opening stencil `roleOpeningStencils` names, each renderer reads it through `editdirective.Directive` with no signature change, and Master's restated script ban is gone.
- Card 1's two `Delete` targets and card 4's `Rename` leave no caller or reference behind (grep over Go and Markdown, outside `_lyx`).
- `hubreconcile`, `editdirective` and `buildvcs` carry `doc.go`, and both new PATTERN entries are present.

### [MEDIUM] A failed walk step leaves written config uncommitted for good

**Location:** `internal/hubreconcile/walk.go:204` (`reconcileWorktree`), and `internal/hubreconcile/walk.go:169` (`reconcileHubWide`)
**Issue:** `configsync.ReconcileAll` stops at the first failing module and returns no results, so every module it already rewrote stays on disk with nothing recorded to commit.
`reconcileWorktree` returns the `*WorktreeError` without restoring its snapshot, and `restoreConfig` runs only on a merge-refused commit.
A commit failure for any other reason leaves the same state.
On the retry those files already match the template, so their results are not `Applied`, they never enter `files`, and the walk stamps the build with them still uncommitted.
Reproduced in a scratch copy of the repo: in `TestEnsure_UnparseablePairConfigNamesPairAndFile`'s fixture, with `loom.yaml` broken, the first `Ensure` rewrites `batcher.yaml` (alphabetically earlier) in the pair's records worktree and fails.
After `loom.yaml` is fixed and committed, the second `Ensure` succeeds and writes the stamp, and `git status` in the pair's records worktree still shows ` M _lyx/config/batcher.yaml`.
The same hole exists in `reconcileHubWide`: a write error from `ReconcileHubWideAt` returns before any commit, with the earlier hub-wide modules already written.
This breaks the package's own contract ("commits everything it wrote") on the error path the plan names as the way forward ("fix the file, then re-run").
The existing test for this path asserts the error and the missing stamp but never checks that the worktree is clean.
**Fix:** After the snapshot, restore it on every failure that follows the write, not only on a merge-refused commit: the `ReconcileAll` error return and the non-merge `Commit` error return in `reconcileWorktree`, and the write error in `reconcileHubWide`'s closure (snapshot the hub-wide files under `boardDir` the same way).
Add an `assertClean` of the pair's records worktree to `TestEnsure_UnparseablePairConfigNamesPairAndFile` after the failed call, which covers the behavior without a new test.

### [LOW] `reconcileRefusal` has an unreachable branch and an unreached one

**Location:** `internal/preflightshed/preflight.go:109`
**Issue:** `worktreeErrorFor` takes `File` and `Module` together from a `*configsync.FileError`, which always sets both, so the `worktreeErr.Module == ""` case cannot occur.
The `worktreeErr.File == ""` case (a commit failure) is reachable but no test in `preflight_integration_test.go` reaches it: the two rows exercise an unparseable file and a held lock.
The generic fallback for a non-`*WorktreeError` (listing failure, lock-dir failure) is likewise unreached.
Under `code-quality` a branch for a state the code's own guarantees rule out is not written, and under `PATTERN-test-economy` an added behavior with no coverage is a finding.
**Fix:** Delete the `Module == ""` case.
Add rows for the commit-failure and generic-error shapes to the existing table-driven reconcile-failure test, or reach them through a small table over `reconcileRefusal`'s inputs.

### [LOW] The loom start and resume refusals are not reached through the loom verbs

**Location:** `internal/loomcli/arm.go:132`
**Issue:** `PATTERN-refusal-way-forward` requires a row in the same commit and a test that reaches the refusal.
The rows landed (`start/resume: hub config reconcile failed` and `start/resume: hub reconcile lock busy` in `contracts/specs/refusal-spec.md`).
The only loom test of the new gate, `TestLoomResumeReconcilesTheHubConfigBeforeArming`, covers success.
The failure shape is reached through `lyx orch start` in `TestOrchIntegration_Refusals`, and through `Ensure` directly in `hubreconcile`, but never through `lyx loom start` or `resume`, so a regression in the loom gate (for example, swallowing the error) would pass.
**Fix:** Add a failure row to `TestLoomResumeReconcilesTheHubConfigBeforeArming`, or a sibling row in the same file's table: an unparseable pair config makes `loom resume` exit non-zero with the file named and `re-run this command`.

### [NIT] Dead case in `snapshotConfig`

**Location:** `internal/hubreconcile/walk.go:310`
**Issue:** `os.ReadFile` returns a non-nil slice on success, so `case data == nil` never matches and the empty-file case is already covered by `default`.
**Fix:** Drop the `data == nil` case; the switch reduces to an `if err != nil` that stores nil and an assignment otherwise.
