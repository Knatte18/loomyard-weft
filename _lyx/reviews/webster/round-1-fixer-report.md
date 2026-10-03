# Webster round 1 — fixer report

Every finding in `round-1-review.md` is fixed, one commit each.
The plan's verify set (build, untagged tests, integration-tagged tests, smoke vet) passes on the final HEAD.

## Fixed

- **F1 (MEDIUM)** — `d165b2410`.
  `frictionengine.Reflect` now budgets only a positive `Deps.Timeout`;
  a zero one reaches the attach and spawn specs unchanged, so shuttle resolves it to `run_timeout_min` as `friction_timeout_min: 0` documents.
  `Deps.Timeout`'s doc, `Reflect`'s doc and the package doc state the zero meaning.
  New test `TestReflect_ZeroTimeout_DefersToShuttle` asserts a settle-path attach and a spawn both carry a zero `Spec.Timeout` and the call reports `reflected`.
- **F2 (LOW)** — `014b6b278`.
  `addVerbs`' and `noteRefusals`' docs and the CONSTRAINTS.md Friction Capture Invariant now state that a refusal cobra raises before the verb's `RunE` (flag parsing, argument count, the persistent pre-run) is not noted.
- **F3 (LOW)** — `83c1d0643`.
  Rewrapped every new and changed Go comment the review listed to one sentence per line, breaking inside a sentence only at an independent-clause boundary.
  Also corrected `validateDeps`' doc, which read "and an empty Deps.StencilsDir, and an empty Deps.TaskSlug".
- **F4 (LOW)** — `c40a80692`.
  `reflectFriction`'s lock paragraph names the halt path (`loomPostRun` under run, `loomAfterStep` under step) and describes the hazard as archiving covered notes out from under the first reflection's agent.
- **F5 (LOW)** — `39dcb0280`.
  The reflection stencil is rewrapped to one sentence per line;
  every `claims_test.go` phrase still sits on one line and the claims test passes.
- **F6 (NIT)** — `68d0820c0`.
  `archiveCovered` moves the report before the record, so a failure part-way always leaves the record beside whatever is left to settle.
  No test drives it: a rename that fails for one entry but not the next cannot be produced portably in a temporary directory.

## Deferred

None.
