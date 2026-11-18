```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: I tested the record's claims, bounds and ordering promises against concrete interleavings and leftover states.

**Findings**

1. MEDIUM — D3, Auto-mode assumptions: "halted" and "finished" are not shed states, so the line's show/hide rule is not pinned.
   - Evidence: `internal/shedengine/status.go` defines `running`, `paused`, `done`, `blocked`, `failed` and `awaiting`. "Halted" is none of them.
   - The record says "finished is one whose status state the shed engine counts as ended". Only `done` is ended, and a cancelled or abandoned pawl run is left unclassified.
   - Counterexample: a pawl run whose status stays `paused` or `failed` and is never resumed keeps its overview line forever, naming a checklist window that may be gone.
   - Fix: name the states. Hide on `done` and on a missing status; show the others, with the state. Say what a cancelled self run leaves behind. I did not trace that: batten cancel removes task run records, and I did not see it reach the self run.

2. LOW — D3 and D2: the window name in the overview line can disagree with the real window.
   - D2 falls back to the role name `loom-status` when the seed is unreadable at add time. D3 computes the name independently from the seed at render time.
   - Counterexample: the seed is unreadable when `ensureRunsStrand` re-adds the strand, so the window is named `loom-status`. Later the seed reads fine and the overview prints the recipe name.
   - There is a second case: in the overview-first order the strand sits in the overview window (named for the runs title after the move), not in a recipe-named window. The line must name the window the strand actually holds.
   - Fix: D3 states that it names the window by the same rule as D2, and accepts the mismatch window, or reads the window from reed.

3. LOW — Goal ("always shows"): a pre-existing race between a hand `lym loom start`/resume and `lym beam start` is not bounded.
   - The two take different locks: loom's bootstrap lock and orch's `start.lock`.
   - Counterexample: the operator closed the checklist, so neither strand exists. Loom's `ensureStatusStrand` reads the strands and sees no runs strand. `beamUp` then adds the runs strand in the overview window. Loom's `AddStrand` with the overview anchor is refused by reed's one-overview rule, `ensureStatusStrand` returns the error, and `lym loom start` fails.
   - D1 does not cause this race, but the Goal's "always" overclaims. Fix: name it in Open risks, or soften the Goal.

4. LOW — Open risks: the binary-mismatch risk is mis-stated, and it is not a risk.
   - D1 writes the sidecar with `lym beam start`'s build, and `resolveStatusStrandAction` keeps a live strand whose sidecar GUID and build match the starting binary.
   - If the loom start binary differs from the one `lym beam start` ran, the next loom start replaces the strand. With the replace spec beside the runs strand, it stays in its own window. That is benign either way.
   - The entry reads as a design note rather than a risk. Drop it or move it to Constraints.

**Claims tested**

- D1, "a live or dead status strand is matched": held. The loop in `ensureRunsStrand` ignores `Live`.
- D1, "every failure is a warning and never undoes the runs strand": held. The re-add comes after the replace, and the sidecar write is separate.
- D1, "only the loom status strand, right after it took that strand's window": held as a bound.
- D1, the sidecar path is right for `BeamUpWorktree`: held. `bringUpLocation` resolves the beam worktree's own location, so `LoomScratchDir(location)` points at the beam.
- D1, "the next loom start keeps the strand": held when the binary matches. When it differs, it is replaced beside the runs strand (finding 4).
- D1 and D2, "covers both orders": held for the pawl flow, where loom start completes before the pawl row runs `BeamUp`. The hand-run race in finding 3 is still open.
- D2, the seed is readable at the bootstrap's spec build: held. `start.go` seeds before `ensureStatusStrand`.
- Constraint, "reed refuses a `Window` group on overview": held. `validateDisplay` in `internal/reedengine/strand.go` refuses it.
- Constraint, "`ReplaceStrand` takes any anchor": held. It removes and adds. `refuseWindowMove` applies only to updates, not to replace.
- D3, "an unreadable seed or status is never an error": held as a stated bound.
- D3, show/hide by state: broken (finding 1).
- D3, the window name matches D2: broken in an edge case (finding 2).
- Goal, "always shows": broken by the hand-run race (finding 3).
- Support log, "`ReplaceStrand`'s slot keeping, which the current code relies on": not verified. It is only in the rejected alternatives, so no finding.

**Tool calls**

- Read shed states, `validateDisplay` and the `orchCLI` fields: the context lacked the run state names, reed's group validation and orch's location field.
- Read `BeamUpWorktree` and `bringUpLocation`, and the seed order in loom start: the context lacked whether orch's location is the beam worktree and whether the seed exists before the status strand spec is built.
- Grepped batten cancel's handling of status files: the context lacked what cancel leaves of a run's status.
