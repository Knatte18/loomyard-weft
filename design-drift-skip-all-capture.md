# A partial scope that skips every test ends in a capture that wrote no file, with no cause named

`drift-skip-all-capture` · Task · bug, drift · running · Code-Burler-Review

A partial Drift scope that skips every test is accepted, and its capture then fails as `exit 0 but wrote no file`, a reason that names neither the skips nor a way out.

## Seen

On the `cs` hub (2026-11-03), the Drift-Baseline agent of `report-significant-digits` recorded `lym drift scope --scope partial` with all six regression cases as `--skip`, so `.lym/drift/skip.txt` held every case `RegressionCases.All` runs.

- The capture ran on the branch point `9017f2d`, exited 0 after 3 s with empty logs, and wrote nothing; the run blocked at Drift-Await:
  `drift: the baseline capture ended failed (exit 0 but wrote no file to …/.lym/drift/baseline), …; way forward: lym drift capture followed by lym loom resume, … or lym drift defer …`
- The tackler, reading that, could not find the cause ("Its capture log and capture-child log are both empty, so the cause is not named from what I read"), chose a plain re-launch, and the same capture failed the same way; its second attempt escalated with the cause still unnamed.
- Neither way forward can succeed: `lym drift capture` repeats the scope, and `lym drift defer` drops the regression the run was meant to have.

## Expected

- A capture that writes nothing while the scope skips tests says so: the failure reason names the scope, the count of skipped tests, and that every test the harness ran may have been skipped.
- Its way forward names the parent's `lym drift scope --scope full|off` (or a shorter `--skip` list) before `lym drift capture` and `lym loom resume`, besides `lym drift defer`.
- A tackler reading the stop learns the cause from the stop alone.
