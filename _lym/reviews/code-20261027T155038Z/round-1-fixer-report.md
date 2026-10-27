# Fixer-report, code review round 1

All five findings fixed as proposed, one commit each, none pushed.

## Changes

- F1: `TestPrepare_CreelMailHookFollowsTheBoardDirectory` in `internal/shuttleengine/claudeengine/settings_test.go` builds the engine with and without a board directory, reads the `settings.json` `Prepare` writes, and asserts one matcher-less `PostToolUse` entry naming the board directory's flag directory, or none when standalone (commit ace9f750).
- F2: the `StartReconcilesAStaleFlagAndAMissingOne` row in `TestDaemon_Lifecycle` (`internal/creelengine/server_test.go`) leaves a stale flag and an unread journaled message before `Run` and asserts both are put right once the daemon serves (commit daa56313).
- F3 (commit b305ab0d):
  - Two rows in `internal/creelengine/deliver_test.go`.
    - `BusySendReleasesTheNoticeToAClaimAndAnswersTheHoldersWait`: the wait is answered and `ClaimAtBoundary` hands the notice out after a busy send.
    - `NoticeClaimedBeforeTheSendIsNeverTyped`: a claim lands between the idle check and the reservation, and nothing is typed.
  - `FlagFollowsThePoolWhenACallbackJournalsThenFails` in `flags_test.go`, driven through `Store.ReadThenFail` in the new `export_test.go`.
  - `creelfake.Delivery.SetIdleHook` in `internal/testkit/creelfake/creelfake.go`, the seam the claimed-first row needs.
- F4: `ReturnMovesTheFlagBackToTheSender` in `internal/creelengine/flags_test.go`. `newFlaggedEnv` now keeps its clock so the row can drive the sweep (commit 1b3fe733).
- F5: `ClaimAtBoundary` counts one more byte per line after the first against `boundaryClaimBudget` (`internal/creelengine/verbs.go`), with the comment and the `doc.go` claim paragraph saying so. The `BudgetCountsTheNewlinesJoiningTheLines` row in `verbs_test.go` pins both sides of the edge (commit 56650b75).

`lym gate test ./internal/creelengine` and `./internal/shuttleengine/claudeengine` pass.

## Self-check

Four forks ran; each returned a report, saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
