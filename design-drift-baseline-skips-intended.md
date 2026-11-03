# The Drift-Baseline stencil skips the tests an intended change drifts

`drift-baseline-skips-intended` · Task · bug, drift, stencil · running · Describe

The Drift-Baseline stencil tells the agent to skip, or scope off, every test an intended change drifts, so an intended change is never compared, never accepted with a reason, and hides any regression in the same tests.

## Seen

On the `cs` hub (2026-11-03), `report-significant-digits` changes the pipe flow report to 4 significant digits on purpose; every regression case renders that report on one line among a dozen raw lines that must not change.

- `contracts/stencils/drift/drift-template-baseline.md` says: "`off` for … one that changes the model's behavior on purpose, so a comparison against the old behavior would only report the intended change", and "`partial` for a task that changes some behavior on purpose; name each test that will drift totally with `--skip <test>`".
- The agent followed it: scope `partial`, all six cases skipped.
  Its own `baseline.md` names the cost: "A later compare that shows drift in any raw line of these cases would be a regression, but the skip hides it, so the formatter's own tests and a manual look at the raw lines have to cover that."
- With every case skipped the capture wrote nothing and the run blocked (see the note on a scope that skips every test).

Drift already has the path for intended drift: the compare reports it, and the Drift-Verify agent runs `lym drift accept <test>` with a reason, which the parent rules on under scope `full`.
In the same trial, `churchill-friction` took that path for a new case and it worked.

## Expected

- The stencil keeps `full` for a task whose intended change the suite reaches, and has the agent name in `baseline.md` the tests and lines it expects to drift, so Drift-Verify accepts each with a reason and catches any other line that moves.
- `--skip` stays for a test the task makes meaningless or nondeterministic, and `off` for a task the suite cannot reach.
