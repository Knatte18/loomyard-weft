# Drift-Baseline pre-accepts drift that contradicts the task

`drift-baseline-preaccepts` · Task · bug, drift, stencil · running · Drift-Baseline

## Seen

On the `cs` hub, task `cached-friction-factor` (Drift trial 2, 2026-11-03) states "every calculated result stays unchanged", while its design computes the friction factor at a rounded Reynolds number, which moves output.
The Drift-Baseline agent saw the contradiction and resolved it in the design's favour, writing in `cached-friction-factor/dotnet/_lym/drift/baseline.md`:

> The goal says every result stays unchanged, but the specified design rounds the Reynolds number […]
> Any drift in these cases is accepted with the reason "friction factor evaluated at the rounded Reynolds number".
> A change larger than a relative 1e-3 in `friction_factor` is not explained by rounding and is a regression.

The baseline report is the expectation Drift-Verify judges against (`drift-template-verify.md`: "Judge each drifted test against the task and the baseline report's expectation").
A report that pre-authorizes acceptance steers the verify agent to accept drift the task says must not happen.
In this run the writer kept the results exact and the compare ran clean, so no acceptance followed; the steering stands for the next run that drifts.

The stencil (`contracts/stencils/drift/drift-template-baseline.md`) tells the agent to name the drift it expects "so the agent that judges the compare later accepts the intended drift with a reason".
It has no rule for a task whose stated invariant and specified design disagree, and it lets the baseline agent write acceptance policy rather than expectation.

## Expected

- A stated output invariant ("results stay unchanged", "behavior-preserving", "pure refactor") wins over a design that would move output: the baseline expects no drift, and any drift is a regression for Drift-Verify to fix or escalate.
- The baseline report names the contradiction for the parent and for Drift-Verify as a risk, instead of resolving it.
- The baseline agent states what it expects and why; it never states what will be accepted. Acceptance stays with Drift-Verify and, under scope `full`, the parent's ruling.
