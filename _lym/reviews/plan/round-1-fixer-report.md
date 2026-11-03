# Plan review round 1 — fixer report

F1 is decision-class and went to the parent; it is left out below.
The parent's decision entry "Store failures at approval: only a done entry warns", delivered as a `ly:LOOM` decision notice, was applied to card 7 as a parent instruction: the done-entry wiring bullet cites the entry and states that every other store failure, an absent entry and an unopenable board included, fails the settle; the done-entry row of `TestApproveDiscussion_Real` says the settle passes with its Warn line.

## Changes

- F2 (fixed as proposed): card 2 logs a failed removal of a removed entry's record file at Warn through `internal/logger` and returns no error, keeping a returned error for the rename alone; the package's unit glyph `internal/logger#` joins its Uses, spelled so after the gate refused the bare directory `internal/logger`.
  Card 5 says the `rename` verb's help and the board skill's rename text state that a saved rename can report an error, and what each error names.
- F3 (fixed as proposed): card 8 drops the judge-whether-the-run-passed text, the or-when-it-errs fallback of `wayCommit`, the `pastErr` fake field and the pastApproval-error row; a `pastApproval` error takes the generic store-error branch, with `hasPlan` and `rowWritten` named as the upstream checks.
  Card 9 folds the `pastApproval` error into `approvalRow`'s recipe-read refusal, drops `pastErr` and the separate row, and names `approvalRow` as the upstream check.
- F4 (fixed as proposed): card 8 reports a not-found `committed()` through the read-error branch with no text of its own and drops the HEAD-does-not-hold row.
- F5 (fixed as proposed, then refined by the self-check): card 2's rename removes `decisionFileName(renamedTo)` when the old file is absent, and adds the `TestDecisionRecordFollowsEntry` row asserting found false.
  A failure of that removal has its own wrap text naming the orphan at the new slug's name, and the post-save error paragraph and card 5's rename help name it.
- F6 (fixed as proposed): card 2's refused-rename row builds its own `boardengine.Config` with a refusing `ValidateSlug`.
- F7 (fixed as proposed): card 5 adds `discussed` to the `boardcli/doc.go` upsert sentence on stored-value-only keys.
- F8 (fixed as proposed): card 7 rewords the whole Plan-Bouncer `approve_seam` comment block, dropping the neither-Discussion-Bouncer-nor-Code-Bouncer sentence, and rewords `TestApproveSeam_FailsToBuild`'s doc comment to name `plan` and `discussion`.
- F9 (fixed as proposed): card 3 says a `lym config board --set` is checked by loading the config after the write and passes when the new value loads.

## Self-check

- fix-consistency, MEDIUM: the F5 target-removal failure reused the move's error text, which names a record at the old slug, and card 2's post-save paragraph and card 5's help went untrue for it. Fixed as proposed: own wrap text, and both siblings name the case.
- fix-consistency, LOW: card 5's ImpactSummary said other verbs are untouched. Fixed as proposed: it names the rename help and skill text.
- fix-claims, MEDIUM: same defect as the fix-consistency MEDIUM, merged with it.
- fix-claims, MEDIUM: card 2 claimed a contract that an error from `RemoveTask`, `Prune` and `MergeTasks` means nothing was saved, which `boardCriticalSection` (`internal/boardengine/board.go:156-164`) does not hold. Fixed with own change: the sentence now says those methods gain no new error after their save.
- fix-proposal, MEDIUM: same defect as the fix-consistency MEDIUM, merged with it.
- fix-affected-set, LOW: card 7 left the `approve_seam` comment at `internal/shedrecipe/entries_bouncer.go:115` naming `env.ApprovePlan` alone. Fixed as proposed: card 7 names that comment beside the doc and switch comments.

## Disputed

None.

## Deferred

None.
