```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: checking every claim the round-2 fix added or rewrote against the code at HEAD.

Claims checked:
- **Hold:**
  - The `Deps.Notice` seam exists (`internal/dobbyengine/loop.go:70-71`).
  - `tick` refills a pid-less record (`loop.go`, the `s.PID == 0` branch).
  - Run cleanup removes run directories (`internal/battencli/cleanup.go:277`).
  - `TidyState.StrandGUID` exists (`internal/dobbyengine/state.go:77-81`).
  - `location.AnchorPath()` is `WorktreePath()/AnchorRel` (`internal/lymcwd/lymcwd.go:59`), which is the same rule `hubgeom.DobbyGeometry` uses for `beamDir`.
  - `loadGate` runs no reconcile (`internal/gatecli/run.go:274-301`).
  - A tidy in flight blocks counting (`internal/dobbyengine/tidy.go`, `countLandings`).
- **Do not hold:** the two findings below.

Findings:

1. **MEDIUM.** Decision 10's new sentence and the matching `lym adopt` acceptance criterion: "`lym adopt`'s warning names `lym fabric reconcile` from the prime before `lym gate test --gate adopt`, since no reconcile ran … so Decision 9's migration has written `adopt` before the hand run reads it."
   Summary: nothing in the code I read shows that `lym fabric reconcile` runs `hubreconcile.Ensure`, which is where the gate migration would run. The phrase "no reconcile ran" is also loose, because `lym adopt` does run an origin reconcile first.
   Evidence:
   - In `internal/fabriccli`, `hubreconcile.Ensure` is called only from `clone.go:144`.
   - `hubreconcile`'s own way forward for retrying the hub config reconcile names `lym loom resume`, `lym batten run` or `lym beam start`, never `lym fabric reconcile` (`internal/hubreconcile/hubreconcile.go:54-55`).
   - `lym adopt` calls `deps.Reconcile` → `fabricengine.ReconcileOrigin` (`internal/adoptcli/adopt.go:194`, `internal/adoptcli/deps.go:65-66`), which is the origin reconcile, not the hub config walk.

   I did not read the body of the `reconcile` verb (`internal/fabriccli/fabric.go:282`). Confirm whether it reaches `hubreconcile.Ensure`; if it does not, name a verb that does.
2. **LOW.** Decision 6's rewritten rationale: "a dead dobby shows in its log within eleven minutes, the interval plus the loop's one-minute idle ceiling."
   Summary: the gap between ticks is bounded by the larger of the idle ceiling and the probe interval, not by the idle ceiling alone, so eleven minutes holds only while `notice_probe_s` is at most 60.
   Evidence: `loop` sets `wait = max(wait, d.probe, wakeFloor)` after `nextWait` caps at `idleCeiling` (`internal/dobbyengine/loop.go:374-378`, and the `loop` body). The recipe default is `notice_probe_s: 30` (`contracts/recipes/batten-recipe.yaml:76`), but the value is configurable.

Tool calls:
- `nextWait` and the `notice_probe_s` default, to check the eleven-minute bound; my context lacked `nextWait` and the config default.
- `adoptcli`'s reconcile and the `fabriccli` reconcile callers, to check the `lym fabric reconcile` claim; neither was in context.
- `lymcwd.AnchorPath`, `TidyState` and `BeamDirName` users, to check Decision 3's path rule and Decision 1's strand GUID; none was in context.
- Callers of `hubreconcile.Ensure`, `ReconcileOrigin` and `adopt.go:185-215`, to check whether adopt or `fabric reconcile` runs the gate migration; not in context.
- `hubreconcile`'s exported doc lines, to see what `Ensure` covers; not in context.
- `hubreconcile`'s way forward and a grep of the `reconcile` verb, to see which verb retries the hub config reconcile; not in context.
