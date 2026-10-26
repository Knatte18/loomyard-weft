---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:170
    summary: "D10 drops the live-orch refusal only from `lyx beam land`, yet Finalize (D9), cancel (D11) and the can-land notice (D12) all call `BeamLandRefusal`, which today refuses while the beam's orch or watcher is live, so as written every pawl Finalize halts, every memberless cancel refuses and the notice never fires"
    fix: "In D10, replace the bullet 'The refusal of a live orch goes, since Beam-Down stops it.' with a bullet saying `BeamLandRefusal` itself loses its orch and watcher blocker (`beamLandRefusal`'s orchLive/watcherLive cases, internal/battencli/beamland.go), since the beam's orch runs until Beam-Down, so every caller (`lyx beam land`, Beam-Await's re-check, Finalize, `lyx batten cancel <beam>` and the dobby's can-land check) judges members and member pairs alone."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:141
    summary: "A failed accept restarts the beam's dobby only when this accept stopped it, so an accept retried after a failed status write, or re-entered after a crash after the dobby stop, finds the dobby already down, leaves it down on a failed re-check and names no way forward"
    fix: "In D9's accept bullet, make the accept record that it stopped the dobby in a marker under the beam worktree's `.lyx/beam/` before stopping it, have the failure path restart the dobby whenever that marker stands and remove the marker after the restart, and have the success path remove it at Done; add an acceptance criterion that an accept re-entered after a crash between the dobby stop and the request's removal, whose re-check fails, leaves the beam's dobby live."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:141
    summary: "The accept stops the dobby before it writes the Parent-Align hold, so in that window `lyx beam start` (D3) or a member's hand `lyx batten run` (`ensureParentDobby`) reads no hold and restarts the dobby, which then stays live to Beam-Down against D1's rule"
    fix: "In D9's accept bullet, reorder the steps to write the run status naming Parent-Align first, then stop the beam's dobby, then re-run the checks, and state that a member a still-live dobby starts between the two is refused by its Worktree-Create's post-pair read; keep the sentence that either that read or the re-check sees the other."
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:141
    summary: "The accept writes the status naming Parent-Align and then re-runs all of `lyx beam land`'s checks (D10), one of which refuses a beam whose pawl run is not at Beam-Await as read from the board's producer half, so the re-check refuses its own accept"
    fix: "In D9's accept bullet, replace 're-runs `lyx beam land`'s checks (D10)' with the list it re-runs: `BeamLandRefusal`'s members and member pairs, the beam's owner and its parent's owner; state that the run-at-Beam-Await check is not re-run, since the row running the accept is that check."
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md:151
    summary: "D9 suppresses InnerRun's driver-dead notice at Beam-Await and says a restarted batten re-spawns the driver, but InnerRun stops respawning a running child after respawnCap respawns with no status-file write between them, and a weeks-long Beam-Await writes none, so a few driver deaths leave the landing wait with no driver and no signal"
    fix: "In D9's InnerRun bullet, keep the quiet-notice suppression but keep the driver-dead notice at Beam-Await once respawnCap is reached (internal/battenshed/innerrun.go:186,445; notice.go:384), naming `lyx loom start` in the beam worktree as today; or state that each Beam-Await re-entry writes the child's status file so the cap counter resets, and add an acceptance criterion for a Beam-Await driver killed more than respawnCap times."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:154
    summary: "D9 lifts the run lock on the beam entry through Beam-Await with no bound, so a board write can change the live beam's project (its parent), kind or recipe, or remove or merge the entry, while its pair, dobby and pawl run exist"
    fix: "In D9's hold bullet, after 'the run lock leaves its fields editable', add that a beam project entry with a pawl run in flight still refuses a change of `project`, `kind`, `recipe` and `beam` and its removal or merge at every row, naming `lyx batten cancel <slug>` as the way forward, while title, brief, body, order and labels stay editable; add an acceptance criterion for the refused re-parent."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:91
    summary: "D5's mixed-version bound says an older build still reads and writes task owners, but today's `normalizeTask` refuses any project carrying a recipe on upsert and on replay, so once `lyx beam create` writes `recipe: pawl` an older build refuses that entry"
    fix: "In D5's upgrade bound, state that an older build refuses a beam project carrying `recipe: pawl` (internal/boardengine/task.go:289,307, replay.go:309), so its board replay refuses that entry and its writes to it fail until the machine is updated through `update-plugins.sh`; keep the `recipe: pawl` write, since the board entry says a beam project's entry carries it."
  - id: F8
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:100
    summary: "Adopting root alone while another machine owns a beam G leaves G's pawl run with no parent dobby on either machine, since the old prime dobby exits and the adopter does not resume a run it reads owned elsewhere, and D6 states no bound for that window"
    fix: "In D6, after the sentence on runs reported owned elsewhere, add a bound: until the same machine owns both G and root, G's pawl run gets no batten pass on either machine and `lyx batten status G` shows it at its last row undriven; the landing stays refused by D10's parent-owner refusal, and `lyx adopt G` on the root owner, or `lyx adopt root` on G's owner, resumes it."
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:184
    summary: "The cancel refusal reads this machine's local Bolt, so a member another machine added while the beam was at Beam-Await is missed and the cancel removes the beam under it, and D11 states no bound"
    fix: "In D11's cancel bullet, state that the cancel first runs the hub-scope origin reconcile and judges `BeamLandRefusal` over the Bolt it pulled, refusing with the reconcile's way forward when origin is unreachable."
  - id: F10
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:165
    summary: "`--anchor` survives only once the first pass wrote batten's run record, so a re-run of `lyx beam create <slug>` after a first pass that died earlier forks the pair at the default anchor"
    fix: "In D10's re-run bound, extend the sentence naming the same `lyx beam create <slug> [--anchor <dir>]` after a covering pause to the flagged-beam-with-no-run-status case, so the operator repeats `--anchor` there as well."
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:146
    summary: "The Finalize backstop halts with the refusal's ways forward per member, but a member with a pair is told to land with `lyx batten run <member>`, which the standing hold refuses, and no route returns a held pawl run to Beam-Await"
    fix: "In D9's Finalize bullet, state that the halt names, per member, only `lyx batten cancel <member> --remove` or moving its `project` off the beam with `lyx board upsert`, then `lyx loom resume` in the beam worktree, never the land-it way forward."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md:37
    summary: "D1's sentence listing every `EnsureLive` caller leaves out Beam-Up, which runs the beam steps, and Beam-Await's failed accept, which restarts the dobby"
    fix: "In D1, add Beam-Up (its beam) and Beam-Await's failed accept (its beam) to the sentence 'Every `EnsureLive` caller starts only the beams it names'."
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md:251
    summary: "The acceptance criterion that a driver re-spawned after `lyx beam stop` posts no notice tests a case the record never produces, since `lyx beam stop` spares the pawl driver"
    fix: "In the acceptance criteria, replace 'a driver re-spawned after `lyx beam stop` posts none' with 'a Beam-Await re-entered with its status naming Beam-Await and no request standing posts none'."
