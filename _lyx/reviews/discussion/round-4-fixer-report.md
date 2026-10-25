# Round 4 fixer-report — discussion

F1, F3 and F4 are `decision`-class and went to the parent, so this report leaves them out.

## Changes

- **F2** (LOW, design), fixed differently in part, in `_lyx/discussion/decision-record.md` D4 and its acceptance bullet.
  As proposed: D4 now defines the owned-elsewhere exit code by the refusal, "every refusal of a pass that reads another machine as the run's owner", through one typed error that `ownedElsewhereRefusal` and the owned arm of `refuseResumeUnreachable` both return.
  The three sites are kept as examples, `refuseStartOwnedElsewhere` is added as the fourth, reached by `ownerPreflight` for a found Worktree-Create status with no pair here, and the acceptance bullet reads "any refusal that names another machine as owner".
  Fixed differently: the proposal said a fresh start's refusal from `claimBoardEntry` stays a refused start under D5.
  I did not write that, because `claimBoardEntry` runs inside the Worktree-Create row's `CreateWorktree` (`internal/battencli/wire.go:812`), after the seed is written, so a claim it loses fails the row, not the start.
  D4 instead says that such a loss fails the row and stops through D7's shed chain, that the code is set only for a refusal before the step loop and never from a row's error, and that a hand pass of the failed run reaches `refuseStartOwnedElsewhere` and exits with the code.
  The fresh-start half of the proposal is kept: a first pass with no status file refused as owned elsewhere exits with the code and stays a refused start.

## Self-check

- fix-consistency, MEDIUM: D7's "a full pass refused as owned elsewhere is neither" and the D4 acceptance bullet contradict the fresh-start refused start. Fixed as proposed, merged with fix-proposal's first finding: D4 states that a first pass with no status file is not yet a run dobby drives, and D7, the D4 acceptance bullet and the refused-start bullet are each scoped to match.
- fix-consistency, MEDIUM: the acceptance bullet covers the fresh start too. Fixed as proposed, merged with the finding above.
- fix-consistency, LOW: "a run it drives" is not defined against a fresh start. Fixed as proposed: D4 now reads "a run it drives, one with a status file".
- fix-claims, LOW: the fresh-start sentence covered every refusal of `refuseStartOwnedElsewhere`, not only the owned-elsewhere one. Fixed as proposed: it now reads "refuses it as owned elsewhere".
- fix-way-forward, MEDIUM: dobby never passes the failed run that `claimBoardEntry`'s loss leaves. Fixed as proposed: D4 names the route, the stop through D7's shed chain carrying the `lyx adopt <slug>` text, then a hand pass that exits with the code.
- fix-proposal, MEDIUM: a fresh start's refusal is handled as both a refused start and an owned-elsewhere pass. Fixed as proposed, merged with fix-consistency's first finding.
- fix-proposal, LOW: the shared typed error is also what `claimBoardEntry` returns from inside the row, so it is ambiguous whether a row error maps to the code. Fixed as proposed: D4 now says that the code is set only for a refusal before the step loop, never from a row's error.

## Disputed

None.

## Deferred

None.
