# Card 16 — finalize-owner-checks

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed#Finalize.pushParent`
- `internal/landingshed#reconcileVerdict`
- `internal/landingshed#TestFinalize_StuckReasons`
- `internal/landingshed#TestFinalize_ReconcileProceedsOrErrors`
- `internal/landingshed#TestFinalize_MergeMarkPushOrderAndVerdict`
- `internal/landingshed#TestMergeIn_RunsStatusCommitThenLockThenSyncThenMergeAndVerify`
- `internal/landingshed/doc.go`
- `internal/loomcli#landingDeps`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`

**Create:**
- `plan:internal/landingshed#ParentOwnership` -> `type ParentOwnership struct`
- `plan:internal/landingshed#OwnerHaltPrefix` -> `const OwnerHaltPrefix = "owner check: "`
- `plan:internal/landingshed#reportVerdict` -> `func reportVerdict(ctx context.Context, producer string, report fabricengine.OriginReport) (shedengine.Outcome, shedengine.OutputPointer, error, bool)`
- `plan:internal/landingshed#ownerHaltReason` -> `func ownerHaltReason(slug, runOwner string, parent ParentOwnership) string`
- `plan:internal/landingshed#Finalize.ownerChecks` -> `func (fz *Finalize) ownerChecks(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error, bool)`
- `plan:internal/landingshed#Finalize.adopterFirstRepair` -> `func (fz *Finalize) adopterFirstRepair(parentHandle parentMerger) string`
- `plan:internal/landingshed#TestFinalize_OwnerChecks` -> `func TestFinalize_OwnerChecks(t *testing.T)`

**Uses:**
- `plan:internal/fabricengine#Fabric.LocalOnlyCommits`
- `internal/landingshed#newTestDeps`
- `internal/landingshed#stuckVerdict`
- `plan:internal/fabricengine#ParentHoldsTaskWork`
- `internal/fabricengine#ParentSlug`
- `internal/fabricengine#EntryOwnedElsewhere`
- `internal/boardengine#ReadOwners`
- `internal/boardengine#Owners.Effective`
- `internal/boardengine#OwnedElsewhere`
- `internal/machinename#Name`

