```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision run end to end on one concrete case, with the data each step actually receives.

**Decisions walked**
- D1, start of beam G from the prime with no root owner (reconcile, root claim, owner and hold refusals, then the beam steps): purpose met. The "every `EnsureLive` caller" list does not match the callers D8 and D9 add (F2).
- D2, recorded session whose transcript is gone (resume, fall back to the note, envelope names the session): purpose met.
- D3, `lyx beam start G` in a member pair, then on G at Parent-Align: purpose met. The `ParentOf` naming and the hold refusal come from the producer half.
- D4, `lyx beam stop G` while the pawl run waits at Beam-Await (dobby ended by its `RoleDobby` record, driver spared): purpose met. One acceptance case cannot arise (F3).
- D5, task in G with a leftover `owner_machine`, effective owner read through `ParentOf` to G: purpose met.
- D6, root then G and G then root on machine B, everything first on A: purpose met.
  - Each order ends with G's pawl run resumed on B by the "owns both" rule.
  - `liftOwnerPauses` lifts only pauses scoped to an adopted slug (`internal/adoptcli/adopt.go:460`). The pause record is one hub file per machine (`internal/pauserecord/doc.go:1`), so no break was built.
- D6, B adopts G while A's pawl run is at Parent-Align: purpose met.
  - B writes G's owner, skips the beam steps and names `lyx adopt root`.
  - A owner-pauses the run, and `lyx adopt root` then resumes it on B.
- D7, A's watcher after `lyx adopt G` on B: purpose met.
- D8, rows run in order on a fresh beam: purpose met.
  - Worktree-Create forks from `ParentBeam(G)`. That skips G itself and walks from G's project (`internal/boardengine/parentbeam.go:21-37`), so a top-level G forks from the prime.
- D9, Beam-Await accept that fails on an open member: met in the uncrashed case.
  - It stops the dobby, writes Parent-Align, re-checks and fails, writes Beam-Await, restarts the dobby and keeps waiting.
  - It breaks on a crash during the accept (F1).
- D9, crash between the request's removal and Done: purpose met. The interrupted-landing notice names `lyx beam start` and `lyx beam land`.
- D9, slot rule: purpose met. Dobby's `TaskView` carries the board `Status` (`internal/dobbyengine/starts.go:24`), so the producer half can be read where slots are counted. `RunView.Producer` alone is batten's row (`internal/dobbyengine/passes.go:29-31`).
- D10, `lyx beam create G` then a cancel without `--remove`, then create again: purpose met.
  - The cancel writes draft (`internal/battencli/cancelcmd.go:461`).
  - The guard's draft check is task-only, and a marked beam pair passes the left-parts check (`internal/battencli/startguard.go:114-116`).
- D11, start guard on a beam project versus a grouping project: purpose met.
- D12, last member torn down, parent owned elsewhere: purpose met.
- D13, promote removal: purpose met (nothing to run).

**Findings**

F1
- Severity: MEDIUM. Class: design.
- Location: decision-record.md D9, accept steps (`_lyx/discussion/decision-record.md:141`, `:144`, `:145`, `:148`).
- Summary: a crash after the accept stopped the beam's dobby leaves the dobby down for good, because the re-entered accept did not stop it itself.
- Counterexample:
  - The driver dies after "first stops the beam's dobby" but before the request is removed, either before or after the Parent-Align write.
  - The re-entered Beam-Await finds the request standing, so it accepts again. It finds the dobby already stopped, so this accept stopped nothing.
  - The re-check fails on a member opened meanwhile. Line 144 restarts the dobby "only when this accept stopped it", so it is not restarted.
  - The failure goes to the orch, but no way forward is named, and the members stay undriven.
  - Only the no-request case (line 148) names `lyx beam start`.
- Purpose missed: "keeps waiting" while the beam's dobby runs its members.
- Fix: let the accept's failure path restart a dobby that is not live while no `lyx beam stop` stands, or have the failure post name `lyx beam start <slug>`.

F2
- Severity: LOW. Class: consistency.
- Location: decision-record.md D1 (`_lyx/discussion/decision-record.md:37`), against D8 Beam-Up (`:126`) and D9 (`:144`).
- Summary: D1's closed list of `EnsureLive` callers leaves out two starters the record itself adds:
  - Beam-Up, which runs "the beam steps (D1)", the dobby start among them;
  - Beam-Await's failure path, which restarts the beam's dobby.
- Effect: a planner reading D1's "Every `EnsureLive` caller" sentence as the full set gets a list that contradicts D8 and D9.

F3
- Severity: LOW. Class: consistency.
- Location: acceptance criteria, line 251 (`_lyx/discussion/decision-record.md:251`), against D4 (`:68`).
- Summary: the criterion "a driver re-spawned after `lyx beam stop` posts none" tests a case the record never produces.
  - D4 ends the dobby by its role alone and spares the pawl run's driver.
  - The driver is re-spawned only by a restarted batten (`:151`), never by a `lyx beam stop`.
- Fix: restate it as "a Beam-Await re-entered with its status at Beam-Await and no request standing posts no notice".

**Tool calls**
1. `boardengine.ParentBeam` and `dobbyengine.TaskView`: needed to know which parent Worktree-Create gets for a beam project, and whether dobby can read the board status. The manifest had only their call sites.
2. `adoptcli/adopt.go`'s owner-pause lift and the `OwnerCaller` writers: needed to walk the two D6 adopt orders through the pause lift.
3. `pauserecord/doc.go` and `liftOwnerPauses`: needed to tell whether an owner pause written on one machine is seen by another.
4. `battencli/cancelcmd.go` around `clearRunClaim` and the `StatusDraft` sites: needed for the D10/D11 path of cancel, then create again.
5. `battencli/startguard.go`'s draft probe and `refuse` order: needed to confirm that path is not refused by the draft or left-parts checks.