---

Every finding is judged against the rubric and the declared `PATTERN-owner-machine` text.
The forks and I reached the four focus questions independently, and the verdicts on them follow the findings.

### [BLOCKING] `BeamLandRefusal` keeps the orch blocker its pawl callers cannot pass

**Location:** `_lyx/discussion/decision-record.md:170`, with D9 (line 146), D11 (line 184) and D12 (line 190).
**Issue:** `BeamLandRefusal` judges the beam's open members, its child pairs and whether its orch strand or watcher is live (`internal/battencli/beamland.go:58-72`, blockers at 86-92).
Under this design the orch runs from Beam-Up to Beam-Down, so it is live at every caller: Finalize's re-check, the cancel refusal and the dobby's can-land check.
D10 drops the live-orch refusal only in `lyx beam land`'s bullet.
Read as written, the pawl run never lands, a memberless beam is never cancelled and the can-land notice never fires.

### [MEDIUM] A failed accept can leave the beam's dobby stopped for good

**Location:** `_lyx/discussion/decision-record.md:141`, 144 and 145.
**Issue:** A failed status write keeps the request, and the next wake retries the accept.
A crash after the dobby stop leaves the request standing, and the re-entered row accepts again.
In both cases the second attempt finds the dobby already down and did not stop it, so on a failed re-check line 144 does not restart it.
The beam is then back at Beam-Await with no hold and no dobby, and the failure post names no `lyx beam start`.
Four lenses raised this independently.

