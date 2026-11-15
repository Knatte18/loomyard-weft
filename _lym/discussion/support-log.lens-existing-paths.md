```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced what each decision's state meets in code that already runs: the readers of the adopt record and the pause callers, the hibernate wake, the network gate and push deferral, the status commit pathspec, and the owner and machine-name readers.

**BLOCKING — D6, D9 (break the Finalize exemption)**
- D6 step 4 and D9 end every process of a subject owned elsewhere and remove its local pair. That breaks the rule that the rest of a Finalize past its parent-side merge finishes on either machine. The record keeps that rule under Constraints (`PATTERN-owner-machine`).
- Evidence: `tendercli.PauseOwnedElsewhere` (`internal/tendercli/deps.go:228-245`) already "skips the end when the run's Finalize is past its parent-side merge".
- D6 and D9 have no such skip. A catch-up or handover on the old owner would kill and remove a landing in flight.
- Fix: carve out runs at a landing row past the parent-side merge in D6 step 4 and D9.

**BLOCKING — D7 (other writers of owner pause entries survive)**
- D7 retires the `owner` caller and drops standing entries at the upgrade, but names only the dobby's path. Two other writers keep making `OwnerCaller` entries after the drop:
  - the hibernate wake's `holdRuns` (`internal/hibernateengine/wake.go:113`), which pauses runs owned elsewhere and ends their processes;
  - `tendercli.PauseOwnedElsewhere` (`internal/tendercli/deps.go:238`).
- `beamcli/dobbydeps.go:567` still reads `OwnerCaller` entries.
- The record says nothing on what these paths become: the act rule's `hand_over` plus D9, or retirement. As written, the upgrade drop is undone at the next wake.

**BLOCKING — D3, D7 (the gate and turn interruption read the local pause record)**
- The run-scope pause intent moves into the tracked record, and the hub `pause` becomes a Bolt hold. The local pause record then keeps only hibernate entries and the interrupted sessions.
- But three things read or write the local record:
  - the network gate (`internal/tendercli/gate.go`, commit `dce431f4`) holds a paused run's own processes by its coverage;
  - fabric's push deferral (`internal/fabricengine/pendingflags.go` `pushDeferral`) reads the same coverage;
  - `tenderengine.Pause` interrupts turns after the grace and records their sessions.
- The record never says that `lym tender pause` still writes the local entry, or that the gate reads the tracked intent or the Bolt hold. If neither happens, a paused run's agents keep running and pushing.
- On another machine, a pause intent read from origin has no local entry at all, so nothing holds that run's processes there.

**MEDIUM — D7, D8 (readers of the retired adopt record)**
- D8 retires the adopt record. Its readers outside `adoptcli` are not addressed:
  - the dobby handover's `UnderAdopt` (`internal/dobbyengine/handover.go:40`), which stops the dobby handing over while an adopt here covers the beam;
  - the orch watcher's `UnderAdoptWatch` (`internal/orchcli/watch.go:88-178`), which keeps the orch up during an adopt;
  - `lym beam status`'s adopt section (`internal/beamcli/adoptsection.go`).
- Without a replacement, the watcher can remove the orch strand mid-adopt. This happens in the window after `lym s` starts and before step 1's owner push lands.
- Fix: these readers read the Bolt `adopt` hold.

**MEDIUM — D2 and D3 disagree on where pause and cancel intents live, and nothing commits them**
- D2 lists "a pause or cancel intent" among the synchronous Bolt writes. D3 puts both under the run's `_lym/shed/<slug>/` on a weft records branch.
- D3 says they are "committed and pushed by D2", but D2 pushes at shed transitions. A pause or cancel from tender or batten makes no shed transition.
- `statusCommitPathspec` (`internal/loomcli/wiring.go:109`) stages only these paths: status, reviews, loom durable, driver records, settle records and usage. The pull-request record, fixer counts and intents are not among them.
- So the record needs to say who commits and pushes an intent outside a transition, and which branch it lives on: the prime's `main-weft` batten record, or the pair's.

**MEDIUM — D11, D2 (tender resume still owes deferred pushes)**
- "With D2 no push is owed at a resume" holds only if the pending-flag deferral retires too.
- `pushDeferral` still leaves pending flags when a pause covers a detached push, for example `commit-records`' path or any push child that remains.
- `tenderengine`'s `spawnOwed` is what pays those debts. If tender resume pushes no pair, flags left by a pause stay unpushed with no reader.
- Fix: retire the pending-flag deferral with D2, or keep `spawnOwed` limited to pairs that have a flag.

**MEDIUM — D4, D5 (the trailer comparison ignores the registry)**
- D4 stops a run when a commit's `Lym-Machine` trailer "names another machine". Only `boardengine.OwnedElsewhere` is said to take the registry.
- So after `lym machine name`, origin commits carrying this machine's former name read as another machine. The machine would then stop its own runs and archive them.
- The same alias gap hits `machinename.Names`' landing count in `beamcli/dobbydeps.go` (`machineLandings`) and the `machine` field of status.json history.

**MEDIUM — D4 (`main-weft` is shared)**
- The prime's `main-weft` holds the batten records of every root run, plus other lym commits: worktree config, `lym loom decision add` on project records, and D10's tracked orch session record.
- One commit there from another machine, such as the old owner's orch-session write or a config write, makes every root run's next rejected push stop.
- The record does not say whether the stop is scoped to the run whose records changed or to the whole branch.

**LOW — D10 (no push path for the orch session record)**
- D10 moves the orch session id into a tracked `_lym/orch/` record written at each launch. That is a weft commit outside any shed transition, and neither D2 nor D10 says who commits and pushes it.
- The session id's readers stay on `orchengine.State`: `launchOrch`, `lym orch stop` and the handoff cleanup segment. The orch routing state stays local.

**LOW — D11 (teardown condition reads the retired cancel marker)**
- The dobby's `strandedEnded` (`internal/dobbyengine/ended.go`) requires "no cancel marker" and "no pause covers the run".
- Both inputs move under D3 and D7, so the widened teardown must read the tracked cancel intent and the hold. The record is silent on this, but it falls out of D1's act rule if `strandedEnded` uses it.

**Paths traced (survives or breaks)**
- D2:
  - statuscommit's synchronous push: survives.
  - `pushDeferral` and tender's `spawnOwed`: break (see the D11, D2 finding).
  - `commit-records`: survives.
- D3:
  - `statusCommitPathspec`: breaks, since the new files are not staged.
  - the cancel marker reader in `strandedEnded`: breaks, silent in the record.
- D4:
  - `pushClassified`'s replay: survives as changed.
  - the shared `main-weft` writers: break the per-run scope.
- D5:
  - `OwnedElsewhere`: survives.
  - `machinename.Names` landings and the D4 trailer: break.
- D6 and D9:
  - `PauseOwnedElsewhere`'s Finalize skip: breaks.
  - dobby `handOver`: survives.
- D7:
  - hibernate `holdRuns`: breaks.
  - `PauseOwnedElsewhere`: breaks.
  - `dobbydeps.go:567`: breaks.
  - the network gate's coverage: breaks.
  - `dobbyengine/passes.go` `Held`: survives through the conversion.
- D8:
  - the `UnderAdopt` readers in handover and the orch watcher, and `beamcli`'s adopt section: break.
  - the fix loop's `on_land` re-check: survives.
- D10:
  - `orchengine.State` readers: survive if mirrored; there is no commit path.
- D11:
  - `tearDownEnded`: survives if it reads the FSM.
  - cleanup's `ReconcileOrigin`: survives.

**Tool calls**
- grep for `adoptrecord.` and the pause caller constants outside their packages: the context had no list of their readers.
- read of the `UnderAdopt` sites, the hibernate wake's `holdRuns` and the `pendingflags` header: the context had only the subagent summaries.
- read of `tendercli/deps.go` `PauseOwnedElsewhere`, the gate's coverage lines and the pathspec function names: the context lacked the Finalize skip and the pathspec.
- read of the body of `statusCommitPathspec` and the gate's lines 40-80: the context needed what the status commit stages.
