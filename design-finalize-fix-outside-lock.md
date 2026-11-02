# Finalize releases the parent lock while its fixer works

`finalize-fix-outside-lock` · Task · enhancement, landing, loom

## Seen (orch, 2026-11-01)

Finalize takes the parent lock (`landingshed/parentguard.go`) before it merges the parent in and verifies, and holds it through the whole row: verify, and on a red verify the fixer session and the re-verify.
`plan-spec-split` held the lock on `main` for 12 minutes: its first verify failed tier0, a fixer worked about 5 minutes, then the re-verify ran every tier.
Meanwhile four green runs queued behind it (`darn-sonnet-helpers`, `readme-profile-in-recipe-column`, `dobby-idle-notice-row-end`, `reed-loses-parts-chair`), each to verify under the lock in turn.

## Design (operator, 2026-11-01)

- Finalize keeps taking the lock before its merge and verify, so a run that verifies green lands what it verified, with no second merge.
- When verify fails, Finalize releases the lock before the fixer starts, so the next run in the queue can land meanwhile.
- When the fixer is done, the run queues for the lock again, merges the parent anew, and verifies under the lock as on its first try.
  When nothing landed on the parent while it fixed, that merge is a no-op and only the re-verify remains, which a red verify runs today too.
- A run back from its fixer goes to the head of the queue, ahead of runs that queued while it fixed (operator, 2026-11-02).
- Taking the lock only at the very end was weighed and rejected: the parent moves between the run's merge and its landing, so the run would merge, verify and fix again, and could starve while much lands.

## Hand-over latency (orch, 2026-11-01)

A queued run polls the lock from one second, doubling to thirty seconds while the holder is unchanged (`parentlock/acquire.go`), so after a long hold the next run takes the lock about 20 to 30 seconds after the release.
Two hand-overs on 2026-11-01 took 23 and 21 seconds (`plan-spec-split` to `darn-sonnet-helpers`, `dobby-idle-notice-row-end` to `reed-loses-parts-chair`).
The release wakes the head of the queue at once, through a notify on the lock directory or a lower poll ceiling of a few seconds.

## Left to the run

How the wait mark and `lym loom status` show a run fixing outside the lock, and how the recovery ladder counts a second failed verify.
