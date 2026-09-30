---
verdict: APPROVED
rationale: "Report APPROVED with 1 MEDIUM, 4 LOW, 3 NIT; the fixer resolved all eight in the cards (card 1 compares with reportHead before the parent count and tests a merge-commit report head, card 2 now edits doc.go, card 6 pins warnings []string and targets envelope_test.go), and a code spot-check finds every cited symbol, fixture and test present with no remaining blocking gap against D1-D7."
---

# Plan-Review round 1 — Bouncer verdict

The round-1 report returned `APPROVED` with eight findings, and it has no `## Focus departures` section, so there is nothing to ratify or reject.
The fixer report says it fixed all eight and deferred none.
I checked each fix against the current cards:

- **F1 (MEDIUM), card 1:** the Intent now compares each commit with `reportHead` before checking its parent count, and it says why the order matters.
  The test list adds a report head that is itself a merge commit, with a second `--no-ff` merge on top.
  D1 now holds for the no-commit-fork case that card 4's scenario leads into.
- **F2 (LOW), card 6:** both `setMergeExit` and `errConflictsWithRecord` gain a `warnings []string` parameter, and the key appears only when the slice is non-empty.
  `internal/fabriccli/envelope_test.go` is now under Edit.
  Its two existing `errConflictsWithRecord` call sites (lines 231 and 274) are the only callers besides `setMergeExit`.
- **F3 (LOW), card 3:** `RecoverResult` moved from Uses to Edit, and the Intent updates its `Warnings` comment.
- **F4 (LOW), cards 2 and 3:** card 2 now edits `doc.go` in the commit that changes record-batch, and card 3 extends that paragraph to recover-batch.
  This satisfies the decision record's "docs in the same commit" constraint.
- **F5 (LOW), card 5:** an unreadable `outcome.yaml` row (a directory at the path) now covers D5's "unreadable" acceptance case.
- **F6–F8 (NIT):** card 6 now names both recipe names, `loom` and `batten`.
  Card 6 now seeds under `h.Location.AnchorPath()`, the path the verb reads.
  Card 4 now names every `RecordDeps` seam that `newRecordFixture` wires.

I ran the following code spot-checks:

- `gitrepo.Repo.MergeHeadPresent`, `CommitParents` and `CurrentSHA` exist.
- `gitwrap_test.go`, `recordbatch_test.go`, `recoverbatch_test.go` and `beginbatch_test.go` are all `//go:build integration`.
- These fixtures and tests exist as the cards name them: `newBeginFixture`, `recordFakeEngine`, `recordFakeSleeper`, `NeverMatches`, `driveRecoverBatch`, `TestRecordBatch_ScopeGuardFindingsLandInWarnings`, `TestRecordBatch_HeadSHAMismatchErrors` and `TestRecoverBatch_ReportHeadSHAMismatchIsHardError`.
- `parseOutcome`, `outcomeDone`, `stateFileName`, `Dir` and `OutcomePath` exist.
- `addWeftVerbs` passes `addMergeVerbs` a getter and has a `*lyxcwd.Location` in scope.
  Only `push` reaches the bypass branch, so `loc()` is always non-nil in `merge-in`.
- The `ly-drive` skill has a `## Repairing` section.
  `cmd/lyx/lydriveskill_test.go` forbids `shedrun.RecipeNames()` and the three literals.

The two existing mismatch tests still pass under the new walk.
Their bogus `head_sha` never lies on HEAD's first-parent chain, so the walk hits a non-merge commit (or the root) and refuses, and the pinned phrase stays in the error.
Both tests match on substrings: `TestRecordBatch_HeadSHAMismatchErrors` checks that the error names HEAD's SHA, and the recover test checks for "does not match the worktree's actual HEAD".

I also checked the plan against the decision record.
Every decision from D1 to D7, every constraint and every acceptance row is carried by some card, and no card adds scope outside the record.
The verify section runs `websterengine`, `fabriccli` and `cmd/lyx` both untagged and under `-tags integration`, which covers every targeted Go package.
Cards 4 and 5 carry no `ImpactSummary`, but both are Create-only cards, and the rubric does not require one there.
The acceptance row "the merge-in-progress probe fires in a linked git worktree" is tested at the `refuseMidMerge` helper, which `RecordBatch` calls unchanged on `WorktreeRoot`.
That is enough and is not a finding.

Nothing left in the plan blocks it.
