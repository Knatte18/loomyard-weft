```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end on one concrete case.

**Decisions walked**

- **D1**, `lyx beam start G` in beam G on its owning machine. The steps (reconcile, claim, owner-pause, dobby, window, orch) chain correctly. **Met.**
- **D2**, `lyx beam stop` then `lyx beam start`: it resumes the recorded id, and refresh gives a fresh session. **Met.**
- **D3**, `lyx beam start` in task pair t1 under G: the refusal names G. **Met.** Its "before it writes anything" disagrees with D1's order (finding 6).
- **D4**, `lyx s` in G: `beam start`, then `reed attach` boots and attaches this worktree's session (`internal/reedcli/attach.go:36-45`). **Met.** For `lyx beam stop G`, a dobby end by role is composable; the purpose is met.
- **D5**, task t1 under beam G when the board records no `prime_owner`. **Not met** (finding 5).
- **D6**, the Neve/OSL-1046 example (G adopted, then root adopted, H stays on Neve):
  - Neve's pass owner-pauses G's pawl run.
  - `lyx beam land G` on OSL-1046 refuses naming adopt root.
  - Adopt root resumes G's pawl run on OSL-1046, which re-enters Beam-Await.
  - H's pawl run is owner-paused by root's dobby on OSL-1046.
  
  **Met.**
- **D7**, `lyx adopt G` on OSL-1046, after which Neve's watcher reads G from Neve's local Bolt. This is bounded by "as fresh as local Bolt". **Met within its bound.**
- **D8**, `lyx beam create G`:
  - Beam-Up marks the pair and runs the beam steps.
  - Beam-Await waits, then Parent-Align, the perch, Describe, Finalize, Beam-Down.
  - Land and teardown follow.
  
  **Met**, apart from findings 1–3.
- **D9**, `lyx beam land G` at Beam-Await, with the re-check, snapshot and dobby stop. **Not met** (findings 1, 3 and 7).
- **D10**, `lyx beam create G` from the prime:
  - The setter writes owner, beam flag and `recipe: pawl`.
  - Batten's claim sets `running · Worktree-Create`.
  - The prime's dobby picks the run up from the prime's batten status files (`internal/dobbyengine/doc.go:37`).
  
  The create itself is **met**; G's life afterwards is **not met** (findings 1 and 2).
- **D11**, `lyx batten cancel G` with no members: the pair's strands are stopped (`internal/pairteardown/cancel.go:214`). **Met.**
- **D12**, the last member t1 lands, its batten run goes done after teardown, and G's dobby checks `BeamLandRefusal` with the orch check dropped (D10). **Met** for run-ending members; **not met** for a member that leaves without a run (finding 8).
- **D13**: promote is removed. **Met.**

**Findings**

1. **BLOCKING · design · D9 last bullet.** For the beam's whole life, no task can join the beam project.
   - From the first pass (claim `running · Worktree-Create`, `internal/battencli/owner.go:195`) onward, the driver writes `<state> · <producer>` to G's board entry (`internal/loomcli/wiring.go:200-216`).
   - G is therefore run-held for weeks.
   - `checkProjectField` refuses any task whose `project` names a run-held project with "is held by a run" (`internal/boardengine/project.go:102`).
   - Case: after `lyx beam create G`, `lyx board upsert {"slug":"t1","project":"G"}` is refused, so no new member can be added.
   - D9 lists the readers the Beam-Await exemption reaches (fork, verbs, start, Project layer, run lock) as a closed list. It states no rule over every run-status reader of a beam entry, and membership is not among them.

2. **BLOCKING · design · D10 / D11 (the parent's dobby drives the pawl run).** The pawl run fills one of its parent dobby's `max_runs` slots for the beam's whole life.
   - Dobby starts a task only "while fewer than `max_runs` runs it drives are in flight: halted runs count" (`internal/dobbyengine/doc.go:58`). The default `max_runs` is 3 (`internal/dobbyengine/template.yaml:1`).
   - Case: root with beams G, H and K. The prime's dobby drives three pawl runs parked at Beam-Await and starts no root task until a beam lands. A nested beam does the same to its parent beam's dobby.
   - No decision exempts a run at Beam-Await from the slot count or bounds this.

3. **MEDIUM · design · D9 "On an accepted request".** A member can fork from the beam after the land checks pass.
   - Order: re-run the checks, write the snapshot, remove the request, stop the dobby, Done.
   - Between the re-check and the dobby stop or the board status leaving `· Beam-Await`, the still-live beam dobby can start a just-released member t2, or a hand `lyx batten run t2` can.
   - t2's Worktree-Create reads `running · Beam-Await` and forks from G (`internal/battencli/beamfork.go:62`).
   - Nothing after Beam-Await re-checks `BeamLandRefusal`, so G goes through Finalize and teardown with t2 forked from it.
   - The dobby should be stopped before the re-check, or a later row should re-check.

4. **MEDIUM · consistency · D9 / D3 / D11, where the hold is read.** "At" and "past" Beam-Await are defined in run-row terms, but every reader judges the board entry's status.
   - The readers are `beamForkPrecondition`, `ResolveBattenPrime`/`beamLanding`, the run lock, layers and `orchcli/refusal.go` (`internal/battencli/refusal.go:100`, `internal/boardengine/runlock.go:76`).
   - The board status is written warn-only, so it can lag the run (`internal/loomcli/wiring.go:226-235`: "a failed board write only warns").
   - The rows before Beam-Await (Worktree-Create, Beam-Up), and halted states at them, are neither "at" nor "past" Beam-Await, so D3 and D9 leave their hold undefined.

5. **LOW · consistency · D5 `Owners.Effective` no-root-owner sentence.** On a board with no root owner, a beam's members do not follow the beam's owner.
   - "every other entry as owned by no other machine" makes t1 under OSL-1046-owned G read as unowned on Neve.
   - This contradicts D5's own "Their effective owner is the owner of `hubgeom.ParentOf`'s answer" and the AC's "a task's effective owner is its beam's owner".
   - Case: a hand `lyx batten run t1` on Neve passes D5's claim check before any prime claim lands.

6. **LOW · consistency · D3 vs D1.** D3 refuses "before it writes anything", but its owner refusal reads "the Bolt the reconcile just pulled".
   - D1 orders the reconcile and the prime claim, which can move branches, remove pairs and write the owner, before any refusal.
   - So the owned-elsewhere refusal follows writes.

7. **LOW · design · D9 restart path.** A crash after the request is removed or the dobby is stopped, but before Done, leaves the beam stuck with no notice.
   - The re-spawned driver re-enters Beam-Await with no request and the beam's dobby down. Members don't start, and no notice says so.
   - Recovery is an unstated second `lyx beam land` or `lyx beam start`.

8. **LOW · design · D12 trigger.** The "can land" check fires only when a run goes done after teardown or leaves through `cancel --remove`.
   - A member that leaves another way never re-arms the notice, though `BeamLandRefusal` now passes: moved off the project with `lyx board upsert`, or marked done by hand, with no run.

Tool calls:
1. `grep internal/dobbyengine/doc.go`: how dobby takes on runs it did not start. The manifest lacked the dobby run-driving rules.
2. `sed internal/dobbyengine/doc.go:28-57`: the pass and in-flight rules, for Beam-Await's driver lock and slot counting.
3. `grep internal/reedcli/attach.go`: `reed attach`'s no-argument behaviour, for D4's `lyx s`.
