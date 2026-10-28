```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state.

Pairs checked:
- D1 × D2 (D2's ids come from D1's headings; a malformed heading has no id): held.
- D1/D2 × `discussionparser.Validate` ordering and darn history (D2 note): held.
- D1 × `lym loom decision add` / `Added after Discussion` (exempt in D1, unchecked in D2): held.
- D1/D2 × carry-over blocks (blocks live only under `## Open risks`, spec line 67): held.
- D3 × D4 token shapes (`#` and `/` split the shapes): held.
- D3 × D4 marks (a marked token gets neither finding): held.
- D3 `<placeholder>` rule × D4 `pkg.Member` shape: broken (F3).
- D4 resolving method × D8 classification: broken at the unstated outcomes (F4).
- D4 quarry outage × D9 (`planContextFindings` returns the error): held.
- D4 × darn (D1 and D2 only; no index needed): held.
- D5 marker set × D6 holding rule: broken (F1).
- D5 × D11 stacking rule × `impactset` guard scan: held.
- D6 × D9 index scope: broken (F2).
- D6/D8 Uses cover × the existing `uses-later-target`: held.
- D6/D8 package self glyph cover × the existing `redundant-package-target`: held.
- D6/D8 later-card cover × D9 done-card skip: held.
- D7 × D9 (gate.yaml tier keys, `language: go` only): held.
- D8 × D9 (index handed to `planContextFindings`): held for D8.
- D10 × D1/D2/D3/D4 (spec placement; darn stencil unchanged): held.
- Acceptance bullets for D1 to D11 × their decisions: held, except where F1 to F4 reach them.
- Constraints (stencil-ownership, sole-parsers, gate-slot parity, refusal-way-forward, `discussionparser` stdlib-only, `planglyph` owns quarry, lymdirs) × D1 to D11: held, except F2 touches "planglyph sole owner" only through D9's wording.

Findings:

F1 — MEDIUM, design, decision-record.md D5 × D6.
D5 marks "one line per struct" a test reflects over, but its scan only checks that some marker is present, never which structs are listed.
`envkit.NilSeams` recurses into every nested struct field of `shedrecipe.Env`, such as `landingshed.Deps`, `websterengine.RunDeps`, `pawlengine.Deps` and `loomshed.DarnDeps`.
D6 holds a card only to a struct named in a marker.
So a card adding a func field to `landingshed.Deps`, which breaks every NilSeams-calling wire-seams test, is held only if someone hand-listed that nested struct, and nothing enforces it.
This is the entry's core "`Deps` or `Env`" case.
D5's rationale says "a marker kept complete by a scan is exact", which the scan does not deliver.
Evidence: internal/testkit/envkit/envkit.go:521-544 (walkNil recurses on reflect.Struct); internal/shedrecipe/recipe.go:187,196,223,276 (nested Deps value fields); decision-record.md:100,104-109,113.

F2 — LOW, consistency, decision-record.md D6 × D9.
D6 holds a card that names "the file that declares the struct", which needs the struct's declaring file, a quarry answer (D4's method reports declaring files).
D9 hands the index to `planContextFindings` "for D8's classification" only, and the seam-marker reader gives only glyphs.
Evidence: decision-record.md:113, 159, 85.

F3 — LOW, consistency, decision-record.md D3 × D4.
D3's `<placeholder>` rule, which D10 puts in the spec for negative scenarios, exempts a `<` token.
D4 repeats the `<` exclusion only for the glyph shape.
The `pkg.Member` shape does not exclude it, so `loomshed.<Gone>` (Member holds an upper-case letter) is resolved as `internal/loomshed#<Gone>`.
Quarry rejects that before resolution, with no stated outcome.
Evidence: decision-record.md:71, 78-79, 165.

F4 — LOW, design, decision-record.md D4 × D8.
D4's method states found, build-constraint-partitioned ambiguous and outage, and D8 reads "declared only in `_test.go` files" from it.
Neither states what an unpartitioned ambiguous answer or a pre-resolution rejection yields: a discussion finding (which one?) or a D8 classification.
The lifted `statusFindings` policy maps these to `glyph-ambiguous` and `glyph-rejected`, which have no discussion-side name.
Evidence: internal/planglyph/resolve.go:84-139; decision-record.md:81-86, 146.

Tool calls:
1. grep of the discussion spec and `discussionparser` for carry-over: my context lacked where carry-over blocks may appear, which the D1/D2 × carry-over pair needs.
