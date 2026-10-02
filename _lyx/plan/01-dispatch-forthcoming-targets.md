# Card 1 — dispatch-forthcoming-targets

**Edit:**
- `internal/planglyph#ValidateDispatch`
- `internal/planglyph#resolvePass`
- `internal/planglyph/planglyph_test.go`
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/runlevel.go`
- `internal/webstercli/validate.go`

**Intent:**
Give dispatch validation a second card set: cards excluded from the resolve pass whose Create targets and Rename New sides still count as forthcoming (#329, decision D1).

`ValidateDispatch` gains a trailing `forthcoming []planparser.Card` parameter beside `completed`.
`completed` keeps its meaning (excluded from the resolve pass and from card-scoped pure findings);
`forthcoming` names the subset of those excluded cards whose work may not have landed.
`resolvePass` takes the forthcoming card IDs as a set, and after canonicalization (and the reload when it rewrote the plan) selects those cards by ID from the current plan, so a handle canonicalization respelled is matched under its new spelling.
Their Create targets join `createTargets` and their Rename New sides join `renameNewTargets`, so a pending card's `Uses:` or target of such a ref is excluded from `statusFindings` exactly as a pending card's own Create target already is.
The forthcoming cards themselves stay out of every pass: they are not resolved, not paired for containment and not checked by the Create inversion.
Update both doc comments to state the two sets and why a forthcoming card's targets are excluded rather than resolved.

The three callers (`BeginBatch`, `Run`, webstercli's `scopedValidate`) pass `nil` for `forthcoming` in this card, so behavior is unchanged until card 2 computes the set.

Bound: a forthcoming ref is excluded from the status check, so a `Uses:` of a target declared by a forthcoming card passes unresolved.
It grants nothing an unbegun pending card's Create target does not already grant, and covers only Create targets and Rename New sides; a forthcoming card's Delete and Edit targets add nothing to either set.

Tests in `planglyph_test.go`, on a `language: go` plan over a fixture repository so the resolve pass runs:
- a pending card's `Uses:` of a Create target declared by a completed-and-forthcoming card that does not exist on disk yields no `glyph-not-found`;
- the same plan with that card completed but not forthcoming reports `glyph-not-found` for the `Uses:`;
- a Rename New side declared by a forthcoming card is excluded the same way;
- a forthcoming card's own Create target that already exists yields no `create-already-exists`, since the card is not resolved.

**ImpactSummary:** `ValidateDispatch`'s signature gains a parameter; its three production callers pass nil, so no behavior changes outside `planglyph`.
