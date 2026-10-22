# Round 4 fixer-report — plan

## Changes

- **F1** (MEDIUM, scope), fixed as proposed: card 10 creates `TestAwaitRunLockOrPass`, planned as an integration-tagged test beside `TestProbeQuietSeesTheGoDriverLock`.
  It runs over a real task worktree whose run lock the test holds with `mustHoldLock`.
  An unfinished step record carrying the test process's pid is the known holder, and `td.sleep` is a counting no-op.
  Its rows are: a hub entry with a parent lock record refusing with `ErrDriverBusy`, naming `lyx tender resume` with no slug and the lift of every live hub entry;
  the slug-entry form naming `lyx tender resume slug`;
  a live waiting marker of the holder's own pid passing with a parent lock record present;
  paused with no parent lock record passing;
  a lock free at once passing with no sleep called.
  The card states what this test pins and what `TestRunLockHolderPasses` pins.
  Two choices were left to me: the known holder comes from an unfinished step record, and "without sleeping" is asserted through the counting sleep.
- **F2** (MEDIUM, consistency), fixed as proposed: card 10's `runLockRefusal`, `awaitRunLockOrPass` and the `awaitOrPass` substrate take a `retry string`.
  Every "the removal again" and "retrying the removal" clause of the refusal now names the retry step, and so does the parent-lock clause "only the retry once that pid ends" (from the self-check).
  The doubled "then then" is gone.
  `EndSession` passes "the removal".
  `TestRunLockHolderPasses` asserts the retry step on every refusing row, and `TestTeardown_CallSequence` asserts that the fake sees "the removal".
  In card 12, `EndBeforeSessionDown` passes "`lyx reed down`" and returns the refusal as it is.
  `TestEndBeforeSessionDown` asserts that the fake sees that retry step on every row that calls it.
  The `*runproc.SurvivorsError` wrap stays.
  I also removed the duplicated "the run lock path and the run lock path" in card 10's test sentence.
- **F3** (LOW, consistency), fixed as proposed with one wording change: card 11's `TestRemoveVerdict` gone-first rows now assert that the refusal contains the other branch's `WayForward` text and not the gone text, and that the closing "then `lyx fabric remove <slug>` again" appears in every row.
  The proposal called the gone text "the sentence ending" in "removes the pair and clears this record".
  That text is a clause followed by more text (`internal/fabricengine/failedpush.go:230`), so the card names it "its clause".
- **F4** (LOW, consistency), fixed as proposed: card 17's Edit list adds `internal/shuttleengine/claudeengine/doc.go`.
  Its corpus paragraph says that the replay-corpus sentence gains `waiting_held`.

## Self-check

- fix-consistency, LOW: card 10's parent-lock clause "only the retry once that pid ends" still had the old wording — fixed as proposed.
- fix-proposal, LOW: the same clause, through which `reed down` could still print a removal — merged with the fix-consistency finding and fixed as proposed. The test sentence now says every refusal names the retry step, and its parent-lock row asserts the retry step.
- fix-claims: no findings.
- fix-affected-set: no findings. It left F3 unchecked because F3's premise is not a set.

## Disputed

None.

## Deferred

None.
