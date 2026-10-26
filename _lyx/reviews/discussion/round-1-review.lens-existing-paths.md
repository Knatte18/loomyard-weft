```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced each state the record changes against the code that already reads or writes it.

**Paths traced, per decision**

- D3 / D9, the hold rebased on "at Beam-Await". A pawl entry now carries a run status for its whole life, so every `IsRunStatus` reader of a beam entry is affected.
  - `beamForkPrecondition` (`battencli/beamfork.go:62`): named in D9; survives as rebased.
  - `ResolveBattenPrime` / `beamLanding` (`battencli/refusal.go:57-104`): named in D9; survives.
  - Orch start's `refuseBeamStart` (`orchcli/refusal.go:37`): replaced by D3; survives.
  - `ComputeLayers` (`boardengine/layer.go:35`): named in D9; survives.
  - `checkRunLocks` / `scopeChanged` (`boardengine/runlock.go:76`): named in D9; survives.
  - `checkProjectField` (`boardengine/project.go:102`): **breaks**. It refuses every write that sets a task's `project` to a run-held project, and D9 does not mention it.
  - `Rename` (`boardengine/rename.go:44`): survives. A beam is refused for rename anyway.
  - Loom peers' `runStatusTasks` and show (`loomcli/peers.go:103,196`): misreads, minor. A beam at Beam-Await lists as an in-flight peer for weeks.
  - `settleClaim` (`pairteardown/claim.go:39,77`): survives. It is only reached at the beam's own teardown or cancel.
  - Cancel's `otherLeft` (`battencli/cancel.go:472`): survives.
- D9, the signal for "at / past Beam-Await". Board status is written by the driver's `SetBoardStatus` seam (`loomcli/wiring.go:200-235`). That write only warns on failure and is deduplicated by `lastBoardStatus`. **Breaks**: the hold readers read the board, which can lag the run, and D9 names neither the signal nor what bounds the lag.
- D10 / D11, the pawl run driven by the parent's dobby. Dobby's start step caps runs in flight at `max_runs` (`dobbyengine/starts.go:51,121`; `doc.go:58`, "halted runs count"; `template.yaml` default 3). **Breaks**: every live beam's pawl run takes one of its parent dobby's slots for the beam's whole life.
- D5, run-held rule dropped.
  - `Effective` (`boardengine/owners.go:121`): named; survives.
  - Tender's `pairOwners` remote listing (`tendercli/deps.go:177`): named; survives. It still lists run-held entries owned elsewhere, which now includes pawl entries, correctly.
  - Adopt's `namedEntries` and `addPair` tests (`adoptcli/adopt.go:254,370`): named; survive under D6.
  - `resumeOwnerClaim` (`battencli/owner.go:241-271`): **breaks**. It writes a task owner on every resume of a run-held task with no owner, which under D5 is every task, and D5's narrowed setter refuses that write.
  - `checkOwnerPayload` (`owners.go:87-104`): survives. A task's effective owner is now its parent's.
- D4, ending the dobby by role.
  - The dobby is registered under the beam slug (`beamcli/dobbydeps.go:65`), and the pawl run's loop is registered under the same slug (`battencli/arm.go:525`). D4's role-scoped end is needed, and the record says so; survives.
  - Tender's owner-pause end, `runproc.End(slug)` (`tendercli/deps.go:321`): survives. That path also ends the beam's dobby, which exits on an owner change anyway.
  - The `lyx reed down` session-end spare of `RoleDobby` (`pairteardown/skip.go:81`): survives.
- D8, recipe rename.
  - `shedrun.RecipeBeam` readers in `loomcli` (`wiring.go:827`, `landingdeps.go:78,249,264,293,303`, `arm.go:198,228`) and `shedrun.Recipes`: survive under D8's rename and seed read.
  - The `beam` config module in `configreg` (`configreg.go:123`) is dobby's `beam.yaml`, not the recipe; untouched, survives.
- D10 / D12 / D11, `BeamLandRefusal`'s orch-live blocker (`beamland.go:73-91`). Removed by D10, so D12's notice and D11's cancel check are not refused by a live orch; survives.
- D7, the watcher and the dobby's own owner exit (`dobbyengine/loop.go:208-249`): survives.
- D1, `EnsureLive` (`dobbyengine/start.go:87`). It has no single-worktree mode; D1 says it is narrowed, so this is plan work; survives.

**Findings**

1. BLOCKING · design · D9 (last bullet). `boardengine`'s membership check refuses any task whose `project` names a run-held project. With the pawl run holding the beam entry for the beam's whole life, no new member can be added to a beam project, before Beam-Await or at it. D9 makes the hold "no landing hold" only for a closed list that leaves this reader out. The rule needs stating over every board reader of a beam entry's run status, not a list. Evidence: `internal/boardengine/project.go:102`.
2. BLOCKING · design · D10/D11 (pawl run driven by the parent's dobby). Dobby counts every run it drives that is not owned elsewhere against `max_runs`, halted runs included, and the default is 3. Each beam's pawl run sits at Beam-Await for weeks and takes a parent-dobby slot the whole time, so three beams under root stop root's dobby starting any task. The record neither exempts a pawl run at Beam-Await from the cap nor bounds the effect. Evidence: `internal/dobbyengine/starts.go:51,121`, `internal/dobbyengine/doc.go:58`, `internal/dobbyengine/template.yaml:1`.
3. MEDIUM · design · D9 / D3. D9 does not name the signal that "at / past Beam-Await" reads. The existing hold readers read the board entry's `<state> · <producer>` status. The driver writes that status warn-only on failure, so it can stay `running · Beam-Await` after the row is Done. Members then still fork from a landing beam and verbs still run inside it, with no stated bound. Evidence: `internal/loomcli/wiring.go:200-235`, `internal/battencli/beamfork.go:62`, `internal/battencli/refusal.go:96-104`.
4. LOW · design · D5. The run-held rule goes "from every reader", but batten's `resumeOwnerClaim` is a writer keyed on `RunHeld` with no owner. Under D5 that is every task, so if the function is kept, every resume asks the narrowed setter for a task owner and is refused. D5 should drop it alongside "Batten's claim writes the run status alone". Evidence: `internal/battencli/owner.go:241-271`.
5. LOW · design · D9. Loom peers treats every run-held entry as an in-flight peer. A beam at Beam-Await therefore lists as a peer for weeks, and `peers show` refuses for its missing records. The record does not say whether a pawl run at Beam-Await is a peer. Evidence: `internal/loomcli/peers.go:99-108,196`.

**Tool calls**

1. `orchcli/refusal.go`, `boardengine/rename.go` and `dobbyengine/starts.go` slot-count lines: the orch beam refusal, the rename lock and the `max_runs` slot rule were not yet in context.
2. Grep of `RecipeBeam` / `"beam"` readers: needed to trace D8's rename against every existing recipe-name reader.
