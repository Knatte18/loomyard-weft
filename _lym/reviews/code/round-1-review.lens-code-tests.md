```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that fails when that behaviour breaks.

**Behaviours and the tests that pin them:**

1. **A beam member under a recorded pull request gets no line when its `land_notice` reads off** (`landnotice.go` `rememberRuns`, condition `if note.Requested`).
   - Pinned by `TestLandNotices/AMemberWhoseLookupReadsOffGetsNoLineUnderAPullRequest` (`landnotice_test.go:78`).
   - Since `f177b00a7`, the harness puts the run's entry on the board as a member of `b` while the run is ahead of Land (`landnotice_test.go:146-149`). If the old membership override came back, this row would send `memberLandedLine` and fail.
   - The mutation report agrees: 2 mutants, 2 killed.
2. **A member whose `land_notice` reads on gets the follow-up under a recorded pull request.**
   - Pinned by `AMemberWhoseLookupReadsOnGetsTheFollowUpUnderAPullRequest` (`landnotice_test.go:79`).
   - The landed-with-no-commit and empty-diff variants are pinned by `AMemberLandedWithNoCommitCarriesTheFollowUp` and `AMemberLandedWithAnEmptyDiffCarriesTheFollowUp` (lines 82-83).
   - The follow-up read in the tick the pull request is recorded is pinned by `AMemberLandingInTheTickThePullRequestIsRecordedGetsTheFollowUp` (line 81).
3. **A requested member with no pull request gets a line with no follow-up.**
   - Pinned by `ABeamWithNoPullRequestSendsARequestedMemberNoFollowUp` (unchanged row).
4. **A requested member's unlanded line carries no follow-up.**
   - Pinned by `AnUnlandedMemberLineCarriesNoFollowUp`. It now sets `requested: true`, as the new rule requires: without it the row would send nothing, so the row still checks what its name says.
5. **A failed landing-record read decides nothing until the read succeeds.**
   - Pinned by `AFailedLandingRecordReadDecidesNothingUntilItReads` (unchanged).
6. **The landing notice no longer reads the board, and its board failure episode is gone.**
   - There is nothing left to assert. Removing `AFailedBoardReadKeepsTheMembership` drops a row whose only subject was the removed membership read.
   - Its `tick()`-returns-an-error check pinned the start step's own board read, not a landing-notice behaviour. No finding.
7. **An older state record carrying a `member` key loads and that key is ignored.**
   - There is no raw-JSON row in `TestLandNotices`, because the `seed` struct can no longer express `member`.
   - Tolerance of unknown keys is pinned generically by `TestState_RoundTripKeepsUnknownKeys` (`internal/dobbyengine/state_test.go:15`).
   - That the key carries no decision follows from behaviour 1's row: the decision reads only `Requested`.
   - I judge this covered and file no finding.
8. **Removed rows.**
   - `ABeamWithNoPullRequestSendsNothingForAnUnrequestedMember`: its behaviour (an off reading sends nothing) is covered by `ALookupReadingOffSendsNothing` (line 68) and the new off-member row.
   - `ANonMemberEntryGetsNoLineUnderAPullRequest` and `ADoneMemberEntryStillGetsTheLine`: these pinned membership only, which the diff removes.
   - `AMemberUnderAPullRequestGetsItsLineWhateverTheLookupReads`: its behaviour is now inverted by the off-member row.
   - None of them pinned a behaviour that is still live and left uncovered.

**Findings:** none.

Tool calls:
- One Bash call read `landnotice_test.go` lines 20-76 and 136-175, and grepped the dobbyengine tests for a test of an older record's `member` key. My context had only the diff hunks, not the unchanged rows, the harness lines or `state_test.go`.
