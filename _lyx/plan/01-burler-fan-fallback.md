# Card 1 — burler-fan-fallback

**Edit:**
- `internal/burlerengine#ResolveFan`
- `internal/burlerengine/config_test.go`
- `internal/burlerengine/template.yaml`
- `internal/burlerengine/doc.go`

**Create:**
- `internal/burlerengine#templateConfig`

**Intent:**
Decision D4: fan and lens resolution reads the operator's `burler.yaml` first and falls back, per name, to the embedded template.
`templateConfig` decodes the embedded `template.yaml` once (strictly, like `LoadConfig`) and returns it.
`ResolveFan` looks the fan name up in `cfg.Fans`, else in the template's fans, and each lens name in `cfg.Lenses`, else in the template's lenses.
A name the operator's file defines is used verbatim, so an operator-defined fan or lens wins over the template's.
A zero `Config` (an absent `burler.yaml`) therefore resolves wholly from the template.
An undefined fan in both is an error naming the fan and listing the known fans, the union of the operator's and the template's names (sorted, through `fanNames` over the merged set);
an undefined lens in both stays the existing "names undefined lens" error.
The "no fans are configured … reconcile" branch goes, since the template always defines fans.
`LoadConfig` stays the direct-read loader and `burler.yaml` stays seed-only for reconcile: the fallback is resolution, never a degrade of a missing file (`PATTERN-config-strictness`).

Bound: the fallback only supplies names absent from the operator's file; a standard fan the operator deleted resolves again from the template, which changes nothing unless a profile or `loom.yaml` names it.

`config_test.go`: `TestResolveFan` replaces the zero-Config "mentions reconcile" case with rows for an operator file without the requested fan resolving it from the template, an operator-defined fan of the same name winning, a zero `Config` resolving from the template, and a name undefined in both failing with an error naming the fan.
The template header comment states the per-name fallback; the `burlerengine` package doc's cluster section states it too.

**ImpactSummary:** ResolveFan gains a template fallback; its callers (Profile.validate, later loomengine) keep the same signature, and only the zero-Config error case changes.
