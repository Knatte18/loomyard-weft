# Card 22 — orch-go-driver-line

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils#wordingClaims`

**Intent:**
D15, as the parent accepted it: a run on the Go driver has no live receiver for run messages, so the orch reaches it through durable verbs until a later task adds one.

`orch-template-role.md`'s `## Fixing a run` section gains one line after the stop-at-a-point line: a go-driven run has no driver to message, so address it through `lyx loom` and `lyx shed` verbs (pause, goto, decision, resume).
This departs from `orch-split`'s routing only for go-driven runs; the existing lines for a run with a live driver stay.

`wordingClaims`' `orch-template-role.md` entry gains a claim that the line reaches the agent: `must` "a go-driven run has no driver to message", in section `## Fixing a run`, with its why.

**ImpactSummary:** one stencil line and one claim row in `contracts/stencils`; no Go behavior changes.
