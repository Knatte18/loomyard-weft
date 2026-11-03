# Drift-Baseline states expected drift, never acceptance policy

The Drift-Baseline stencil told the agent to name the drift it expects "so the agent that judges the compare later accepts the intended drift with a reason".
It had no rule for a task whose stated invariant ("every result stays unchanged") and specified design disagree,
so a baseline agent could resolve the contradiction in the design's favour and write acceptance policy: a reason to accept the drift and a tolerance under which it passes.
Drift-Verify judges against that report, so the report steered it to accept drift the task said must not happen.

## What changes

`contracts/stencils/drift/drift-template-baseline.md`:

- Step 1: a stated output invariant (results unchanged, behavior-preserving, pure refactoring) wins over any part of the design that would move output.
  The agent expects no drift from that design and names the contradiction as a risk instead of resolving it.
- Step 2: the `full` scope bullet says the verify agent judges each drifted test against the task and the expectation, instead of saying it accepts the drift the baseline expects.
- Step 4: the report states what the agent expects and why, never what will be accepted: no acceptance reason and no tolerance under which drift passes.
  Judging and accepting drift belong to Drift-Verify and the run's parent.
  Under a stated invariant the report says no drift is expected and names each contradiction as a risk.

`contracts/stencils/drift/drift-template-verify.md`, Step 2: the baseline report is expectation, never permission to accept.
A task that states its output stays unchanged intends no drift, so any drift in it is a defect, even where the baseline report names the contradicting design as a risk.

## Design decisions

- The verify stencil changes too, not only the baseline stencil.
  Hubs keep running with baseline reports written under the old text, and the verify side's rule keeps those from steering an acceptance.
- No marker or Go change: both stencils keep their substitution markers, so `BaselineTable` and `VerifyTable` render them unchanged.
