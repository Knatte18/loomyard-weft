# Fixer report — plan, round 2

Every finding's premise held against the cards and code it cites, and each was fixed as proposed.
`lyx loom validate-plan` over the edited plan returns `ok: true`, with only the informational `create-new-unit` findings it already reported.

## Changes

- **F1** (BLOCKING): card 16 lists `PATTERN.md` under `Edit:`.
  Its Intent replaces the false "the background says" sentence with two prescribed edits.
  The first rewrites `PATTERN-driver-choice-single-site.md`'s opening sentence to name loom's `start` and `resume` and `lyx loom drive`'s own package as readers, each selecting only the driving surface.
  The second changes the `PATTERN.md` line to "in its bootstrap verbs (loom's `start` and `resume`)".
  The ImpactSummary names the widened invariant text.
- **F2** (MEDIUM): card 11's Intent says that, for a `friction: failed` done, `handBackRun` writes the report, sends the notice under the `OrchRecorded` rule and records the outcome, but calls no `commitAndPark`.
  `TestDriveHandBack` gains a row asserting the report with its send line and no park marker.
  Card 12's `done` bullet builds a `driveReport`, takes `NewReportPath` and calls `handBackRun` after `CommitRecords`, and its `done` test row asserts the report on disk and no park marker.
- **F3** (MEDIUM): card 20's `binaryRestep` now calls `Spawn` first and writes the marker only after `Spawn` returns nil.
  A refused spawn writes no marker.
  Start's driver-lock check bounds a batten crash between the spawn and the marker write.
  The test gains the refused-spawn-then-retry row.
- **F4** (LOW): card 12's step 4 skips the recheck for `done` and the busy exit.
  The busy row now runs with the run lock held by another process and asserts that the holder is logged.
- **F5** (LOW): card 2's `TestPrepareRunSettings` bullet gains the `loom.tackler_max_spawns: -1` refusal row.
- **F6** (LOW): card 15's wiring says `VerifyRecord` reads `verifytree.ReadPublishFailure` and maps `Row`, `Head`, `Target` and `WrittenAt` onto `battenshed.ChildPublishFailure`, as `readChildPublishFailure` does.
  It also says `EscalationPending` wraps `PendingEscalation` and drops the brief path.
- **F7** (LOW): card 13 writes the `spawn` record for `tackle` and `arbitrate` only after the call returns a fresh attempt number, and writes none for a refusal, a reused verdict or an attached attempt.
  The Budgets rows and the restart rows now assert no `spawn` record, and the general assertion covers only rows that start a fresh attempt.
  This leaves card 7's `Closed` correct, since it compares against the attempt record's `SpawnedAt`, not the settle record.

## Disputed

None.

## Deferred

None.
