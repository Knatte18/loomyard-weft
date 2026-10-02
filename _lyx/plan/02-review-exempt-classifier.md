# Card 2 — review-exempt-classifier

**Create:**
- `internal/planparser/reviewclass.go`
- `plan:internal/planparser#ReviewExempt` -> `func ReviewExempt(plan *Plan) bool`
- `internal/planparser/reviewclass_test.go`

**Intent:**
Add the one classification function D6 names, shared later by PR-Rework (which records the class) and the Plan-Bouncer skip seam (which re-checks it).

`ReviewExempt` reports true only when the plan has at least one card, every card's every `TargetGroups` entry is `Prosa`, and no `Prosa` target is a source file.
A `Prosa` target is a source file when:

- under a glyph-enabled `language:`, it parses (through the package's existing `parseGlyph` helper) to a self glyph whose `Glyph.UnitPath` carries no file extension (a whole package or directory), or to a file self glyph whose extension is a source extension of the plan's language (`.go` for `go`);
- under `language: none`, it carries no file extension (a directory), since `none` declares no source extensions.

Anything that fails to parse counts as a source file, so the function over-covers rather than under-covers.
Declare the per-language source-extension set once in this file.
Ref-shape decisions route through the package's existing helpers (`planLanguage`, `parseGlyph`, `hasFileExtension`) and glyph-to-path conversion through `Glyph.UnitPath` only, per the Ref-Shape Registry and Glyph Conversion Chokepoint invariants.
The function reads card type labels and target paths alone, never a label the plan writer chooses for itself, which is why it can be trusted to route a risky card to review (D6).

Tests (table-driven over hand-built and parsed plans): all-`Prosa` cards on `.md` and `.yaml` files → exempt; a card carrying `Edit`, `Create`, `Delete`, `Rename`, `Move` or `Custom` → required; a multi-label `Prosa`+`Edit` card → required; a `Prosa` card on a `.go` file → required; a `Prosa` card on a whole-package self glyph (`internal/foo#`) → required; a `language: none` plan's `Prosa` card on a directory → required and on `docs/a.md` → exempt; a plan with no cards → required.
