```yaml
lens: fix-way-forward
outcome: reported
```

Way-forward check: whether every refusal, halt, next step and route the fix introduced can be carried out from the state it names, and leads somewhere that makes progress.

**MEDIUM — D4 in-flight bullet "Darn-Write, or any run resumed without that goto"**
Summary: for a run resumed from Drift-Baseline without the goto, the way forward Darn-Write names keeps a baseline captured while the record was missing.
Evidence:
- Nothing refuses a missing record before Darn-Write. The drift subject only puts the record path into the agent's prompt: "the decision record at `%s`" in `subjectFields.reading`, `internal/driftengine/seats.go:62`.
- So Drift-Baseline runs its baseline agent against a file that does not exist, and the run reaches Darn-Write's refusal.
- That refusal's route (`goto --to Record-Write`, `step`, `goto --to Darn-Write`, `resume`) skips Drift-Baseline on purpose, so the baseline the agent captured with no task to read stays in place.
- For such a run no writer has committed and HEAD is still at the branch point, so `goto --to Record-Write` then `resume` would recapture the baseline correctly.

The halt reason should name both routes, chosen by whether an earlier Darn-Write spawn committed work. Otherwise the bullet should state that this run keeps its blind baseline.

**LOW — D6 "in the form the final-summary spec requires of every change description so the `description` gate takes it as any other"**
Summary: a no-parent stop report must also pass the `verify` gate, which runs before `description`, and D6 does not say it will.
Evidence:
- Darn-Write's gates are `discussion`, `verify` and `description`, in that order (`contracts/recipes/darn-recipe.yaml:97-106`).
- The verify gate "runs the hub's `darn-write` key, or the tree-scoped command … and lints the comments from the merge base on every attempt".
- A stop on an unchanged tree depends on that command passing on the base tree. D6's text and its acceptance row only cover the description gate.

Checked and holding:
- `lym loom step` drives exactly one producer (`internal/loomcli/cli.go:332`), so the D4 route is runnable.
- Darn-Write has no `on_stuck` (`contracts/recipes/darn-recipe.yaml:87-89`), so D6's Stuck halts the run and `resume` returns to Darn-Write.
- `shed.yaml`'s `driver_kind` is `CategoryNone` (`internal/sheddriver/config.go:78`), so D13's way forward to the pair's local `shed.yaml` is admissible.
- Re-running the generic Preflight is a content-free environment check (`internal/preflightshed/preflight.go:1-2`), so D10's map to Preflight is reachable.

Tool calls:
1. `internal/loomcli/cli.go:325-345` and `contracts/recipes/darn-recipe.yaml:87-110`: whether `lym loom step` runs one row and whether Darn-Write has an `on_stuck`. Neither was in context.
2. Grep for `driver_kind` categories and the Preflight constructor: whether a local `shed.yaml` admits `driver_kind`. Not in context.
3. `internal/preflightshed/preflight.go` doc lines: whether Preflight re-runs safely on a resumed run. Not in context.
4. `internal/driftengine/seats.go:46-90`: what Drift-Baseline does with a missing record subject. Not in context.
