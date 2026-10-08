# Card 3 — rename-interface-method

**Edit:**
- `internal/planglyph#renameDeclSource`
- `internal/planglyph/glyphchain_test.go`
- `internal/planglyph/glyphchain_integration_test.go`

**Intent:**
Fix the defect the D1 probe found: a `Rename` whose Old side is an interface method can never canonicalize.
quarry answers an interface method with kind `method` and its bare method spec as the signature (`IfaceMethod() int`, no `func`, no receiver), so `renameDeclSource` derives `IfaceMethodRenamed() int`, which `quarry.Name` rejects as unparseable, and the plan gate reports a blocking `handle-name-failed` on a correct plan.

When the resolved Old symbol is a method whose signature does not open with `func`, `renameDeclSource` derives the declaration as a method on its owner instead: `func (<owner>) ` followed by the renamed spec, the owner taken from the symbol's parsed `Glyph.Owner`.
`quarry.Name` names `func (Iface) IfaceMethodRenamed() int` as `shapes#Iface.IfaceMethodRenamed`, the glyph the delta then reports as created.
Every other rename keeps today's derivation.

Add the interface-method row's Rename leg (`shapes#Iface.IfaceMethod` -> `plan:dirname#Iface.IfaceMethodRenamed`) to both glyph-chain files: the plan gate validates clean with the handle canonicalized to `plan:shapes#Iface.IfaceMethodRenamed`, and the delta leg binds it and passes its done-check.

**ImpactSummary:** One branch in `renameDeclSource`, reached only by a Rename of an interface method, which failed canonicalization before; every other Rename derives its declaration unchanged.
