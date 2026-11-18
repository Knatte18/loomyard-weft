```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked each bound, guarantee and code claim in the decision record and support log.

**Findings**

1. **MEDIUM — D2, "falls back to today's replace … so the beam overview still comes up".** This guarantee breaks in step 1's main failure mode.
   - Counterexample: `ReplaceStrand` in `internal/reedengine/replace.go` validates first, then calls `removeStrandLocked` and kills the old pane, and only then calls `addStrandLocked`.
   - When the add of the moved `loom-status` fails (for example a launch or new-window error), the old `loom-status` GUID is already gone from the table.
   - The fallback "replace `loom-status` by the runs strand" then targets a GUID that no longer exists and fails. The session ends with neither the checklist nor the beam overview.
   - Fix: after a failed move, re-read the strands. Replace the `loom-status` strand if it is still present (a validation-stage failure), and otherwise add the runs strand.
   - The acceptance criterion on "the move's replace fails" should cover both failure stages.

2. **MEDIUM — D2 and Goal, concurrent `lym loom start`.** The record states no ordering against a hand `lym loom start` in the beam worktree.
   - Counterexample: loom's `ensureStatusStrand` runs under loom's bootstrap lock, and `ensureRunsStrand` runs under orch's `start.lock`. These are different locks.
   - Both can call `ReplaceStrand` on the same `loom-status` GUID. Reed's op lock serializes the two calls, but the second one finds the GUID gone and fails.
   - When orch's call is the second one, its fallback also fails (finding 1), so the overview is lost.
   - Both paths also write the status sidecar, and the last writer wins. A stale GUID only costs one relaunch on the next start, so that part is LOW.
   - The fix in finding 1 covers the loss. The record should say which outcome is accepted in the race.

3. **LOW — Goal, "while a beam's pawl run lives, the session shows the checklist".** The Goal promises more than the record provides.
   - Counterexample: after a reed server restart, both strands are dead.
   - `lym beam start` takes the `runs != nil` branch, replaces only the runs strand and leaves `loom-status` dead. D2 never fires, because a runs strand exists.
   - Only `lym loom start` restores the checklist. Today that holds because only loom's `start` calls `ensureStatusStrand`.
   - Open risks names only the case of a session started on an older build. Either narrow the Goal, or name this path as well.

4. **LOW — D2, trigger "a `loom-status` strand when no runs strand exists".** This holds, but the acceptance wording overstates what the code can see.
   - `StrandStatus.OwnWindow` is true for both an overview strand and an own-window strand, and `StrandStatus` carries no display.
   - So `ensureRunsStrand` cannot tell whether `loom-status` sits in the overview window. A `loom-status` that is already in its own window (for example after the runs strand was removed) gets replaced and relaunched again. That is harmless churn.
   - The acceptance criterion's phrase "in the overview window" describes a precondition the code does not check.

5. **LOW — D2 step 1, the viewer after the move.** The record says nothing about where the viewer lands.
   - `ReplaceStrand` calls `selectStrandWindowLocked` only when the replacement is an overview strand.
   - An operator viewing the overview window when `loom-status` moves out (with `KeepViewer`) has their current window end. Reed's fallback then picks the viewer's window before the runs strand exists.
   - This is minor. The plan or tests should state the expected viewer landing.

6. **LOW — D3, window name from the recipe.** A name collision is not bounded.
   - Reed treats the window named `shell` (`shellWindowName` in `internal/reedengine/windoworder.go`) as the shell window when it checks and repairs the window order.
   - A `Title` equal to `shell` would collide with it. No recipe is named that today, so this stays theoretical. A one-line bound, refusing or ignoring a `Title` equal to a reserved window name, would close it.

**Claims attacked**

- D1, "loomcli imports orchcli": held (`bootstrap.go` imports orchcli).
- D1, "`StrandStatus` carries no command or display": held (`internal/reedengine/lifecycle.go`).
- D2, "Beam-Up row runs inside the live pawl run": held for the pawl path. `lym beam start` also reaches `ensureRunsStrand`, which feeds findings 3 and 4.
- D2, fallback guarantee: broken (finding 1).
- D2, the sidecar makes the next start keep the strand: held, except under the race in finding 2.
- D2, "never fails start": held.
- D2, "every other strand untouched": held.
- D3, "`Title` read only for an overview strand today": held (`overviewWindowTitle`; the `spawn.go` and `ownwindow.go` break-out use `strandLabel`).
- D3, validation accepts `Title` on an own-window strand: held (`render.ValidateDisplay` checks only `WindowSize`).
- D3, the name survives later renames: held. Reed renames only the shell window, and a `-n` name turns off automatic rename.
- D3, name collision: unbounded (finding 6).
- D4, "the prime holds no run of its own": not verified, but plausible, since the prime's batten runs are keyed by slug and not `SelfRunID`.
- D4, the hide rule for no seed, no status or a done run: held.
- Open risk, older-build sessions: held but incomplete (finding 3).

**Tool calls**

- Read `ReplaceStrand` in `internal/reedengine/replace.go`: my context did not have its remove-then-add order, which D2's fallback depends on.
- Read `validateDisplay` and grepped `rename-window` and `strandLabel` sites: my context lacked whether reed refuses `Title` on an own-window strand, and every place a window is named.
- Read `render.ValidateDisplay` and grepped `window_name` readers: my context lacked whether anything validates or matches on window names.
- Grepped window-name comparisons in reed's window files: my context lacked whether reed finds any window by its name (it does, for `shell`).
