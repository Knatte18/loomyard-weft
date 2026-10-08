# Card 12 — reset-start-archives

**Edit:**
- `internal/websterengine#Run`
- `internal/websterengine/reset.go`
- `internal/websterengine/archive.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/reset_test.go`
- `internal/websterengine/reset_integration_test.go`
- `internal/websterengine/archive_test.go`
- `internal/webstercli#websterCLI.resetCmd`
- `internal/webstercli#standaloneBranch`
- `internal/webstercli/reset_test.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `plan:internal/gitrepo#Repo.ResetKeep`
- `plan:internal/websterengine#resetKeep`
- `internal/websterengine#RemoveRecoveryStrands`
- `internal/websterengine#archiveStateFile`
- `internal/websterengine#archiveReportsDir`
- `internal/websterengine#clearRenderedPrompts`

**Intent:** `reset --to start` becomes the one start-over verb, and standalone mode performs the reset instead of naming a git command (D3, D4).

The archive: the in-place archive `run --fresh` performs (state.json renamed to `state-<stamp>.json`, the reports dir to `reports-<stamp>`, the rendered prompts cleared) moves out of `Run`'s fresh branch into one exported function in `archive.go` that both `Run --fresh` and the reset call, so the two archive identically and the next plain `lyx webster run` finds no state and starts a new run.
The decision record names `ArchiveRunRecord` for this; that function needs a told destination outside the webster dir and moves its whole content, which reset has no destination for, so the reset uses the archive `run --fresh` performs, which is the decision's intent ("the same archive `run --fresh` uses").
`--to start` archives after the branch reset and the state save succeed.

The archive-only path: when the start cannot be moved to, `PlanReset` returns a plan marked archive-only with its reason instead of refusing, for exactly this closed set: no batch recorded a start, a recorded start is missing from the repository, the recorded starts share no single oldest commit and no common ancestor, or the resolved start is not an ancestor of HEAD.
It keeps the refusals that come before the target resolves (run lock held, no `state.json`, merge in progress, foreign branch) and drops the missing-commit, no-common-ancestor, ancestor and dirty-tree refusals for `start` only, which guard a branch move it does not make; `pre-fix` and the card-11 targets keep all of them.
The verb removes the run's live recovery strands first exactly as a moving reset to start does (refusing with the `lyx reed remove` way forward when one cannot be removed), runs no git, archives, and reports `moved: false` with `reason`; a moving reset reports `moved: true`.
Bound: the archive-only path moves the run record into archive names, never deletes it, and moves no commit; it lets through a branch rewritten under the run and any uncommitted changes, which the next run starts over.

Standalone: `resetCmd` no longer refuses when there is no fabric; it calls a new websterengine function in `reset.go` that performs the planned move through `resetKeep` (card 10), touches no remote, and for `--to start` archives as above.
When git's keep form refuses, the refusal carries git's message and a way forward per dirty path the move would overwrite (the dirty tracked paths that differ between HEAD and the target): `git checkout -- <path>` for a path in the plan's `OwnPaths`, commit or restore for any other, then re-run the reset.
Every reset to start, in either mode, and every standalone reset lists the uncommitted paths it left (the worktree's `DirtyPaths` after the move) in an `uncommitted` envelope key, so the operator sees what the next run starts over: in a pair the run's own tracked changes are discarded and untracked files stay; standalone keeps every change git carries across; the archive-only path keeps everything.
`standaloneBranch`'s comment no longer says the verb refuses.

Grep for `standalone mode has no task pair` and the start-target `run --fresh` refusal texts in `PlanReset`/`resolveResetSHA`; their tests join this card.
`reset_test.go` gains rows for each archive-only cause returning the plan with its reason and for the pre-run refusals still refusing; `reset_integration_test.go` asserts a moving reset to start archives the record and a following state load finds none, the archive-only path under a rewritten branch, and a standalone keep-reset moving HEAD, keeping an unrelated dirty file and refusing over an overwritten one with the per-path way forward; `archive_test.go` pins the factored archive; `webstercli/reset_test.go` asserts `moved`, `reason` and `uncommitted` in the envelope.
`refusal-spec.md` drops the start rows the archive-only path replaces and the standalone refusal row, and gains the standalone keep-refusal row; the package doc's reset section states the archive, the archive-only set and the per-mode trees.

**ImpactSummary:** Changes what reset --to start leaves behind (an archived run record) and makes standalone reset act; Run --fresh calls the factored archive with unchanged behavior.
