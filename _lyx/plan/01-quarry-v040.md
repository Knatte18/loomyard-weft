# Card 1 — quarry-v040

**Edit:**
- `go.mod`
- `go.sum`
- `internal/planglyph#CanonicalizeHandles`
- `internal/planglyph#renameDeclSource`

**Intent:** Raise `github.com/Knatte18/quarry` from `v0.2.1` to `v0.4.0` (D1): run `go get github.com/Knatte18/quarry@v0.4.0` and `go mod tidy`, so `go.mod` pins `v0.4.0` and `go.sum` carries its hashes.
`v0.3.0`'s Enclose comes in unused; no check reads it.

Fix the library break: `quarry.Name` rejects a `quarry.Declaration` whose `Lang` is zero, per answer, so every declaration LoomYard builds sets `Lang`.
In `CanonicalizeHandles`, a Create handle's declaration and a re-sign arrow's declaration set `Lang` to the language `plan.GlyphLanguage()` returns, which is `glyph.Go` for every plan this card can see.
In `renameDeclSource`, every derived declaration (the var and const synthesis included) sets `Lang` to the resolved symbol's own `sym.Glyph.Lang`.
No call site leaves `Lang` zero.

Coverage is the existing Go binding tests, which fail with `handle-name-failed` or `resign-head-mismatch` on every row when `Lang` is zero: `TestCanonicalizeHandles_Rewrites`, `TestGlyphChain_PlanGate`'s Create, Rename and re-sign legs, and `TestGlyphChain_ResignScenarios`.
Every Go plan fixture keeps the findings it has today.

**ImpactSummary:** module-wide dependency raise; the only source change is the `Lang` field on the three declaration sites inside `internal/planglyph`'s handle canonicalization, with no signature change.
