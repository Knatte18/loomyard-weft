# Gate list: several gates per producer row

A producer row now carries an ordered list of gates instead of one.
Each gate has its own attempt budget and a `pass_on_cap` flag, so a gate can be a check that must pass or a one-time request that lets the run through once its budget is spent.
This lays the shape `parent-review` builds its review gate on.
No existing gate changes behaviour: every gated row in `contracts/recipes/loom-recipe.yaml` migrates to the list form and keeps its re-prompt count, findings file, re-prompt text and Done/Stuck mapping.

This task resolves no GitHub issue, so there is no `Closes` line.

## What changes

- `shuttleengine.GateSpec` is an ordered list of entries (`Name`, `Gate` closure, `Attempts`, `PassOnCap`), carried as one value through every hop (`StartGated`, `RunGated`, `AttachGated`, `RunOpts.Gate`, `websterengine`'s `StartMaster`).
  An empty or nil list is ungated.
- The Wait loop's gated-Done branch and `finalize`'s verdict site evaluate the list.
- `GateOutcome` keeps `Passed`, `Attempts` and `FindingsPath` and gains a per-gate report: name, re-prompts sent, and state at the final arrival (passed, failed, let through, off, not reached).
- `shedrecipe` reads a `gates` list key in place of `gate`/`gate_attempts`, which are now rejected as unknown keys.
  Per element: `name` (existing vocabulary, same `Env` requirements), `attempts` (required, non-negative), optional `pass_on_cap`.
  An empty list, a duplicate name and a negative `attempts` are construction errors naming the row.
- `burlerengine`'s report-repair wrapper applies to every entry in the list.
- Docs: the recipe file's comments, the doc comments on the changed types, and a `gates` paragraph in `contracts/specs/shed-recipe-spec.md`.

## Design decisions

- **Evaluation order.** At each arrival (a gated `OutcomeDone`) entries run in list order and stop at the first failure.
  `finalize` reads the memoised verdict when Wait left one and otherwise evaluates afresh, but never re-prompts or changes a count.
- **Counting.** Each entry counts its own consecutive failures; a pass resets that entry's count.
- **Cap.** Without `PassOnCap`, a capped entry's failure ends the loop and the run finalizes Done with a failed outcome, as today.
  With `PassOnCap`, a capped entry is let through and never runs again on this run, so a later entry's re-prompt cannot re-fire it.
- **`attempts: 0` is off.** The old "0 means default 3" rule and `defaultGateAttempts` are gone, so `attempts` is required in the recipe.
  An off entry still resolves its closure and `Env` at construction, so a typo fails loudly.
  A list of only off entries is still a gated run with a passing `Result.Gate`.
- **Aggregates.** `Passed` is true only when every non-`PassOnCap`, non-off entry passed at the final arrival; `Attempts` is the sum of re-prompts sent.
  Consumers (`shedadapters`, `burlerengine`, shed status history) are untouched.
- **One findings file.** `gate-findings.md` is overwritten by whichever entry failed; re-prompt text is unchanged.
  The re-prompt Warn log gains the failing entry's name.

## Limits to know

- Counts are in memory per shuttle run.
  After an attach a `PassOnCap` entry starts from zero and can fire again; a request that must not repeat makes its own closure idempotent.
- Two non-cap gates that alternate failures reset each other and can re-prompt until `spec.Timeout`.
  No current row has two gates.
- `rework-generations` also edits `loom-recipe.yaml` (`PR-Rework` rows carry `gate: rework-plan`); whichever lands second must keep its row changes in the `gates:` form.

## To check

- The integration suite passed at the final head; nothing was left unrun.
- By hand: confirm `loom-recipe.yaml` has no `gate:` or `gate_attempts:` key, and that a real loom run through a gated row (for example `Discussion-Write`) re-prompts and maps to Stuck exactly as before.
