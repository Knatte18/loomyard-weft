---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:156-158
    summary: D9 removes the driver-dead notice below respawnCap on a false premise, and the cap it leaves as the bound never trips for a Beam-Await driver that writes its status and then dies
    fix: In decision-record.md D9's InnerRun bullet (lines 156-158), delete "and no driver-dead notice below `respawnCap`" and keep only the quiet-notice suppression at Beam-Await. State that InnerRun's existing driver-dead notice, sent once per episode at the first probe that finds a running child's driver dead, applies at Beam-Await as at any row, and that `respawnCap` only changes its text. Delete the sentences that make the cap the point at which the notice starts, and keep the sentence about the respawn count resetting. Drop "and without driver-dead notices below `respawnCap`" from the `lyx beam create` acceptance criterion (line 254), and reword the criterion at line 260 to a Beam-Await driver found dead sending the driver-dead notice naming `lyx loom start`. If the writer keeps the suppression instead, the Decision states a bound that a driver which re-enters the row, writes its status and dies again cannot slip under, such as a respawn count the entry write does not reset, and its own acceptance criterion.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:147
    summary: D9's failed-accept path restarts the beam's dobby but says nothing about a restart that fails, so the stop Beam-Await imposes on the dobby has no bound there
    fix: In decision-record.md D9's failed-re-check sentence (line 147), add what follows a failed dobby restart. The dobby-stop marker stays standing, the request is still removed, and the failure posted to the beam's orch names both the re-check's blockers and `lyx beam start <slug>`, which now passes because the hold is lifted. Add that the next accepted request restarts the dobby on its own failed re-check, since the marker still stands. Add an acceptance criterion that a failed restart leaves the marker and names `lyx beam start` in the orch notice.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:92
    summary: D5's mixed-version bound says an older build's writes to a beam project entry carrying recipe pawl fail, but only upsert, patch, merge and replay refuse it, while run-status and owner writes skip normalizeTask and succeed
    fix: In decision-record.md D5's last Upgrade sentence (line 92), narrow "its writes to that entry fail" to the paths that call `normalizeTask`, naming them as an upsert, patch or merge of the entry and the replay of that build's own local commits that touch it. Add that its `SetRunStatus` and owner-setter writes to the entry still succeed, so an older machine can still overwrite the beam project's run status and owner until it is updated through `update-plugins.sh`. Keep "its other entries unaffected" and the `update-plugins.sh` way forward.
---

### [MEDIUM] D9 removes the driver-dead notice below respawnCap on a false premise, and the bound it leaves is bypassable

**Location:** `_lyx/discussion/decision-record.md:156-158`, with acceptance criteria at lines 254 and 260.

**Issue:**
D9 says InnerRun "sends no quiet notice for a child at Beam-Await, as it sends none while a parent lock wait record stands, and no driver-dead notice below `respawnCap`", then sends the driver-dead notice "as at any row" once the cap passes.
The code does not behave that way at any row.
`noticeCondition` in `internal/battenshed/notice.go` returns a driver-dead finding for every running child whose driver is found dead (`if !alive { finding := noticeFinding{condition: noticeDriverDead, ...} }`).
Only the text changes at the cap: below it, `conditionText` supplies "driver strand is dead".
So the Decision is a Beam-Await-specific suppression of today's notice, not a description of current behavior.

It also states no bound that holds.
The bound it names is the cap: respawns count up only while no status-file write falls between them (`internal/battenshed/innerrun.go`, `baseline.After(p.respawnStatusTime)` resets `p.respawns`).
D9's own next sentence says a re-spawned driver that writes its status file on re-entering the row resets the count, "so the respawns go on".
A Beam-Await driver that re-enters, writes, and then dies, whether by a panic in the wait loop or a kill, is respawned on every pass indefinitely, and with the notice suppressed below the cap it is never reported.
The guard is removed, and the replacement bound covers only a driver that dies before its write.
The attack-surface question applies: the Decision that lowers a guard states what it lets through and what bounds it.

### [MEDIUM] A failed dobby restart on the failed-accept path is unspecified

**Location:** `_lyx/discussion/decision-record.md:147-148`.

**Issue:**
Beam-Await stops the beam's dobby on an accepted request.
On a failed re-check, D9 "restarts the beam's dobby whenever the marker stands and then removes the marker, removes the request, posts the failure ... and keeps waiting".
Retry and crash re-entry are covered through the marker, and the status write comes before the restart, so the restart never runs with a hold standing.
The record is silent on a restart that returns an error.
Either reading breaks something.
If the marker is removed regardless, the beam is left without a dobby, no hold stands, and nothing tells the orch, since the posted failure names only the re-check's blockers.
If the marker stays while the request is removed, nothing retries it until the next accepted request.
The acceptance criterion at line 259 promises a live dobby after a failed re-check and has no case for this.

### [LOW] D5's older-build bound overstates what an older build refuses

**Location:** `_lyx/discussion/decision-record.md:92`.

**Issue:**
`checkProjectFields` refuses a project with `recipe`, and `normalizeTask` runs it, as the record says (`internal/boardengine/task.go`).
`normalizeTask` is reached from `NewTask`, `ApplyPatch` (the upsert and merge paths) and the two replay paths in `replay.go`.
`Store.SetStatus`, which `Board.SetRunStatus` calls, and the owner setter's `writeOwners` do not call it, so an older build's run-status and owner writes to the entry succeed.
`Store.Load` does not normalize either, so the board still loads.
"Its writes to that entry fail" is therefore too broad.
"Its other entries unaffected" holds, since `validateWrite` and `checkProjectRules` do not re-run `normalizeTask` over other entries.

## Rejected

- Focus 1, callers of `BeamLandRefusal`: not a defect.
  D10's last bullet states once that the refusal loses the `orchLive` and `watcherLive` cases and that every caller (`lyx beam land`, Beam-Await's re-check, Finalize, `lyx batten cancel <beam>` and the dobby's can-land check) judges members and member pairs alone.
  In `internal/battencli/beamland.go`, `beamMembers` skips the beam's own slug and `beamChildPairs` skips the beam's own pair, so nothing the pawl run itself owns refuses it, and D9 and D11 word the same condition.
- Focus 2, order of the accept in D9: the order holds.
  The hold write comes before the marker and the dobby stop, so a crash before the marker leaves the dobby live and a crash after it restarts on a failed re-check.
  A start between the hold write and the stop meets the Worktree-Create hold read.
  A failed status write on the failure path retries the whole accept with the hold still standing, so the restart never runs under a hold.
  The re-check names members, member pairs and the two owners, none of which reads the run status, so the accept's own write makes none false.
  The one gap is the restart failure, filed as F2.
- Focus 3, the board run lock and the cancel refusal's read of local Bolt: stated and bounded.
  D9's last bullets say which fields stay editable until the hold, and which changes are refused at every row.
  D11 has the cancel run the hub-scope origin reconcile first and refuse with its way forward when origin is unreachable.
  The driver's respawn and notice behavior is F1.
- Focus 4: filed as F3.
