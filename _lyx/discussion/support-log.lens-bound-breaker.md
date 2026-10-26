```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked the record's bounds, guards and claims about existing code.

**Findings**

1. **BLOCKING — D11 (cancel) together with D10 and D13: a beam can now be cancelled and removed while it has members.**
   - Today a beam has no batten run until `lyx beam land`, which first passes `BeamLandRefusal` (no open members, no member pairs).
     So `lyx batten cancel <beam> --remove` could only reach a beam that was already empty.
   - Under pawl, the beam's batten run exists for the beam's whole life.
     `cancel.go` has no beam or member guard (its `cancelRun`, `endRunPhase` and `clearRunPhase` show none).
   - Counterexample: G has two member pairs forked from G's branch and running under G's dobby, and the operator runs `lyx batten cancel G --remove`.
     The cancel kills G's dobby and orch, then tears down G's pair and branch, and the members are left with a dead parent.
   - D11 states only that the cancel ends the beam's dobby and orch.
     It needs a bound: either cancel `--remove` of a pawl run refuses while `BeamLandRefusal` would refuse, or the record says what happens to the members.

2. **MEDIUM — D12: the "beam can land" condition is false at the moment dobby sees a landing.**
   - Dobby counts landings from the batten history's Land Done (`dobbyengine/tidy.go` `countLandings`).
   - Worktree-Teardown runs after Land, so at Land Done the member's pair still exists.
     "No open member and no member pair" therefore reads false, and the notice never fires on the last landing.
   - A member that leaves through `lyx batten cancel --remove` is not a landing either, so it never triggers the check.
   - Fix: re-check on the member run reaching done (after teardown) or on any probe tick while the episode is unarmed, rather than at the landing count.

3. **MEDIUM — D8 Beam-Up "through D1's steps" against D6 "from the dobby start on": the two disagree about Beam-Up.**
   - D1's steps include `hubreconcile.Ensure`, a hub-scope origin reconcile, the prime claim and owner-pausing every owned_elsewhere run.
   - Run from inside a pawl child driver, that is a hub-wide reconcile that can move branches and pause other runs, all from one run's producer.
   - The orch doc says `pauseOwned` is installed only by the root pre-run, and a nil one is skipped, so whether the pause happens depends on the process.
   - D6 already skips the reconcile for adopt. Beam-Up should say the same ("from the dobby start on"), or state the bound of a reconcile running inside a run.

4. **MEDIUM — D10 `lyx beam create` under a hub pause, or a first pass that dies, leaves a beam no one starts.**
   - Create records the owner, flag and `recipe: pawl`, then spawns a detached first pass.
     Per the dobby doc, a first pass refused for a covering pause writes no status.
   - D11 says dobby starts tasks only, so nothing starts the pawl run at the lift.
   - The `--anchor` value rides only in that first pass's record, so it is lost unless the operator repeats it.
   - The record states no bound. It should say that the envelope names the pause and a re-run of `lyx beam create <slug> [--anchor …]` after the lift, or that dobby starts a flagged beam project whose status is empty.

5. **LOW — D5, "a board with no root owner reads every entry as owned by no other machine, as today".**
   - This also blanks a recorded beam owner: `Effective` returns "" whenever `o.Prime == ""`.
   - Once owners live only on projects, a board with beam owners but no `prime_owner` (a fresh hub where `lyx beam create` ran before any `EnsureLive` claim) makes every beam read as anyone's.
   - Either have create also claim root when none is recorded, or keep a recorded beam owner even when no root owner exists.

6. **LOW — D13, removing the start guard's promote refusal.**
   - A task pair carrying the beam mark, left by an interrupted promote from an older binary, would become startable as a task run.
   - `hubgeom.DobbyStart` would also list it as a beam and start a dobby there.
   - The record should state that the leftover state is assumed absent ("promote is not used" per note `beam-promote-keeps-discussion`), or keep a refusal for a task whose pair is marked.

7. **LOW — D5, the mixed-version window.**
   - A machine still on the old binary reads a run-held task with no recorded owner as owned by no other machine.
   - Once the new binary drops task owners on load, that old machine could claim or resume those tasks until it is updated.
   - The record does not state the window; one line ("both machines deploy before …") would bound it.

**Claims I attacked**

- D9, owned-elsewhere yield then respawn: **held.** InnerRun respawns an unconfirmed running child, and a spawn refused with `ChildOwnedElsewhereError` owner-pauses the run, so the loop is bounded by the existing mechanism, provided the child bootstrap's owner check reads the beam's own owner, as D11 says.
- D9, no quiet or driver-dead notice at Beam-Await, with respawn still working: **held.** Respawn does not depend on the notice.
- D9, land request seen within a minute: **held.**
- D6 example, Neve owner-pauses G's pawl run: **held**, through the same spawn refusal or a full pass.
- D6, adopting root while G is owned elsewhere resumes G's pawl run, which then owner-pauses on the new machine and lifts once G is adopted: **held.**
- D7, watcher stops the orch, including races with adopt and with a lost owner write at create: **held.**
- D10, land refusal while the batten run is owner-paused but the child is at Beam-Await: **held.** The request waits for the lift.
- D8, a seed recording `beam` resumes as pawl at Beam-Describe or Finalize: **held.** Finalize now goes on to Beam-Down.
- D5, a task whose origin record names the prime although its board project moved under a beam: **held.** It is consistent with `ParentOf`.
- D4, `lyx s` refusal and attach behaviour: **held.**
- D11 cancel: **broken** (finding 1).
- D12 trigger: **broken** (finding 2).
- D10 create under a pause: **broken** (finding 4).
- D5 with no root owner: **broken** (finding 5).
- D8 against D6, Beam-Up's steps: **inconsistent** (finding 3).

**Tool calls**

- Read `internal/battenshed/innerrun_owner.go` (head): needed to know whether a pass checks the owner and how a respawn refusal is read; the context lacked this.
- Grepped `internal/battencli/cancel.go` for beam guards: needed to know whether cancel `--remove` is guarded for beams; the context lacked this.
- Grepped `internal/dobbyengine/tidy.go` and `internal/battenshed/innerrun.go`: needed the landing-count trigger and the child respawn-confirmation rule; the context lacked both.
