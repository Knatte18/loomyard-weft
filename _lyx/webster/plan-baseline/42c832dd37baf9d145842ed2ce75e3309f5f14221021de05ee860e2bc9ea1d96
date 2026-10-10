# Card 20 — stencil-drift-once

**Create:**
- `internal/stencilstore#refreshNotice`

**Edit:**
- `internal/stencilstore#decision`
- `internal/stencilstore#decide`
- `internal/stencilstore#reconcileOne` -> `func reconcileOne(path, name string, state State, onDisk, shipped []byte, mode Mode, source Source, force bool) (bool, refreshNotice, error)`
- `internal/stencilstore#reconcileAll`
- `internal/stencilstore#warnPortBackDrift`
- `internal/stencilstore/reconcile_test.go`
- `internal/stencilstore/doc.go`

**Rename:**
- `internal/stencilstore#TestWarnPortBackDrift_EmitsClassAndRemedyPerDifferingStencil` -> `internal/stencilstore#TestWarnPortBackDrift_EmitsOneLineListingTheDriftedStencils`

**Uses:**
- `internal/stencilstore#pendingWrites`

**Intent:** D10's second half: the stencil drift notice is one Warn per process listing the drifted stencils, instead of one line per stencil.
`warnPortBackDrift` collects the differing stencils with their drift class and remedy and emits one Warn whose message names the count and lists `<stencil>: <class>` pairs, followed by the remedies that apply, once; a pass where only behind-class stencils differ logs that one line at Info, as a behind class logs today.
The refresh notices flow back instead of logging per stencil: the `decision` struct's `logNotice func()` becomes a `notice refreshNotice` value, the stencil name with its case (not-older production build, dev build, unstamped build, edited behind) and the recorded and running revisions where the case has them; `decide` keeps its signature and fills that value where it built a closure; `reconcileOne` returns the decision's notice beside whether it wrote; `reconcileAll` collects every notice of the pass and emits one line per case listing its stencils, at the case's level today and with its remedy once.
`pendingWrites`, the `WritesDue` path, keeps calling `decide` and reads only the action, so it logs nothing, as today.
Reconcile runs once per process pre-run, so one line per case per pass is one per process.
The renamed test asserts one drift line naming both differing stencils and their classes; `reconcile_test.go`'s refresh cases assert the single listing line per case, and that `WritesDue` logs nothing.

**ImpactSummary:** `reconcileOne`'s one caller is `reconcileAll` in the same file, and `decision` is read only by `reconcileOne` and `pendingWrites`; log output only, with the reconcile decisions and writes unchanged.
