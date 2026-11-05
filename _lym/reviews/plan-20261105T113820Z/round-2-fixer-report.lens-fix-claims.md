```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: checking that every claim the fix added or rewrote holds against the code at HEAD.

**[LOW]** Card 19, the added allowlist sentence: "the proposal parser and its finding type, told its paths, resolving no geometry."
- **Summary:** `raddleparser` is told no paths, so the comment line the card prescribes would be false.
- **Evidence:** card 12 makes the package pure. `Parse(data []byte)`, `Hash(data []byte)`, `Check(p Proposal, record []byte, board BoardContext)` and `RecheckBoard(p Proposal, board BoardContext)` all take bytes or values, never a path. The paths go to loomshed's `RaddleCheck` (card 19) instead.
- **Fix:** drop "told its paths" and keep "the proposal parser and its finding type, resolving no geometry".

Every other added claim holds:
- **Card 21, `stencil.FillWith`:** the function takes an `optional` list and refuses unfilled top-level markers not in it (`internal/stencil/stencil.go:35,53`).
- **Card 21, `Table.Validate`:** it rejects a blank table value not named in `Optional` (`internal/seatengine/table.go:218,347`).
- **Card 21, `DiscussionTable`:** it sets `Optional` (`internal/loomengine/discussiontable.go:169`).
- **Card 21, stencil wording:** the bound words "never", "do not" and "before" are in `askOperatorBounds` (`contracts/stencils/stencils.go:807`).
- **Card 12, `PATTERN-tolerant-records`:** the background keeps agent-written contracts strict and says a future JSON contract "validates its keys some other way" (`pattern/PATTERN-tolerant-records.md:9,11`).
- **Card 25, `pawlDeps`:** it fills `Paths: hubgeom.PawlPaths(location)` and nothing else path-related (`internal/loomcli/pawlwiring.go:51`).
- **Card 25, `TestWire_BeamRecipe`:** the test compares `c.env.Pawl.Paths` with `hubgeom.PawlPaths(loc)` (`internal/loomcli/wiring_beam_test.go:147`).
- **Card 25, marker constant:** `AwaitMarkerName` is in `internal/pawlengine/landrequest.go:24`.
- **Card 25, `TestDobbyGeometry`:** the test checks each named pawl path against the dobby's `.lym/beam/` directory (`dobbygeom_test.go:112`).
- **Card 22, `boardengine`:** none of its dependencies imports loomshed (the parent's `go list -deps` counted 0).
- **Card 22, `SplicePart` inputs:** card 12 defines the four unknown checks, and card 22 has `Check` run before step 2 in the same `Call`.

Further tool calls:
1. A grep of `internal/stencil` `FillWith`, the tolerant-records background and `pawlwiring.go` lines 40–70. My context had none of `FillWith`'s optional rule, the pattern's background text or the `pawlDeps` body.
2. A grep locating `FillWith` and its optional and empty handling. The first grep's path glob missed the file.
