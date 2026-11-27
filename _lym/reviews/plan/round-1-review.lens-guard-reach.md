```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. Every guard, refusal, branch and test row the cards add, checked for whether a real entry point can reach it.

**Per guard, reachable or not**

- **Card 2.** `LoadConfig` refusing an unknown `window_placement` or a tile size below 1: reachable through a machine-layer value. `UnknownPlacementRefused` and `ZeroTileWidthRefused` reach it.
- **Card 3.**
  - `AfterPlacement`'s early return on a blank command: reachable (`UnsetCommandRunsNothing`).
  - Config load error: reachable (`window_wait_ms: -1`).
  - Cancel ending the command and its group: reachable (`CancelEndsTheCommandAndItsGroup`).
- **Card 4.** `Open` with `separate` true on a launcher with no `separate`: not reachable through `OpenWindow`, since `PreLaunch(false)` fails first. It is a fail-loud wiring guard, which needs no reaching test, so it is no finding.
- **Card 6.** Every `PreLaunch` check: reachable through the probe rows.
- **Card 7.**
  - Script lock past 30 s, and past `within`: reachable (rows).
  - A leftover in `kwinscript.loaded` still loaded, giving `LeftoverScriptError`: reachable (`loaded-true` recording).
  - Load answering `-1`: reachable in the test through `load-taken`. In production it needs a plugin-name collision; the decision record requires the guard anyway.
  - `ErrScriptTimeout`, cancelled-context unload, and `no-output` giving `ErrNoTilingOutput`: reachable (rows and recordings).
  - `kwinquery.json` reuse for the same output after `after`: reachable.
  - Abandoned acquire releasing late: reachable. In the child it can land while `window_command` runs after a lock timeout. `TestAcquireWithin` reaches it.
- **Card 8.**
  - Absent or undecodable file read: reachable.
  - `Settle` with and without `seenOn`: reachable.
  - Undecodable-file heal (`ErrDecode`, then remove, then retry): reachable, but the heal runs outside `UpdateJSON`'s lock, so two writers can both reach it (finding 5).
  - The settled test `SettledBy == PID`: can read true right after `RecordOpen` (finding 4).
- **Card 9.**
  - `Anchor` answering false for a missing or unsettled entry: reachable.
  - Empty target past the last desktop: reachable.
  - Home gone, so the start falls to the anchor: reachable.
- **Card 10.**
  - Probe failure, and `ErrNoTilingOutput`, settling the entry: reachable (rows).
  - Lock budget spent: reachable (row).
  - Parent unsettled under the lock, and the fallback anchor past 180 s: reachable (rows).
  - Cancel before or during `afterPlace`: reachable (rows).
  - A homed window whose home the locked snapshot no longer holds: reachable, but no `TestFlow_Place` row (finding 3).
  - The hand-move rule fed by mode `off`'s record outside the lock: it fires on a state that only a stale snapshot creates (finding 1).
- **Card 11.**
  - Resolution-failure branch: reachable only through a refused `beam.window_placement` or desktop key. The `UnknownWorktree` row does not reach it, and no row reaches the refused-config inputs (finding 2).
  - Failed registration only warning: a raw I/O error, exempt.
  - Prime child not registered: reachable (`PrimeWindow`).
- **Card 12.**
  - `handTerminalWindow` with a directory in no worktree: reachable (tier1 row).
  - Hand-off failure running `placeWindow`: reachable (seam row).
- **Card 13.** Hand-off failure as a hub warning: reachable (seam row).

**Findings**

1. **BLOCKING, design.** `_lym/plan/10-kwinplace-place-flow.md`, step 3. Mode `off` runs `RecordReport` from the window-wait snapshot before `kwinplace.lock` is taken.
   - `updatePlaces`' hand-move rule says a settled window whose desktop differs from `LastDesktop` was moved by hand. Here it runs over a snapshot that another child's locked apply can supersede before the `off` child's write lands.
   - A pre-apply snapshot then shows the windows that apply just moved on their old desktops, and the rule rewrites their homes to those stale desktops.
   - The guarded write recurs outside the lock that guards it everywhere else.
   - D10 puts the query, plan, apply and record steps under the lock, and D9 lists "a record-only run under `off`" beside the post-apply write.
   - Fix: take `kwinplace.lock` and run a fresh `Query` before the `off` record.
   - Evidence: card 10 lines 721–733 in my plan dump; decision record lines 197–202 and 239.

2. **BLOCKING, design.** `_lym/plan/11-place-window-child.md`, the `UnknownWorktree` row and the "resolution that fails" branch. The row's input cannot reach the branch it claims:
   - `lymcwd.NamedWorktree` answers anchor `"."` for an unrecorded name (`internal/lymcwd/lymcwd.go:185-193`).
   - `hubgeom.TaskSettingChain` skips a missing worktree root (`internal/hubgeom/tasksetting.go:31-41`).
   - `hubgeom.ParentOf` answers `""` for a slug the board does not hold (`internal/hubgeom/parentof.go:27-32`, `internal/boardengine/parentbeam.go:18-19`).

   So none of the card's resolution calls fails for an unknown name, and the child runs the normal flow. Its "no marker" assertion would pass for another reason: `window_command` started in a directory that does not exist.

   The branch is reachable only through a refused `beam.window_placement` or a refused desktop key. No row reaches either input or asserts the way-forward warning the card prescribes.

   Fix: replace or add rows whose child meets a refused `beam.window_placement` and a refused desktop key, asserting the settled entry and the warning. Either drop `UnknownWorktree`, or have the verb check the name against `fabricengine.List` and say so.

3. **LOW, consistency.** `_lym/plan/10-kwinplace-place-flow.md`, step 5 ("A homed window whose home the fresh snapshot no longer holds needs an anchor from then on"). The state is reachable: the operator removes a desktop between the wait and the lock. No `TestFlow_Place` row reaches this skip-then-anchor return to the parent wait; card 9's `TestPlan` covers only the planner's fallback. Evidence: card 10 lines 729–736 and the row list at 758–778.

4. **LOW, design.** `_lym/plan/08-kwinplace-places.md`, `RecordOpen`. It sets the pid and keeps `SettledBy`, and `Settled` is `SettledBy == PID`.
   - A new window whose pid equals the entry's previous pid reads as settled at once: pid reuse after a reboot, which D2's rationale names.
   - A child waiting on that parent then anchors on the parent's stale entry.
   - Fix: `RecordOpen` clears `SettledBy`.
   - Evidence: card 8 lines 574–581; decision record line 192.

5. **LOW, design.** `_lym/plan/08-kwinplace-places.md`, the `ErrDecode` heal. Removing the file by its path and retrying runs outside `state.UpdateJSON`'s lock (`internal/state/state.go:157-178`). Two concurrent writers that both meet the undecodable file can each remove and rewrite it, and the second removal drops the first writer's freshly written entry. Fix: run the remove and rewrite under the places lock, through one locked step.

Tool calls:
- One Bash call read `lymcwd.anchorRelOrRoot`, the `hubgeom.TaskSettingChain` body, `hubgeom.ParentOf` and `boardengine.ParentBeam`. My context held only the first lines of these functions, not their branches for an unknown worktree name.
