```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guarantee and code claim in `decision-record.md` and `support-log.md`. One claim broke, one held only in part, and the rest held.

**Claims attacked**
- Goal: "the checklist is gone until a later `lym loom start` adds it back". **Held** (`internal/orchcli/runs.go:88-91`).
- D1: "`internal/loomcli` imports `internal/orchcli`". **Held** (`internal/loomcli/bootstrap.go:13`).
- D1: "`reedengine.StrandStatus` carries neither command nor display". **Held** (`internal/reedengine/lifecycle.go:72-100`).
- D1: "loomcli's recipe defaults to tapestry before a seed exists". **Held** (`internal/loomcli/sharedbootstrap.go:206-208`).
- D1: "a strand kept by its sidecar is never re-specced". **Held** (`internal/loomcli/bootstrap.go:226-236`).
- D2 case 1: "`StrandStatus` does not tell own-window from overview apart". **Held**: `OwnWindow` is true for both (`internal/reedengine/lifecycle.go`, the `OwnWindow` comment).
- D2 case 2: "a reed restart leaves both strands dead, and `lym beam start` otherwise brings back no checklist". **Held**: `Up` never launches strands (`internal/reedengine/lifecycle.go:743-746`).
- D2 failures: a replace can fail before or after the old strand is removed. **Held**: validation runs before `removeStrandLocked`, and `addStrandLocked` runs after it (`internal/reedengine/replace.go:45-85`).
- D2 race: "a concurrent `lym loom start` that put `loom-status` back in the overview window makes the runs add fail". **Held**: `AddStrand` refuses a second overview strand (`internal/reedengine/doc.go:432`). The re-entry window is the gap between the replace and the sidecar write; the record states it.
- D2 bound: "acts only on `loom-status` and the runs strand; never fails `start`". **Held** (`internal/orchcli/start.go:483-485`).
- D2: "the viewer ends on the beam overview window, as after today's replace". **BROKEN**, see finding 1.
- D3: "reed names an own window by its role". **Held** for the break-out (`internal/reedengine/ownwindow.go:405`, which uses `strandLabel`, the role). New windows take the `name` argument (`internal/reedengine/spawn.go:262`).
- D3: "`Display.Title` is read only for an overview strand". **Held** (`internal/reedengine/render/types.go:66-68`, `internal/reedengine/spawn.go:281-282`).
- D3 reserved names: the only names reed relies on are `shell` and `overview`. **Held**:
  - The shell window is found by its console pane and renamed to `shell` (`internal/reedengine/windoworder.go:204`).
  - `OpenWindow` skips `@lym_own` windows in its same-name lookup (`internal/reedengine/doc.go:390`).
  - No recipe in `shedrun.RecipeNames` equals either name (`internal/shedrun/seed.go:89-90`).
- D3: two strands sharing one `Title`. **Held, but the record never says so**:
  - tmux allows duplicate window names.
  - Reed targets windows by id, or by a per-call temporary name (`internal/reedengine/doc.go:379-385`, `internal/reedengine/window.go:109`).
- D3: "every path that names such a window: the new window and a break-out". **Held**: a relaunch and `Resume` both open through `openOwnWindowLocked` (`internal/reedengine/spawn.go:255-262`). The only `rename-window` calls are the shell repair and the detached-window temporary (`internal/reedengine/windoworder.go:204`, `internal/reedengine/window.go:109`).
- D4: "reads seed and status without locks, like the frame's other lines". **Held** (`internal/battencli/overview.go:446-449`).
- Constraint: the sidecar stays at the `loomengine.LoomScratchDir` path. **Held** (`internal/loomcli/statusbuild.go:12-16`).
- Scope Out: "an older build left no checklist strand; the next `lym loom start` restores it". **Held**: with a runs strand present, `statusStrandAddSpec` returns the beside spec (`internal/loomcli/bootstrap.go:31-43`).

**Findings**

1. BLOCKING · design · `_lym/discussion/decision-record.md` D2 ("The viewer ends on the beam overview window, as after today's replace") and the matching acceptance criterion.
   The claim breaks under reed's own viewer rules, and D2 gives no mechanism that would select the overview.
   Counterexample: a viewer on the `loom-status` overview window, with `ensureRunsStrand` running case 1.
   - `ReplaceStrand` with a non-overview spec calls `killStrandPanes(..., selectOnEnd=true)`. The viewer moves to `relevantWindow`, which is the newest live agent window, else the overview, else none and tmux decides. No overview exists at that moment, because the runs strand is not added yet. Evidence: `internal/reedengine/replace.go:70-71`, `internal/reedengine/strand.go:835-856`, `internal/reedengine/ownwindow.go:456-470`.
   - The new `loom-status` window carries `KeepViewer`, so it is never selected (`internal/reedengine/ownwindow.go:479-480`).
   - The runs strand's `AddStrand` opens an overview window, which "is never selected here" (`internal/reedengine/spawn.go:256`).
   - Today's replace selects the new window only because `endedCurrent && replacingWithOverview` holds (`internal/reedengine/replace.go:97-98`), and D2 removes that path.
   So the viewer lands on an agent window or wherever tmux falls back, not on the beam overview. The criterion "After the move the viewer is on the beam overview window" cannot pass without a reed select step that no Decision names.

2. LOW · design · `_lym/discussion/decision-record.md` D3, on two strands sharing one `Title`.
   The claim holds, but D3 depends on these tmux and reed facts without stating them:
   - tmux allows duplicate window names.
   - Reed addresses windows by id, or by a unique per-call temporary name.
   - The `OpenWindow` same-name lookup skips `@lym_own` windows.
   - The shell window is found by its console pane, not by its name.

   Evidence: `internal/reedengine/doc.go:372-390`, `internal/reedengine/window.go:109`, `internal/reedengine/windoworder.go:204`. Without these stated, Plan has to re-derive them; the fix is to record them with this evidence in D3.

**Tool calls**
- Searched reedengine for `rename-window`, `automatic-rename`, `Up` and `Resume`, and orchcli for `reedUp`. The context lacked any window-rename path that could undo a `Title` name, and lacked whether `lym beam start` relaunches dead strands.
- Read `resolveSeedRecipe`, `addStrandLocked`'s head and the second-overview refusal. The context lacked the source of D1's tapestry-default claim, and where the D2 race's add refusal comes from.
- The first call's greps for `func (c *orchCLI) reedUp` and `func resolveSeedRecipe` in `internal/loomcli/arm.go` returned nothing; the second call found `resolveSeedRecipe` in `internal/loomcli/sharedbootstrap.go`.
