# Round 3 fixer report — discussion

## Changes

- **F1** (MEDIUM, fixed differently in part), `_lyx/discussion/decision-record.md` D9's InnerRun bullet and two acceptance criteria.
  Removed the driver-dead-notice suppression below `respawnCap` and kept the quiet-notice suppression at Beam-Await.
  D9 now states that InnerRun's existing driver-dead notice applies at Beam-Await as at any row, once per state episode of the child, and that a respawn count reset by a status write lets the respawns go on.
  Dropped "and without driver-dead notices below `respawnCap`" from the `lyx beam create` criterion.
  Not made as proposed: the reviewer's reworded criterion, "a Beam-Await driver found dead sending the driver-dead notice naming `lyx loom start`".
  Below the cap the notice text is the plain "driver strand is dead" (`conditionText`, `internal/battenshed/notice.go`), and only the cap's text carries `loomStartRemedy`.
  The criterion now reads that a Beam-Await driver found dead sends one driver-dead notice in its state episode, also when it dies after re-entering the row and writing its status.
  Also not carried over: the proposal's "sent once per episode at the first probe that finds a running child's driver dead", reworded as "a probe that finds the running child's driver dead sends it, once per state episode", without claiming which probe.
- **F2** (MEDIUM, fixed as proposed), D9's failed-re-check bullet.
  A failed dobby restart leaves the dobby-stop marker standing and still removes the request.
  The orch failure then names both the re-check's blockers and `lyx beam start <slug>`, which passes because the status write lifted the hold.
  The next accepted request restarts the dobby on its own failed re-check.
  Added an acceptance criterion for this case.
- **F3** (LOW, fixed as proposed), D5's last Upgrade sentence.
  Narrowed the older build's failing writes to an upsert, patch or merge of the entry, and to the replay of its own local commits that touch it.
  Added that its `SetRunStatus` and owner-setter writes skip `normalizeTask` and still succeed.
  "Its other entries unaffected" and the `update-plugins.sh` way forward are kept.
- Self-check fixes: D9 now states that the cap's text naming `lyx loom start` goes out only when the cap is reached before any driver-dead notice of the episode, and is otherwise never sent.
  D5 now states that the older build's run-status and owner writes succeed locally, and reach origin only when their push needs no replay.

`_lyx/discussion/support-log.md` is unchanged: none of these findings touches it.

## Self-check

- fix-consistency, LOW, D9's "`respawnCap` changes only its text, which names `lyx loom start`" read as if the cap notice always arrives: fixed as proposed, merged with the fix-claims and fix-proposal findings below.
  D9 now says the cap's text goes out only when the cap is reached before any driver-dead notice of the episode.
- fix-claims, MEDIUM, same claim: the cap text can never be sent once a notice below the cap went out, since the episode key is the same (`notice.go` `noticeCondition`/`noticeStep`). Fixed as proposed, taking the first of its two options; the second, a build change, is not needed to make the record true.
- fix-proposal, MEDIUM, same claim: fixed by the same change.
- fix-proposal, LOW, F3's "still succeed" contradicts the refused replay of the same local commits: fixed as proposed by qualifying the sentence as "succeed locally, and reach origin whenever their push needs no replay".
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.
