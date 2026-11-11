```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1–F4 as carried out, looking for cases where a proposal is wrong or breaks something.

**Proposals fixed as proposed:** F1 (skip past-Land listed runs in `flagMembers`), F2 (seam test), F3 (membership-based request, with a variation), F4 (rows).

Findings:

1. **MEDIUM**, `flagMembers` skip (`landnotice.go`, `pastLand` check, F1 proposal).
   - Summary: the skip covers only runs still listed, so the double line survives once the run drops from the listing while its entry is open.
   - Evidence: F1's own premise is that a run past Land (teardown row) can have an open entry.
   - Case: the run lands and its line is posted. The flag is cleared and the memory is dropped. Teardown finishes and the run is no longer in `runs`, but the entry is still open. `flagMembers` then sees an open member with no flag and no listed past-Land run, so it flags it again. When the entry turns done, `takeDoneEntries` takes it as flagged, done and unremembered, and posts a second line.
   - The reviewer's alternative, "or whose slug has landing-notice memory", doesn't close this gap either, because the memory is deleted after settle. The fix is partial and the `TestLandNotices` row never lists the run as gone.
   - A durable marker, such as keeping a "landed" memory until the entry is done or gone, would be needed.

2. **LOW**, `rememberRuns` (`requested := carriesField || (followUp != "" && (note.Member || ...))`, F3 proposal).
   - Summary: a member that was never flagged now gets a line under per-task whenever the pull request is recorded at landing time. That includes members that merely sat ahead of Land as non-requested when the pull request appeared, and done members.
   - Evidence: the row `ADoneMemberEntryIsNotFlagged` asserts `want: memberLandedLine` for an entry that was never flagged and was done from the start. That is the intended F3 behavior, but it is a broader trigger than F3's "member turns done in the tick after".
   - Case: `member` is true for entries of any status, so a done member entry with a pull request recorded sends a line it never asked for.

3. **LOW**, `ClearOwed = carriesField` (F3 proposal, "keep the clear owed only when the entry still carries the field").
   - Summary: `carriesField` includes the remembered `note.Requested`, which can be stale, so a clear is owed for an entry that no longer carries the field.
   - Evidence: `note.Requested` is only refreshed while the entry is on the board. This matches the old behavior, so it is not a regression.

No further findings.

Tool calls: none. My context already held every changed file.
