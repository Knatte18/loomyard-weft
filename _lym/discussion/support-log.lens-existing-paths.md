```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: tracing the code paths that already touch what D1, D2 and D3 change, and saying for each path whether the decision's result survives it.

**Paths traced**

- D1 → `loomcli.ensureStatusStrand` on a later loom start or resume, through `resolveStatusStrandAction`, which matches the role and keeps the strand when the sidecar's guid and build match: survives, because D1 writes the sidecar with the shared path and the `statuspane.WriteSidecar` helper. On a build mismatch it replaces the strand with D2's beside-runs spec, since the runs strand is present: survives.
- D1 → a later `lym beam start` with the runs strand current: returns early and leaves the status strand alone. Survives.
- D1 → a later `lym beam start` with the runs strand stale: the `runs != nil` case replaces only the runs strand. Survives.
- D1 → reed `ReplaceStrand`, then `AddStrand` of the own-window spec: `validateDisplay` (`internal/reedengine/strand.go`) accepts a `Window` group on own-window. Survives. `refuseWindowMove` only guards updates, which D1 does not use.
- D1 → the pawl row's `BeamUp(location)` from the beam worktree, against `BeamUpWorktree(hub, slug)` and `lym beam start` from the prime: survives only if orchcli takes the status sidecar path and the seed from the beam's worktree, not from the location the verb was called in. See finding 2.
- D1 → no path removes the loom status strand, so a status strand left over from a finished run can meet the `loomStatus != nil` case. See finding 3.
- D2 → `statusStrandAddSpec` callers, which are `ensureStatusStrand` only, and D1: survives under one builder.
- D2/D3 → the window name D3 prints against the window reed creates: a strand kept by sidecar from an older build keeps a window named by its role, while D3 prints the recipe name. See finding 4.
- D3 → the finished-run test against the existing readers: `fabriccli.runStatusUnfinished` treats a run as finished only when its state is `StateDone`. `beamcli` `ChildStatus` and `RunID` read the same self-run status and seed and are unaffected. See finding 1.
- D3 → the prime's `lym batten overview` also reads the prime worktree's self run. That is harmless, and the record's "beam overview" wording does not exclude it.

**Findings**

1. **MEDIUM — D3, finished run (and Auto-mode assumption 3).**
   - The record leaves "finished" vague: "a status state the shed engine counts as ended".
   - The existing code already defines it: `runStatusUnfinished` in `internal/fabriccli/anchordeps.go` returns `status.State != shedengine.StateDone`.
   - The record should pin finished as `shedengine.StateDone` and drop the assumption, so Plan does not invent a second definition.
2. **MEDIUM — D1 and D2, which worktree.**
   - `ensureRunsStrand` is reached from the beam worktree (pawl's `BeamUp(location)`) and from the prime (`BeamUpWorktree(hub, slug)` and `lym beam start &lt;slug&gt;`).
   - The record does not say that the seed D2 reads and the sidecar D1 writes are the beam worktree's: `LoomScratchDir` and `shedrun.ReadSeed` of the session's own worktree.
   - If the caller's location is used, a start from the prime writes the sidecar to the prime. The next loom start in the beam then misses it and replaces the strand. That is churn, not loss, but D1's stated purpose of keeping the strand breaks.
   - Evidence: `internal/loomcli/pawlwiring.go` calls both `BeamUp(location, …)` and `BeamUpWorktree(hub, slug)`, and `ensureRunsStrand` keeps its sidecar under `c.paths.Dir`.
3. **LOW — D1, a status strand from a finished run.**
   - D1 re-adds the checklist whenever it displaces a loom status strand, live or dead, whatever the state of the run.
   - The status strand is never removed when a run ends, so a session that loses its runs strand after pawl is done gets a checklist window for a finished run.
   - D3 hides its line for such a run, so the two disagree. Gating the re-add on the self run being unfinished would align them, or the record could state that this is accepted.
4. **LOW — D2 and D3, the window name.**
   - D3 prints the window name by D2's rule.
   - A status strand added before this change, and kept by `resolveStatusStrandAction` on a sidecar match, sits in a window named `loom-status`, so the overview line names a window that does not exist until the strand is replaced.
   - This only happens in sessions that predate the change. The record should either state it as accepted or have D3 print the name of the strand's actual window.

**Tool calls**

- grep for `LoomStatusStrandName`, `statusSidecarFile` and `SelfRunID` across `internal`: to find the other readers and writers of the strand role, the sidecar and the self run, which my context did not list.
- Read `beamcli/deps.go` (the self-run readers), `fabriccli.runStatusUnfinished` and `reedengine.validateDisplay`/`refuseWindowMove`: to check the existing finished-run definition and whether reed accepts a window group on own-window, which my context did not show.