### [MEDIUM] The dobby can be restarted between the accept's stop and its hold write

**Location:** `_lyx/discussion/decision-record.md:141`, against D1 line 38 and D3 line 60.
**Issue:** Until the Parent-Align write no hold stands, so `lyx beam start` passes D3 and a member's hand pass restarts the dobby through `ensureParentDobby` (`internal/battencli/cli.go:506-540`).
Nothing stops that dobby again before Beam-Down.
Writing the hold first closes the window: the member's post-pair read still catches a start the dobby makes before it stops.

### [MEDIUM] The accept's re-check includes the check the accept just made false

**Location:** `_lyx/discussion/decision-record.md:141`, against D10 line 169 and D9 line 153.
**Issue:** The accept writes Parent-Align and then re-runs `lyx beam land`'s checks.
Those include "a beam whose pawl run is not running at Beam-Await", and D9 makes every reader judge the row from the board's producer half, which now names Parent-Align.

### [MEDIUM] The Beam-Await driver stops being respawned with no signal

**Location:** `_lyx/discussion/decision-record.md:151`.
**Issue:** InnerRun respawns a running child with no driver at work at most `respawnCap` (3) times with no status-file write between respawns, and then spawns nothing (`internal/battenshed/innerrun.go:186`, 347-348, 445-447).
Past the cap, the driver-dead notice is the only signal (`innerrun.go:392`, `notice.go:384-385`), and D9 suppresses it at Beam-Await.
Beam-Await is meant to wait for weeks, and the record names no status-file write during the wait.
Its premise that "a restarted batten re-spawns the driver, which re-enters the row" fails after the cap.

### [MEDIUM] Lifting the run lock at Beam-Await has no bound

**Location:** `_lyx/discussion/decision-record.md:154`.
**Issue:** `checkRunLocks` refuses a held entry's removal and every `scopeChanged` field: Project, Kind, Recipe, Beam and others (`internal/boardengine/runlock.go:68-86`, 107-125).
D9 lifts that lock wholesale until Parent-Align.
Only the beam flag (`project.go` beam check) and the slug (`rename.go:47`) stay guarded on their own.
The rubric's attack-surface rule asks a downgraded guard to state what it now lets through and what bounds it, and D9 states neither.

### [MEDIUM] The mixed-version bound is false for `recipe: pawl`

**Location:** `_lyx/discussion/decision-record.md:91`, with D5 line 87 and D10 line 164.
**Issue:** `normalizeTask` calls `checkProjectFields` for every project, and that function refuses a project carrying a recipe (`internal/boardengine/task.go:289`, 301-307).
`normalizeTask` runs on upsert (`task.go:388`, 445) and on replay of origin's changes (`replay.go:309`, 391).
So an older build refuses a beam project carrying `recipe: pawl`: it does not "still read and write task owners" as D5 claims.
Dropping the write is no fix, since the board entry says a beam project's entry carries `recipe: pawl`.

### [LOW] Adopting root alone leaves a beam owned elsewhere with an undriven pawl run

**Location:** `_lyx/discussion/decision-record.md:100-105`.
**Issue:** Machine A owns root and G, and machine B runs `lyx adopt root`.
A's prime dobby exits on its owner read, so no batten pass of G's pawl run is scheduled on A.
B reads the run as owned elsewhere and does not resume it (line 105).
The landing stays safely refused by D10's parent-owner refusal, and either machine's adopt resumes the run.
D6 still states no bound for this window.

