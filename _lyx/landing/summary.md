# webster: a plan fixer clears mechanical plan conflicts in the run

A `begin-batch` refusal with `plan_conflict`, such as a card still naming a symbol or path an earlier batch renamed, no longer ends the Webster run and reroutes it to Plan-Burler-Review at once.
Master first runs the new `lyx webster fix-plan`, which spawns a small cold plan-fixer strand.
The fixer edits the cards the conflict names to match the tree, runs `lyx webster rebaseline --card NN` for each card it edited, and Webster goes on.
Only a conflict whose fix would change what a card does still ends Master stuck, and under loom still reroutes to plan review.

## What changes

- `lyx webster fix-plan` is a new role-audience verb, modelled on `fix-verify`.
  It spawns or attaches to the plan-fixer strand for the conflict record `begin-batch` left, blocks until the strand ends, and returns `{"status": "done", "verdict": "fixed"}` or `"plan_writer"`.
- A new stencil, `webster-template-plan-fix.md`, is the fixer's prompt, rendered at call time.
- Master's stencil: its plan-conflict rung runs `fix-plan` backgrounded and, on `fixed`, calls `begin-batch` again for the same batch.
  Any other verdict, a dead fixer or a refusal writes `outcome: stuck` quoting the `begin-batch` refusal, as before.
  `plan_drifted` still ends stuck with no fixer.
- `state.json` gains a plan-fixer record list, loaded absent as nil.
  Run respawn stops a leftover non-terminal plan fixer and marks it dead; `reset --to start` stops every recorded one.
- Docs: `internal/websterengine/doc.go`, `internal/webstercli/doc.go`, `contracts/specs/llm-model-spec.md` and `docs/overview.md`.

## Design decisions

- The fixer is a cold strand, not a Master fork.
  A fork inherits Master's whole context, and Master's rules forbid it editing the plan.
- It runs at the `master` role's model choice with `recovery_timeout_min` as its timeout.
  No new `webster.yaml` key is added.
- The fixer decides whether a fix is mechanical.
  Its reply file starts with `verdict: fixed` or `verdict: plan_writer`, and a reply without a valid verdict line counts as `plan_writer`.
  When `rebaseline` refuses, it runs `restore-plan` and reports `plan_writer`.
- `fix-plan` is bounded.
  It refuses, spawning nothing and naming the way forward (write `outcome: stuck`), when no conflict record exists, when an earlier plan fixer of the run got the same findings, or when the batch already had two plan fixers.
  A re-entrant call attaches to a non-terminal fixer instead of spawning another.
- The fixer's session is not in the run audit's session list, since its one legitimate write, to the plan directory, would count as a violation there.
  In its place `fix-plan` records HEAD and the uncommitted paths at spawn.
  A `fixed` verdict from a fixer that moved HEAD or dirtied a path that was clean at spawn is overruled to `plan_writer` with a warning.
- The conflict record stays on disk until the next `begin-batch`, so the loom recipe, the shed Webster adapter and the plan-review instructions are unchanged.
- A `plan_defect` conflict also goes to the fixer; its verdict, not the cause, decides.
- The cold verify-fixer's spawn and await code in `internal/websterengine` is factored into `coldfixer.go` and shared with the plan fixer.

## To check by hand

- Not run: an end-to-end Webster run where a later card names a renamed symbol, to see the fixer clear it and `begin-batch` succeed.
- Risk: a fixer that misjudges a design change as mechanical can land a card that does something other than the plan writer meant, since `begin-batch`'s re-resolution only checks that cards match the tree.
- Risk: a file the fixer writes that git ignores escapes the HEAD and clean-tree check.