**Intent:**
D7: a run lands only on the machine that owns it and its parent, checked in the landing producer, so loom and darn runs and an alignment approve that lands with no orch turn all pass it; plus D6's halt note for an adopter-first landing.
`Deps` gains `Slug string`, `ParentHoldsTaskWork func() (bool, error)`, `ParentOwner func() (ParentOwnership, error)`, `OriginParentHoldsWork func() (bool, error)` and `ParentCommitsOriginLacks func() ([]string, error)`, each nil-is-absent like `Reconcile`: a nil `ParentHoldsTaskWork` reads not past the merge, and a nil `ParentOwner` reads the parent owned here, so a struct-literal or `newTestDeps` Finalize with no owner seams keeps today's landing; `ParentOwnership` holds the parent's `Slug` (empty for the prime), its `Owner` and whether this machine owns it (`Here`).
`reconcileVerdict` keeps its signature and maps the report through `reportVerdict`, which holds its ended and stopped mapping, so Merge-In proceeds on an owned_elsewhere verdict as before (D7's Bound).
`Finalize.Call` replaces its step 1b3 with `ownerChecks`: `ParentHoldsTaskWork` first; a read error halts through the halt path, prefixed `OwnerHaltPrefix`, naming the error and "re-run `lyx loom resume` in the task worktree", so a doubt never skips the checks; a parent already holding the task's work (a Finalize past its parent-side merge, such as the driver's in-process re-step after a transient push failure) runs `reconcileVerdict` as today, skipping both owner checks and the unreachable halt.
Otherwise it runs `Reconcile` itself, a nil `Reconcile` judging nothing: an error is returned; an unreachable origin halts, prefixed `OwnerHaltPrefix`, naming a re-run of `lyx loom resume` once origin answers; an ended or stopped entry maps through `reportVerdict`; the run's owner is the owned_elsewhere entry's owner, and the parent's comes from `ParentOwner`, read from the local Bolt the reconcile left, a read error halting prefixed `OwnerHaltPrefix` naming the error and the re-run; a failed check halts with `ownerHaltReason`, prefixed `OwnerHaltPrefix`, through `stuckVerdict` with halt set, so the loom halt note is written (`PATTERN-friction-capture`).
Bound: a run whose pair has no batten run record reads its run owned here, since the reconcile judges only recorded pairs; its parent's owner is still read through `ParentOwner`.
`ownerHaltReason` words D7's three messages: parent not owned names the parent's owner and either adopting the parent here, `lyx adopt --prime` for the prime or `lyx adopt <parent-slug>`, which resumes the halted run, or `lyx adopt <slug>` on the parent's owner, adding that it stops at its first step with `--keep-local` and `--drop-local` named when that machine drove the run before and holds local commits or a live pair for it; run not owned names the run's owner and `lyx adopt <slug> --keep-local` here; both name both owners, `lyx adopt <slug>` on the parent's owner, and `lyx adopt --prime <slug> --keep-local` or `lyx adopt <parent-slug> <slug> --keep-local` here.
`pushParent`'s non-transient failure appends `adopterFirstRepair`: only when `OriginParentHoldsWork` reads true does it list `ParentCommitsOriginLacks`, name the task's squash by the parent head's SHA, and name `git reset --keep origin/<parent-branch>` in the parent's worktree when the squash is the only commit, or `git rebase --onto <squash>~1 <squash> <parent-branch>` there with the SHA filled in and, on a conflict, `git rebase --abort` and a drop of the squash by hand, when another commit is listed; then `lyx fabric remove <slug>`, leaving the owner pause standing; when origin's parent lacks the work or a read fails it adds nothing, keeping today's reason.
`landingDeps` fills the new fields: `Slug` with `seedSlug`, `ParentHoldsTaskWork` with `fabricengine.ParentHoldsTaskWork` over the prime and the pair, `ParentOwner` over `boardengine.ReadOwners` of the hub board, `fabricengine.ParentSlug` and `machinename.Name`, `OriginParentHoldsWork` with `ReadParentTips(true)` and `HoldsWork` on the upstream tip, and `ParentCommitsOriginLacks` with `LocalOnlyCommits` over the parent pair; `TestLandingDeps_EveryFieldPopulated` checks them.
`TestFinalize_OverRealHub` and the `finalize_verify_test.go` tests build `Deps` without the owner seams, so the nil-is-absent rule keeps them landing unchanged.
`doc.go` documents the checks, the past-merge skip and the repair note.
`TestFinalize_OwnerChecks` is a table: run and parent owned here lands; run owned elsewhere halts naming the owner and `--keep-local`; parent owned elsewhere halts naming its owner and both adopt forms, `--prime` for the prime and the parent slug for a pair parent; both halt naming both owners and both forms; unreachable halts naming the re-run; a parent already holding the work lands under an owned-elsewhere and an unreachable report; a `ParentHoldsTaskWork` error halts; a `ParentOwner` error halts; each halt merges nothing and carries `OwnerHaltPrefix`; a Finalize halted then resumed with the parent not holding the work meets both checks again.
`TestFinalize_StuckReasons` keeps its ended and stopped rows through `reportVerdict`; `TestFinalize_ReconcileProceedsOrErrors` drops its "unreachable origin proceeds" row, which `TestFinalize_OwnerChecks` now owns as a halt and as a past-merge landing.
`TestFinalize_MergeMarkPushOrderAndVerdict` gains failed-push rows for the repair note: the squash alone names the reset, another commit listed names the rebase with the SHA, origin's parent lacking the work names no repair, and a read failure names no repair.
`TestMergeIn_RunsStatusCommitThenLockThenSyncThenMergeAndVerify` gains a row: an owned_elsewhere report proceeds to the merge.

**ImpactSummary:** landingshed's Finalize and its deps, loomcli's assembly; Merge-In, Publish, PR-Align and PR-Gate proceed unchanged.

**Commit:** 16: landing: Finalize lands only on the machine that owns the run and its parent
