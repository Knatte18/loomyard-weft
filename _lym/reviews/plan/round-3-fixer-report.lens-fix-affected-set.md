```yaml
lens: fix-affected-set
outcome: reported
```

I checked that the fix covered every member of each finding's set (fix-affected-set lens).

Findings I left unchecked: none.

1. **MEDIUM.** Changed text: card 12, `TestReconcileOrigin_OwnedElsewhere`, the sentence "Its rows take `OwnerOf` from `pairteardown.OwnerReader` with a fixed identity, over the owners the board's own entries record".
   - Summary: the moved owned_elsewhere set has two subject rows that are not the empty-subject row, and the card does not say how the real reader produces their subjects.
   - Evidence: `internal/fabricengine/originreconcilemoves_integration_test.go:659-662` lists "a task member of a beam names the beam" (subject `beam`) and "a root task names root" (subject `root`).
   - Today the scripted `OwnerOf` at `:668-670` returns those subjects directly.
   - `pairteardown.OwnerReader` instead derives the subject from `owners.AdoptSubject(slug, resolve)` over its `parentOf` argument (`internal/pairteardown/originseams.go:94-118`).
   - The card names a fixed identity but no `parentOf`, and no beam or root membership in the board entries, so those two rows cannot be built as written.
2. **LOW.** Changed text: the same OwnedElsewhere text, "so no row scripts an owner answer", together with the kept nil-`OwnerOf` row.
   - Summary: the card is silent on how this row observes that the reader was never asked.
   - Evidence: the row at `originreconcilemoves_integration_test.go:731-741` (no Bolt origin head) asserts that `OwnerOf` was never asked, through a scripted `quiet` reader that sets `asked`.
   - Under the new rule that row needs a recording wrapper around the real reader, which the card does not mention.
3. **No finding: F21's teardown set.** The scripted `teardownRecorder` replies sit at `internal/fabricengine/originreconcileended_integration_test.go:128`, `:304` and `:327`.
   - `:304` is the busy rows and `:327` the failing-teardown row; card 12 deletes both.
   - `:128` is the landed row; after the fix the card stops asserting its scripted Tags and Kept lines. Every member is covered.
4. **No finding: F19's set.** No card still mentions `newReplayHub`, `pushPrimeRecords`, `commitOnOrigin` or `addPushedPair`.
   - Of fabric's clone-commit-push helpers, the two that boardengine_test would have restated, `originCommit` and `originCommitFile`, are both on the Edit list.
   - Other copies inside fabric (`boltFixture.moveUpstream` in `internal/fabricengine/boltpullfirst_integration_test.go:48-55`) are duplicates within one package, outside the finding's per-package premise.
5. **No finding: F20's set.** No card still mentions `pushkit` or `RequireEntry`.

Tool calls:
- grep for `"clone"` in the fabricengine and boardengine tests: reran F19's search for git plumbing duplicated across packages, which my context did not hold.
- grep for `OwnerOf` and `Teardown` seams and `teardownRecorder` uses in the reconcile tests: reran F21's search for scripted seams, which my context did not hold.
- read of the scripted-reply rows, the owner-subject rows and `boltFixture.moveUpstream`: checked what each set member does, which my context did not show.
- grep across every card for the removed helper names, `pushkit`, `RequireEntry` and `CommitOnBare`: checked for stale references in other cards, which my context did not hold.
