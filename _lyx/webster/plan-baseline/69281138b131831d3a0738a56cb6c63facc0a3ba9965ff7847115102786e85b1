# Card 1 — pattern: fill a directive from stencil text

**Create:**
- `plan:internal/pattern#FillDirective` -> `func FillDirective(stencilName, stencilText, overview string) (string, error)`

**Edit:**
- `internal/pattern#Directive`
- `internal/pattern/doc.go`

**Intent:**
Extract the in-memory half of `Directive` into an exported `FillDirective`, so a caller holding a directive stencil's bytes and a `PATTERN.md`'s bytes from somewhere other than disk renders the directive exactly as `Directive` does.
`tools/tokencount` needs this in card 7 to reconstruct the orchestrator directive at a run's base commit (D4).

`FillDirective` returns `("", nil)` for a whitespace-only overview, and otherwise strips the stencil's leading banner with `stencil.StripLeadingComment` and fills its `pattern_overview` marker with the overview through `stencil.Fill`.
It wraps a fill error as `Directive` wraps it today, `pattern: fill directive stencil %q: %w` over `stencilName`, which names the stencil only in that error text.
`Directive` keeps its order (empty worktree root, unknown role, overview read, inactive overview, then the stencil read) and calls `FillDirective` with its stencil's name for the fill, so it still reads no stencil for an inactive overview and its fill-error text is unchanged.
No import changes, so `PATTERN-leaf-packages`' closed import set for `pattern` holds.
`doc.go` names `FillDirective` beside `Directive` as the fill a caller with stencil and overview bytes in hand uses.

This is a pure extraction: `Directive`'s existing tests in `internal/pattern/pattern_test.go` cover the fill, and card 7's calibration test covers `FillDirective` through tokencount.

**ImpactSummary:** `Directive`'s behavior is unchanged; one new export in a leaf package, no caller changes.

**Commit:** 1: pattern: FillDirective renders a directive from stencil and overview bytes
