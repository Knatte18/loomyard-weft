# Card 40 — slices-cleanup

**Edit:**
- `fabriccli/cloneconfigcommit_integration_test.go`
- `fabricengine/diff_integration_test.go`
- `stencilstore/reconcile_test.go`
- `gitrepo/parity_test.go`
- `reedengine/reconcile_test.go`

**Intent:**
Replace the remaining hand-written slice helpers with the standard library (#324 §6): each `containsPath` becomes `slices.Contains` and `reedengine`'s `equalStringSlices` becomes `slices.Equal` (card 16 already removed reedcli's copy).

Check each copy's semantics before replacing it: a helper that normalises paths (slash conversion, cleaning) before comparing keeps that normalisation at the call site, so no assertion becomes looser or stricter.
After this card, `grep -rn 'func containsPath\|func equalStringSlices' internal cmd` finds nothing.
Apply D10 to every touched test.

**ImpactSummary:** Five test files swap private slice helpers for slices.Contains/slices.Equal; comparisons are unchanged.
