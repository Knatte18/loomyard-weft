# Card 10 — resign-head-match

**Edit:**
- `internal/planglyph#CanonicalizeHandles`
- `plan:internal/planglyph#planGatePass`
- `internal/planglyph/doc.go`
- `internal/planglyph/glyphchain_test.go`

**Prosa:**
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `plan:internal/planparser#CardResign`
- `internal/planparser#Card`
- `internal/planglyph#answerSymbols`

**Intent:**
Check every re-sign arrow's head against its member (D5): `resign-head-mismatch` at every gate, and `resign-interface-method` at the plan gates only.

`CanonicalizeHandles` takes each card's `Resigns` whose target parses as a member glyph (not a self glyph, not a `plan:` handle) as a second declaration source in its one batched `quarry.Name` call: `{Unit: <the glyph's Unit>, Decl: <the head>}`.
Build these sources inline in `CanonicalizeHandles`, whose row in `status_enforcement_test.go`'s allowlist already covers its `Unit` reads; a helper would need a row of its own.
Each re-sign answer is compared with its glyph and then discarded: it never enters `canonicalOwners`, the `handle-canonical-collision` check or the `RewriteRefs` substitutions, since the head names an existing member, not a handle.
A `Name` error, or an ID different from the glyph's own `String()`, is the blocking `resign-head-mismatch` on that card.
Its detail names the head, the glyph and what `Name` answered, and the way forward: write the member's new declaration head, receiver included for a method.
A receiver change from value to pointer keeps the member path and matches; a changed receiver type names another member and does not.
The check reads only plan text, so its verdict cannot change mid-run, and it runs wherever `resolvePass` does, `ValidateDispatch` included.

`planGatePass` adds the blocking `resign-interface-method`: an arrow whose member glyph resolves, through `answerSymbols`, to a method whose signature does not open with `func`, quarry's answer for an interface method, whose own spec is no declaration.
Its way forward is to drop the arrow and state the signature change in the card's Intent.
It reads the tree, so it stays out of `ValidateDispatch`.
What dispatch lets through: an arrow on an interface method that first appears after the plan gate passes, since its head can name the interface method's own glyph and so clear `resign-head-mismatch`.
What bounds it: the arrow's only mechanical effect is marking the member for caller coverage, which dispatch does not run either, and webster's per-card build and test gate fails the card that breaks a caller.
A struct field already fails as `glyph-not-found`, and every head written for it fails to parse or names another member.

What this lets through: a head that names the right member but misstates its signature passes, since nothing downstream compares it with the built declaration.
What bounds it: it misleads only the implementer reading the card, and the card's Intent and webster's per-card build and test gate bound it.

`doc.go` gains both bullets.
The spec gains both rows (`resign-interface-method` emitted at `ValidateFormat`, `Validate` and `ValidateRework` only), renumbered with every `row N` cross-reference fixed.

`glyphchain_test.go` gains an arrow leg per shape row: a head re-signing the member validates clean for every shape a Go declaration names (the value-method row through a pointer-receiver head, and the `shapes#Generic` row with `shapes#Generic.GenericMethod` beside it on the same card, the member whose receiver card 11 counts as a caller), the interface-method row reports `resign-interface-method` from `ValidateFormat` and not from `ValidateDispatch`, and the struct-field row reports `glyph-not-found` beside `resign-head-mismatch`.
Scenario rows: a head naming another member, a head that fails to parse, and a changed receiver type each report `resign-head-mismatch` from both `ValidateFormat` and `ValidateDispatch`; the same member re-signed on two cards validates clean with no `handle-canonical-collision`, and both card files stay byte-identical across `ValidateFormat`.

**ImpactSummary:** Adds two blocking checks on the new arrow only; plans without an arrow get the same batched Name call and the same findings as before.
