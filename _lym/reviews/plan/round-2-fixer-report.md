---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "The proposed new card 5 row (a second Run started by a spawn within the interval waits out its rest) is the behaviour card 5's existing row already pins (a second Run started 20 s after the last attempt, waiting 40 s). One test per behaviour, so the existing row now says it is started as a spawn starts it, with no push-now mark, and no duplicate row was added. Every other part of the fix was made as proposed."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
---

# Round 2 fixer-report

## Changes

- **F1** (card 9, card 19): card 9 edits `Topology.removePair`, `remoteTeardownJob`, `runRemoteTeardown` and `archiveWeftTip`.
  `removePair` reads the hold through `recordsPushCeiling` before its first mutation and carries it on the job.
  `archiveWeftTip` pushes the records branch only up to `<hold>^`.
  Card 19's dropped-landing row of `TestCommand_RemoveRefusesWhileALandingStands` runs plain and with `--remote`, and asserts that origin does not reach the held commit after the removal.
  Changed again by the self-check: the archive tag is also made at `<hold>^` (see Self-check).
- **F2** (cards 9, 10, 11): card 10 edits `PushRun` and `hubLandDeps`, lists `internal/beamcli/doc.go` and `pushledger#OwedTips` under Uses, and edits the two planned tests.
  With `CodeCeiling` empty on a beam pair, the ceiling is the first parent of the oldest first-parent commit past origin that is an owed tip or a `LandingCommits` commit, and `PushBeam` refuses on either.
  Card 9 now points at card 10's widening, and its beamcli doc line no longer names landings alone.
  Card 11 wires the parent seams for every beam pair target, one holding no landing included, and the "pair with no landing runs no parent step" clause is gone.
  New tier1 rows sit in `TestPushRun_StopsShortOfAHeldRecordsCommit` and `TestHubLandDeps_PushBeamRefusesWhileTheBeamHoldsLandings`.
- **F3** (cards 11, 19): `LandingStands`' trailer clause applies only with git and pushes both on, and skips a run whose board entry is `done`.
  Card 11 applies the same `done` skip to `settleParent`'s entry condition and the landing target's `Ahead`.
  New rows: card 19 removes a pushes-off landing's pair through `Teardown.RemovePair` with the variable set and unset; card 11's `TestParentPush` reads `Ahead` false over a board-done landing commit.
- **F4** (cards 4, 5, 6, 11, 19): card 4's `Entry` gains `Dropped`, mapping a dropped run to its landing commit, held and not owed.
  Cards 5 and 6 keep an entry that holds `Dropped`.
  In card 11, step 1 prunes `Dropped` once `LandingCommits` no longer lists the commit.
  Step 2 records the drop and resets the parent through `ResetToOrigin` before the hand-over for a run owned elsewhere, and a push carrying a commit still in `Dropped` stays refused.
  The re-record, step 4 and card 19's `LandingStands` read `Dropped` in place of the pair's `HeldRecords`.
  The false claim that `HandOverOwnedElsewhere` resets the parent is gone.
  New `TestParentPush` rows: an owned-elsewhere drop leaves the parent at origin's tip; a dropped run owned here with a newer landing on it, then removed, is neither re-recorded nor pushed.
- **F5** (overview, cards 10, 11, 19): a re-recorded landing holds the oldest commit of the pair's records branch past its tracking ref.
  Card 10 creates `fabricengine#KeepHeldRecords`, which writes `refs/held-records/<slug>`.
  `recordsPushCeiling` reads that ref when the ledger has no hold, and `Topology.removePair` deletes it.
  The drop in card 11 calls `KeepHeldRecords`.
  The overview decision and card 19's removal text name the ref.
  `TestParentPush` rows: the ledger-lost row now commits the done transition and asserts that origin lacks Finalize's records commit, and a new row deletes the ledger after a drop and checks that the pair push stops short.
- **F6** (cards 5, 6, 11, 19): every write of a records target's entry sets `Worktree` and `Slug`, in card 5's pass, card 6's starter and `--push-now`, and card 19's mark.
  Card 11's drop sets them on an existing entry too.
  `TestStart` asserts both fields, and `TestParentPush`'s drop row starts from an entry that already exists.
