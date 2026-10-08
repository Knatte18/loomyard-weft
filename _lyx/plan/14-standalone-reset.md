# Card 14 — standalone-reset

**Create:**
- `internal/websterengine/keepreset.go`

**Edit:**
- `internal/websterengine#ResetDeps`
- `internal/websterengine#PlanReset`
- `internal/websterengine/doc.go`
- `internal/websterengine/reset_test.go`
- `internal/webstercli#websterCLI.resetCmd`
- `internal/webstercli#standaloneBranch`
- `internal/webstercli/reset_test.go`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/gitrepo#Repo.ResetKeep`
- `plan:internal/websterengine#ArchiveRunAfterReset`
- `plan:internal/websterengine#UncommittedPaths`
- `internal/gitrepo#Repo.WorktreeChangedFiles`
- `internal/gitrepo#Repo.ChangedFilesSince`

**Intent:** Standalone mode performs the reset instead of naming a git command (D4).
`resetCmd` no longer refuses when there is no fabric.
Its standalone branch performs the planned move itself, as its pair branch calls `fab.ResetPairCode`: `gitrepo.New(c.geom.WorktreeRoot).ResetKeep(plan.SHA)` (card 10), touching no remote; for `--to start` it removes the live recovery strands first and archives through `ArchiveRunAfterReset` as card 13's pair branch does, including the archive-only path, which runs no `ResetKeep`.
The call sits in `internal/webstercli`, never in `internal/websterengine`, whose production source `cmd/lyx/rawgitmutation_test.go` keeps free of `gitrepo.New(` outside `gitwrap.go`'s read-only allowlist entry, so webster's engine still runs no mutating git (`PATTERN-fabric-git`, `PATTERN-gitrepo-client-boundary`: the keep-reset is a `gitrepo` method on the `gitexec` side, pinned by card 10) and `reset.go`'s file header stays true.

In standalone `PlanReset` skips its foreign-dirty-path refusal (`ResetDeps` gains `Standalone bool`, which the CLI sets only when it has no fabric), because git's keep form is the guard there (D4's bound): it carries an uncommitted change across or refuses, and never discards one.
What that lets through is a standalone reset over foreign tracked changes the move does not touch, which stay in the tree; bounded by keep's own refusal, which leaves HEAD and every file unchanged when a dirty path differs between HEAD and the target, and by every other `PlanReset` refusal, which standalone keeps.
In a pair the refusal stays, since fabric's reset discards the run's own tracked changes and must not reach a foreign one.

When git's keep form refuses, `resetCmd` reads the paths the move would overwrite (the worktree's changed paths, `WorktreeChangedFiles`, that also differ between HEAD and the target, `ChangedFilesSince`) and returns the error of `KeepResetRefusal(plan ResetPlan, cause error, overwritten []string) error`, the one function of the new `keepreset.go` (its file header says so), which the CLI alone calls: git's message and a way forward per such path, `git checkout -- <path>` for a path in the plan's `OwnPaths`, commit or restore for any other, then re-run the reset.
`KeepResetRefusal` is pure, so its per-path classing is tested without git.
Every standalone reset lists the uncommitted paths it left (`UncommittedPaths` after the move) in the `uncommitted` envelope key: standalone keeps every change git carries across, the run's own included.
`standaloneBranch`'s comment no longer says the verb refuses, and `resetCmd`'s long help replaces "In standalone mode it refuses and names the git reset --keep command to run" with the standalone keep-reset, its refusal and the `uncommitted` key.

Tests:
- `reset_test.go` gains `TestKeepResetRefusal`, a table pinning the per-path way forward for a run-written and a foreign path, and a row in card 11's `TestPlanReset_Resolution` for the standalone flag skipping the foreign-dirty-path refusal; `test-budget.yaml` raises `internal/websterengine` by one.
- `TestResetCmd` in `webstercli/reset_test.go` replaces its "standalone names git reset keep" refusal row with two standalone steps, a move asserting HEAD and the `uncommitted` key, and a keep refusal asserting the per-path way forward reaches the envelope.
  The keep semantics themselves (an unrelated dirty file carried across, a refusal leaving HEAD and the file untouched) stay pinned once, by card 10's `TestResetKeep`.

`refusal-spec.md`, by row:
- `reset: standalone mode`: deleted, standalone now performs the reset.
- `reset: dirty tracked path the run did not write`: narrowed to a pair reset; standalone skips it, keep's refusal guarding instead.
- Added: `reset: standalone keep refused` (git's keep form refuses over a dirty path the move would overwrite; correctness halt; `git checkout -- <path>` for a run-written path, commit or restore any other, then re-run the reset).
The package doc's reset section states the standalone keep-reset and its tree.

**ImpactSummary:** Makes standalone reset act from webstercli through gitrepo's keep-reset and adds one ResetDeps flag only the CLI sets; the pair reset is unchanged.
