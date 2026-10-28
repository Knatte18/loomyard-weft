```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: whether each guard, refusal and test row the cards add can be reached from a real entry point.

**Findings**

1. **BLOCKING** — Card 8 (`seam-test-check`), D6/D9. The `TestPlanWriteEntry_GateConfig` row panics before it can report `seam-test-unlisted`.
   - That row runs the gate built through `planContextOf` over a worktree that has a marked struct and a card that edits it.
   - With a marker present, `seamTestFindings` calls `index.ResolveGlyphs`.
   - The shedrecipe tests give the gate `noFindingsPlanIndex{}`, or `placeholderIndex{}` from `newTestEnv`. Both embed a nil `planindex.Index`, so any method they don't define panics.
   - Card 8 adds no `ResolveGlyphs` to either fake.
   - Card 3's line "the fakes that embed `planindex.Index` … inherit the method and need no change" is false for any row that reaches the new checks.
   - Evidence: `internal/shedrecipe/fixture_test.go:71-72` ("any other method panics"), `internal/shedrecipe/fixture_test.go:179` (`PlanIndex: placeholderIndex{}`), `internal/shedrecipe/entries_planwrite_test.go:314,335`, `internal/shedrecipe/recipe.go:277-279` (shedrecipe links no tree-sitter, so its tests cannot switch to `planglyph.NewIndex`).

2. **MEDIUM** — Card 9 (`doc-go-check`), D8. The test-code rule wrongly flags a test-only card that creates a new test function by handle, and no row reaches that case.
   - The common test-only card is a `Create` of `plan:&lt;dir&gt;#TestNew`.
   - Card 9 counts an unresolved member as test code only when it "is a `Rename` new side whose old side is test code or whose member name begins `Test`…". The `Test` prefix there attaches to the Rename new side.
   - So `plan:internal/d#TestNew` comes back `GlyphMissing`, counts as non-test code, and the card gets `doc-go-unlisted`.
   - D8's bound implies the prefix rule covers any unresolved member, not only a Rename new side ("A card creating a test helper by member glyph whose name begins with none of those prefixes is not test-only"). The acceptance criterion also says a test-only card passes.
   - The rows cover a Rename to `plan:internal/d#TestDoes` and an Edit of `internal/d#fixture`, but no Create of a `Test…` handle.
   - Evidence: `_lym/plan/09-doc-go-check.md` Intent "A ref is test code when…" and its Tests list; decision record D8, lines 139 and 145.

3. **MEDIUM** — Card 4 (`citation-check`), D4. No row reaches the `GlyphAmbiguous` → `citation-symbol-ambiguous` ("cite the declaring file") branch.
   - The only ambiguity row, `x.Foo`, comes from two `internal/dup/{a,b}/x` package directories sharing a base name. That is a different branch, decided before `ResolveGlyphs` runs.
   - The branch where quarry answers ambiguous without a build-constraint partition can occur (card 3's `p#G` fixture produces it), but no citation row cites such a glyph.
   - Evidence: `_lym/plan/04-citation-check.md` Intent "a `GlyphAmbiguous` answer is one `citation-symbol-ambiguous` finding naming the candidates' files" against its Rows list.

4. **MEDIUM** — Card 4, D3. The `_lym`/`.lym` exclusion is never actually reached.
   - The `_lym/plan/00-overview.md` row passes on whichever branch applies first. If the `gitkit.CopyRepo` fixture does not track a `_lym` root entry, the token passes as "first segment not a tracked root", and the `lymdirs.LymDirName` exclusion never runs.
   - No row uses a `.lym/` token.
   - The row only proves the exclusion if the fixture commits a `_lym/` entry and the cited file is absent.
   - Evidence: `_lym/plan/04-citation-check.md` Path bullet and Rows ("`_lym/plan/00-overview.md` … pass").

5. **LOW** — Card 4, D4. The quarry-outage row only reaches `ResolveGlyphs` if its record cites a glyph or qualified symbol, and the card doesn't say so.
   - `citationFindings` makes the call "only when there is at least one".
   - A record with no symbol citation passes without calling the fake, so the error branch goes unreached.
   - Evidence: `_lym/plan/04-citation-check.md` Intent "resolved in one `index.ResolveGlyphs` call, made only when there is at least one" against the row "a fake index … whose `ResolveGlyphs` returns an error".

6. **LOW** — Card 6 (`seam-marker-scan`), D5. The "marked with both is not a violation" `NilSeams` row needs the fixture package to declare `type Env struct` and `type Inner struct`, and the card doesn't say so.
   - Without them, the marker-names-no-struct check fires on `p#Env`/`p#Inner`, and the row reports a violation.
   - Evidence: `_lym/plan/06-seam-marker-scan.md` third bullet ("declares no `type &lt;Name&gt; struct`") against the fixture rows passing `envStructs` `[]string{"p#Env", "p#Inner"}`.

**Tool calls**
- Read `internal/loomcli/wiring.go` (wireLightweight): to confirm `validate-discussion`'s env fills `WorktreeRoot` and `PlanIndex`; my context didn't show that function's body.
- Read `internal/shedrecipe/entries_planwrite_test.go` (`TestPlanWriteEntry_GateConfig`) and the index fakes in `fixture_test.go`: to see which index the planned shedrecipe rows run on.
- Read the discussion records in `internal/loomcli/parity_test.go` and `smoke_gate_test.go`: to check whether their no-git worktrees would hit card 4's tracked-root read.
- Read `symbolRenamePlan`, the `noFindingsPlanIndex` uses, and the shedrecipe/loomrecipe plan-gate tests: to check whether existing go plans reach `ResolveGlyphs` on a fake.
