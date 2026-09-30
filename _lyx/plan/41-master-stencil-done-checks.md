# Card 41 — master-stencil-done-checks

**Prosa:**
- `contracts/stencils/webster/webster-template-master.md`

**Edit:**
- `internal/websterengine/template_test.go`

**Uses:**
- `internal/websterengine/recordbatch.go`

**Intent:**
Fix PR #321 finding 6: the Master stencil's "A done-check failure arrives as `batch_failed`" section still lists "a symbol this batch deleted that the remaining plan still references" as a `batch_failed` done-check.
`postBatchChecks` now records that finding as a later-card-drift warning on the batch, so the stencil teaches Master to expect a failure that never comes.

Remove that item from the section's list of done-checks, and add one sentence after the list: a finding about a later card (drift, or a symbol this batch deleted that a later card still references) comes back on the envelope's `warnings` and the batch still records; that later card's own `begin-batch` refuses it, naming "edit the plan, then `lyx webster rebaseline`".
Keep the section's semantic line breaks and change nothing else in the stencil.

`template_test.go` gains an assertion on the rendered master template: the done-check section no longer carries `a symbol this batch deleted that the remaining plan still references`, and it carries the later-card warning sentence.

**ImpactSummary:** One stencil section's done-check list and one template test change; the stencil's markers are unchanged.
