# Review Drift as a whole and propose one reworked design

Adds `docs/research/drift-whole-review-2026-11-04.md`, a review of Drift as one system, written from the two C# trial reports, the Drift code, the two stencils and the recipes.
No code, stencil or config changes.

## What the report holds

- A map of every Drift row, verb, config key, stencil and record, and a split of what code decides, what an agent decides and what the parent decides.
- Each trial fault placed on that map, with the class of fault that remains after its fix.
- Six faults the trials did not show, found by reading the code: `lym drift accept` and `--skip` check no test name; an acceptance covers a whole test while the log shows one line of it; the accepted profile is captured after the agent's fixes and so certifies the final output; scope `off` is an agent's call with no recorded reason in the verdict; the writer never sees the expectation or a compare; the expectation file is not sealed.
- Answers to the three review questions: Drift-Recheck needs a row but not an engine of its own; acceptances can lapse per cell once code diffs the profiles; the accepted-profile wiring has a test now.
- One reworked design: code diffs baseline and current profiles cell by cell, an agent writes a structured expectation after the capture and before any code, code decides containment, and a separate agent judges only the values inside the expected cells.
- The guarantees the flow must give, what the design removes and adds, five decisions for the operator, and a build order a tapestry can take.

## Design decisions of the review

- The report proposes and builds nothing; the operator decides the design, as the plan on the board entry says.
- It runs no Drift run. Where the design depends on the `cs` harness writing line-oriented text profiles, it cites trial 1's setup and names the assumption as the first decision.
- Tapestry's fix loop depends on Webster's gate seam, which the review did not read; the report says so instead of guessing.
