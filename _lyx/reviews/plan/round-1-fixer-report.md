# Plan-Review round 1 — fixer report

All four findings are fixed.
`lyx loom validate-plan` passes after the edits (`ok: true`, only the three informational `create-new-unit` findings).
`approved:` in `00-overview.md` is untouched.

## Fixed

- **F1 (LOW), card 2 ImpactSummary.**
  Rewritten to count and name `foreignMergeStatePresent`'s 6 call sites (MergeIn, Merge, recheckMergePreconditionsUnderLock, mergeStateOrForeignErr for MergeContinue/MergeAbort, MergeStageResolved, Fabric.Commit), all in fabricengine, each seeing an unchanged answer and error wrapping.
  It also notes that `doc.go` is comment-only and that `MidMerge` has no caller until card 3.
- **F2 (LOW), card 3 ImpactSummary.**
  Rewritten as a blast-radius conclusion: one production caller of `runDriverSpawnAndWait`, which constructors set the new field, which test receivers reach the call and with what fake, `startCmd` as help-text only, and batten's `childNotParked` treating the new kind as a hard error.
- **F3 (NIT), card 2 reader placement.**
  The extracted four-probe reader now goes in `mergestate.go` directly above `foreignMergeStatePresent`, and `MidMerge` in `midmerge.go` calls it, so every foreign-state probe stays in the file whose header says it holds them.
  No new Create entry was added: the reader is part of the `Fabric.foreignMergeStatePresent` edit, exactly as it was part of the `midmerge.go` create before.
- **F4 (NIT), card 3 test rename.**
  The conditional rename is now definite: `TestNewLoomCLI_SetsBothInjectedSeams` becomes `TestNewLoomCLI_SetsInjectedSeams`, with its doc comment updated.

## Deferred

None.
