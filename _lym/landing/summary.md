# Drift-Baseline states expected drift, never acceptance policy

The Drift-Baseline stencil let the baseline agent write acceptance policy into its report.
Drift-Verify judges against that report, so a baseline that pre-authorizes drift steers it to accept drift the task says must not happen.

On the `cs` hub, task `cached-friction-factor` stated that every calculated result stays unchanged, while its design evaluated the friction factor at a rounded Reynolds number.
The baseline agent resolved the contradiction in the design's favour and wrote "any drift in these cases is accepted with the reason …".
That run's compare came out clean, so nothing was wrongly accepted, but the next run that drifts would have been.

Closes the board entry `drift-baseline-preaccepts`.

## Changes

`contracts/stencils/drift/drift-template-baseline.md`:

- Step 1 gains a rule: a stated output invariant ("results stay unchanged", "preserves behavior", "pure refactoring") wins over any part of the design that would move output.
  The agent expects no drift and never resolves the contradiction itself; it names it as a risk.
- Step 2 no longer says the verify agent "accepts each test that drifts as you expect with a reason"; it says the agent judges each drifted test against the task and the expectation.
- Step 4 asks the report to state what the agent expects and why, never what will be accepted: no acceptance reason and no tolerance under which drift passes.
  Under an invariant the report says no drift is expected and names each contradiction between invariant and design, what the design would move, and that the invariant forbids it.

`contracts/stencils/drift/drift-template-verify.md`:

- States that the baseline report records what its agent expected, never what Drift-Verify may accept.
- A task with a stated output invariant intends no drift, so any drift in it is a defect, even where the baseline report names a part of the design as a risk.

## Design decisions

- The invariant wins in both stencils, not only in the baseline.
  Fixing the baseline alone would leave Drift-Verify trusting any older or hand-written report that still carries acceptance language.
- Acceptance stays with Drift-Verify and, under scope `full`, the parent's ruling; the baseline agent has no say in it.

## To check by hand

Only the stencil text changed, with no Go code and no tests.
No regression check ran and none of this was run against a live task.
Check that the new wording reads unambiguously to an agent, ideally on a run whose task states an invariant that its design contradicts.
