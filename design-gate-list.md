# Gate list: several gates per row, each with its own budget and pass_on_cap

`gate-list` · Planned · feature · done

## Goal

A producer row carries a list of gates instead of one, each with its own attempt budget and a `pass_on_cap` flag, so a gate can be a check that must pass or a one-time request that lets the run through afterwards.
Split out of `parent-review` (operator, 2026-10-02), which builds its review gate on this; this task changes no gate's behaviour.

## Shape

- A gate is Go code run before the producer may finish (`internal/shuttleengine/gate.go`, `internal/shedrecipe/entries_gate.go`); it returns pass or fail, and what it checks is up to the program it calls.
  It is mechanical in the sense its name means: the LLM inside the producer cannot decide it is done; the gate decides.
- `GateSpec` becomes a list; each gate has its own `attempts` and `pass_on_cap`.
  With `pass_on_cap`, a gate that has used its attempts lets the run through on its next arrival instead of failing into Stuck; `attempts: 0` turns a gate off.
- Ordering, kept simple: at each arrival the gates run in list order and stop at the first that fails, so a later gate runs only when every earlier one passed on that arrival.
- Each gate counts its own attempts as consecutive failures, and a pass resets its count to zero.
  So a validator listed before a review re-runs at every arrival, including the one after the writer addressed the review, and its budget starts afresh.
- The rows gated today migrate to the list form (`gate:`/`gate_attempts:` in `contracts/recipes/loom-recipe.yaml`); every existing gate behaves exactly as before, which is this task's test.
- `GateSpec` is carried as one value through every hop (`RunGated`, `AttachGated`, a producer's `RunOpts.Gate`), so the list shape travels the same way; `GateOutcome` reports per gate.
- Out of scope: parallel gates, dependencies between gates, and any new gate (the review gate is `parent-review`).
