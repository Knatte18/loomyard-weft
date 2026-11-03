```yaml
lens: bound-breaker
outcome: reported
```

The record's claims and bounds, attacked one by one: one is broken (the old-build bound in D5); the rest hold.

**Claims attacked**
- **D1 bound, `lym reed add` refuses the overview anchor:** held. `internal/reedcli/add.go:77-82`.
- **D1 bound, no shuttle caller sets the overview anchor:** held. `lym shuttle run --anchor` refuses overview (`internal/shuttlecli/run.go:113`). `Spec.validate` only defaults an empty anchor to own-window (`internal/shuttleengine/spec.go:209`). No recipe or seat config names an anchor.
- **D1 bound, the runs strand and the status strand are the only overview spawners:** held. `render.AnchorOverview` is set only at `internal/loomcli/bootstrap.go:32` and `internal/orchcli/runs.go:44`.
- **D1 bound, the explicit-name refusal stops a second copy:** held. Concurrent loom starts are serialised by the bootstrap lock (`start.go:538`) and beam starts by the start lock. The status strand's replace removes the old strand before its add.
- **D1 placement for a relaunch or replacement whose old window is still listed:** held. The old window's key is not higher than its own, so the new window lands next to it, or after an equal-key sibling, which matches "the order they last opened".
- **D1, `UpdateStrand` is the only way to change a live overview display's key:** held. The only other writer of a live display is `moveShellWindowStrandsLocked`, which never holds an overview strand.
- **D1 tie rule across a server restart:** held. Relaunch follows the strand table, and `ReplaceStrand` keeps the table slot (`replace.go:88`), so the windows still sit in the order they last opened.
- **D2, no overview window is an agent window:** held. `agentWindowPanes` checks `isOverview` before own-window (`ownwindow.go:439-446`).
- **D2 rationale, the lowest-index overview is the beam overview:** held. It fails only while an old-key status strand ties with the runs strand, which D5's bound already states.
- **D3, the held recipe equals the seed's recipe:** held. `armAt` calls `resolveRecipe` (`arm.go:266`) before `RunE`. Both use the tapestry default when no seed exists. `WriteSeed` validates the recipe (`internal/shedrun/seed.go:174`), so no seed can carry an empty title.
- **D4, a current runs strand is kept and a stale one replaced, as today:** held. `runs.go:82-87`.
- **D5, the next start finds a sidecar naming another build:** held. `BuildIdentity` is the binary's path, size and mtime (`internal/statuspane/build.go`), so any upgraded binary differs and `StrandCurrent` fails.
- **D5, an older lym's display reads as key zero and a rewrite drops the key:** held. `Display` is persisted verbatim as JSON.
- **D5, "an older lym process still doing reed ops refuses a second overview add of its own":** broken. See F1.

**Findings**

F1 — MEDIUM, design. Location: `decision-record.md` D5, Bound.
- **Summary:** the bound understates what a live old-build process does. The process that matters is the pawl run's own Go driver (`lym loom drive`). It calls `orchcli.BeamUp` in-process at its Beam-Up row (`internal/loomcli/pawlwiring.go:106-107`), and its BeamUp-family calls include `BeamUpWorktree` (`pawlwiring.go:123`). If that driver was started by the old build and is still live after the upgrade, it runs the old `ensureRunsStrand` (`internal/orchcli/runs.go:84-91`).
- **Case 1, runs strand recorded by the new build:** the old driver sees a sidecar that does not match its own build and calls `ReplaceStrand(runs)`. The old `ReplaceStrand` removes the runs strand and kills its pane (`replace.go:64-71`) before `addStrandLocked` refuses the overview add (`strand.go:404-409`), because the new-placed status strand holds the overview. The beam overview window is lost, not just "refused".
- **Case 2, no runs strand:** the old driver replaces the status strand with the runs strand, so the checklist disappears again.
- **Why the heal does not cover it:** D5 says a hand `lym loom start` "keeps a live driver", so the old driver survives the very start that is meant to heal the session.
- **Fix:** the bound should state both outcomes and name what restores the runs strand: `lym beam start` under the new build, which D4 makes add beside the status strand. It should also state that the old driver must end for the old placement to stop coming back.

**Tool calls**
1. `withRevivalFirst` and the watchdog's relaunch path, to see whether a long-lived process relaunches overview strands. The manifest did not cover revival.
2. Searched recipe, seat and config files for anchor settings, and found `resolveRecipe`'s call site. Needed to test the "no caller sets overview" bound and the D3 recipe claim one hop further.
3. `internal/loomcli/arm.go` around line 266 and `ReadSeed`, to confirm `start` resolves the recipe before `RunE`.
4. Recipe validation in `decodeSeed`/`WriteSeed` and the `BuildIdentity` definition, for the empty-recipe title edge and D5's sidecar claim.
5. Searched for callers of `BeamUp`/`ensureRunsStrand`, to find long-lived processes that reach `ensureRunsStrand`.
6. Searched pawlengine and loomcli for `orchcli` use, which showed the pawl driver calls `BeamUp` in-process (`internal/loomcli/pawlwiring.go`).