- **F7** (overview, cards 5, 7): `pushSpawnCommand` and `SpawnSync` leave `PushNow` unset, and the bound says the second process exits at the lock.
  The test lines and the overview decision now describe the line without `--push-now`.
  Card 5's existing second-`Run` row now says it is started by a spawn with no mark (see the frontmatter).
- **F8** (cards 6, 7): `SpawnSync` passes `--worktree` with the board directory.
  With `--board` or `--landing`, card 6's pusher locates the hub from `--worktree` and builds the target over the prime.
  Card 7 pins the line, and `TestPusherCmd_PushNowWakesAndExitsAtAHeldLock` gains a row that runs `SpawnSync`'s exact arguments.
- **F9** (card 20): `PushSummaryDeps` gains `Waits`, which `hubDobbyDepsStarting` fills from `battencli.HubRunSlugs` and `sheddriver.ReadDriveWait`.
  The owed period starts from the waits' `Since`.
  The Uses list gains `HubRunSlugs`, `ReadDriveWait` and `DriveWaitRecord`.
  A new `TestPushSummary` row covers a beam member's wait.
- **F10** (card 17): `catchUpPushSeams` gains `LockWait`, which `HubCatchUp` fills with `StatusPushLockWait` and the test sets to a few milliseconds.
  The test stays at about six seconds, which now holds because the held lock no longer waits out 30 s.

`lym loom validate-plan` reports `ok: true` with informational findings only, both after the fixes and after the self-check edits.

## Self-check

- fix-proposal, BLOCKING, the F2 seams on a beam with no landing leave a routine landing refused for good: fixed with own change. `SettleRefused` also clears a step-4 `held_back` record once every landing commit it named is recorded or gone. A `TestParentPush` row now covers the clear (card 11).
- fix-affected-set, BLOCKING, the F3 `done` skip is missing from card 10's trailer readers: fixed as proposed. `recordsPushCeiling`'s trailer read and the beam ceiling/`PushBeam` refusal skip a run whose board entry is `done`. A `TestPushPairAnchored` row covers it.
- fix-proposal, MEDIUM, F7 strands a beam's own commits behind a wait on the mark alone: fixed with own change. Card 5's wait on the mark alone now applies to the landing target only, and a beam pair target waits the interval in force.
- fix-consistency, MEDIUM, card 8's `RunCommitsAhead` does not read the new held-records ref: fixed with own change. Card 10 has `RunCommitsAhead` read its hold through `recordsPushCeiling` and adds a `TestRunCommitsAhead` row. The sub-point on card 17's `Held` seam is disputed, `premise false` as a defect: `Held` only picks the target handed to `Start`. A pair held by the ref alone pushes up to the hold and reads level (card 8 via card 10), so starting no pusher for it is correct.
- fix-consistency, LOW, cards 20 and 21 lack the held-only `Dropped` case: fixed as proposed, in both cards' text and rows.
- fix-consistency, LOW, card 3 says the summary reads waits from `RunView`: fixed as proposed.
- fix-consistency, LOW, the overview spawn decision's "Applies to" is stale: fixed as proposed; it now lists cards 6, 7, 11, 12 and 19.
- fix-claims, LOW, card 6's `--board` resolution is unspecified against `ResolveWorktree`: fixed as proposed. With either flag, the pusher skips the worktree resolution and takes `--worktree`'s parent directory as the hub.
- fix-affected-set, LOW, `landLocally`'s and `WakePair`'s entry writes do not name `Worktree`/`Slug`: fixed as proposed, in cards 12 and 11.
- fix-proposal, LOW, the archive tag still carries the held commit to origin: fixed as proposed. Card 9 makes the tag at `<hold>^` too, and card 19's row asserts that no branch or archive tag on origin reaches the held commit.
- fix-affected-set, LOW (information, outside the changed hunks), `HandOverTips`' code push of a beam pair can carry landing commits: deferred, see below.

## Disputed

None of the review's findings. One self-check sub-point is disputed in the Self-check section: card 17's `Held` seam.

## Deferred

- Self-check note (fix-affected-set, LOW): `HandOverTips` pushes a pair's owed code tip with `PushCommit` or a lease move (`internal/fabricengine/handover.go:290`), and no card caps that push for a beam pair carrying landing commits. This round's change set never touched this text, so the fix-check rubric puts it out of scope. It is a candidate finding for the next review round.

## Rulings

None reached this round.
