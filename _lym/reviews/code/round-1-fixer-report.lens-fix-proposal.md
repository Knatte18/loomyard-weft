```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 fix, the new `GroupingProjectWhoseRunFiledItsFamilyResumesOnlyWithTheFilingMark` subtest.

no findings

I looked for a case where the proposal is wrong and found none.
- **Shared state:** the mark is written under `h.Location`, which `hubforge.CopyHub` gives this test alone. The subtests run serially, since only the parent calls `t.Parallel()`, so there is no race.
- **Effect on later subtests:** the mark is an untracked, empty file, so it can't affect the tracked-dirty check. `MemberHoldsAPairOnlyThroughARegisteredWorktree` and `BeamWithMembersIsRefusedNamingEachOpenEntry` read the board and worktrees, not the mark, and `g` is not used again.
- **Test strength:** the "before" half asserts the `grouping project` refusal for `g`, a non-beam project with `found=true`. The "after" half asserts a pass once the mark exists. Dropping the `FilingMarked` wiring, or keying it by another run ID or location, fails the "after" half. A mark that is always true would fail the "before" half.
- **Gate:** the subtest passed in the tier-1 log (`--- PASS: ...GroupingProjectWhoseRunFiledItsFamilyResumesOnlyWithTheFilingMark`), so it ran and was not skipped.

Tool calls: none. My context already held every changed file and the gate output.
