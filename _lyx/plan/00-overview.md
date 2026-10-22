---
format: 5
approved: true
language: go
---

# Plan: lenses resolve from a language-neutral base plus a picked language layer

A hub's lenses resolve from a language-neutral base template plus an embedded language layer (`go`, `csharp`, `python`) its base `lenses.yaml` picks with `lens_set`, and a new `remove` key drops lenses.
`configengine` gains a generic template overlay whose effective template every load, write and reconcile path judges against, so lenses stays a leaf package and every other module behaves as before.

## Card Index

1 — configengine-overlay-load — add the spec overlay, its one composer and the base-layer selector, and load, merge-below and annotate against the effective template
2 — configengine-overlay-writes — make Set and Edit judge against the effective template, scaffold without per-name entries, and refuse the selector in a hub worktree file
3 — lenses-lens-set — wire `lens_set` into the lenses module and move today's five Go-worded lenses into the `go` layer verbatim
4 — lenses-language-layers — reword the base neutrally and ship the `csharp` and `python` layers
5 — configsync-effective-template — judge every strip, prune, reconcile and template-equal drop against the effective template
6 — lenses-remove — add `remove`, applied in `lenses.Load`, with the emptied-fan refusal and a once-per-process warning for a name matching no lens

## Shared Decisions

### Decision: pattern-overview-headroom

- **Decision:** card 1 adds the declared `PATTERN-template-overlay` line and cuts at least as many bytes from `PATTERN.md`: the overview's intro line and the blank line after it, and the two entry clauses the record's `PATTERN changes` declare (`PATTERN-sparse-hub-base`'s walk-strip clause, `PATTERN-config-layers`' machine-layer clause), each recorded after Discussion.
- **Rationale:** the overview is already past the headroom line `patternlint.Lint` enforces at the verify gate, so any growth fails the run; each cut clause is already stated in its entry's background file, so no rule is lost.
- **Applies to:** card 1

### Decision: effective-template-entry-points

- **Decision:** `configengine.EffectiveTemplate(spec, base, hub)` composes from a base document; `configengine.TemplateUnder(dirs, spec)` reads the base layer and calls it; `configsync` uses the first where it holds the base document it judges and the second for a file above the base.
- **Rationale:** D4 keeps composition and layer-file reading in `configengine` (`PATTERN-config-layers`), and the `hub` flag only picks the unknown-value refusal's way forward.
- **Applies to:** cards 1, 2, 5

## PATTERN changes

- `PATTERN-template-overlay` — adds: Judging a config document against its template: only against `configengine`'s effective template, the overlay picked from the base layer.
- `PATTERN-sparse-hub-base` — changes: Writing a hub's base file: only the units the hub tuned, never a template fill.
- `PATTERN-config-layers` — changes: Resolving a module's config: only configengine reads, stacks and locates layer files.
