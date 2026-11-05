# Round 2 fixer report

Every finding's premise was checked against the code at HEAD (`internal/pawlengine/beamawait.go`, `internal/shedengine/run.go`, `internal/landingshed/pullrequestmode.go`, `internal/beamcli/doc.go`, `internal/pullrequest/doc.go`) and holds.
`lym loom validate-plan` reports `ok: true` after the changes, with informational findings only.

## Changes

- **F1** (MEDIUM), fixed as proposed, with one claim made exact.
  - Card 10 Intent: the entry finds an accepted landing in any of three states; the third is the dobby-stop marker standing with no land request, the state a resume of a halted run leaves. It feeds the same two branches, and the lifted-hold branch is now reachable by the board hold and by the dobby-stop marker.
  - The removal-order bullet names the window after `landChecks` and before `recheckStanding`'s first removal, which the third detection settles.
  - Reaching inputs and `TestBeamAwait` gain the two rows the fix asked for: `Recheck` passing ends Done, and no ready record lifts the hold and starts the dobby.
  - Overview Shared Decisions bullet names all three signals.
  - The proposed claim that a marker with no land request always means a landing the loop left is not exact: `refuse` leaves the marker after a failed dobby restart (`beamawait.go:347-356`). The card names that case and its outcome: the entry re-runs the re-check, which refuses again and retries the restart, or lands under the final ready record.
- **F2** (MEDIUM), fixed as proposed. Card 15 step 5 refuses a `Halt` with a message built from `landingshed.OriginMissing(origin)`, naming the demand, the missing fact and `lym beam ready <slug>` again. `OriginMissing` added to Uses. The no-upstream `TestMarkReady` row asserts `lym beam ready` and not `lym loom resume`; a `merge` parent row with the same fact is added. `validate-plan` passes with the Uses line.
- **F3** (LOW), fixed as proposed. Card 15 step 6 refuses a `PullRequestSide` or `ReadMergeState` error naming the cause, `lym beam ready <slug>` and `lym beam pr status <slug>`, with no record written; `TestMarkReady` gains the failing-`Status` row.
- **F4** (LOW), fixed as proposed. Card 3's `blocked` cause names a required check only when its conclusion is not `success`, `neutral` or `skipped`; `TestReadMergeState` gains the skipped-check row.
- **F5** (LOW), fixed as proposed. Card 12's `judgeAwaitingRun` drops the done branch and passes a run at Beam-Await in every state (running, paused, failed, blocked and awaiting); the `TestDescribe` row "a done run" becomes "a run at Beam-Down".
- **F6** (LOW), fixed as proposed. Card 15 rewrites `internal/beamcli/doc.go`'s ready paragraph; card 3 states that `internal/pullrequest/doc.go`'s first sentence names no caller list.

## Self-check

- fix-consistency, LOW: card 10's ordinary re-entry bullet and its `TestBeamAwait` row defined a re-entry by the await marker alone, which disagreed with the third detection when both markers stand. Fixed as proposed: the bullet and row now say no dobby-stop marker stands.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