### [LOW] The cancel refusal reads a possibly stale local Bolt

**Location:** `_lyx/discussion/decision-record.md:184`.
**Issue:** A member upserted on machine B while G is at Beam-Await is not on A's local Bolt until A pulls.
A cancel on A then passes `BeamLandRefusal` and removes the beam under that member.

### [LOW] `--anchor` is lost when a re-run follows a died first pass

**Location:** `_lyx/discussion/decision-record.md:165-167`.
**Issue:** The anchor rides into batten's run record at the first pass, and D10's re-run case "a flagged beam with no run status, left by a first pass ... that died" gets a fresh first pass.
A re-run of `lyx beam create <slug>` without `--anchor` forks at the default anchor, and the bound names repeating `--anchor` only after a covering pause.

### [LOW] The Finalize backstop names a way forward the hold refuses

**Location:** `_lyx/discussion/decision-record.md:146`.
**Issue:** For a member with a pair, `beamLandRefusal` names `lyx batten run <member>` (`internal/battencli/beamland.go:80`).
Under the standing hold, D9 refuses that member's Worktree-Create, and no route returns the held pawl run to Beam-Await.

### [LOW] D1's `EnsureLive` caller list omits callers the record adds

**Location:** `_lyx/discussion/decision-record.md:37`.
**Issue:** Beam-Up runs the beam steps (D8, line 126), and Beam-Await's failed accept restarts the dobby (line 144), yet D1's sentence presents its callers as the full set.

### [LOW] One acceptance criterion tests a case that cannot arise

**Location:** `_lyx/discussion/decision-record.md:251`.
**Issue:** D4 ends the dobby by its role and spares the pawl driver, so `lyx beam stop` never re-spawns that driver.
The criterion's intent is a re-entry with no request standing and the status at Beam-Await.

## Focus verdicts

- Beam entry run hold: one rule over the producer half covers each reader the directive names.
  `checkProjectField`, `ComputeLayers`, `beamForkPrecondition`, `ResolveBattenPrime`, D3's refusal and `lyx loom peers` each read `IsRunStatus` and switch cleanly at the hold.
  The run lock is the exception: lifting it is unbounded (F6).
  The other `IsRunStatus` readers survive: rename, cancel, `settleClaim` and run_settings.
- Parent dobby capacity: D9's last bullet states the bound.
  A pawl run takes no slot through Beam-Await, halted or not, and takes one from Parent-Align on.
  Dobby's `TaskView` carries the board status the rule reads, so a long Beam-Await does not keep the parent dobby from starting ready tasks.
  This is not a defect.
- Beam-Await accept ordering: a member's start or fork after the hold write is caught, either by the re-check or by its Worktree-Create's post-pair read.
  The stop-before-hold window lets the dobby be restarted (F3).
  The re-check refuses its own accept (F4).
  A crash between the dobby stop and the request's removal leaves a state the re-entered row does not fully recover from, since the dobby stays down (F2).
  A crash between the request's removal and Done is recovered by the interrupted-landing notice.
- Adopt and ownership: in both orders, root then G and G then root, G's pawl run ends with one driver, on the adopting machine.
  Adopting G while its landing is in flight elsewhere writes the owner and owner-pauses the run on the old machine, and `lyx adopt root` resumes it on the new one, with no second driver.
  Adopting root alone leaves a beam owned elsewhere with no parent dobby on either machine, a window D6 leaves unbounded (F8).

## Rejected

- Handler candidate: the owner pause of a pawl run names `lyx adopt <beam>` (`internal/battenshed/innerrun_owner.go:57`), which looked like a dead end on the adopter. It is a valid way forward on the machine whose orch receives the notice: there it takes the beam back and the run resumes under D6's owns-both rule.
- Bound-breaker F2's alternative fix, to stop writing `recipe: pawl`: the board entry says a beam project's entry carries `recipe: pawl`, so F7 keeps the write and corrects the bound instead.
