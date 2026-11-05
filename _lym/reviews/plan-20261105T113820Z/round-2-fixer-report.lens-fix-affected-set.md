```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covers every member of each finding whose premise is a set: F1, F2, F5, F3 and F8.

Unchecked: F3 and F8. Each premise is one table or one sentence, so I spent no search on them. F4, F6 and F7 have no set premise.

**[LOW] Card 25 states that only `hubgeom.PawlPaths` fills `HandOffMarker`, but two other `pawlengine.Paths` fillers it edits are left out.**
- Changed text: card 25 Intent, the added `hubgeom.PawlPaths sets HandOffMarker …` sentence, and the rewritten `beamUpEntry requires the new seam and path; pawlDeps fills the HandOff seam, and the path reaches it through …` sentence.
- Search: `pawlengine.Paths{` is built in four places:
  - `internal/hubgeom/dobbygeom.go:76`, which the fix now covers.
  - `internal/pawlengine/world_test.go:63`. The Intent covers it ("world and world.deps gain … the marker path").
  - `internal/shedrecipe/fixture_test.go:66` (`validPawlDeps`).
  - `internal/testkit/envkit/envkit.go:377` (`FullEnv`).
- Gap: card 25 lists `validPawlDeps` and `FullEnv` as Edit targets, but its Intent only says their fake `Notify` reports posted. It never says their `Paths` literal gains `HandOffMarker`.
- Effect: since `beamUpEntry` "requires the new seam and path", a `validPawlDeps` without the marker would make shedrecipe's entry tests refuse the full deps.
- Fix: one clause saying `validPawlDeps` and `FullEnv` set `HandOffMarker` beside `AwaitMarker`.

Confirmed with no finding:
- **F1:** of the cards that touch loomshed (18–23), only 19 and 22 bring a new import into loomshed production code, `raddleparser` and `boardengine`, and both now edit `loomshedAllowedImports`. Card 18's `boardengine` read goes through `loomcli.recordWriteDeps`, not loomshed. Card 20's members live in loomcli.
- **F5:** the only production site that writes a recipe name as a literal and that card 23 touches is the `internal/shedcli/table.go:55` map key, which the fix now covers. The other sites the fix names already use constants or are comments.

Tool calls beyond my context:
1. grep for `pawlengine.Paths{`, to rerun F2's filler search.
2. grep of the plan cards for loomshed targets, to rerun F1's search for loomshed production imports.
3. grep for `"motif"` in production Go, to rerun F5's literal-site search.
4. `validPawlDeps` in `fixture_test.go` and card 18/20 loomshed lines, to confirm the third `Paths` filler and to check cards 18 and 20 for new loomshed imports.
5. Card 18's head, to see its targets for a loomshed import.
6. grep of cards 18 and 21 for `boardengine`, to confirm where card 18's `boardengine` use lives.
