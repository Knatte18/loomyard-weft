# Fixer report — discussion round 4

## Changes

All in `_lyx/discussion/decision-record.md`.

- **F1** (fixed as proposed, and extended): D5 Fresh launch now says `withdraw` acts on the receiver by name whether or not creel lists the strand and answers an empty result when nothing of the caller's is queued, and that `lyx orch start` runs the first withdraw before it removes the old strand.
  A new acceptance criterion covers a continue or adopt after `lyx orch stop`.
  Added for the twin case the proposal left open: `Store.Release` (`internal/creelengine/verbs.go:560-583`) refuses an unlisted name the caller holds no hold on, so D5 now says `lyx orch start` sends no first release when no orch strand is listed, and the second release ends any hold left under the fixed strand name.
- **F2** (fixed as proposed): the Fresh launch bound now covers a failed release: ORCHWATCH's hold stays on the relaunched orch, only its mail is reminded, and the hold lapses within the Hold bullet's one hour because the idle watcher renews none.
  The criterion on a refused second withdraw now reads "second withdraw or a release fails, refused or unable to reach creel" and pins that the hold lapses.
- **F3** (fixed as proposed): the TTL clause now says the exposed inject is typed on the first idle, unpaused probe only if that probe comes within the ten-minute TTL, which runs while the session is busy, and expires otherwise.
- **F4** (fixed as proposed): D5 says whichever caller's withdraw removes a pause-resume line, `lyx orch start` or the watcher's retried or phase-reset withdraw, clears the resume-noticed mark under the record lock, and the orchengine creel seam's withdraw returns the removed ids and subjects.
  D9's repost sentence names both watcher paths.

## Self-check

- fix-consistency, LOW: the line "that bound covers a failed second withdraw" was left stale by F2. Fixed as proposed: it now names a failed release too.
- fix-claims: no findings.
- fix-proposal, MEDIUM: a post-stop continue hit a refused first release on every run, so a failure was recorded as routine. Fixed with my own change: the proposed carve-out in `release` was not made. Instead `lyx orch start` sends no first release when no orch strand is listed, so creel's release keeps refusing a mistyped name. The post-stop criterion now says no fresh-launch failure is recorded.
- fix-way-forward, LOW: the same finding as fix-proposal's, merged with it and fixed by the same change.

## Disputed

None.

## Deferred

None.
