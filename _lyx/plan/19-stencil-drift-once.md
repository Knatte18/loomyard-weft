# Card 19 — stencil-drift-once

**Edit:**
- `internal/stencilstore#reconcileAll`
- `internal/stencilstore#decide`
- `internal/stencilstore#warnPortBackDrift`
- `internal/stencilstore/reconcile_test.go`
- `internal/stencilstore/doc.go`

**Rename:**
- `internal/stencilstore#TestWarnPortBackDrift_EmitsClassAndRemedyPerDifferingStencil` -> `plan:internal/stencilstore#TestWarnPortBackDrift_EmitsOneLineListingTheDriftedStencils`

**Intent:** D10's second half: the stencil drift notice is one Warn per process listing the drifted stencils, instead of one line per stencil.
`warnPortBackDrift` collects the differing stencils with their drift class and remedy and emits one Warn whose message names the count and lists `<stencil>: <class>` pairs, followed by the remedies that apply, once; `decide` returns its refresh notices (the dev-build, unstamped-build and edited-behind cases) to `reconcileAll` instead of warning per stencil, and `reconcileAll` emits them as one Warn listing the stencils per remedy.
Reconcile runs once per process pre-run, so one Warn per pass is one per process.
The renamed test asserts one drift line naming both differing stencils and their classes; `reconcile_test.go`'s refresh cases assert the single listing line.
`doc.go` states the one-line notices.

**ImpactSummary:** Log output only; the reconcile decisions and writes are unchanged.
