# Plan-Review round 1 — fixer report

All three findings were validated against the code and the decision record and fixed.

## Fixed

- **F1** (LOW, decision) — `02-record-batch-resolves-head-sha.md`.
  Confirmed: `git rev-parse --disambiguate=7B3D140C` resolves with exit 0, and D1 resolves any hex string of 4 to 39 characters.
  The resolver bullets now take a 40-hex value of either case unchanged, lowercase a 4-to-39-digit hex value of either case before `CommitsNamedBy`, and return the lowercase full SHA it answers;
  "any other value" now reads as one holding a non-hex character, or longer than 40.
  The record-batch abbreviation row covers an uppercase abbreviation beside the lowercase one.
  The open "confirm git's exit status" note is replaced by the confirmed fact: a prefix naming nothing prints nothing and exits 0.
- **F2** (LOW, consistency) — `05-derived-overview-frame.md` ImpactSummary.
  It now says three helpers (`fingerprintMismatchWayForward`, `PlanEditError`, `batchCardEditError`) gain the webster dir and two (`overviewIndexOnly`, `overviewRefusal`) take the recorded frame in place of the state.
- **F3** (NIT, consistency) — `01-pattern-fill-directive.md`.
  Confirmed against `internal/pattern/pattern.go:129`: the wrap names the stencil.
  `FillDirective` gains a `stencilName` parameter, used only in the fill-error wrap `pattern: fill directive stencil %q: %w`, so `Directive` passes its stencil's name and its error text stays unchanged.
  Card 7's `merriamTexts` bullet passes `directiveStencilPath` as that name.

## Disputed

None.

## Deferred

None.
