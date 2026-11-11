# Darn verify gate: stop at first failed tier, rerun failures, time budget

Darn-Write's verify gate cost a darn run 35 to 40 minutes in two observed runs.
It ran every tier after a failed one, re-ran the whole scope on each attempt, and nothing told anyone it was long.
A halt after a gate that never re-prompted also read `gate did not pass after 0 attempts`.
This change fixes those four faults together, since they share the gate's code path.

Closes #92

## What changes

- **Stop at the first failed tier.**
  `lym gate test` gains `--stop-on-fail`, which ends the call at its first failed or build-blocked run.
  No later tier, vet or guard run starts, and one stderr line names the failed run and every skipped one.
  The lint and report steps still run.
  A run that crashed its test runner in its run and its rerun judged no code, so it stops nothing.
  The default stays as it was: running past a failed tier is deliberate, so a failure the change did not cause hides no later check.
  Only the `darn-write` gate key's told and tree-scoped commands add the flag; the landing call and every other key keep the whole call.
- **Rerun only the failed tests between attempts.**
  When a failed run's failure report names failed tests alone, the verify gate keeps them.
  The next attempt first runs those tests alone, one `lym gate test --wait` call per tier with a `-run` pattern.
  If they fail again, the fixer and the writer get only those findings, and the full command waits.
  If they pass, or fail naming no test, the full command runs.
  A timeout, a runner crash, a build error and a failure outside any test keep nothing, so the next attempt runs the full command.
  `gateslot.FailedTests` reads the report and `gateslot.FailedTestsCall` renders the command.
- **A time budget.**
  A new `darn.yaml` key `verify_budget_min` (template default 30, 0 turns it off) sets how long a verify gate may run from its first check.
  Once spent, the run's parent orch gets one notice naming the gate, the attempt and the failed runs so far.
  The notice only informs: the attempts go on and `verify_attempts` still bounds them.
  A timer sends it while a verify command runs, so a single long pass raises it too.
- **A truthful zero-attempt halt.**
  `GateOutcome` gains `EndedEarly`, the reason a run ended while its failing entry still had re-prompts left.
  The causes are a re-prompt that could not be sent, a pane with no input box past the bound, an entry's wait that ran out,
  and a session or deadline that ended first.
  The Stuck reason now reads `gate failed its first check and the session was never re-prompted, since <cause>` at zero attempts,
  and names the attempts made plus the cause otherwise.
  A gate that spent its budget keeps the old wording.

## Design decisions

- The stop-on-fail flag is opt-in per call, not a new default, to keep `lym gate test --upto` and the landing gate reporting every failure.
- The failed-tests state and the budget clock live in the gate closure, so a restarted driver starts both afresh.
- A spent budget raises a notice rather than stopping the attempts or handing the run over; the operator decides from the notice.
- `FailedTestsCall` runs a failed test name in every package of its tier that the call lists, so a name that failed in one package also runs in the others.
- The `lym gate test` help text no longer carries the line `-C names the run directory, with explicit targets only.`

## To check by hand

No regression check ran for this task, and none of the following was run against a live darn run:

- A Darn-Write verify gate with a failing tier 0 stops there and starts no tier 1.
- A second attempt after named test failures runs those tests alone first, then the full command once they pass.
- A gate left running past `verify_budget_min` sends one creel notice to the orch and goes on.
- A run halted with no gate re-prompt shows the new reason, not `after 0 attempts`.
