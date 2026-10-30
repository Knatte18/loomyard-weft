# Fixer-report, round 1

## Changes

- **F1** — fixed as proposed, with the marker the review suggested: `LandState` gains `Refused`.
  A refused `Land` posts its notice once and records `{Refused: true}`; the episode stays open, so each tick asks `Land` again and posts nothing more, until it succeeds (`recordLand(false)`) or the reading stops passing.
  The parent-owner notice still closes the episode.
  Test: a new `TestLandBeam` row with one refusal then a success, asserting one notice and a landing; the refusal-once row no longer pins the land count.
- **F2** — fixed as proposed, as a tier1 test, not tier0: `hubforge` is tier1 throughout `internal/beamcli`.
  `TestHubDobbyDeps_LandsOnlyAFlaggedBeam` drives the real dobby deps over a hubforge hub: an unflagged beam reads as not passing and `Land` refuses naming `lym beam ready b` with no request written; after `markReady`, the reading passes and `Land` writes the land request.
- **F3** — fixed as proposed (idempotent commit).
  The `MarkReady` dep skips the write when the record already flags the beam and still commits; `markReady` always calls the seam.
  Tests: `TestMarkReady` rows for a failing `MarkReady` dep and for a flagged beam that still commits (`markCalls`), and a tier1 test that an uncommitted flagged record is committed and not rewritten.
- **F4** — fixed as proposed: `TestLand` rows for owner-before-ready, ready-before-work-remaining and an unreadable ready record; `landWorld` gained `readyErr` and `markErr`; `TestMarkReady` rows for each error's way forward.
- **F5** — fixed as proposed for the row: `unflagged` is set on the every-failure row and the ready line is last in its `want`.
  `pawlRecheck`'s own read is not driven by a test: it cannot be reached over a hermetic worktree without the whole recheck, and the review allowed leaving it.
- **F6** — fixed as proposed: the `tick` comment ends with "the beam's landing step".
- **F7** — fixed as proposed: the `internal/loomcli/doc.go` re-check paragraph names the ready record and `lym beam ready <slug>`.

The commits group findings by package: F1 and F6, F2, F3 and F4 together (they share `land_test.go`), F5 and F7, then the self-check follow-up.

## Self-check

- fix-consistency, LOW: the `doc.go` "Bound" bullet and the `State.Land` comment read as if a refusal closed the episode — fixed as proposed.
- fix-affected-set, LOW: the `State.Land` comment — fixed, same change as above.
- fix-affected-set, LOW: `doc.go` lines opening the landing bullet read as if every notice closes the episode — fixed by qualifying them.
- fix-affected-set, LOW: the `doc.go` "Bound" bullet is ambiguous — fixed, same change as the consistency finding.
- fix-code-correctness, LOW: nothing pinned the commit of an already flagged record — fixed with a tier1 test.
- fix-proposal: no findings.

All four forks returned a report.

## Disputed

None.

## Deferred

None.
