# Drift-Baseline stencil compares intended drift instead of skipping it

The Drift-Baseline stencil (`contracts/stencils/drift/drift-template-baseline.md`) told the agent to scope `off` or `--skip` every test an intended change drifts.
A skipped test is never compared, so the intended change was never accepted with a reason, and a regression in the same test went unseen.
On the `cs` hub, `report-significant-digits` changed one line of a report on purpose; the agent skipped all six regression cases, which also hid the dozen raw lines in each that must not change.
With every case skipped the capture wrote nothing and the run blocked.

Drift already handles intended drift: the compare reports it, and the Drift-Verify agent runs `lym drift accept <test>` with a reason, which the parent rules on under scope `full`.
The stencil now sends the agent down that path.

## What changes in the stencil

- `full` is the scope for every task the suite reaches, a pure refactoring and a task that changes behavior on purpose alike.
- `off` stays for a task the suite cannot reach, and no longer covers a task that changes behavior on purpose.
- `partial` with `--skip <test>` is limited to tests the task makes meaningless or nondeterministic, such as a test of a removed feature or one whose output now varies between runs.
- The stencil says outright never to skip a test because the task changes its output on purpose.
- The baseline report must name each test expected to drift, the lines or values that will change and why, and state that every other line of the test must stay as it is.
  It must also name each skipped test and why it is skipped.
  Drift-Verify can then accept the intended drift with a reason and treat any other movement as a regression.

## Design decisions

- The change is stencil text only; no Go code, verb or Drift behavior changes.
- Intended drift is judged at verify time, where `lym drift accept` already exists, not excluded up front at baseline time.

## To check by hand

- Run a Drift-Baseline on a task that changes output on purpose, and confirm the agent picks `full` and names the expected drift in `baseline.md` rather than skipping.
- This was not run against a live agent here.
