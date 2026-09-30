---
verdict: APPROVED
rationale: "All nine round-1 findings are fixed in the current cards (card 1 skips the budget arm for an exempt Stuck and tests the spent-budget case; cards 5 and 7 add real-pair integration tests of the records pathspec; card 4 adds reedengine.ErrNoSession and card 9 maps it to not live; cards 10 and 15 cover the stale docs; card 10 bounds the done-seen marker; card 13 adds CleanupResult.ArchiveSkippedReason; card 14 pins the untracked drive-report case), and a re-check against the decision record and the code finds no new defect"
---

# Plan-Review round 1: Bouncer verdict

The round-1 report raised one BLOCKING, three MEDIUM, four LOW and one NIT finding, and the fixer report claims all nine are fixed.
I checked each fix against the current card text and, where a fix rests on code, against the code.

- **F1 (BLOCKING), exempt Stuck blocked by a spent budget.**
  Card 1 now applies `Shed.stepLocked`'s budget arm to counted Stucks only, says why skipping the count alone is not enough, and tests an exempt Stuck after exactly N counted Stucks.
  Card 10's shedengine-driven tests seed N counted Stucks (budget fully spent) for both the done wait and the awaiting wait.
  Resolved.
- **F2 (MEDIUM), records pathspec never exercised against git.**
  Card 5 targets `wiring_commitstatus_integration_test.go` with a friction-note-plus-drive-report case and an archive-rename addition-plus-deletion case over the real `loomCommitStatusDeps`.
  Card 7 creates `commitrecords_integration_test.go`, covering commit-and-push and the clean-tree no-op.
  Card 16 names those tests as the proof of the link its `Spawn` stub stands in for.
  Resolved.
- **F3 (MEDIUM), absent reed session read as an error.**
  I confirmed in `internal/reedengine/lifecycle.go` that `Status` calls `requireSessionLocked`, which returns a plain `errors.New(noSessionMessage(...))`, and that `StatusResult.Strands` carries `Name` and `Live`.
  Card 4 wraps a new `ErrNoSession` sentinel on that return only, and tests it over the existing `newColdScratchEngine` fixture.
  Card 9 factors `driverAliveFrom` and maps the sentinel to "not live" with a table test.
  The Shared Decision is updated, and `./internal/reedengine/...` is on both `go test` lines.
  Resolved.
- **F4 (MEDIUM), stale docs.**
  Card 10 covers `battenshed/doc.go`'s report path and never-liveness sentence, the overview's halted-child-restart and 12-hour sentences, and the `maxStatusHistoryEntries` comment, which it also adds as a target.
  Card 15 adds the overview's teardown remedy and archive-tag sentences.
  A grep for `.lyx/loom/friction` and the old drive-report location finds only `battenshed/doc.go` and `loomcli/run.go`, which cards 10 and 3 cover.
  `manifest/designs/` does not name these paths.
  Resolved.
- **F5 (LOW), done-seen marker lifetime.**
  Card 10 removes the marker on Done and on any non-done Call, makes removal failures hard errors, and asserts the past-grace warning through `logger.SetOutput`, the pattern `internal/loomshed/gatefindings_test.go` already uses.
  Resolved.
- **F6 (LOW), no-origin skip reason in Cleanup.**
  Card 13 targets `CleanupResult` with a once-per-verb `ArchiveSkippedReason` and a no-origin test.
  Resolved.
- **F7 (LOW), sentinel test case.**
  Card 14's dirty file is now a new untracked file in a not-yet-tracked drive-reports directory.
  Resolved.
- **F8 (LOW), card 1 ImpactSummary.**
  The summary now names both `episodeStuckCount` callers, and the intent carries the status-rendering half of the open risk.
  Resolved.
- **F9 (NIT), start-on-awaiting premise.**
  Card 10's intent states the verified `start.go` premise.
  Resolved.

The report has no `## Focus departures` section, so there is nothing to ratify or reject.
Its settled focus items 2 and 6 (whole `_lyx/loom` pathspec; card 9 and 10 granularity) are not defects, and I agree with both.

## Fresh pass

- **Decision-record fidelity.**
  Every Decision and Constraint maps to a card.
  Awaiting watch, per-approval resume and done wait are in cards 9 and 10.
  The exempt flag is in card 1.
  Friction and drive-report relocation are in cards 2, 3 and 6.
  The status commit and `commit-records` are in cards 5, 7 and 8.
  The archive on every weft-branch deletion is in cards 11 to 13.
  The teardown remedy is in cards 14 and 15.
  The recipe header, `driver_exit_grace_s` and CONSTRAINTS.md are in cards 10 and 12.
  No card adds unlicensed scope.
  Card 12 places the archive before `removePortal`, while the dirty refusal fires after portal teardown, so a retry after `commit-records` archives a new tip under a new tag.
  The archive-tag-shape Shared Decision anticipates exactly that retry.
- **Code premises.**
  `landingshed.ReadApproval`/`WriteApproval`, `fabricengine.WeftWarpSlug`, `battencli.createRefusal`, `wireForHub`, `recordSpawnConfirmed`/`SpawnConfirmedFile` and `defaultInnerRunPollIntervalS` all exist as the cards assume.
- **Verify coverage.**
  Every targeted Go package is run plain and under `-tags integration`.
  The ly-drive skill is tested from `cmd/lyx`, and the batten recipe from `battenrecipe`.
- **Granularity.**
  Card 4 pairs a pure constant relocation (no tests of its own) with the reed sentinel.
  Both are prerequisites for the same liveness probe, and the relocation has no independently testable behavior, so this is acceptable.
- **Custom cards and ImpactSummary.**
  No `Custom` card exists.
  Every `Edit`/`Delete` ImpactSummary states a real blast-radius conclusion.
