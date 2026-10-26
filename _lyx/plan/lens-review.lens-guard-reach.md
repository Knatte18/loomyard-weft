```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: tracing each guard, refusal and test row the cards add to a real entry point, and checking whether the guarded site recurs unguarded.

**Findings**

- **MEDIUM — Card 18, D9 member hold read at Worktree-Create.** The hold read only runs right after `Topology.Add`, so a resumed Worktree-Create whose pair is already complete never reaches it.
  - In `internal/battencli#battenCLI.wire`, `CreateWorktree` returns early when `taskWorktreeComplete` reports complete (`if complete { … return reconcileTaskPair(location, slug) }`, wire.go ~883).
  - That return comes before both `beamForkParent`, with its precondition, and the new post-`Add` read.
  - So a member that crashed after `Add` and before the read resumes past Worktree-Create, even though its beam now holds.
  - Fix: run the hold read, `memberHoldRefusal` plus pair removal, on the complete-pair branch too for a member of a beam.

- **MEDIUM — Card 15, D9 failed re-check path.** A failed write of the Beam-Await status after a failed re-check is left unhandled.
  - D9 says the writes that cross the hold are retried at each wake until one succeeds. Card 15 only retries the entry write and the Parent-Align write.
  - If the Beam-Await write fails, the board still reads `· Parent-Align`. The dobby restart then goes through `EnsureLive`, whose card 7 `LandingHeld` refuses a held beam.
  - The marker stays and the request is removed. The posted way forward, `lyx beam start <slug>`, is also refused by `BeamUp`'s hold refusal (card 10).
  - The hold stands with no request and the dobby stopped, until some later write succeeds.
  - Fix: retry this write at each wake as well, and post `lyx beam start` only once it has succeeded.

- **LOW — Card 15, interrupted-landing entry branch.** This branch fires only on a running-to-running re-entry.
  - `Shed.Step` writes no status before `Call` when the status reads running. It does call the resume write `persistMarked` → `CommitStatus` when the run was halted, paused or blocked (internal/shedengine/run.go:286-297, 907).
  - The loom commit-status seam then writes `running · Beam-Await` onto the board entry (`internal/loomcli#newCommitStatusSeam`).
  - So a run halted or paused between the request's removal and Done re-enters with the board already at Beam-Await. The one interrupted-landing notice D9 promises is never posted.
  - The hold is lifted either way. The card should state this bound, or make the notice independent of the board producer, for example a mark written beside the request.

- **LOW — Card 11, D3 refusals.** The card does not say that `startCmd` and `stopCmd` bypass `beamcli#runVerb`.
  - `runVerb` calls `battencli.ResolveBattenPrime` first. That refuses inside a task pair with "re-run it from <beam>", with no path.
  - It also refuses inside a beam whose landing hold stands with "re-run this verb from <parent>" (`internal/battencli#beamLanding`, as card 5 edits it), not with `lyx batten status <slug>`.
  - Routed through `runVerb`, `resolveBeamTarget`'s task-pair refusal and `BeamUp`'s hold refusal would be unreachable from inside those worktrees.
  - The card should state that these verbs resolve their location themselves.

- **LOW — Card 7, D1 "none of them starts a dobby for a held beam".** The guarded site still recurs unguarded in the anchor move.
  - The beam branch of `internal/fabriccli#startDobby` starts a beam's dobby through the `StartDobby` seam (`hubAnchorDeps`: `dobbyengine.SpawnDetached(geometry)`, anchordeps.go ~255). It does not go through `EnsureLive`, so it skips `LandingHeld`.
  - A beam whose pawl run is halted at Finalize holds the hold with no live run, so its anchor move is admitted and restarts the dobby.
  - Card 7 only narrows `EnsureDobbies`. The beam branch should go through `EnsureLive` naming the beam, or read the hold itself.

The other guards trace to a real entry point and are not repeated elsewhere on an unguarded path.

**Further calls**

- `grep CommitStatus` in internal/shedengine: my context lacked when shed persists status around a producer call.
- `sed -n 880,910p` of internal/shedengine/run.go plus a grep: same reason, to read the persist path that calls `CommitStatus`.
- `grep` for the transition and entered-row writes in run.go: same reason, to find the writes made before a call.
- `grep` for `CommitStatus` and `EnteredRow` in run.go: same reason.
- `sed -n 255,310p` of internal/shedengine/run.go: my context lacked whether a resumed running row gets a status write before `Call`.
